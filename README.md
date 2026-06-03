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
```

###Step 2: Establish Remote WinRM Management (If Running Remotely)
If you intend to run this playbook from an external control engine (like a dedicated Linux server or a remote Mac/WSL instance over your local network), activate the Windows Remote Management (WinRM) listener parameters inside that same Administrator PowerShell:

```
winrm quickconfig -q
winrm set winrm/config/service/Auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```

💡 Note: If running the playbook directly from within the machine via localized tools (WSL targeting localhost), configuring unencrypted network ports is completely optional.

## 📂 Project Directory Structure
Organize your configuration setup files inside your workspace folder like this:
```
dev-workstation-setup/
├── inventory.ini           # Defines target connections and environments
├── setup_pc.yml            # Main playbook container managing tasks
└── README.md               # Setup reference guide (This file)
```

## 📄 File Profiles
1. inventory.ini
Create this file to define targeting paths. For running directly on your current workstation machine locally, use the local engine definition block:

```
[workstation]
localhost ansible_connection=local

# Alternative configuration block for a remote machine over the network:
# my-windows-pc ansible_host=192.168.1.50 ansible_user=Administrator ansible_password=SecretPassword ansible_connection=winrm ansible_winrm_server_cert_validation=ignore
```

## 2. setup_pc.yml
Save the primary task orchestration blueprint code here:

```
---
- name: Configure Windows Workstation
  hosts: workstation
  gather_facts: no
  
  vars:
    git_username: "Your Name"           # <-- Change to your actual name
    git_email: "your.email@example.com" # <-- Change to your actual email
    
    # Precise marketplace extension identifiers
    vscode_extensions:
      - pkief.material-icon-theme          # Material UI Icons
      - redhat.java                        # Java Language Engine
      - vscjava.vscode-java-pack           # Extension Pack for Java 
      - JohnPapa.Angular2                  # Angular Snippets
      - eamodio.gitlens                    # GitLens Code History
      - Postman.postman-for-vscode         # Postman Client Native
      - cweijan.vscode-mysql-client2       # MySQL Database Explorer

  tasks:
    - name: Install core development applications via Chocolatey
      win_chocolatey:
        name:
          - googlechrome
          - git
          - cmder
          - vscode
          - mysql.workbench
        state: present

    - name: Configure global Git username
      win_shell: "git config --global user.name '{{ git_username }}'"
      args:
        executable: powershell

    - name: Configure global Git email
      win_shell: "git config --global user.email '{{ git_email }}'"
      args:
        executable: powershell

    - name: Install VS Code Extensions
      win_shell: "code --install-extension {{ item }} --force"
      args:
        executable: powershell
      loop: "{{ vscode_extensions }}"
      register: extension_result
      changed_when: "'was successfully installed' in extension_result.stdout"

    - name: Install MySQL Server
      win_chocolatey:
        name: mysql
        state: present

    - name: Ensure MySQL Service is running
      win_service:
        name: MySQL
        start_mode: auto
        state: started

    - name: Install Microsoft OpenJDK 21
      win_chocolatey:
        name: microsoft-openjdk
        version: '21.0.0'
        state: present

    - name: Install NVM for Windows
      win_chocolatey:
        name: nvm
        state: present

    - name: Install latest LTS Node.js using NVM
      win_shell: |
        nvm install lts
        nvm use lts
      args:
        executable: powershell

    - name: Install Global NPM Packages (Angular CLI)
      win_shell: |
        npm install -g @angular/cli
      args:
        executable: powershell
```


###🚀 Execution Guide
Once your initialization parameters inside inventory.ini and variables inside setup_pc.yml are modified, launch the environment builder utilizing the ansible-playbook execution runner command:

```
ansible-playbook -i inventory.ini setup_pc.yml
```
