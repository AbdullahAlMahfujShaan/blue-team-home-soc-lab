# Wazuh Server Setup (`BTL-WAZUH`)

This document covers the deployment, OS configuration, and all-in-one installation of the central Wazuh SIEM server (`BTL-WAZUH`) running on Ubuntu Server ARM64.

---

## Hardware & Virtual Machine Configuration

| Parameter | Configuration |
| :--- | :--- |
| **Hostname** | `btl-wazuh` |
| **Guest OS** | Ubuntu Server ARM64 |
| **vCPU** | 4 Cores |
| **RAM** | 8 GB |
| **Storage** | 80 GB Virtual Disk |
| **Network Adapter** | VMware NAT |

---

## Operating System Preparation & Updates

Before deploying the Wazuh stack, update system repositories, upgrade all packages, and apply kernel updates:

```bash
# Update repository index and perform full system upgrade
sudo apt update
sudo apt full-upgrade -y

# Reboot to apply core updates
sudo reboot
```

After the server reboots, clear out orphaned dependencies:

Bash

```
# Remove unused packages
sudo apt autoremove -y
```

Wazuh All-in-One Installation
-----------------------------

The Wazuh installation assistant automates the deployment of the Wazuh Manager, Wazuh Indexer, and Wazuh Dashboard on a single host.

1.  Download the official Wazuh installation script:

    Bash

    ```
    curl -L -o wazuh-install.sh [https://packages.wazuh.com/4.14/wazuh-install.sh](https://packages.wazuh.com/4.14/wazuh-install.sh)
    ```

2.  Run the automated installation script:

    Bash

    ```
    sudo bash ./wazuh-install.sh -a
    ```

> **Security Note:** Upon script completion, the terminal displays the generated `admin` password and login details. Do **not** commit passwords, secret keys, or API tokens to GitHub repositories.

Dashboard Access
----------------

Once the installer finishes initializing services, access the Web UI from any host on the network:

Plaintext

```
https://<WAZUH-IP>
```

Log in with the `admin` username and the unique credential generated during setup.

References & Documentation
--------------------------

-   **Official Deployment Guide:** [Wazuh Quickstart](https://documentation.wazuh.com/current/quickstart.html)
