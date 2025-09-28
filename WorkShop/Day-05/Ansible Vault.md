# Ansible Vault Demo: Deploy MySQL with your playbook and storing password in Vault

---

## Prerequisite (one-liner)

You need Ansible installed on your control node. If you're testing locally on the same machine, this will work with `ansible_connection=local`.

Install Ansible (Ubuntu example):

```bash
sudo apt update && sudo apt install -y ansible
```

---

## Step 1 — Create a working directory

```bash
mkdir -p ~/ansible-mysql-demo && cd ~/ansible-mysql-demo
```

---

## Step 2 — Create `inventory/hosts.ini` (use local test or real hosts)

**Local test (single machine):**

```bash
mkdir -p inventory
cat > inventory/hosts.ini <<'EOF'
[database]
localhost ansible_connection=local ansible_become=yes
EOF
```

**Remote example (replace IPs/user):**

```ini
# inventory/hosts.ini
[database]
db1 ansible_host=10.0.0.11

[database:vars]
ansible_user=ubuntu
ansible_become=yes
```

---

## Step 3 — Create `database.yaml` (use vaulted passwords)

Run this command to create the playbook file modified to **use vaulted variables** instead of hardcoded `password` strings. The rest of the playbook structure is unchanged — only password references use `{{ vault_deploy_password }}` and `{{ vault_mysql_root_password }}`.

```bash
cat > database.yaml <<'EOF'
- name: Ansible Playbook for Installing MySQL on Database Server
  hosts: database
  become: yes
  tasks:
    - name: Install MySQL Packages
      action: apt pkg={{ item }} state=latest
      with_items:
       - python3-mysqldb
       - mysql-server
       - mysql-client

    - name: Start the MySQL service
      action: service name=mysql state=started

    - name: Create deploy user for mysql
      mysql_user: user="deploy" host="%" password={{ vault_deploy_password }} priv=*.*:ALL,GRANT
      ignore_errors: yes

    - name: Ensure anonymous users are not in the database
      mysql_user: user='' host={{ item }} state=absent
      ignore_errors: yes
      with_items:
                  - 127.0.0.1
                  - ::1
                  - localhost

    - name: Update mysql root password for all root accounts
      mysql_user: name=root host={{ item }} password={{ vault_mysql_root_password }}
      ignore_errors: yes
      with_items:
                  - 127.0.0.1
                  - ::1
                  - localhost

    - name: Replace 127.0.0.1 with 0.0.0.0
      replace:
        path: /etc/mysql/mysql.conf.d/mysqld.cnf
        regexp: '127.0.0.1'
        replace: 0.0.0.0

    - name: Start the MySQL service
      action: service name=mysql state=restarted

    - name: Transfer SQL file on Database Server
      copy: src=dump.sql dest=/tmp/dump.sql mode=0755

    - name: Import file.sql similar to mysql -u <username> -p <password> < hostname.sql
      mysql_db:
       state: import
       name: all
       login_user: deploy
       login_password: {{ vault_deploy_password }}
       target: /tmp/dump.sql
EOF
```

### Step 3.1 — Create and encrypt the Vault file (store passwords)

Create the vault variables file and encrypt it so passwords are not in plaintext.

```bash
mkdir -p inventory/group_vars/vault
ansible-vault create inventory/group_vars/vault/vault.yml
```

When the editor opens, paste the following (replace values with your desired passwords):

```yaml
vault_deploy_password: "DeployS3cret"
vault_mysql_root_password: "SuperSecretRootP@ss"
```

Save and exit — the file will be encrypted.

**Run the playbook with vault** (interactive vault prompt):

```bash
ansible-playbook -i inventory/hosts.ini database.yaml --ask-vault-pass
```

Or, create a vault password file (demo only) and run non-interactively:

```bash
echo 'my_vault_password' > ~/.vault_pass.txt
chmod 600 ~/.vault_pass.txt
ansible-playbook -i inventory/hosts.ini database.yaml --vault-password-file ~/.vault_pass.txt
```

## Step 4 — Create `dump.sql` (schema file)

— Create `dump.sql` (schema file)

```bash
cat > dump.sql <<'EOF'
CREATE DATABASE IF NOT EXISTS company_db;
USE company_db;
CREATE TABLE IF NOT EXISTS employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  department VARCHAR(50),
  salary DECIMAL(10,2)
);
EOF
```

---

## Step 5 — Run the playbook

If you're running locally and using `ansible_connection=local`, run:

```bash
ansible-playbook -i inventory/hosts.ini database.yaml
```

If control -> remote hosts require privilege escalation and password prompt, run:

```bash
ansible-playbook -i inventory/hosts.ini database.yaml --ask-become-pass
```

Notes:

* The playbook as provided uses plaintext `password` for users. It will attempt to set root and `deploy` passwords to the string `password`.
* If you get failures related to `mysql_user` or authentication, install the `community.mysql` collection (see optional step below).

---

## Optional Step — Install community.mysql collection (if mysql modules fail)

If tasks using `mysql_user` or `mysql_db` fail because the module is not available, install the collection on control node:

```bash
ansible-galaxy collection install community.mysql
```

Then re-run the playbook.

---

## Step 6 — Verify the database & table (quick checks)

**Using Ansible ad-hoc (control node):**

```bash
# show databases via mysql client as root (not secure to put password on CLI, demo only)
ansible database -i inventory/hosts.ini -m ansible.builtin.command -a "mysql -u root -ppassword -e 'SHOW DATABASES;'" --become

# show tables in company_db as deploy
ansible database -i inventory/hosts.ini -m ansible.builtin.command -a "mysql -u deploy -ppassword -e 'USE company_db; SHOW TABLES;'" --become
```

**Manual on the host (local):**

```bash
sudo mysql -u deploy -ppassword -e "USE company_db; SHOW TABLES;"
```

---

## Step 7 — Cleanup (remove dump file)

```bash
ansible database -i inventory/hosts.ini -m ansible.builtin.file -a "path=/tmp/dump.sql state=absent" --become
```

---

## Final notes

* This file uses the exact playbook you supplied without changing its structure or passwords.
* It is a simple demo for learning. For production use, convert plaintext passwords to Ansible Vault and make the playbook idempotent and robust.

EOF
