# 🚀 Windows Developer Workstation Automation Setup

This directory contains a unified Ansible infrastructure-as-code solution to automate the initialization, installation, and deployment configuration of a complete Windows-based software development machine. 

By executing this automated setup, your system installs web tools, terminal management environments, database runtime instances, backend compilers, language runtimes, and individual custom settings cleanly without manual intervention.

---

## 🛠️ System Prerequisites

Before running the Ansible orchestration engine, the target Windows host must be modified to allow program installation hooks and shell script processing permissions.

### Step 1: Install Chocolatey & Set Execution Policy
Open a **PowerShell terminal as an Administrator** on the target Windows system and execute the following payload bootstrap script:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('[https://community.chocolatey.org/install.ps1](https://community.chocolatey.org/install.ps1)'))
