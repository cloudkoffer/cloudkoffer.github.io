# Cloudkoffer

## Repositories

- [:simple-github: Infrastructure as Code (IaC)](https://github.com/cloudkoffer/IaC)
- [:simple-github: Configuration as Code (CaC)](https://github.com/cloudkoffer/CaC)
- [:simple-github: GitOps](https://github.com/cloudkoffer/GitOps)

## Building a Cluster

- Boot Talos in maintenence mode.
    - **Keyboard F12**
        - Ensure the nodes are shutdown.
        - Connect a keyboard to a node and press its power button.
        - Press F12 (repeatedly) during the boot process to trigger the network boot.
    - **Local Medium**
        - Prepare bootable USB flash drive or SD card (e.g. [balenaEtcher](https://www.balena.io/etcher)) using `talos-amd64.ios` from Talos GitHub [release page](https://github.com/siderolabs/talos/releases).
        - Ensure the nodes are shutdown.
        - Connect the local medium to a node and press its power button.
        - Select `Reset Talos installation` if Talos was previously installed, otherwise `Talos ISO`.

- Apply `Infrastructure as Code (IaC)`
- Apply `Configuration as Code (CaC)`
