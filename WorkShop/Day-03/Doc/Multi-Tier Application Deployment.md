# Multi-Tier Application Deployment using Ansible

This guide explains how to set up and deploy a **multi-tier application architecture** using Ansible, consisting of a **Database Server** and a **Backend Server**.

---

## 1. Launch Instances & Initial Setup

1. **Launch 4 Instances** on AWS (or your cloud provider).

   * Configure Security Group → **Allow All** traffic.

2. **Login as Root User** and install Ansible:

   ```bash
   sudo su -
   apt update && apt -y install ansible
   ```

3. **Create Ansible directory**:

   ```bash
   mkdir /etc/ansible
   ```

4. **Copy PEM file**:

   ```bash
   cp ansible.pem /etc/ansible/kubernetes.pem
   ```

5. **Disable Host Key Checking**:

   ```bash
   export ANSIBLE_HOST_KEY_CHECKING=False
   ```

---

## 2. Configure Ansible Inventory

Create the **hosts** file:

```ini
[frontend]
107.20.48.131

[backend]
52.55.80.236

[database]
172.31.21.151

[all:vars]
ansible_ssh_port=22
ansible_ssh_user=ubuntu
#ansible_ssh_password=password
ansible_ssh_private_key_file=/etc/ansible/kubernetes.pem
```

### Test Connectivity

```bash
ansible -m ping all
```

---

## 3. Database Setup (MySQL)

### Create **database.yaml**

```yaml
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
      mysql_user: user="deploy" host="%" password=password priv=*.*:ALL,GRANT
      ignore_errors: yes

    - name: Ensure anonymous users are not in the database
      mysql_user: user='' host={{ item }} state=absent
      ignore_errors: yes
      with_items:
                  - 127.0.0.1
                  - ::1
                  - localhost

    - name: Update mysql root password for all root accounts
      mysql_user: name=root host={{ item }} password=password
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

    - name: Import SQL file
      mysql_db:
       state: import
       name: all
       login_user: deploy
       login_password: password
       target: /tmp/dump.sql
```

### Create **dump.sql**

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

### Run the Playbook

```bash
ansible-playbook database.yaml
```

### Validate Database

```bash
mysql -u deploy -p
Enter password: password
mysql> show databases;
```

Or connect from **Master Node**:

```bash
sudo apt update
sudo apt install mysql-client-core-8.0 -y
mysql -h <DB_PUBLIC_IP> -P 3306 -u deploy -p
```

### Insert Test Data

```sql
INSERT INTO employees (first_name, last_name, department, salary)
VALUES ('Rahul', 'Nayak', 'DEVOPS', 45000.00);

SHOW TABLES;
SELECT * FROM employees;
```

---

## 4. Backend Setup (NodeJS)

### Create **backend.yaml**

```yaml
---
- hosts: backend
  become: yes
  vars:
    - dbhost: 3.89.180.33
    - dbusername: deploy
    - dbpassword: password
  tasks:
     - name: Update apt cache
       apt:
        update_cache: yes

     - name: Install NodeJS and Dependencies
       apt: name={{ item }} state=latest
       loop:
         - nodejs
         - unzip
         - npm

     - name: Copy NodeJS code on Application servers
       get_url: url=https://github.com/anujdevopslearn/NodeJSMySQLRestAPI/archive/refs/heads/master.zip dest=/tmp/artifact.zip

     - name: Extract Code
       unarchive: src=/tmp/artifact.zip dest=/opt/ remote_src=True

     - name: Replace Host
       replace:
        path: /opt/NodeJSMySQLRestAPI-master/src/config/db.config.ts
        regexp: '^(.*)HOST(.*)'
        replace: 'HOST: "{{ dbhost }}",'

     - name: Replace Username
       replace:
        path: /opt/NodeJSMySQLRestAPI-master/src/config/db.config.ts
        regexp: '^(.*)USER(.*)'
        replace: 'USER: "{{ dbusername }}",'

     - name: Replace Password
       replace:
        path: /opt/NodeJSMySQLRestAPI-master/src/config/db.config.ts
        regexp: '^(.*)PASSWORD(.*)'
        replace: 'PASSWORD: "{{ dbpassword }}",'

     - name: Run Node Install & Start Service
       shell: |
        npm install
        npm install forever -g
        npm run build
        forever stop backend-api | true
        forever start --uid "backend-api" -a ./build/server.js
        forever list
       args:
         chdir:  /opt/NodeJSMySQLRestAPI-master
```

### Pre-setup Database for Backend

```sql
create database testdb; use testdb;

CREATE TABLE IF NOT EXISTS tutorials (
 id int NOT NULL PRIMARY KEY AUTO_INCREMENT,
 title varchar(255) NOT NULL,
 description varchar(255),
 published boolean DEFAULT false
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO tutorials (title, description, published)
VALUES
 ('Introduction to SQL', 'Learn the basics of SQL, including queries and joins.', true),
 ('Advanced SQL Techniques', 'Explore subqueries, indexes, and optimization.', false),
 ('Node.js for Beginners', 'Start building backend applications using Node.js.', true),
 ('REST APIs with Express', 'Learn how to create RESTful APIs using Express.js.', false),
 ('Frontend Basics', 'HTML, CSS, and JavaScript essentials.', true),
 ('Angular Crash Course', 'Build dynamic web applications using Angular framework.', false),
 ('React Fundamentals', 'Understand React components, state, and props.', true),
 ('Database Design', 'Learn how to design relational databases efficiently.', false),
 ('Docker Essentials', 'Containerize your applications with Docker.', true),
 ('DevOps Introduction', 'Basics of CI/CD, Jenkins, and cloud deployments.', false);
```

### Run Backend Playbook

```bash
ansible-playbook backend.yaml
```

### Security Group Update

Modify **Backend Security Group**:

| Type       | Protocol | Port Range | Source    |
| ---------- | -------- | ---------- | --------- |
| Custom TCP | TCP      | 8080       | 0.0.0.0/0 |

### Validate Backend Service

Open in Browser:

```
http://<BACKEND_PUBLIC_IP>:8080/api/tutorials
```

Check backend service:

```bash
forever list
```

To restart backend:

```bash
forever restart 0
```

---

✅ **At this point, your Database and Backend services are successfully deployed using Ansible!**
