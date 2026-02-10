# Installing Ansible: Prerequisites & Setup

Ansible is famous for being "agentless," which makes getting started incredibly fast. However, the **Control Node** (your management machine) does have specific requirements. This guide covers the prerequisites and installation steps, including fixes for modern Linux environments.

---

## 1. Prerequisites (The Control Node)

The **Control Node** is the machine where you run the `ansible` commands.

### OS Requirements
*   **Supported:** Any POSIX-compliant operating system.
    *   Linux (Red Hat, Debian, Ubuntu, CentOS, Fedora, etc.)
    *   macOS
    *   BSD
    *   Windows Subsystem for Linux (WSL)
*   **NOT Supported:** Native Windows (PowerShell/CMD). You *cannot* run the Ansible Control Node directly on Windows. You **must** use WSL (Windows Subsystem for Linux).

### Software Requirements
*   **Python:** Python 3.10 or newer is recommended.
*   **OpenSSH:** You need an SSH client to connect to managed nodes.

---

## 2. Prerequisites (The Managed Nodes)

The **Managed Nodes** are the servers you are automating.

*   **Linux/Unix:**
    *   **SSH:** Port 22 must be open, and you must have valid credentials (SSH key or Password).
    *   **Python:** Python 2.7+ or Python 3.5+ installed. (Note: Most modern Linux distros come with this post-installed).
*   **Windows:**
    *   **WinRM:** Windows Remote Management must be enabled (Ansible acts as a WinRM client).
    *   **PowerShell:** PowerShell 3.0 or newer.
*   **Network Devices:**
    *   **SSH enabled:** No Python required on the device itself.

---

## 3. Installation Methods

Modern Linux distributions (Ubuntu 23.04+, Debian 12+) implement **PEP 668**, which prevents `pip` from installing packages globally to avoid breaking system tools.

You will see this error: `error: externally-managed-environment`.
**To fix this, choose one of the methods below:**

### Method A: Using `pipx` (Recommended)
`pipx` installs applications in isolated environments but makes their commands available globally. This is the **best way** to get the latest Ansible without breaking your system.

1.  **Install pipx:**
    ```bash
    sudo apt update
    sudo apt install pipx
    pipx ensurepath
    ```
    *(You may need to restart your terminal after `ensurepath`)*

2.  **Install Ansible:**
    ```bash
    pipx install --include-deps ansible
    ```
    *(Note the `--include-deps` flag is crucial to expose the `ansible`, `ansible-playbook` commands)*.

3.  **To Upgrade later:**
    ```bash
    pipx upgrade ansible
    ```

### Method B: Using OS Package Manager (Easiest)
This is the simplest method but may install an older version.

*   **Ubuntu/Debian:**
    ```bash
    sudo apt update
    sudo apt install ansible
    ```

*   **RHEL/CentOS/Fedora:**
    ```bash
    sudo dnf install ansible
    ```

*   **macOS (via Homebrew):**
    ```bash
    brew install ansible
    ```

### Method C: Python Virtual Environment (For specific projects)
If you want to isolate Ansible dependencies for a specific project.

1.  **Create a virtual environment:**
    ```bash
    python3 -m venv venv
    ```
2.  **Activate it:**
    ```bash
    source venv/bin/activate
    ```
3.  **Install Ansible inside the venv:**
    ```bash
    pip install ansible
    ```
    *(Note: You must activate the venv every time you want to run ansible)*.

---

## 4. Troubleshooting

### Command 'ansible' not found
### Command 'ansible' not found
1.  **Check if only `ansible-community` was installed:**
    If `pipx list` shows only `ansible-community`, run:
    ```bash
    pipx install --include-deps ansible --force
    ```
2.  **Check PATH:**
    Run `pipx ensurepath` and restart your terminal.

---

## 5. Verification

Once installed, verify that Ansible is working by checking the version.

```bash
ansible --version
```

**Output should look like this:**
```text
ansible [core 2.15.0]
  config file = None
  configured module search path = ['/home/user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/user/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 ...
```

## 6. Summary

*   **Control Node:** Needs Linux/macOS/WSL + Python 3.
*   **Managed Node:** Needs Python + SSH (Linux) or WinRM (Windows).
*   **Best Installation:** Use `pipx install --include-deps ansible` on modern Linux to avoid Python environment errors.
