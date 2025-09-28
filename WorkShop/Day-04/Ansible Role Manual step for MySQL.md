# 🚀 Ansible Role Demo: MySQL Setup

This demo shows how to create an **Ansible role for MySQL** and use it in a playbook to set up a database, user, and import schema.

---

## 📂 Folder Structure

```
/etc/ansible/
│── main.yaml
│── roles/
    └── mysql/
        ├── tasks/
        │   └── main.yml
        ├── files/
        │   └── dump.sql
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
```

---

## 1️⃣ Initialize Role

Run inside your roles directory:

```bash
ansible-galaxy init mysql
```

---

## 2️⃣ Create Main Playbook

**`main.yaml`**

```yaml
- name: Database Playbook
  hosts: database
  become: yes
  roles:
    - mysql

- name: Backend Playbook
  hosts: backend
  roles:
    - backend

- name: Frontend Playbook
  hosts: frontend
  roles:
    - frontend
```

---

## 3️⃣ Add MySQL Tasks

**`roles/mysql/tasks/main.yml`**

```yaml
---
# tasks file for mysql

- name: Install MySQL Packages
  apt:
    name: "{{ item }}"
    state: latest
  loop:
    - python3-mysqldb
    - mysql-server
    - mysql-client
  become: yes

- name: Start the MySQL service
  service:
    name: mysql
    state: started
    enabled: yes
  become: yes

- name: Create deploy user for mysql
  mysql_user:
    name: deploy
    host: '%'
    password: "password"
    priv: '*.*:ALL,GRANT'
    state: present
  ignore_errors: yes
  become: yes

- name: Ensure anonymous users are not in the database
  mysql_user:
    name: ''
    host: "{{ item }}"
    state: absent
  loop:
    - 127.0.0.1
    - ::1
    - localhost
  ignore_errors: yes
  become: yes

- name: Update mysql root password for all root accounts
  mysql_user:
    name: root
    host: "{{ item }}"
    password: "password"
    state: present
  loop:
    - 127.0.0.1
    - ::1
    - localhost
  ignore_errors: yes
  become: yes

- name: Replace 127.0.0.1 with 0.0.0.0 in mysqld.cnf
  replace:
    path: /etc/mysql/mysql.conf.d/mysqld.cnf
    regexp: '127\\.0\\.0\\.1'
    replace: '0.0.0.0'
  become: yes
  notify:
    - Restart mysql

- name: Restart the MySQL service
  service:
    name: mysql
    state: restarted
    enabled: yes
  become: yes

- name: Transfer SQL file to database server
  copy:
    src: dump.sql
    dest: /tmp/dump.sql
    mode: '0755'
    owner: root
    group: root
  become: yes

- name: Import SQL dump into MySQL
  mysql_db:
    state: import
    name: company_db
    login_user: "{{ db_username }}"
    login_password: "{{ db_password }}"
    target: /tmp/dump.sql
  become: yes
```

---

## 4️⃣ Add SQL Schema

**`roles/mysql/files/dump.sql`**

```sql
CREATE DATABASE IF NOT EXISTS company_db;
USE company_db;
CREATE TABLE IF NOT EXISTS employees (
  id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50),
  department VARCHAR(50),
  salary DECIMAL(10,2)
);
```

---

## 5️⃣ Define Defaults

**`roles/mysql/defaults/main.yml`**

```yaml
---
db_username: deploy
db_password: password
```

---

## 6️⃣ Add Handlers

**`roles/mysql/handlers/main.yml`**

```yaml
---
# handlers file for mysql
- name: Restart mysql
  service:
    name: mysql
    state: restarted
```

---

## 7️⃣ Run the Playbook

```bash
ansible-playbook -i inventory main.yaml
```

---

## ✅ End Result

* MySQL installed & running
* `deploy` user created with full privileges
* Root password updated
* Remote access enabled (`0.0.0.0`)
* Database `company_db` created with `employees` table

---
