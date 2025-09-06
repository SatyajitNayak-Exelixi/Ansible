# Ansible Command Cheat Sheet

This guide provides the most commonly used **Ansible commands** along with their descriptions. Use it as a quick reference or teaching material.

---

## 🔹 Installation & Setup

```bash
ansible --version
```

👉 Check the installed Ansible version.

```bash
sudo apt update
sudo apt install ansible -y
```

👉 Install Ansible on **Ubuntu/Debian systems**.

```bash
sudo yum install ansible -y
```

👉 Install Ansible on **RHEL/CentOS systems**.

```bash
pip install ansible
```

👉 Install Ansible using **pip** (works across systems, but not recommended for production).

---

## 🔹 Inventory Management

```bash
cat /etc/ansible/hosts
```

👉 Default inventory file location.

```bash
ansible all --list-hosts
```

👉 List all hosts from the inventory.

```bash
ansible all -i inventory.ini --list-hosts
```

👉 Use a custom inventory file.

```bash
ansible all -m ping
```

👉 Test connectivity to all hosts.

---

## 🔹 Ad-Hoc Commands

```bash
ansible all -m ping
```

👉 Check connectivity (ping module).

```bash
ansible all -m command -a "uptime"
```

👉 Run a command on all hosts.

```bash
ansible web -m shell -a "df -h"
```

👉 Run a shell command on `web` group.

```bash
ansible db -m yum -a "name=httpd state=present"
```

👉 Install a package using the `yum` module.

```bash
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```

👉 Copy a file to remote hosts.

```bash
ansible all -m file -a "path=/tmp/test state=directory"
```

👉 Create a directory.

```bash
ansible all -m service -a "name=httpd state=started enabled=yes"
```

👉 Start and enable a service.

---

## 🔹 Playbooks

```bash
ansible-playbook site.yml
```

👉 Run a playbook.

```bash
ansible-playbook site.yml --syntax-check
```

👉 Check playbook syntax.

```bash
ansible-playbook site.yml --list-tasks
```

👉 List all tasks in a playbook.

```bash
ansible-playbook site.yml --list-hosts
```

👉 Show hosts targeted by a playbook.

```bash
ansible-playbook site.yml --start-at-task="Install Packages"
```

👉 Start execution from a specific task.

```bash
ansible-playbook site.yml --step
```

👉 Run playbook step by step.

```bash
ansible-playbook site.yml -C
```

👉 Run in check (dry-run) mode.

---

## 🔹 Roles & Galaxy

```bash
ansible-galaxy init myrole
```

👉 Create a new Ansible role.

```bash
ansible-galaxy install geerlingguy.apache
```

👉 Install a role from Ansible Galaxy.

```bash
ansible-galaxy list
```

👉 List installed roles.

---

## 🔹 Ansible Vault (Secrets Management)

```bash
ansible-vault create secrets.yml
```

👉 Create a new encrypted file.

```bash
ansible-vault edit secrets.yml
```

👉 Edit an encrypted file.

```bash
ansible-vault view secrets.yml
```

👉 View an encrypted file.

```bash
ansible-vault encrypt file.yml
```

👉 Encrypt an existing file.

```bash
ansible-vault decrypt file.yml
```

👉 Decrypt a file.

```bash
ansible-playbook site.yml --ask-vault-pass
```

👉 Run a playbook with vault password prompt.

```bash
ansible-playbook site.yml --vault-password-file .vault_pass.txt
```

👉 Run playbook with a password file.

---

## 🔹 Troubleshooting & Debugging

```bash
ansible all -m ping -vvv
```

👉 Increase verbosity for debugging.

```bash
ansible-playbook site.yml -vvv
```

👉 Debug playbook execution.

```bash
ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook site.yml
```

👉 Print detailed task output.

---

## 🔹 Useful Configurations

```bash
cat /etc/ansible/ansible.cfg
```

👉 Default configuration file.

```ini
[defaults]
inventory = ./inventory
host_key_checking = False
retry_files_enabled = False
```

👉 Common settings in `ansible.cfg`.

---

