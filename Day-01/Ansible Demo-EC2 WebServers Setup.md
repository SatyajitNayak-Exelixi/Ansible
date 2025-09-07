# Ansible Demo: EC2 **WebServers** Setup & Ad‑hoc Commands

This walkthrough shows how to prepare an Ansible inventory for Ubuntu EC2 instances, fix SSH issues, and run ad‑hoc commands (including installing Java). It mirrors the exact flow you’ll demo, including the initial failure and the fix.

> **Working directory:** `/etc/ansible`

---

## 1) Prerequisites

* Ansible installed on the control node (the machine you’re running commands from).
* Your PEM key placed at: `/etc/ansible/Ansible.pem`.
* Target hosts (Ubuntu) reachable on port 22 with the `ubuntu` user.

Disable host key checking for the demo to avoid interactive prompts:

```bash
export ANSIBLE_HOST_KEY_CHECKING=False
```

> ⚠️ For production, consider keeping host key checking enabled.

---

## 2) Create the inventory and show the **initial failure**

Create (or edit) `/etc/ansible/hosts` **without** connection vars first:

```ini
[WebServers]
172.31.30.74
172.31.30.89
172.31.25.66
172.31.28.241
```

Test connectivity (this will **fail** due to missing user/key info):

```bash
ansible -m ping WebServers
```

You should see an error like `UNREACHABLE!` or `Permission denied (publickey)`, demonstrating the initial problem.

---

## 3) Add connection variables (the fix)

Append these group vars to `/etc/ansible/hosts`:

```ini
[WebServers]
172.31.30.74
172.31.30.89
172.31.25.66
172.31.28.241

[WebServers:vars]
ansible_ssh_port=22
ansible_ssh_user=ubuntu
# ansible_ssh_password=password
ansible_ssh_private_key_file=/etc/ansible/Ansible.pem
```

> ✅ Tip: `ansible_user` is the modern alias for `ansible_ssh_user`, but either works.

---

## 4) Fix key permissions

Your PEM key must not be group/other-readable:

```bash
chmod 400 /etc/ansible/Ansible.pem
```

---

## 5) Verify connectivity

Ping all hosts:

```bash
ansible -m ping WebServers
```

Run a simple command on all hosts (prints each hostname):

```bash
ansible -m shell -a "hostname" WebServers
```

> If you keep inventories elsewhere, add `-i /path/to/hosts`. Here we rely on `/etc/ansible/hosts`.

---

## 6) Demonstrate a command that **fails without sudo**

Try installing Java **without** privilege escalation (expect failure):

```bash
ansible -m shell -a "apt update && apt -y install default-jdk" WebServers
```

You’ll see permission errors because `apt` requires elevated privileges.

---

## 7) Run the same with **sudo** (become)

Now run with `--become` so Ansible uses sudo:

```bash
ansible -m shell -a "apt update && apt -y install default-jdk" WebServers --become
```

> ✅ Best practice (optional to show): use the `apt` module instead of `shell`:

```bash
ansible -m apt -a "update_cache=true name=default-jdk state=present" WebServers --become
```

If your remote user needs a password for sudo, add `--ask-become-pass` (or configure passwordless sudo for the demo).

---

## 8) Common pitfalls to call out during the demo

* **Group name mismatch:** Use `WebServers` consistently (not `WebServer`).
* **Wrong key perms:** Ensure `chmod 400` on the PEM or SSH will refuse the key.
* **User mismatch:** Use the correct default SSH user for your AMI (`ubuntu` for Ubuntu, `ec2-user` for Amazon Linux, etc.).
* **Security note:** Disabling host key checking is for demos; enable it outside of demos.

---

## 9) Quick reference (cheat sheet)

```bash
# 1) Disable host key checking (demo only)
export ANSIBLE_HOST_KEY_CHECKING=False

# 2) Inventory path (default)
cat /etc/ansible/hosts

# 3) Fix key perms
chmod 400 /etc/ansible/Ansible.pem

# 4) Connectivity tests
ansible -m ping WebServers
ansible -m shell -a "hostname" WebServers

# 5) Package install (will fail without sudo)
ansible -m shell -a "apt update && apt -y install default-jdk" WebServers

# 6) Package install with sudo
ansible -m shell -a "apt update && apt -y install default-jdk" WebServers --become
# or (best practice)
ansible -m apt -a "update_cache=true name=default-jdk state=present" WebServers --become
```

---
