Ansible - basic playbook
========================

Example basic playbook:

```yaml
---

- name: "Show Ansible facts for a host"
  hosts: all
  roles:
    - debug
```
