# Ansible Cheatsheet

Complete Quick Reference Guide for Ansible Automation — 350+ Commands, 500+ Examples, Production Ready.

## 🔧 Basics & Installation

### Installation & Setup

Install Ansible:

```bash
# Using pip
pip install ansible

# Debian/Ubuntu
sudo apt update
sudo apt install ansible

# RHEL/CentOS
sudo yum install ansible

# macOS (Homebrew)
brew install ansible
```

Check Ansible version:

```bash
ansible --version
```

### Inventory File

Basic inventory format:

```ini
[webservers]
web1.example.com
web2.example.com ansible_user=ubuntu

[databases]
db1.example.com ansible_port=2222
db2.example.com

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Group variables:

```ini
ansible_host=192.168.1.1
ansible_port=22
ansible_user=root
ansible_password=secret
ansible_ssh_private_key_file=/path/to/key
ansible_connection=ssh
```

## ⚡ Ad-Hoc Commands

### Running Commands

Execute module:

```bash
ansible all -m ping
ansible webservers -m command -a "uptime"
ansible all -m shell -a "df -h" -b
```

Common modules:

```bash
ansible all -m ping                                     # Test connectivity
ansible all -m setup                                    # Gather facts
ansible webservers -m command -a "uptime"                # Run a command
ansible webservers -m shell -a "echo $HOME"               # Run via shell
ansible webservers -m copy -a "src=file.txt dest=/tmp/file.txt"
ansible webservers -m file -a "path=/tmp/test state=directory"
ansible webservers -m apt -a "name=nginx state=present" -b
ansible webservers -m service -a "name=nginx state=started" -b
ansible all -m user -a "name=deploy state=present" -b
ansible all -m reboot -b
```

## 📜 Playbook Structure

Basic playbook:

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

Run playbook:

```bash
ansible-playbook site.yml
ansible-playbook site.yml -i hosts
ansible-playbook site.yml -u ubuntu
ansible-playbook site.yml -v          # Verbose
ansible-playbook site.yml -vvv        # Very verbose
ansible-playbook site.yml --check     # Dry run
ansible-playbook site.yml --syntax-check
```

Using variables:

```yaml
- hosts: all
  vars:
    http_port: 80
    app_name: myapp
  tasks:
    - name: Print variables
      debug:
        msg: "App {{ app_name }} on port {{ http_port }}"

    - name: Use facts
      debug:
        msg: "OS is {{ ansible_os_family }}"
```

Variable files:

```yaml
# vars/main.yml
app_name: myapp
app_port: 8080
app_env: production
```

```yaml
# playbook.yml
- hosts: all
  vars_files:
    - vars/main.yml
  tasks:
    - name: Show app name
      debug:
        msg: "{{ app_name }}"
```

```bash
# Pass an extra vars file at runtime
ansible-playbook site.yml -e "@vars/main.yml"
```

## 📁 Role Structure

Create role:

```bash
ansible-galaxy role init myrole
```

```text
myrole/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
└── vars/
    └── main.yml
```

Use roles:

```yaml
---
- hosts: webservers
  roles:
    - myrole
    - { role: nginx, nginx_port: 8080 }

  tasks:
    - name: Include role dynamically
      include_role:
        name: myrole
```

## 📦 Popular Modules

File operations:

```yaml
- name: Copy file
  copy:
    src: files/app.conf
    dest: /etc/app/app.conf
    owner: root
    group: root
    mode: '0644'

- name: Create directory
  file:
    path: /opt/app
    state: directory
    mode: '0755'

- name: Create symlink
  file:
    src: /opt/app/current
    dest: /opt/app/latest
    state: link

- name: Template a config file
  template:
    src: templates/app.conf.j2
    dest: /etc/app/app.conf

- name: Remove file
  file:
    path: /tmp/old_file
    state: absent
```

Package management:

```yaml
- name: Install package
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Install multiple
  apt:
    name:
      - nginx
      - curl
      - git
    state: present

- name: Remove package
  apt:
    name: apache2
    state: absent
```

Service management:

```yaml
- name: Start service
  service:
    name: nginx
    state: started
    enabled: yes

- name: Restart service
  service:
    name: nginx
    state: restarted

- name: Stop service
  service:
    name: nginx
    state: stopped
```

## 🔁 Conditionals & Loops

If conditions:

```yaml
- name: Install on Debian only
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"

- name: Install on RedHat only
  yum:
    name: nginx
    state: present
  when: ansible_os_family == "RedHat"

- name: Run only if variable is defined
  debug:
    msg: "{{ my_var }}"
  when: my_var is defined

- name: Multiple conditions (AND)
  debug:
    msg: "Matched"
  when:
    - ansible_distribution == "Ubuntu"
    - ansible_distribution_major_version == "22"
```

Loops:

```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git

- name: Create multiple users
  user:
    name: "{{ item.name }}"
    uid: "{{ item.uid }}"
  loop:
    - { name: alice, uid: 1001 }
    - { name: bob, uid: 1002 }

- name: Loop with index
  debug:
    msg: "{{ item.0 }} - {{ item.1 }}"
  with_indexed_items: "{{ my_list }}"
```

## ✨ Advanced Features

Handlers (notify):

```yaml
tasks:
  - name: Update nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx

handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
```

Filters:

```yaml
- name: Use default filter
  debug:
    msg: "{{ my_var | default('fallback') }}"

- name: Uppercase a string
  debug:
    msg: "{{ 'hello' | upper }}"

- name: Convert to JSON
  debug:
    msg: "{{ my_dict | to_json }}"

- name: Select items from a list
  debug:
    msg: "{{ users | selectattr('active', 'equalto', true) | list }}"

- name: Join a list into a string
  debug:
    msg: "{{ my_list | join(', ') }}"
```

## ✅ Best Practices

- Use roles for organization - Structure code in reusable roles
- Use variables for flexibility - Make playbooks configurable
- Test with `--check` - Preview changes before executing
- Use tags for selective runs - Run specific parts of playbook
- Leverage handlers - Use notify for service restarts

## 🔍 Troubleshooting

Debug mode:

```bash
ansible-playbook site.yml -vvv    # Very verbose output
ansible-playbook site.yml --step  # Interactive, step through tasks
```

Test connection:

```bash
ansible all -i hosts -m ping
ansible all -i hosts -m setup
```

Common Commands:

```bash
ansible-inventory --list    # List inventory
ansible-doc module_name     # Module documentation
```

## 📋 Quick Reference

| Command | Description | Example |
|---|---|---|
| `ansible` | Ad-hoc command | `ansible all -m ping` |
| `ansible-playbook` | Run playbook | `ansible-playbook site.yml` |
| `ansible-galaxy` | Manage roles | `ansible-galaxy role init` |
| `ansible-inventory` | Manage inventory | `ansible-inventory --list` |
| `ansible-doc` | Module docs | `ansible-doc apt` |
| `ansible-vault` | Encrypt data | `ansible-vault create secret.yml` |
| `--check` | Dry run | `ansible-playbook site.yml --check` |
| `-v`, `-vv`, `-vvv` | Verbosity | `ansible-playbook site.yml -vvv` |

## 🧩 Ansible Concepts

**File Modules**: `copy`, `file`, `template`, `lineinfile`, `unarchive`

**System Modules**: `service`, `user`, `group`, `cron`, `mount`

**Package Modules**: `apt`, `yum`, `dnf`, `pip`, `package`

**Network Modules**: `uri`, `get_url`, `ios_config`, `nxos_facts`, `junos_command`

---

*Source: adapted from the Ansible cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
