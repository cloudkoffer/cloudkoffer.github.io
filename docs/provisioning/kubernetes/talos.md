<!-- markdownlint-disable MD046 -->
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

        === "Cloudkoffer v3"

            ``` shell title="File: .envrc" linenums="1"
            TALOS_VERSION=v1.7.4
            KUBERNETES_VERSION=1.30.1
            NODES_CONTROLPLANE=(
              192.168.1.1
              192.168.1.2
              192.168.1.3
            )
            NODES_WORKER=(
              192.168.1.4
              192.168.1.5
              192.168.1.6
              192.168.1.7
              192.168.1.8
              192.168.1.9
              192.168.1.10
            )
            ```

        === "Cloudkoffer v2"

            ``` shell title="File: .envrc" linenums="1"
            TALOS_VERSION=v1.7.4
            KUBERNETES_VERSION=1.30.1
            NODES_CONTROLPLANE=(
              192.168.1.1
              192.168.1.2
              192.168.1.3
            )
            NODES_WORKER=(
              192.168.1.4
              192.168.1.5
              192.168.1.6
              192.168.1.7
              192.168.1.8
              192.168.1.9
              192.168.1.10
            )
            ```

        === "Cloudkoffer v1"

            ``` shell title="File: .envrc" linenums="1"
            TALOS_VERSION=v1.7.4
            KUBERNETES_VERSION=1.30.1
            NODES_CONTROLPLANE=(
              192.168.1.1
              192.168.1.2
              192.168.1.3
            )
            NODES_WORKER=(
              192.168.1.4
              192.168.1.5
            )
            ```

    === "Terraform"

        === "Cloudkoffer v3"

            ``` shell title="File: .envrc" linenums="1"
            TF_VAR_talos_version=1.7.4
            TF_VAR_kubernetes_version=1.30.1
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

        === "Cloudkoffer v2"

            ``` shell title="File: .envrc" linenums="1"
            TF_VAR_talos_version=1.7.4
            TF_VAR_kubernetes_version=1.30.1
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

        === "Cloudkoffer v1"

            ``` shell title="File: .envrc" linenums="1"
            TF_VAR_talos_version=1.7.4
            TF_VAR_kubernetes_version=1.30.1
            TF_VAR_nodes='{
              "controlplane"=[
                "192.168.1.1",
                "192.168.1.2",
                "192.168.1.3"
              ],
              "worker"=[
                "192.168.1.4",
                "192.168.1.5"
              ]
            }'
            ```

