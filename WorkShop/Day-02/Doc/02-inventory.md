# Ansible Inventory

Ansible **inventory** is a critical component that defines the **hosts (remote systems)** and the **groups** they belong to. It tells Ansible *which machines* to manage and *how to connect* to them.

Inventories can be **static** (manually defined) or **dynamic** (fetched automatically from a source like AWS or Azure).

---

## 🗂️ Static Inventory

A **static inventory** file is a simple text file, typically named `hosts` or `inventory`, written in either **INI** or **YAML** format.

### 🔹 INI Format Example

```
# inventory file: hosts

[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=/path/to/key
```

➡️ Here:

* **[webservers]** and **[dbservers]** are groups.
* The `all:vars` section defines variables applied to all hosts.

---

### 🔹 YAML Format Example

```
# inventory file: hosts.yaml

all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /path/to/key
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    dbservers:
      hosts:
        db1.example.com:
        db2.example.com:
```

➡️ YAML format is more structured and preferred for complex inventories.

---

## ☁️ Dynamic Inventory

A **dynamic inventory** is generated programmatically — perfect for **cloud environments** where servers frequently change.

It retrieves host information dynamically from providers like **AWS**, **GCP**, or **Azure**.

### Example: AWS EC2 Dynamic Inventory Script (Python)

```
#!/usr/bin/env python3

import json
import boto3

def get_aws_ec2_inventory():
    ec2 = boto3.client('ec2')
    instances = ec2.describe_instances()

    inventory = {
        'all': {
            'hosts': [],
            'vars': {
                'ansible_user': 'ec2-user',
                'ansible_ssh_private_key_file': '/path/to/key'
            }
        },
        '_meta': {
            'hostvars': {}
        }
    }

    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] == 'running':
                public_ip = instance.get('PublicIpAddress')
                if public_ip:
                    inventory['all']['hosts'].append(public_ip)
                    inventory['_meta']['hostvars'][public_ip] = {
                        'ansible_host': public_ip
                    }

    print(json.dumps(inventory, indent=2))

if __name__ == '__main__':
    get_aws_ec2_inventory()
```

➡️ Save this file as `aws_ec2_inventory.py`, make it executable, and use it as your dynamic inventory.

---

## ⚙️ Using the Inventory

Run any Ansible **playbook** or **ad-hoc command** using your inventory file:

```
ansible-playbook -i hosts site.yml
```

Or with a dynamic inventory script:

```
ansible-playbook -i aws_ec2_inventory.py site.yml
```

---

## ✅ Summary

| Type                  | Description                                    | Use Case                           |
| --------------------- | ---------------------------------------------- | ---------------------------------- |
| **Static Inventory**  | Manually defined hosts in INI/YAML             | Small or fixed environments        |
| **Dynamic Inventory** | Generated dynamically using plugins or scripts | Cloud or auto-scaling environments |

Ansible’s inventory flexibility allows it to manage everything from a few servers to thousands of dynamic cloud instances efficiently.
