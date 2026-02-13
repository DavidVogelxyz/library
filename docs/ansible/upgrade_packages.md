Ansible - upgrade packages on different distros
===============================================

[Back to Ansible main page](README.md)

Update package lists:

```yaml
---

- name: apk - update
  when: ansible_facts['pkg_mgr'] == "apk"
  become: true
  community.general.apk:
    update_cache: true

- name: apt - update
  when: ansible_facts['pkg_mgr'] == "apt"
  become: true
  ansible.builtin.apt:
    update_cache: true

- name: pacman - update
  when: ansible_facts['pkg_mgr'] == "pacman"
  become: true
  community.general.pacman:
    update_cache: true
```

Upgrade packages:

```yaml
---

- name: apk - upgrade
  when: ansible_facts['pkg_mgr'] == "apk"
  become: true
  community.general.apk:
    upgrade: true

- name: apt - upgrade
  when: ansible_facts['pkg_mgr'] == "apt"
  become: true
  ansible.builtin.apt:
    name: "*"
    state: latest

- name: dnf - upgrade
  when: ansible_facts['pkg_mgr'] == "dnf"
  become: true
  ansible.builtin.dnf:
    name: "*"
    state: latest

- name: pacman - upgrade
  when: ansible_facts['pkg_mgr'] == "pacman"
  become: true
  community.general.pacman:
    upgrade: true
```

Autoremove (`apt`):

```yaml
---

- name: apt - autoremove
  when: ansible_facts['pkg_mgr'] == "apt"
  become: true
  ansible.builtin.apt:
    autoremove: yes
```