- Install and configure [talosctl](https://www.talos.dev/v1.7/introduction/getting-started/#talosctl).

    === "CLI"

        - <https://www.talos.dev/latest/talos-guides/install/talosctl/>

        ``` shell title="Shell"
        curl -sL https://talos.dev/install | sh
        ```

    === "Terraform"

        ``` terraform title="File: provider.tf" linenums="1"
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

- Boot the nodes using either USB sticks or a network boot (F12).

- Wait until the nodes have entered maintenance mode.

    !!! info "Note for the Terraform workflow"

        The `talosctl` CLI is required to carry out the following step.
        The installation steps can be found in the **CLI** tab of the **Install and configure talosctl** step.

    === "Cloudkoffer v3"

        ``` shell title="Shell"
        for node in {1..10}; do
          echo -n "Node ${node}: "
          talosctl get machinestatus \
            --nodes="192.168.1.${node}" \
            --output=jsonpath='{.spec.stage}' \
            --insecure
        done
        ```

    === "Cloudkoffer v2"

        ``` shell title="Shell"
        for node in {1..10}; do
          echo -n "Node ${node}: "
          talosctl get machinestatus \
            --nodes="192.168.1.${node}" \
            --output=jsonpath='{.spec.stage}' \
            --insecure
        done
        ```

    === "Cloudkoffer v1"

        ``` shell title="Shell"
        for node in {1..5}; do
          echo -n "Node ${node}: "
          talosctl get machinestatus \
            --nodes="192.168.1.${node}" \
            --output=jsonpath='{.spec.stage}' \
            --insecure
        done
        ```

- Create talos machine secrets.

    === "CLI"

        ``` shell title="Shell"
        talosctl gen secrets \
          --output-file=secrets.yaml
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf" linenums="1"
        variable "talos_version" {
          description = "The talos version for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="File: main.tf" linenums="1"
        resource "talos_machine_secrets" "this" {
          talos_version = var.talos_version
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Create talos configuration patches.

    ??? abstract "File: all.yaml"

        === "Cloudkoffer v3"

            ``` yaml linenums="1"
            cluster:
              discovery:
                registries:
                  service:
                    disabled: true
                  kubernetes:
                    disabled: false
              network:
                cni:
                  # Use custom cni
                  # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#machine-config-preparation
                  name: none
              proxy:
                # Disable kube-proxy
                # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#machine-config-preparation
                disabled: true
            machine:
              install:
                disk: /dev/nvme0n1
                extraKernelArgs:
                  # Setting cpu scaling governor
                  # https://www.talos.dev/latest/learn-more/knowledge-base/#setting-cpu-scaling-governor
                  - cpufreq.default_governor=performance
              kubelet:
                extraArgs:
                  # Enable metrics server
                  # https://www.talos.dev/latest/kubernetes-guides/configuration/deploy-metrics-server/
                  rotate-server-certificates: true
                extraMounts:
                  # Enable local storage
                  # https://www.talos.dev/latest/kubernetes-guides/configuration/local-storage/
                  - destination: /var/mnt
                    type: bind
                    source: /var/mnt
                    options:
                      - bind
                      - rshared
                      - rw
              files:
                # Expose containerd metrics
                # https://www.talos.dev/latest/talos-guides/configuration/containerd/#exposing-metrics
                - content: |
                    [metrics]
                      address = "0.0.0.0:11234"
                  path: /etc/cri/conf.d/20-customization.part
                  op: create
            ```

        === "Cloudkoffer v2"

            ``` yaml linenums="1"
            cluster:
              discovery:
                registries:
                  service:
                    disabled: true
                  kubernetes:
                    disabled: false
              network:
                cni:
                  # Use custom cni
                  # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#machine-config-preparation
                  name: none
              proxy:
                # Disable kube-proxy
                # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#machine-config-preparation
                disabled: true
            machine:
              install:
                disk: /dev/nvme0n1
                extraKernelArgs:
                  # Setting cpu scaling governor
                  # https://www.talos.dev/latest/learn-more/knowledge-base/#setting-cpu-scaling-governor
                  - cpufreq.default_governor=performance
              kubelet:
                extraArgs:
                  # Enable metrics server
                  # https://www.talos.dev/latest/kubernetes-guides/configuration/deploy-metrics-server/
                  rotate-server-certificates: true
                extraMounts:
                  # Enable local storage
                  # https://www.talos.dev/latest/kubernetes-guides/configuration/local-storage/
                  - destination: /var/mnt
                    type: bind
                    source: /var/mnt
                    options:
                      - bind
                      - rshared
                      - rw
              files:
                # Expose containerd metrics
                # https://www.talos.dev/latest/talos-guides/configuration/containerd/#exposing-metrics
                - content: |
                    [metrics]
                      address = "0.0.0.0:11234"
                  path: /etc/cri/conf.d/20-customization.part
                  op: create
            ```

        === "Cloudkoffer v1"

            ``` yaml linenums="1"
            cluster:
              discovery:
                registries:
                  service:
                    disabled: true
                  kubernetes:
                    disabled: false
              network:
                cni:
                  # Use custom cni
                  # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#machine-config-preparation
                  name: none
              proxy:
                # Disable kube-proxy
                # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#machine-config-preparation
                disabled: true
            machine:
              install:
                disk: /dev/sda
                extraKernelArgs:
                  # Setting cpu scaling governor
                  # https://www.talos.dev/latest/learn-more/knowledge-base/#setting-cpu-scaling-governor
                  - cpufreq.default_governor=performance
              kubelet:
                extraArgs:
                  # Enable metrics server
                  # https://www.talos.dev/latest/kubernetes-guides/configuration/deploy-metrics-server/
                  rotate-server-certificates: true
                extraMounts:
                  # Enable local storage
                  # https://www.talos.dev/latest/kubernetes-guides/configuration/local-storage/
                  - destination: /var/mnt
                    type: bind
                    source: /var/mnt
                    options:
                      - bind
                      - rshared
                      - rw
              files:
                # Expose containerd metrics
                # https://www.talos.dev/latest/talos-guides/configuration/containerd/#exposing-metrics
                - content: |
                    [metrics]
                      address = "0.0.0.0:11234"
                  path: /etc/cri/conf.d/20-customization.part
                  op: create
            ```

    ??? abstract "File: controlplane.yaml"

        ``` yaml linenums="1"
        cluster:
          apiServer:
            certSANs:
              - 192.168.1.101
              - kube.case.local
          extraManifests:
            # Install metrics server
            # https://www.talos.dev/latest/kubernetes-guides/configuration/deploy-metrics-server/
            # https://github.com/alex1989hu/kubelet-serving-cert-approver/releases
            - https://raw.githubusercontent.com/alex1989hu/kubelet-serving-cert-approver/v0.8.4/deploy/standalone-install.yaml
            # https://github.com/kubernetes-sigs/metrics-server/releases
            - https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.1/components.yaml
          inlineManifests:
            # Install cilium cni
            # https://www.talos.dev/latest/kubernetes-guides/network/deploying-cilium/#method-5-using-a-job
            - name: cilium-install
              contents: |
                ---
                apiVersion: rbac.authorization.k8s.io/v1
                kind: ClusterRoleBinding
                metadata:
                  name: cilium-install
                roleRef:
                  apiGroup: rbac.authorization.k8s.io
                  kind: ClusterRole
                  name: cluster-admin
                subjects:
                - kind: ServiceAccount
                  name: cilium-install
                  namespace: kube-system
                ---
                apiVersion: v1
                kind: ServiceAccount
                metadata:
                  name: cilium-install
                  namespace: kube-system
                ---
                apiVersion: batch/v1
                kind: Job
                metadata:
                  name: cilium-install
                  namespace: kube-system
                spec:
                  backoffLimit: 10
                  template:
                    metadata:
                      labels:
                        app: cilium-install
                    spec:
                      restartPolicy: OnFailure
                      tolerations:
                        - operator: Exists
                        - effect: NoSchedule
                          operator: Exists
                        - effect: NoExecute
                          operator: Exists
                        - effect: PreferNoSchedule
                          operator: Exists
                        - key: node-role.kubernetes.io/control-plane
                          operator: Exists
                          effect: NoSchedule
                        - key: node-role.kubernetes.io/control-plane
                          operator: Exists
                          effect: NoExecute
                        - key: node-role.kubernetes.io/control-plane
                          operator: Exists
                          effect: PreferNoSchedule
                      affinity:
                        nodeAffinity:
                          requiredDuringSchedulingIgnoredDuringExecution:
                            nodeSelectorTerms:
                              - matchExpressions:
                                  - key: node-role.kubernetes.io/control-plane
                                    operator: Exists
                      serviceAccount: cilium-install
                      serviceAccountName: cilium-install
                      hostNetwork: true
                      containers:
                      - name: cilium-install
                        image: quay.io/cilium/cilium-cli-ci:latest
                        env:
                        - name: KUBERNETES_SERVICE_HOST
                          valueFrom:
                            fieldRef:
                              apiVersion: v1
                              fieldPath: status.podIP
                        - name: KUBERNETES_SERVICE_PORT
                          value: "6443"
                        command:
                          - cilium
                          - install
                          - --helm-set=ipam.mode=kubernetes
                          - --set
                          - kubeProxyReplacement=true
                          - --helm-set=securityContext.capabilities.ciliumAgent={CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}
                          - --helm-set=securityContext.capabilities.cleanCiliumState={NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}
                          - --helm-set=cgroup.autoMount.enabled=false
                          - --helm-set=cgroup.hostRoot=/sys/fs/cgroup
                          - --helm-set=k8sServiceHost=localhost
                          - --helm-set=k8sServicePort=7445
                          - --helm-set=hubble.relay.enabled=true
                          - --helm-set=hubble.ui.enabled=true
                          - --helm-set=hubble.ui.ingress.enabled=true
                          - --helm-set=hubble.ui.ingress.className=true
                          - --helm-set=hubble.ui.ingress.hosts={hubble.cluster.cloudkoffer.dev}
        machine:
          network:
            interfaces:
              - interface: eth0
                mtu: 1500
                dhcp: true
                # Configure virtual (shared) ip
                # https://www.talos.dev/latest/talos-guides/network/vip/
                vip:
                  ip: 192.168.1.101
        ```

- Create talos client and machine configuration.

    === "CLI"

        ``` shell title="Shell"
        talosctl gen config cloudkoffer https://192.168.1.101:6443 \
          --config-patch="@../patches/all.yaml" \
          --config-patch-control-plane="@../patches/controlplane.yaml" \
          --install-image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
          --kubernetes-version="${KUBERNETES_VERSION}" \
          --with-docs=false \
          --with-examples=false \
          --with-secrets=secrets.yaml

        export TALOSCONFIG="$(pwd)/talosconfig"
        talosctl config endpoint 192.168.1.1 192.168.1.2 192.168.1.3
        talosctl config node 192.168.1.1
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf" linenums="1"
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

        ``` terraform title="File: main.tf" linenums="1"
        data "talos_client_configuration" "this" {
          client_configuration = talos_machine_secrets.this.client_configuration
          cluster_name         = "cloudkoffer"
          endpoints            = var.nodes.controlplane
          nodes                = [var.nodes.controlplane[0]]
        }

        data "talos_machine_configuration" "controlplane" {
          cluster_endpoint = "https://192.168.1.101:6443"
          cluster_name     = "cloudkoffer"
          machine_secrets  = talos_machine_secrets.this.machine_secrets
          machine_type     = "controlplane"

          config_patches = [
            file("../patches/controlplane.yaml"),
            file("../patches/all.yaml"),
          ]

          docs               = false
          examples           = false
          kubernetes_version = var.kubernetes_version
          talos_version      = talos_machine_secrets.this.talos_version
        }

        data "talos_machine_configuration" "worker" {
          cluster_endpoint = "https://192.168.1.101:6443"
          cluster_name     = "cloudkoffer"
          machine_secrets  = talos_machine_secrets.this.machine_secrets
          machine_type     = "worker"

          config_patches = [
            file("../patches/all.yaml"),
          ]

          docs               = false
          examples           = false
          kubernetes_version = var.kubernetes_version
          talos_version      = talos_machine_secrets.this.talos_version
        }
        ```

        ``` terraform title="File: outputs.tf" linenums="1"
        output "talosconfig" {
          value     = data.talos_client_configuration.this.talos_config
          sensitive = true
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

        ``` shell title="Shell"
        terraform output -raw talosconfig > talosconfig
        export TALOSCONFIG="$(pwd)/talosconfig"
        ```

- Apply talos machine configuration.

    === "CLI"

        ``` shell title="Shell"
        for node in "${NODES_CONTROLPLANE[@]}"; do
          talosctl apply-config \
            --nodes="${node}" \
            --file=controlplane.yaml \
            --insecure
        done

        for node in "${NODES_WORKER[@]}"; do
          talosctl apply-config \
            --nodes="${node}" \
            --file=worker.yaml \
            --insecure
        done
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf" linenums="1"
        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }
        ```

        ``` terraform title="File: main.tf" linenums="1"
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

        ``` terraform title="File: variables.tf" linenums="1"
        variable "nodes" {
          description = "A map of node data."
          type = object({
            controlplane = list(string)
            worker       = list(string)
          })
          nullable = false
        }
        ```

        ``` terraform title="File: main.tf" linenums="1"
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

    !!! info "Note for the Terraform workflow"

        The `talosctl` CLI is required to carry out the following step.
        The installation steps can be found in the **CLI** tab of the **Install and configure talosctl** step.

    ``` shell title="Shell"
    talosctl health
    ```

- Retrieve kubeconfig.

    === "CLI"

        ``` shell title="Shell"
        talosctl kubeconfig kubeconfig
        export KUBECONFIG="$(pwd)/kubeconfig"
        ```

    === "Terraform"

        ``` shell title="Shell"
        terraform output -raw kubeconfig_raw > kubeconfig
        export KUBECONFIG="$(pwd)/kubeconfig"
        ```

## Maintenance Steps

``` shell title="Shell"
export TALOSCONFIG="$(pwd)/talosconfig"
export KUBECONFIG="$(pwd)/kubeconfig"
```

- Upgrade Talos.

    !!! info

        Perform the upgrade one by one for each node.

    ``` shell title="Shell"
    TALOS_VERSION=v1.7.4

    talosctl upgrade \
      --image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
      --nodes=192.168.1.x
    ```

- Stage-Upgrade Talos.

    !!! tip

        Use if the above upgrade fails due to a process holding a file open on disk.

    ``` shell title="Shell"
    TALOS_VERSION=v1.7.4

    talosctl upgrade \
      --image="ghcr.io/siderolabs/installer:${TALOS_VERSION}" \
      --nodes=192.168.1.x \
      --stage

    talosctl reboot \
      --nodes=192.168.1.x \
      --wait
    ```

- Upgrade Kubernetes.

    ``` shell title="Shell"
    KUBERNETES_VERSION=1.30.1

    talosctl upgrade-k8s \
      --to="${KUBERNETES_VERSION}"
    ```
