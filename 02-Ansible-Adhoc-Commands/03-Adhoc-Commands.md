# Ansible Ad-Hoc Commands: Remote Operations at Speed

Ad-Hoc commands are the "Quick Wins" of Ansible. They allow you to execute a single task on one or more managed nodes without writing a full Playbook.

---

## 1. When to Use Ad-Hoc Commands?
Ad-Hoc commands are perfect for tasks you do rarely or just once.
*   **Rebooting** a set of servers.
*   **Checking Disk Space** across your entire infrastructure.
*   **Copying a single file** to 20 servers.
*   **Installing a quick patch** or security update.
*   **Managing Users** (resetting passwords, adding a new admin).

> [!TIP]
> If you find yourself running the same ad-hoc command repeatedly, it's time to put it into a **Playbook**.

---

## 2. The Anatomy of an Ad-Hoc Command

```bash
ansible [pattern] -i [inventory] -m [module] -a "[arguments]" -b
```

| Component | Meaning |
| :--- | :--- |
| `ansible` | The command line tool. |
| **[pattern]** | Which hosts to target (e.g., `web`, `db`, `all`, `192.168.1.1`). |
| `-i [inventory]` | Path to your inventory file. |
| `-m [module]` | The module to run (e.g., `ping`, `apt`, `shell`). |
| `-a "[args]"` | Arguments passed to the module. |
| `-b` | **Become** (run as sudo/root). |

---

## 3. Essential Modules & Scenarios

### A. System & Connection Check (`ping`)
The most basic ad-hoc command to verify connectivity and Python availability.
```bash
ansible all -i inventory.ini -m ping
```

### B. Command & Shell Execution (`command`, `shell`)
Used to run raw Linux commands.
*   **`command`**: Default module. Doesn't support pipes or variables.
*   **`shell`**: Recommended when you need logic (`|`, `>`, `*`, `$`).

**Scenario: Checking uptime and disk usage**
```bash
ansible web -i inventory.ini -m command -a "uptime"
ansible db -i inventory.ini -m shell -a "df -h | grep /dev/sda1"
```

### C. Package Management (`apt`, `yum`)
Install, update, or remove software.

**Scenario: Updating Apache on all web servers**
```bash
ansible web -i inventory.ini -m apt -a "name=apache2 state=latest" -b
```

### D. File Operations (`copy`, `file`, `fetch`)
*   `copy`: Move files from Control Node to Managed Nodes.
*   `file`: Manage permissions, symlinks, and directories.
*   `fetch`: Pull files from Managed Nodes to Control Node.

**Scenario: Creating a directory with specific permissions**
```bash
ansible all -i inventory.ini -m file -a "path=/var/www/html/assets state=directory mode=0755 owner=www-data" -b
```

### E. Service Management (`service`, `systemd`)
Start, stop, or restart background services.

**Scenario: Restarting Nginx after a config change**
```bash
ansible web -i inventory.ini -m service -a "name=nginx state=restarted" -b
```

### F. User Management (`user`)
Manage Linux user accounts.

**Scenario: Creating a new developer account**
```bash
ansible all -i inventory.ini -m user -a "name=john state=present groups=sudo" -b
```

---

## 4. In-Depth Scenarios

### Scenario 1: Mass Log Retrieval
You suspect a security breach and need the last 50 lines of `/var/log/auth.log` from every server in your inventory.
```bash
ansible all -i inventory.ini -m shell -a "tail -n 50 /var/log/auth.log" -b > fleet_auth_logs.txt
```
*   **Wait!** Using `>` redirects the output of the local command to a local file. This gives you a consolidated report of all server logs in one file.

### Scenario 2: Emergency Service Stop
A database update went wrong, and you need to stop the `postgresql` service on all DB servers immediately to prevent data corruption.
```bash
ansible db -i inventory.ini -m service -a "name=postgresql state=stopped" -b
```

### Scenario 3: Checking System Facts (Infrastructure Audit)
You need to know the OS version and IP addresses of every server for an audit.
```bash
ansible all -i inventory.ini -m setup
```
*   **Note:** This returns a huge JSON of everything Ansible knows about the hosts (Facts). You can filter it:
```bash
ansible all -i inventory.ini -m setup -a "filter=ansible_distribution*"
```

---

## 5. Ad-Hoc Command Best Practices

1.  **Use `check_mode` first**: Add `-C` or `--check` to see what would happen without actually changing anything (dry run).
2.  **Be careful with `all`**: Double-check your command before running it against your entire infrastructure.
3.  **Limit output**: Use `-o` (one-line) for cleaner reading when running against hundreds of servers.
4.  **Use specific users**: If you aren't logging in as root, use `-u username` and `-b` to escalate.

---
