# Infrastructure-as-Code-with-Ansible-for-LAMP-Stack


```markdown
# 📦 Infrastructure as Code with Ansible for LAMP Stack

This project demonstrates how to automate the deployment of a LAMP (Linux, Apache, MySQL, PHP) stack using **Ansible** with a **role-based structure**. It supports multiple environments and uses best practices like handlers, tags, templates, and modular roles.

---

## ✅ Objective

- Install **Apache**, **MySQL**, and **PHP**
- Use **handlers**, **notify**, **tags**, **registers**, **facts**
- Support for **dev**, **staging**, and **prod** environments

---

## 🔧 Step 1: Project Directory Structure

```bash
lamp-ansible/
├── ansible.cfg
├── inventory/
│   ├── dev.ini
│   ├── staging.ini
│   └── prod.ini
├── playbook.yml
├── roles/
│   ├── apache/
│   │   ├── tasks/main.yml
│   │   ├── handlers/main.yml
│   │   └── templates/index.html.j2
│   ├── mysql/
│   │   ├── tasks/main.yml
│   │   ├── handlers/main.yml
│   └── php/
│       ├── tasks/main.yml
│       └── handlers/main.yml
```
---

### `file-setup.py`

```python

import os

# Base directory
base_dir = "/home/lilia/VIDEOS/lamp-ansible"

# Define content for each file
file_contents = {
    "ansible.cfg": """[defaults]
inventory = inventory
host_key_checking = False
retry_files_enabled = False
""",

    "inventory/dev.ini": """[web]
192.168.56.10 ansible_user=ubuntu
""",

    "inventory/staging.ini": """[web]
192.168.56.20 ansible_user=ubuntu
""",

    "inventory/prod.ini": """[web]
192.168.56.30 ansible_user=ubuntu
""",

    "playbook.yml": """- name: Install LAMP Stack
  hosts: web
  become: yes
  vars:
    mysql_root_password: "root123"
  roles:
    - { role: apache, tags: ["web", "apache"] }
    - { role: mysql, tags: ["db", "mysql"] }
    - { role: php, tags: ["php"] }
""",

    "roles/apache/tasks/main.yml": """- name: Install Apache
  apt:
    name: apache2
    state: present
    update_cache: yes
  notify: Restart Apache

- name: Deploy custom homepage
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
""",

    "roles/apache/handlers/main.yml": """- name: Restart Apache
  service:
    name: apache2
    state: restarted
""",

    "roles/apache/templates/index.html.j2": """<html>
  <head><title>Welcome</title></head>
  <body>
    <h1>Hello from {{ inventory_hostname }}</h1>
  </body>
</html>
""",

    "roles/mysql/tasks/main.yml": """- name: Install MySQL Server
  apt:
    name: mysql-server
    state: present
    update_cache: yes

- name: Ensure MySQL is running
  service:
    name: mysql
    state: started
    enabled: true

- name: Set root password
  mysql_user:
    name: root
    password: "{{ mysql_root_password }}"
    login_unix_socket: /var/run/mysqld/mysqld.sock
    host_all: true
    state: present
  tags: mysql_setup
""",

    "roles/mysql/handlers/main.yml": """- name: Restart MySQL
  service:
    name: mysql
    state: restarted
""",

    "roles/php/tasks/main.yml": """- name: Install PHP and modules
  apt:
    name:
      - php
      - libapache2-mod-php
      - php-mysql
    state: present
    update_cache: yes
  notify: Restart Apache
""",

    "roles/php/handlers/main.yml": """- name: Restart Apache
  service:
    name: apache2
    state: restarted
"""
}

# Create the file structure and write contents
for relative_path, content in file_contents.items():
    full_path = os.path.join(base_dir, relative_path)
    os.makedirs(os.path.dirname(full_path), exist_ok=True)
    with open(full_path, "w") as f:
        f.write(content)

"All Ansible playbook files and contents have been successfully created"


```



---

## ⚙️ Step 2: Configuration Files

### `ansible.cfg`

```ini
[defaults]
inventory = inventory
host_key_checking = False
retry_files_enabled = False
```

### `inventory/dev.ini`

```ini
[web]
192.168.56.10 ansible_user=ubuntu
```

_Update staging and prod with your actual host IPs._

---

## 📄 Step 3: Playbook Definition

### `playbook.yml`

```yaml
- name: Install LAMP Stack
  hosts: web
  become: yes
  vars:
    mysql_root_password: "root123"
  roles:
    - { role: apache, tags: ["web", "apache"] }
    - { role: mysql, tags: ["db", "mysql"] }
    - { role: php, tags: ["php"] }
```

---

## 🧱 Step 4: Role Definitions

### Apache

#### `roles/apache/tasks/main.yml`

```yaml
- name: Install Apache
  apt:
    name: apache2
    state: present
    update_cache: yes
  notify: Restart Apache

- name: Deploy custom homepage
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
```

#### `roles/apache/handlers/main.yml`

```yaml
- name: Restart Apache
  service:
    name: apache2
    state: restarted
```

#### `roles/apache/templates/index.html.j2`

```html
<html>
  <head><title>Welcome</title></head>
  <body>
    <h1>Hello from {{ inventory_hostname }}</h1>
  </body>
</html>
```

---

### MySQL

#### `roles/mysql/tasks/main.yml`

```yaml
- name: Install MySQL Server
  apt:
    name: mysql-server
    state: present
    update_cache: yes

- name: Ensure MySQL is running
  service:
    name: mysql
    state: started
    enabled: true

- name: Set root password
  mysql_user:
    name: root
    password: "{{ mysql_root_password }}"
    login_unix_socket: /var/run/mysqld/mysqld.sock
    host_all: true
    state: present
  tags: mysql_setup
```

#### `roles/mysql/handlers/main.yml`

```yaml
- name: Restart MySQL
  service:
    name: mysql
    state: restarted
```

---

### PHP

#### `roles/php/tasks/main.yml`

```yaml
- name: Install PHP and modules
  apt:
    name:
      - php
      - libapache2-mod-php
      - php-mysql
    state: present
    update_cache: yes
  notify: Restart Apache
```

#### `roles/php/handlers/main.yml`

```yaml
- name: Restart Apache
  service:
    name: apache2
    state: restarted
```

---

## 🧪 Step 5: Running the Playbook

To apply the playbook to the **dev** environment:

```bash
ansible-playbook -i inventory/dev.ini playbook.yml --tags "web,db,php"
```

### Example Use of Tags

| Command | Description |
|--------|-------------|
| `--tags "web"` | Run only Apache tasks |
| `--tags "php"` | Run only PHP installation |
| `--skip-tags "mysql"` | Skip all MySQL tasks |
| `--tags "web,php"` | Run only Apache and PHP |

---

## 📘 Explanation of Ansible Concepts Used

| Feature        | Description |
|----------------|-------------|
| **Handlers**   | Trigger actions only when notified by tasks |
| **Tags**       | Run or skip specific parts of the playbook |
| **Registers**  | Store output of a task for use in logic (extendable) |
| **Facts**      | Use system information like `ansible_hostname`, etc. |
| **Role-based** | Makes the playbook modular, reusable, and organized |

---

## 🧩 Next Steps

- [ ] Add testing support with Molecule or Testinfra
- [ ] Encrypt credentials using Ansible Vault
- [ ] Parameterize IPs and credentials using `group_vars`

---

