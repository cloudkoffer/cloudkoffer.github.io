<!-- markdownlint-disable MD046 -->
# Flux

## Description

Flux is a tool for keeping Kubernetes clusters in sync with sources of configuration (like Git repositories and OCI artifacts), and automating updates to configuration when there is new code to deploy.

Flux version 2 ("v2") is built from the ground up to use Kubernetes' API extension system, and to integrate with Prometheus and other core components of the Kubernetes ecosystem. In version 2, Flux supports multi-tenancy and support for syncing an arbitrary number of Git repositories, among other long-requested features.

Flux v2 is constructed with the GitOps Toolkit, a set of composable APIs and specialized tools for building Continuous Delivery on top of Kubernetes.

## Installation Steps

- Checkout the [:simple-github: provisioning-gitops-flux](https://github.com/cloudkoffer/provisioning-gitops-flux) repository.

    ``` shell title="Shell"
    git clone https://github.com/cloudkoffer/provisioning-gitops-flux
    cd provisioning-gitops-flux
    ```

- Install and configure [age](https://github.com/FiloSottile/age), [kubectl](https://kubernetes.io/docs/tasks/tools/) and [flux](https://fluxcd.io/flux/cmd/).

    === "CLI"

        - <https://github.com/FiloSottile/age>
        - <https://kubernetes.io/docs/tasks/tools/>
        - <https://fluxcd.io/flux/cmd/>

    === "Terraform"

        ``` terraform title="File: variables.tf" linenums="1"
        variable "github_token" {
          description = "The personal access token to authenticate to GitHub."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="File: provider.tf" linenums="1"
        terraform {
          required_providers {
            # https://github.com/clementblaise/terraform-provider-age/blob/main/CHANGELOG.md
            age = {
              source  = "clementblaise/age"
              version = "0.1.1"
            }
            # https://github.com/hashicorp/terraform-provider-local/blob/main/CHANGELOG.md
            local = {
              source  = "hashicorp/local"
              version = "2.5.1"
            }
            # https://github.com/hashicorp/terraform-provider-kubernetes/blob/main/CHANGELOG.md
            kubernetes = {
              source  = "hashicorp/kubernetes"
              version = "2.30.0"
            }
            # https://github.com/integrations/terraform-provider-github/releases
            github = {
              source  = "integrations/github"
              version = "6.2.1"
            }
            # https://github.com/fluxcd/terraform-provider-flux/blob/main/CHANGELOG.md
            flux = {
              source  = "fluxcd/flux"
              version = "1.3.0"
            }
          }
        }

        provider "kubernetes" {
          config_path = "~/.kube/config"
        }

        provider "github" {
          owner = "cloudkoffer"
          token = var.github_token
        }

        provider "flux" {
          kubernetes = {
            config_path = "~/.kube/config"
          }

          git = {
            url = "https://github.com/cloudkoffer/gitops-flux.git"
            http = {
              username = "git"
              password = var.github_token
            }
          }
        }
        ```

- Create SOPS age key.

    === "CLI"

        ``` shell title="Shell"
        age-keygen --output age.key
        ```

    === "Terraform"

        ``` terraform title="File: main.tf" linenums="1"
        resource "age_secret_key" "this" {}
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Create SOPS configuration.

    === "CLI"

        ``` shell title="Shell"
        cat <<EOF > sops.yaml
        creation_rules:
          - path_regex: .*.yaml
            encrypted_regex: ^(data|stringData)$
            age: $(age-keygen -y age.key)
        EOF
        ```

    === "Terraform"

        ``` terraform title="File: variables.tf" linenums="1"
        variable "cluster_name" {
          description = "The name for the Talos cluster."
          type        = string
          nullable    = false
        }
        ```

        ``` terraform title="File: main.tf" linenums="1"
        resource "local_file" "this" {
          filename = "sops.yaml"
          content  = <<-EOT
          creation_rules:
            - path_regex: .*.yaml
              encrypted_regex: ^(data|stringData)$
              age: ${age_secret_key.this.public_key}
          EOT
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Create SOPS Kubernetes secret.

    === "CLI"

        ``` shell title="Shell"
        kubectl create namespace flux-system &>/dev/null || true
        kubectl create secret generic sops-age \
          --namespace=flux-system \
          --from-file=age.key
        ```

    === "Terraform"

        ``` terraform title="File: main.tf" linenums="1"
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
            # created: 2024-01-01T00:00:00+01:00
            # public key: ${age_secret_key.this.public_key}
            ${age_secret_key.this.secret_key}
            EOT
          }
        }
        ```

        ``` shell title="Shell"
        terraform apply
        ```

- Bootstrap flux.

    === "CLI"

        ``` shell title="Shell"
        export GITHUB_TOKEN=<github-token>

        flux bootstrap github \
          --owner=cloudkoffer \
          --repository=gitops-flux \
          --path=cluster \
          --token-auth
        ```

    === "Terraform"

        ``` terraform title="File: main.tf" linenums="1"
        resource "flux_bootstrap_git" "this" {
          path           = "cluster"
          version        = "v2.3.0" # https://github.com/fluxcd/flux2/releases
          log_level      = "info"   # debug, info, error
          network_policy = false

          depends_on = [
            kubernetes_secret.sops_age,
          ]
        }
        ```

        ``` shell title="Shell"
        export TF_VAR_github_token=<github-token>
        terraform apply
        ```

## Maintenance Steps

[:simple-github: gitops-flux](https://github.com/cloudkoffer/gitops-flux)
