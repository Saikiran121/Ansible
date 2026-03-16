# Ansible Built-in Modules: The Power of `ansible.builtin`

Ansible comes with a rich collection of **built-in modules** that require no extra installation. They are the essential building blocks for any automation.

Since Ansible 2.10, it is a best practice to use the **Fully Qualified Collection Name (FQCN)** (e.g., `ansible.builtin.copy` instead of just `copy`).

---

## 1. File Management Modules

These modules manage files, directories, and their properties (permissions, ownership).

### A. `ansible.builtin.file`
Used to create, delete, and set permissions for files and directories.

**Scenario: Ensure a directory exists with correct permissions**
```yaml
- name: Create web directory
  ansible.builtin.file:
    path: /var/www/my_app
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
```

### B. `ansible.builtin.copy`
Copies files from the control node to managed nodes.

**Scenario: Push a local config file to a server**
```yaml
- name: Push custom configuration
  ansible.builtin.copy:
    src: files/settings.conf
    dest: /etc/my_app/settings.conf
    backup: yes # Creates a backup of the original file if it changes
```

### C. `ansible.builtin.template`
Similar to `copy`, but uses **Jinja2** to inject variables into the file before copying.

**Scenario: Dynamic configuration based on hostname**
```yaml
- name: Configure Nginx from template
  ansible.builtin.template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

---

## 2. Package Management Modules

Manage software installations across different operating systems.

### A. `ansible.builtin.apt` (Debian/Ubuntu) & `ansible.builtin.yum` (RHEL/CentOS)
The standard tools for installing software.

**Scenario: Install multiple essential tools**
```yaml
- name: Install utility packages
  ansible.builtin.apt:
    name:
      - curl
      - htop
      - git
    state: present
    update_cache: yes
```

### B. `ansible.builtin.package`
A **generic** module that calls the OS's specific package manager automatically. Useful for multi-OS playbooks.

---

## 3. System & Service Modules

### A. `ansible.builtin.service` / `ansible.builtin.systemd`
Manage background services (start, stop, restart, enable on boot).

**Scenario: Restart a service after a config update**
```yaml
- name: Restart Apache
  ansible.builtin.service:
    name: apache2
    state: restarted
    enabled: yes
```

### B. `ansible.builtin.user` & `ansible.builtin.group`
Manage Linux accounts and permissions.

**Scenario: Create a deployment user**
```yaml
- name: Add 'deploy' user
  ansible.builtin.user:
    name: deploy
    shell: /bin/bash
    groups: sudo
    append: yes
```

---

## 4. Special Purpose Modules

### A. `ansible.builtin.lineinfile`
Modifies specific lines within a file. Perfect for small config tweaks.

**Scenario: Disable Root SSH Login**
```yaml
- name: Disable Root Login
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
    state: present
  notify: Restart SSH
```

### B. `ansible.builtin.debug`
Prints statements or variable values to the console during execution. Essential for **troubleshooting**.

**Scenario: Inspect host facts**
```yaml
- name: Show the OS distribution
  ansible.builtin.debug:
    msg: "The OS is {{ ansible_distribution }}"
```

### C. `ansible.builtin.shell` vs `ansible.builtin.command`
*   `command`: Safest, doesn't use the shell env (no pipes, no logic).
*   `shell`: Allows full shell features like `|`, `>`, and `$HOME`.

---

## 5. Summary Table

| Module Category | Examples | Purpose |
| :--- | :--- | :--- |
| **Files** | `file`, `copy`, `template`, `lineinfile` | Manage identity and content of files. |
| **Packages** | `apt`, `yum`, `package`, `pip` | Manage software installation. |
| **Commands** | `command`, `shell`, `script` | Run raw commands or scripts. |
| **Services** | `service`, `systemd`, `cron` | Manage background life. |
| **System** | `user`, `group`, `hostname` | Manage OS-level settings. |

---

> [!IMPORTANT]
> Always check for **Idempotency**. Modules like `copy`, `file`, and `apt` are naturally idempotent. `shell` and `command` are NOT; you should use `creates` or `removes` arguments to make them safe.
