# Talos

## Description

Talos is a modern OS for running Kubernetes: secure, immutable, and minimal. Talos is fully open source, production-ready, and supported by the people at Sidero Labs All system management is done via an API - there is no shell or interactive console. Benefits include:

- **Security**: Talos reduces your attack surface: It's minimal, hardened, and immutable. All API access is secured with mutual TLS (mTLS) authentication.
- **Predictability**: Talos eliminates configuration drift, reduces unknown factors by employing immutable infrastructure ideology, and delivers atomic updates.
- **Evolvability**: Talos simplifies your architecture, increases your agility, and always delivers current stable Kubernetes and Linux versions.

### Features

- **Minimal** - Talos consists of only a handful of binaries and shared libraries: just enough to run containerd and a small set of system services. This aligns with NIST's recommendation in the Application Container Security Guide.
- **Hardened** - Built with the Kernel Self Protection Project configuration recommendations. All access to the API is secured with Mutual TLS. Settings and configuration described in the CIS guidelines are applied by default.
- **Immutable** - Talos improves security further by mounting the root filesystem as read-only and removing any host-level such as a shell and SSH.
- **Ephemeral** - Talos runs in memory from a SquashFS, and persists nothing, leaving the primary disk entirely to Kubernetes.
- **Current** - Delivers the latest stable versions of Kubernetes and Linux.

## Installation Steps

