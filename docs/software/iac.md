# Infrastructure as Code

## Talos

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
