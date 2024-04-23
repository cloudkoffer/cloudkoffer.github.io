# Talos

## Description

[Talos Linux](https://www.talos.dev) is Linux designed for Kubernetes – secure, immutable, and minimal.

- Supports cloud platforms, bare metal, and virtualization platforms
- All system management is done via an API. No SSH, shell or console
- Production ready: supports some of the largest Kubernetes clusters in the world
- Open source project from the team at Sidero Labs

### Why Talos Linux?

- **Security** - Talos reduces your attack surface. It's minimal, hardened and immutable. All API access is secured with mutual TLS (mTLS) authentication.
- **Predictability** - Talos eliminates configuration drift, reduces unknown factors by employing immutable infrastructure ideology, and delivers atomic updates.
- **Evolvability** - Talos simplifies your architecture, increases your agility, and always delivers current stable Kubernetes and Linux versions.

### Features

- **Minimal** - Talos consists of only a handful of binaries and shared libraries: just enough to run containerd and a small set of system services. This aligns with NIST's recommendation in the Application Container Security Guide.
- **Hardened** - Built with the Kernel Self Protection Project configuration recommendations. All access to the API is secured with Mutual TLS. Settings and configuration described in the CIS guidelines are applied by default.
- **Immutable** - Talos improves security further by mounting the root filesystem as read-only and removing any host-level such as a shell and SSH.
- **Ephemeral** - Talos runs in memory from a SquashFS, and persists nothing, leaving the primary disk entirely to Kubernetes.
- **Current** - Delivers the latest stable versions of Kubernetes and Linux.

## Installation Steps

- Checkout the [:simple-github: provisioning-gitops-flux](https://github.com/cloudkoffer/provisioning-gitops-flux) repository.

    ``` shell
    git clone https://github.com/cloudkoffer/provisioning-cluster-talos
    cd provisioning-cluster-talos
    ```

- Install talosctl.

    === "CLI"

        ``` shell
        curl -sL https://talos.dev/install | sh
        ```

    === "Terraform"

        ``` terraform title="provider.tf"
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

        ``` shell
        curl -sL https://talos.dev/install | sh
        ```

- Configure environment variables.

    === "CLI"

        ``` shell
        CLUSTER_NAME="talos-cloudkoffer-v3"
        CLUSTER_ENDPOINT="https://192.168.1.101:6443"
        TALOS_VERSION="v1.7.0"
        KUBERNETES_VERSION="1.30.0"

        case "${CLUSTER_NAME}" in
          talos-cloudkoffer-v1)
            ALL_NODES=5
            CONTROLPLANE_NODES=1
            ;;
          talos-cloudkoffer-v2)
            ALL_NODES=10
            CONTROLPLANE_NODES=3
            ;;
          talos-cloudkoffer-v3)
            ALL_NODES=10
            CONTROLPLANE_NODES=3
            ;;
        esac
        ```

    === "Terraform"

        ``` shell
        CLUSTER_NAME=talos-cloudkoffer-v3
        ```

- Boot the nodes using either USB sticks or a network boot (F12).

- Wait until the nodes have entered maintenance mode.

    ``` shell
    for i in {1..${ALL_NODES}}; do
      echo -n "Node ${i}: "
      talosctl get machinestatus \
        --insecure \
        --nodes "192.168.1.${i}" \
        --output jsonpath='{.spec.stage}'
    done
    ```

- Create talos machine secrets.

    === "CLI"

        ``` shell
        talosctl gen secrets \
          --output-file=secrets.yaml \
          --talos-version="${TALOS_VERSION}"
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "talos_version" {
          description = "The talos version for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="main.tf"
        resource "talos_machine_secrets" "this" {
          talos_version = var.talos_version
        }
        ```

        ``` shell
        terraform apply -var-file="configs/${CLUSTER_NAME}.tfvars"
        ```

- Create talos client configuration.

    === "CLI"

        ``` shell
        talosctl config endpoint 192.168.1.1 192.168.1.2 192.168.1.3 \
          --talosconfig=talosconfig
        talosctl config node 192.168.1.1 \
          --talosconfig=talosconfig
        talosctl config merge talosconfig
        talosctl config use-context "${CLUSTER_NAME}"
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "cluster_name" {
          description = "The name for the Talos cluster."
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
        ```

        ``` terraform title="main.tf"
        data "talos_client_configuration" "this" {
          client_configuration = talos_machine_secrets.this.client_configuration
          cluster_name         = var.cluster_name
          endpoints            = var.nodes.controlplane
          nodes                = [var.nodes.controlplane[0]]
        }
        ```

        ``` terraform title="outputs.tf"
        output "talosconfig" {
          value     = data.talos_client_configuration.this.talos_config
          sensitive = true
        }
        ```

        ``` shell
        terraform apply -var-file="configs/${CLUSTER_NAME}.tfvars"

        terraform output -raw talosconfig > talosconfig
        talosctl config merge talosconfig
        talosctl config use-context "${CLUSTER_NAME}"
        ```

- Create talos machine configuration.

    === "CLI"

        ``` shell
        talosctl gen config "${CLUSTER_NAME}" https://192.168.1.101:6443 \
          --endpoints="${CLUSTER_ENDPOINT}" \
          --with-secrets=secrets.yaml \
          --config-patch="@../patches/${CLUSTER_NAME}/all.yaml" \
          --config-patch-control-plane="@../patches/${CLUSTER_NAME}/controlplane.yaml" \
          --config-patch-worker="@../patches/${CLUSTER_NAME}/worker.yaml" \
          --install-disk=/dev/nvme0n1 \
          --install-image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
          --kubernetes-version="${KUBERNETES_VERSION}" \
          --talos-version="${TALOS_VERSION}"
          --with-docs=false \
          --with-examples=false
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "cluster_name" {
          description = "The name for the Talos cluster."
          type        = string
          nullable    = false
        }

        variable "cluster_endpoint" {
          description = "The endpoint for the Talos cluster."
          type        = string
          default     = "https://192.168.1.101:6443"
          nullable    = false
        }

        variable "kubernetes_version" {
          description = "The kubernetes version for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="main.tf"
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

        ``` shell
        terraform apply -var-file="configs/${CLUSTER_NAME}.tfvars"
        ```

- Apply talos machine configuration.

    === "CLI"

        ``` shell
        for i in {1..${CONTROLPLANE_NODES}}; do
          talosctl apply-config \
            --insecure \
            --nodes="192.168.1.${i}" \
            --file=controlplane.yaml
        done

        for i in {$((CONTROLPLANE_NODES+1))..$((ALL_NODES))}; do
          talosctl apply-config \
            --insecure \
            --nodes="192.168.1.${i}" \
            --file=worker.yaml
        done
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }
        ```

        ``` terraform title="main.tf"
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

        ``` shell
        terraform apply -var-file="configs/${CLUSTER_NAME}.tfvars"
        ```

- Bootstrap kubernetes cluster.

    === "CLI"

        ``` shell
        talosctl bootstrap
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }
        ```

        ``` terraform title="main.tf"
        resource "talos_machine_bootstrap" "this" {
          client_configuration = talos_machine_secrets.this.client_configuration
          node                 = var.nodes.controlplane[0]

          depends_on = [
            talos_machine_configuration_apply.controlplane,
          ]
        }
        ```

        ``` shell
        terraform apply -var-file="configs/${CLUSTER_NAME}.tfvars"
        ```

- Wait until cluster is healthy.

    ``` shell
    talosctl health
    ```

- Retrieve kubeconfig.

    ``` shell
    talosctl kubeconfig
    kubectl config use-context "admin@${CLUSTER_NAME}"
    ```

## Maintenance Steps

- Configure environment variables.

    ``` shell
    CLUSTER_NAME="talos-cloudkoffer-v3"
    TALOS_VERSION="v1.7.0"
    KUBERNETES_VERSION="1.30.0"
    ```

- Upgrade Talos.

    !!! info

        Perform the upgrade one by one for each node.

    ``` shell
    talosctl upgrade \
      --image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
      --context="${CLUSTER_NAME}" \
      --nodes 192.168.1.x
    ```

- Stage-Upgrade Talos.

    !!! tip

        Use if the above upgrade fails due to a process holding a file open on disk.

    ``` shell
    talosctl upgrade \
      --image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
      --stage \
      --context="${CLUSTER_NAME}" \
      --nodes 192.168.1.x

    talosctl reboot \
      --wait \
      --context="${CLUSTER_NAME}" \
      --nodes 192.168.1.x
    ```

- Upgrade Kubernetes.

    ``` shell
    talosctl upgrade-k8s \
      --to="${KUBERNETES_VERSION}" \
      --context="${CLUSTER_NAME}"
    ```
