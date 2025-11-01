# Understanding YAML

YAML (**YAML Ain't Markup Language**) is a **human-readable data serialization format** that is widely used for configuration files, automation tools like **Ansible**, and data exchange between systems that use different data structures.

---

## YAML Syntax Basics

YAML represents data in a structured and easy-to-read way using **key-value pairs**, **lists**, and **dictionaries**.

### 🟢 Strings, Numbers, and Booleans

```
string: "Hello, World!"
number: 42
boolean: true
```

* Strings can be written with or without quotes.
* Numbers and booleans are written directly (no quotes needed).

---

### 🍎 Lists

Lists (or arrays) are written using a hyphen `-` followed by a space.

```
fruits:
  - Apple
  - Orange
  - Banana
```

This represents a list of three items: Apple, Orange, and Banana.

---

### 👤 Dictionary (or Mapping)

Dictionaries represent key-value pairs and are indented for hierarchy.

```
person:
  name: John Doe
  age: 30
  city: New York
```

This describes a `person` object with `name`, `age`, and `city` keys.

---

### 🧩 List of Dictionaries

YAML allows you to nest lists and dictionaries to describe more complex data structures.

```
family:
  parents:
    - name: Jane
      age: 50
    - name: John
      age: 52
  children:
    - name: Jimmy
      age: 22
    - name: Jenny
      age: 20
```

In this example:

* `parents` is a list of two dictionaries.
* `children` is also a list of dictionaries.

---

## ⚙️ Key Rules to Remember

* **Indentation matters** – use spaces (not tabs).
* **Consistent spacing** – 2 spaces per level is common.
* **Colons (`:`)** separate keys and values.
* **Hyphens (`-`)** indicate list items.
* **Comments** start with a `#` symbol.

Example:

```
# This is a YAML comment
server:
  host: localhost
  port: 8080
```

---

## ✅ Summary

YAML is simple, flexible, and widely adopted in DevOps tools like **Ansible**, **Docker**, and **Kubernetes**.
It provides a clear and concise way to represent structured data while remaining easy for humans to read and write.
