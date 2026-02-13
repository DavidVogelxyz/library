Ansible - basic role
====================

Example basic role:

```yaml
---

- name: Show Ansible facts for a host
  ansible.builtin.debug:
    var: ansible_facts
```
