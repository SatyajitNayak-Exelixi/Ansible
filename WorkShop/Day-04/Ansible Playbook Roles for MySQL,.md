# Ansible Playbook for MySQL, Backend, and Frontend Setup

This document explains how MySQL is installed on the **database servers**, backend services on the **backend servers**, and frontend services on the **frontend servers** using **Ansible roles**.

---

## Step 1: Install Required Role from Ansible Galaxy

```bash
ansible-galaxy role install geerlingguy.mysql
```

* This command downloads and installs the **geerlingguy.mysql** role from **Ansible Galaxy** into your local Ansible roles directory.
* The role contains predefined tasks, handlers, templates, and default variables to automate the installation and configuration of **MySQL**.

---

## Step 2: Inventory File (`hosts`)

```ini
[frontend]
54.196.210.144

[backend]
3.91.205.203

[database]
54.160.160.110

[all:vars]
ansible_ssh_port=22
ansible_ssh_user=ubuntu
#ansible_ssh_password=password
ansible_ssh_private_key_file=/etc/ansible/Mamuni.pem
```

* **frontend** group → contains frontend server IP.
* **backend** group → contains backend server IP.
* **database** group → contains database server IP.
* Global variables under `[all:vars]` → define SSH connection details for all groups.

---

## Step 3: Main Playbook (`main.yaml`)

```yaml
- name: Database Playbook
  hosts: database
  become: yes
  roles:
    - geerlingguy.mysql

- name: Backend Playbook
  hosts: backend
  roles:
    - backend

- name: Frontend Playbook
  hosts: frontend
  roles:
    - geerlingguy.k8s_manifests
```

### Explanation

1. **Database Playbook**

   * Runs on hosts in the `[database]` group.
   * Uses the role `geerlingguy.mysql` which automates MySQL installation and setup.

2. **Backend Playbook**

   * Runs on hosts in the `[backend]` group.
   * Applies the custom role `backend` (found under `/etc/ansible/roles/backend`).

3. **Frontend Playbook**

   * Runs on hosts in the `[frontend]` group.
   * Uses the role `geerlingguy.k8s_manifests` (for Kubernetes manifest management).

---

## Step 4: Variables File (`vars.yaml`)

This file contains custom variables passed at runtime.

Example:

```yaml
mysql_root_password: mysecurepassword
mysql_databases:
  - name: myapp_db

mysql_users:
  - name: myapp_user
    host: "%"
    password: myapppassword
    priv: "myapp_db.*:ALL"
```

* Sets the **MySQL root password**.
* Creates a database `myapp_db`.
* Creates a user `myapp_user` with access to the database.

---

## Step 5: Running the Playbook

```bash
ansible-playbook -i hosts main.yaml -e @vars.yaml
```

### Explanation

* `-i hosts` → specifies the inventory file.
* `main.yaml` → the main playbook that runs database, backend, and frontend roles.
* `-e @vars.yaml` → passes external variables from `vars.yaml` file.

---

## How MySQL Gets Installed

1. When you run the playbook, Ansible connects to the **database server** (`54.160.160.110`) via SSH using the provided private key.
2. The **`geerlingguy.mysql`** role is executed, which:

   * Installs MySQL server package.
   * Configures root password.
   * Creates databases and users as defined in `vars.yaml`.
   * Ensures the MySQL service is running.
3. Backend and frontend roles are also executed on their respective servers.

---

✅ With this setup:

* Database server automatically gets MySQL installed and configured.
* Backend services are deployed on the backend server.
* Frontend services are deployed on the frontend server.
