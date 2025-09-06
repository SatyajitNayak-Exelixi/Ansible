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

👉 Use a custom inventory file (`inventory.ini`).

```bash
ansible all -m ping -i inventory.ini
```

👉 Test connectivity to all hosts using a custom inventory.

---

## 🔹 Ad-Hoc Commands

```bash
ansible all -m ping -i inventory.ini
```

👉 Check connectivity (ping module).

```bash
ansible all -m command -a "uptime" -i inventory.ini
```

👉 Run a command on all hosts.

```bash
ansible web -m shell -a "df -h" -i inventory.ini
```

👉 Run a shell command on `web` group.

```bash
ansible db -m yum -a "name=httpd state=present" -i inventory.ini
```

👉 Install a package using the `yum` module.

```bash
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts" -i inventory.ini
```

👉 Copy a file to remote hosts.

```bash
ansible all -m file -a "path=/tmp/test state=directory" -i inventory.ini
```

👉 Create a directory.

```bash
ansible all -m service -a "name=httpd state=started enabled=yes" -i inventory.ini
```

👉 Start and enable a service.

---

## 🔹 Playbooks

```bash
ansible-playbook main.yml -i inventory.ini
```

👉 Run a playbook.

```bash
ansible-playbook main.yml --syntax-check -i inventory.ini
```

👉 Check playbook syntax.

```bash
ansible-playbook main.yml --check -i inventory.ini
```

👉 Run playbook in check (dry-run) mode.

```bash
ansible-playbook main.yml --list-tasks -i inventory.ini
```

👉 List all tasks in a playbook.

```bash
ansible-playbook main.yml --list-hosts -i inventory.ini
```

👉 Show hosts targeted by a playbook.

```bash
ansible-playbook main.yml --start-at-task="Install Packages" -i inventory.ini
```

👉 Start execution from a specific task.

```bash
ansible-playbook main.yml --step -i inventory.ini
```

👉 Run playbook step by step.

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
ansible-playbook main.yml --ask-vault-pass -i inventory.ini
```

👉 Run a playbook with vault password prompt.

```bash
ansible-playbook main.yml --vault-password-file .vault_pass.txt -i inventory.ini
```

👉 Run playbook with a password file.

---

## 🔹 Troubleshooting & Debugging

```bash
ansible all -m ping -vvv -i inventory.ini
```

👉 Increase verbosity for debugging.

```bash
ansible-playbook main.yml -vvv -i inventory.ini
```

👉 Debug playbook execution.

```bash
ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook main.yml -i inventory.ini
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
inventory = ./inventory.ini
host_key_checking = False
retry_files_enabled = False
```

👉 Common settings in `ansible.cfg`.
