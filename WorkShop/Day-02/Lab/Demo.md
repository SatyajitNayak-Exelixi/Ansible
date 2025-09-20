# Ansible Apache Web Server Installation Guide

## Step 1: Install Apache (apache.yaml)

**apache.yaml**

```yaml
---
- hosts: development
  tasks:
     - name: Install apache2 package
       apt: name=apache2 update_cache=yes state=latest

     - name: Enable Mod Rewrite
       apache2_module: name=rewrite state=present

     - name: Restart Service
       service: name=apache2 state=restarted
```

Run the playbook:

```bash
ansible-playbook apache.yaml -b
```

This will:

* Install Apache2
* Enable mod\_rewrite
* Restart Apache service

---

## Step 2: Cleanup Apache (Optional)

Remove Apache completely:

```bash
ansible -m shell -a "apt -y remove --purge apache2" webservers -b
```

> Stop Nginx before running this if it is active.

---

## Step 3: Deploy Website (site.yml)

**site.yml**

```yaml
---
- hosts: webservers
  become: yes
  vars:
    git_repo: "https://github.com/mdn/beginner-html-site-styled.git"
    git_version: "gh-pages"

  tasks:
    - name: Run apt-get update
      apt:
        update_cache: yes

    - name: Install apache2 package
      apt:
        name: apache2
        state: latest
        update_cache: yes

    - name: Enable Mod Rewrite
      apache2_module:
        name: rewrite
        state: present

    - name: Restart Apache service
      service:
        name: apache2
        state: restarted

    - name: Clean Old Code (remove /var/www/html)
      file:
        path: /var/www/html
        state: absent

    - name: Checking out code from GitHub
      git:
        repo: "{{ git_repo }}"
        dest: /var/www/html
        version: "{{ git_version }}"
```

Run the playbook:

```bash
ansible-playbook site.yml
```

This will:

* Update apt cache
* Install Apache2 if missing
* Enable mod\_rewrite
* Restart Apache service
* Remove old code in `/var/www/html`
* Clone MDN Beginner HTML Site from GitHub to `/var/www/html`

---

## Step 4: Verify Deployment

Check the website:

```bash
curl http://<server-ip>/
```

Or open in browser:

```
http://<server-ip>/
```
