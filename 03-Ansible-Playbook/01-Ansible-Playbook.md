# Ansible Playbooks: Orchestration and Configuration

While Ad-Hoc commands are for quick tasks, **Ansible Playbooks** are the heart of Ansible. They allow you to define complex configurations as "Infrastructure as Code" (IaC) in a repeatable, human-readable format.

---

## 1. What is a Playbook?
A Playbook is a YAML file containing one or more "Plays".
*   **A Play** maps a group of hosts to a set of tasks.
*   **Infrastructure as Code**: Playbooks are version-controlled, documented, and reusable.
*   **Idempotency**: A well-written playbook ensures that running it multiple times results in the same state without causing unnecessary changes.

---

## 2. Playbook Structure & YAML
Ansible uses YAML (Yet Another Markup Language). It is sensitive to indentation (use spaces, not tabs).

### The Basic Hierarchy:
1.  **Playbook** (The file)
2.  **Play** (Defines *who* is targeted and *what* settings apply)
3.  **Tasks** (List of actions to perform)
4.  **Modules** (The actual tools used in tasks)

```yaml
---
- name: Install and Start Apache      # Play Name
  hosts: web                          # Target Hosts
  become: yes                         # Run as Sudo
  
  tasks:                              # List of Tasks
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present

    - name: Ensure Apache is running
      service:
        name: apache2
        state: started
```

---

## 3. Core Components

### A. Hosts and Become
*   `hosts`: Specifies which servers from your inventory should run this play.
*   `become`: Set to `yes` to perform actions with root privileges (sudo).

### B. Variables (`vars`)
Store values you want to reuse across your playbook.
```yaml
vars:
  http_port: 80
  app_dest: /var/www/html
```

### C. Handlers
Handlers are tasks that only run when "notified" by another task. They are typically used for restarting services after a config change.
```yaml
tasks:
  - name: Copy Nginx Config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: Restart Nginx

handlers:
  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

---

## 4. Real-World Example: Multi-Tier Deployment
In this scenario, we set up a Web Server and a Database Server in a single playbook.

```yaml
---
# Play 1: Setup DB Servers
- name: Configure Database Servers
  hosts: db
  become: yes
  tasks:
    - name: Install MariaDB
      apt:
        name: mariadb-server
        state: present

# Play 2: Setup Web Servers
- name: Configure Web Servers
  hosts: app
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
    - name: Deploy application code
      copy:
        src: ./index.html
        dest: /var/www/html/index.html
```

---

## 5. Advanced Scenarios

### Scenario 1: Loops (`with_items` or `loop`)
Instead of writing 5 tasks to install 5 packages, use a loop.
```yaml
tasks:
  - name: Install multiple packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - curl
      - git
      - vim
      - python3-pip
```

### Scenario 2: Conditionals (`when`)
Run a task only if a specific condition is met (e.g., only on Ubuntu servers).
```yaml
tasks:
  - name: Install Apache on Debian-based systems
    apt:
      name: apache2
      state: present
    when: ansible_os_family == "Debian"
```

### Scenario 3: Error Handling (`ignore_errors`)
Sometimes you want the playbook to keep running even if a specific task fails.
```yaml
tasks:
  - name: Run a risky command
    shell: /usr/bin/some-script.sh
    ignore_errors: yes
```

---

## 6. How to Run a Playbook?
Use the `ansible-playbook` command:
```bash
ansible-playbook -i inventory.ini my-playbook.yml
```

### Pro-Tip: Syntax Check & Dry Run
*   **Syntax Check**: `ansible-playbook --syntax-check my-playbook.yml`
*   **Check Mode (Dry Run)**: `ansible-playbook -C my-playbook.yml`

---

## 7. Summary: Ad-Hoc vs. Playbooks

| Feature | Ad-Hoc Commands | Ansible Playbooks |
| :--- | :--- | :--- |
| **Complexity** | Simple, one-task. | Complex, multi-task. |
| **Persistence** | Not saved (CLI history only). | Saved as YAML files. |
| **Reusuability** | Low. | High. |
| **Control** | Immediate execution. | Full orchestration (Order, Handlers, Loops). |
| **Best For** | Reboots, Disk checks, Quick pings. | Full server provisioning, App deployment. |