- Checkout the [:simple-github: provisioning-k8s-talos](https://github.com/cloudkoffer/provisioning-k8s-talos) repository.

    ``` shell title="Shell"
    git clone https://github.com/cloudkoffer/provisioning-k8s-talos
    cd provisioning-k8s-talos
    ```

- Configure environment variables.

    === "CLI"

        ``` shell title="Shell"
        CLUSTER_NAME="talos-cloudkoffer-v3"
        CLUSTER_ENDPOINT="https://192.168.1.101:6443"
        TALOS_VERSION="v1.7.0"
        KUBERNETES_VERSION="1.30.0"
        NODES_CONTROLPLANE=(
          "192.168.1.1"
          "192.168.1.2"
          "192.168.1.3"
        )
        NODES_WORKER=(
          "192.168.1.4"
          "192.168.1.5"
          "192.168.1.6"
          "192.168.1.7"
          "192.168.1.8"
          "192.168.1.9"
          "192.168.1.10"
        )
        ```

    === "Terraform"

        ``` shell title="Shell"
        TF_VAR_cluster_name="talos-cloudkoffer-v3"
        TF_VAR_cluster_endpoint="https://192.168.1.101:6443"
        TF_VAR_talos_version="1.7.0"
        TF_VAR_kubernetes_version="1.30.0"
        TF_VAR_nodes='{
          "controlplane"=[
            "192.168.1.1",
            "192.168.1.2",
            "192.168.1.3"
          ],
          "worker"=[
            "192.168.1.4",
            "192.168.1.5",
            "192.168.1.6",
            "192.168.1.7",
            "192.168.1.8",
            "192.168.1.9",
            "192.168.1.10"
          ]
        }'
        ```

- Install and configure [talosctl](https://www.talos.dev/v1.7/introduction/getting-started/#talosctl).

    === "CLI"

        - <https://www.talos.dev/v1.7/introduction/getting-started/#talosctl>

    === "Terraform"

        ``` terraform title="File: provider.tf"
        terraform {
          required_providers {
            # https://github.com/siderolabs/terraform-provider-talos/releases
            talos = {
              source  = "siderolabs/talos"
              version = "0.5.0"
            }
          }
        }

        provider "talos" {}
        ```

        ``` shell title="Shell"
        curl -sL https://talos.dev/install | sh
        ```

- Boot the nodes using either USB sticks or a network boot (F12).

- Wait until the nodes have entered maintenance mode.

    ``` shell title="Shell"
    for node in "${NODES_CONTROLPLANE[@]}"; do
      echo -n "Node ${node}: "
      talosctl get machinestatus \
        --insecure \
        --nodes "${node}" \
        --output jsonpath='{.spec.stage}'
    done

    for node in "${NODES_WORKER[@]}"; do
      echo -n "Node ${node}: "
      talosctl get machinestatus \
        --insecure \
        --nodes "${node}" \
        --output jsonpath='{.spec.stage}'
    done
    ```

- Create talos machine secrets.

    === "CLI"

        ``` shell title="Shell"
        talosctl gen secrets \
          --output-file=secrets.yaml \
          --talos-version="${TALOS_VERSION}"
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf"
        variable "talos_version" {
          description = "The talos version for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="File: main.tf"
        resource "talos_machine_secrets" "this" {
          talos_version = var.talos_version
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Create talos client and machine configuration.

    === "CLI"

        ``` shell title="Shell"
        talosctl gen config "${CLUSTER_NAME}" https://192.168.1.101:6443 \
          --config-patch="@../patches/${CLUSTER_NAME}/all.yaml" \
          --config-patch-control-plane="@../patches/${CLUSTER_NAME}/controlplane.yaml" \
          --config-patch-worker="@../patches/${CLUSTER_NAME}/worker.yaml" \
          --install-disk=/dev/nvme0n1 \
          --install-image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
          --kubernetes-version="${KUBERNETES_VERSION}" \
          --talos-version="${TALOS_VERSION}"
          --with-docs=false \
          --with-examples=false \
          --with-secrets=secrets.yaml

        talosctl config endpoint 192.168.1.1 192.168.1.2 192.168.1.3 \
          --talosconfig=talosconfig
        talosctl config node 192.168.1.1 \
          --talosconfig=talosconfig
        talosctl config merge talosconfig
        talosctl config use-context "${CLUSTER_NAME}"
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf"
        variable "cluster_name" {
          description = "The name for the Talos cluster."
          type        = string
          nullable    = false
        }

        variable "cluster_endpoint" {
          description = "The endpoint for the Talos cluster."
          type        = string
          nullable    = false
        }

        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }

        variable "kubernetes_version" {
          description = "The kubernetes version for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="File: main.tf"
        data "talos_client_configuration" "this" {
          client_configuration = talos_machine_secrets.this.client_configuration
          cluster_name         = var.cluster_name
          endpoints            = var.nodes.controlplane
          nodes                = [var.nodes.controlplane[0]]
        }

        data "talos_machine_configuration" "controlplane" {
          cluster_endpoint = var.cluster_endpoint
          cluster_name     = var.cluster_name
          machine_secrets  = talos_machine_secrets.this.machine_secrets
          machine_type     = "controlplane"

          config_patches = [
            file("../patches/${var.cluster_name}/controlplane.yaml"),
            file("../patches/${var.cluster_name}/all.yaml"),
          ]

          docs               = false
          examples           = false
          kubernetes_version = var.kubernetes_version
          talos_version      = talos_machine_secrets.this.talos_version
        }

        data "talos_machine_configuration" "worker" {
          cluster_endpoint = var.cluster_endpoint
          cluster_name     = var.cluster_name
          machine_secrets  = talos_machine_secrets.this.machine_secrets
          machine_type     = "worker"

          config_patches = [
            file("../patches/${var.cluster_name}/all.yaml"),
          ]

          docs               = false
          examples           = false
          kubernetes_version = var.kubernetes_version
          talos_version      = talos_machine_secrets.this.talos_version
        }
        ```

        ``` terraform title="File: outputs.tf"
        output "talosconfig" {
          value     = data.talos_client_configuration.this.talos_config
          sensitive = true
        }
        ```

        ``` shell title="Shell"
        terraform apply

        terraform output -raw talosconfig > talosconfig
        talosctl config merge talosconfig
        talosctl config use-context "${CLUSTER_NAME}"
        ```

- Apply talos machine configuration.

    === "CLI"

        ``` shell title="Shell"
        for node in "${NODES_CONTROLPLANE[@]}"; do
          talosctl apply-config \
            --insecure \
            --nodes="${node}" \
            --file=controlplane.yaml
        done

        for node in "${NODES_WORKER[@]}"; do
          talosctl apply-config \
            --insecure \
            --nodes="${node}" \
            --file=worker.yaml
        done
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf"
        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }
        ```

        ``` terraform title="File: main.tf"
        resource "talos_machine_configuration_apply" "controlplane" {
          for_each = toset(var.nodes.controlplane)

          client_configuration        = talos_machine_secrets.this.client_configuration
          machine_configuration_input = data.talos_machine_configuration.controlplane.machine_configuration
          node                        = each.key
        }

        resource "talos_machine_configuration_apply" "worker" {
          for_each = toset(var.nodes.worker)

          client_configuration        = talos_machine_secrets.this.client_configuration
          machine_configuration_input = data.talos_machine_configuration.worker.machine_configuration
          node                        = each.key
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Bootstrap kubernetes cluster.

    === "CLI"

        ``` shell title="Shell"
        talosctl bootstrap
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf"
        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }
        ```

        ``` terraform title="File: main.tf"
        resource "talos_machine_bootstrap" "this" {
          client_configuration = talos_machine_secrets.this.client_configuration
          node                 = var.nodes.controlplane[0]

          depends_on = [
            talos_machine_configuration_apply.controlplane,
          ]
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Wait until cluster is healthy.

    ``` shell title="Shell"
    talosctl health
    ```

- Retrieve kubeconfig.

    === "CLI"

        ``` shell title="Shell"
        talosctl kubeconfig
        kubectl config use-context "admin@${CLUSTER_NAME}"
        ```

    === "Terraform"

        ``` shell title="Shell"
        terraform output -raw kubeconfig_raw > kubeconfig
        ```

## Maintenance Steps

- Upgrade Talos.

    !!! info

        Perform the upgrade one by one for each node.

    ``` shell title="Shell"
    TALOS_VERSION="v1.7.0"

    talosctl upgrade \
      --image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
      --nodes 192.168.1.x
    ```

- Stage-Upgrade Talos.

    !!! tip

        Use if the above upgrade fails due to a process holding a file open on disk.

    ``` shell title="Shell"
    TALOS_VERSION="v1.7.0"

    talosctl upgrade \
      --image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
      --stage \
      --nodes 192.168.1.x

    talosctl reboot \
      --wait \
      --nodes 192.168.1.x
    ```

- Upgrade Kubernetes.

    ``` shell title="Shell"
    KUBERNETES_VERSION="1.30.0"

    talosctl upgrade-k8s \
      --to="${KUBERNETES_VERSION}"
    ```