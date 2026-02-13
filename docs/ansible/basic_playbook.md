Ansible - basic playbook
========================

[Back to Ansible main page](README.md)

Example basic playbook:

```yaml
---

- name: "Show Ansible facts for a host"
  hosts: all
  roles:
    - debug
```
