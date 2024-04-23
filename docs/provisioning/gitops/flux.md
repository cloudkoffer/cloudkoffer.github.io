# Flux

## Description

## Installation Steps

- Checkout the [:simple-github: provisioning-gitops-flux](https://github.com/cloudkoffer/provisioning-gitops-flux) repository.

    ``` shell
    git clone https://github.com/cloudkoffer/provisioning-gitops-flux
    cd provisioning-gitops-flux
    ```

- Install and configure [age](https://github.com/FiloSottile/age), [kubectl](https://kubernetes.io/docs/tasks/tools/) and [flux](https://fluxcd.io/flux/cmd/).

    === "CLI"

        ``` shell
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "github_token" {
          description = "The personal access token to authenticate to GitHub."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="provider.tf"
        terraform {
          required_providers {
            # https://github.com/clementblaise/terraform-provider-age/blob/main/CHANGELOG.md
            age = {
              source  = "clementblaise/age"
              version = "~> 0.1.1"
            }
            # https://github.com/hashicorp/terraform-provider-kubernetes/blob/main/CHANGELOG.md
            kubernetes = {
              source  = "hashicorp/kubernetes"
              version = "2.29.0"
            }
            # https://github.com/integrations/terraform-provider-github/releases
            github = {
              source  = "integrations/github"
              version = "6.2.1"
            }
            # https://github.com/fluxcd/terraform-provider-flux/blob/main/CHANGELOG.md
            flux = {
              source  = "fluxcd/flux"
              version = "1.2.3"
            }
          }
        }

        provider "kubernetes" {
          config_path = "~/.kube/config"
        }

        provider "github" {
          owner = "qaware"
          token = var.github_token
        }

        provider "flux" {
          kubernetes = {
            config_path = "~/.kube/config"
          }

          git = {
            url    = "ssh://git@github.com/${data.github_repository.this.full_name}.git"
            ssh = {
              username    = "git"
              private_key = tls_private_key.this.private_key_pem
            }
          }
        }
        ```

- Configure environment variables.

    ``` shell
    CLUSTER_NAME="talos-cloudkoffer-v3"
    export GITHUB_TOKEN="<github-token>"
    ```

- Create SOPS age key.

    === "CLI"

        ``` shell
        if [ ! -f "configs/${CLUSTER_NAME}.agekey" ]; then
          age-keygen --output "configs/${CLUSTER_NAME}.agekey"
        fi
        ```

    === "Terraform"

        ``` shell
        resource "age_secret_key" "this" {}
        ```

- Create SOPS configuration.

    === "CLI"

        ``` shell
        cat <<EOF > "../../deployment-stack/configs/${CLUSTER_NAME}.sops.yaml"
        creation_rules:
          - path_regex: .*.yaml
            encrypted_regex: ^(data|stringData)$
            age: $(age-keygen -y "configs/${CLUSTER_NAME}.agekey")
        EOF
        ```

    === "Terraform"

        ``` shell
        resource "local_file" "this" {
          content = <<-EOT
          creation_rules:
            - path_regex: .*.yaml
              encrypted_regex: ^(data|stringData)$
              age: ${age_secret_key.this.public_key}
          EOT
          filename = "../../deployment-stack/configs/${var.cluster_name}.sops.yaml"
        }
        ```

- Create SOPS Kubernetes secret.

    === "CLI"

        ``` shell
        kubectl create namespace flux-system &>/dev/null || true
        kubectl create secret generic sops-age \
          --namespace=flux-system \
          --from-file="configs/${CLUSTER_NAME}.agekey" \
          --dry-run=client \
          --output=yaml \
          --save-config | \
        kubectl apply -f -
        ```

    === "Terraform"

        ``` shell
        resource "kubernetes_namespace" "flux_system" {
          metadata {
            name = "flux-system"
          }
        }

        resource "kubernetes_secret" "sops_age" {
          metadata {
            name      = "sops-age"
            namespace = kubernetes_namespace.flux_system.metadata.0.name
          }

          data = {
            "age.agekey" = <<-EOT
            # created: 2023-01-01T00:00:00+01:00
            # public key: ${age_secret_key.this.public_key}
            ${age_secret_key.this.secret_key}
            EOT
          }
        }
        ```

- Bootstrap flux.

    === "CLI"

        ``` shell
        flux bootstrap github \
          --owner=cloudkoffer \
          --repository=gitops-flux \
          --path="clusters/${CLUSTER_NAME}" \
          --read-write-key
        ```

    === "Terraform"

        ``` terraform title="variables.tf"
        variable "cluster_name" {
          description = "The name for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="main.tf"
        resource "flux_bootstrap_git" "this" {
          path           = "clusters/${var.cluster_name}"
          interval       = "1m0s"
          version        = "v2.2.3" # https://github.com/fluxcd/flux2/releases
          log_level      = "info"   # debug, info, error
          network_policy = false

          depends_on = [
            github_repository_deploy_key.this,
            kubernetes_secret.sops_age,
          ]
        }
        ```

        ``` shell
        terraform apply
        ```

## Maintenance Steps

[:simple-github: gitops-flux](https://github.com/cloudkoffer/gitops-flux)
