# Sysmon Installation & Telemetry Validation

This document outlines the deployment, configuration, and telemetry verification of Microsoft System Monitor (Sysmon) on `BTL-WIN11`.

---

## Directory Structure & Prerequisites

Sysmon binaries and configuration files were deployed to a dedicated directory on the local file system:

* **Target Directory:** `C:\Tools\Sysmon`
* **Files:**
  * `Sysmon64a.exe` *(ARM64 binary)*
  * `sysmonconfig.xml` *(Custom configuration file)*

---

## Installation Process

1. Open an elevated PowerShell or Command Prompt window as `labadmin`.
2. Navigate to the installation directory:
   ```cmd
   cd C:\Tools\Sysmon
   ```

1.  Install Sysmon using the ARM64 executable and load the configuration file:

    DOS

    ```
    .\Sysmon64a.exe -accepteula -i .\sysmonconfig.xml
    ```

2.  Verify the `Sysmon64` service status:

    PowerShell

    ```
    Get-Service -Name Sysmon64
    ```

Event Viewer Log Path
---------------------

Once installed, Sysmon logs operational events to the following Windows Event Viewer location:

Plaintext

```
Applications and Services Logs
  └── Microsoft
        └── Windows
              └── Sysmon
                    └── Operational

```

Telemetry Verification & Testing
--------------------------------

To confirm that Sysmon is accurately capturing endpoint activity, the following validation commands were executed and verified within the Event Viewer:

### 1\. Network Connection Telemetry (Event ID 3)

-   **Command:**

    PowerShell

    ```
    Test-NetConnection example.com -Port 443
    ```

-   **Captured Log:** Sysmon **Event ID 3** (*Network Connection*), detailing outbound connection parameters, destination IP, and source process ID.

### 2\. DNS Query Telemetry (Event ID 22)

-   **Command:**

    PowerShell
    
    ```
    Resolve-DnsName example.com
    ```

-   **Captured Log:** Sysmon **Event ID 22** (*DNS Query*), capturing the query name, query results, and requesting process image path.

### 3\. Process Creation Telemetry (Event ID 1)

-   **Commands:**

    DOS

    ```
    ping 127.0.0.1
    whoami
    hostname
    ipconfig

    ```

-   **Captured Logs:** Sysmon **Event ID 1** (*Process Creation*), confirming full command-line arguments, parent process details, hashes, and process tree context for `ping.exe`, `whoami.exe`, `hostname.exe`, and `ipconfig.exe`.

References & Credits
--------------------

-   **Official Documentation:** [Microsoft Sysmon Documentation](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

-   **Configuration Base:** [sysmon-modular by Olaf Hartong](https://github.com/olafhartong/sysmon-modular)
