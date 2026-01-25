+++
date = '2026-01-25T00:35:54+03:00'
draft = false 
title = 'Using Ansible facts to manage state'
tags = [ "devops", "ansible", "linux" ]
authors = [ "Yusuf Yakubov" ]
+++

Sometimes you need to declaratively manage packages on your target system but absense of state in ansible makes it inconvinient

Here is the simple pattern based on **ansible_local facts** that i found on the internet that may help you

Basically it holds state information in a file **`/etc/ansible/facts.d/*.fact`** in a json/ini format and to reference it u can use **`{{ ansible_local.* }}`**

In this example we manage package creation but you can adapt it to yout own needs like maybe managing ssh authorized_keys file, etc.

```yaml
- name: Defining packages to install
  set_fact:
    packages_to_install:
      - curl
      - git
      - vim

- name: Installing a bunch of packages
  apt:
    name: "{{ packages_to_install }}"
    state: present

- name: Checking if state has changed
  apt:
    name: "{{ item }}"
    state: absent
  loop: "{{ ansible_local.packages | default(packages_to_install) | difference(packages_to_install) }}"
  register: removed_packages

- name: Ensuring that facts.d directory exists
  file:
    state: directory
    path: /etc/ansible/facts.d
    recurse: true
    owner: root
    mode: "0644"

- name: Adding current state to packages.fact
  copy:
    content: "{{ packages_to_install | to_json }}"
    dest: /etc/ansible/facts.d/packages.fact
    owner: root
    mode: "0644"
  when: removed_packages.changed
```

## References

1. Discovering variables: facts and magic variables # adding custom facts - <https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_vars_facts.html#adding-custom-facts>
