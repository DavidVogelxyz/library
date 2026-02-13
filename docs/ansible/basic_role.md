Ansible - basic role
====================

[Back to Ansible main page](README.md)

Example basic role:

```yaml
---

- name: Show Ansible facts for a host
  ansible.builtin.debug:
    var: ansible_facts
```
