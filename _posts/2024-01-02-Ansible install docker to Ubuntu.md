---
title: Ansible install docker to Ubuntu 
date: 2024-01-02 11:11:00 -0700
categories: [Linux]
tags: Ansible
Pin:
---

#### About ansible

Ansible is a radically simple IT automation system. It handles configuration management, application deployment, cloud provisioning, ad-hoc task execution, network automation, and multi-node orchestration. https://github.com/ansible/ansible

#### Install ansible with `pip`

please make sure you have install python3.

To verify whether `pip` is already installed for your preferred Python

```shell
python3 -m pip -V
```

I use the python enve environment install ansible

```shell
python3 -m venv .ansible_python # new venv environment named ansible_python

source .ansible_python/bin/activate # activate the environment
```

Then install the ansible

```
pip install ansible
```

#### Set ansible configration

```shell
vim /home/USER1/.ansible/ansible.cfg
```

```shell
[web_server]
# name ansible_host ansible_ssh_user ansible_ssh_private_key_file 
web1 ansible_host=192.168.1.21 ansible_ssh_user=user1 ansible_ssh_private_key_file=/home/USER1/.ssh/id_ed25519
web2 ansible_host=192.168.1.46 ansible_ssh_user=user1 ansible_ssh_private_key_file=/home/USER1/.ssh/id_ed25519
web3 ansible_host=172.xxx.xxx.xxx ansible_ssh_user=root ansible_ssh_private_key_file=/home/USER1/.ssh/id_ed25519
```

more information please visit https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups

#### My first ansible test

```
ansible -i /home/USER1/.ansible/ansible.cfg web1 -m shell -a "df -h"
```

All server 

```
ansible -i /home/USER1/.ansible/ansible.cfg all -m shell -a "df -h"
```

#### Use `ansible-playbook` install docker to the server

Add new ansible-playbook configuration file

```shell
vim /home/USER1/.ansible/docker_install_playbook.yml
```

```yaml
---
- hosts: all
  become: true

  tasks:
    - name: Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: true

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
        state: latest
        update_cache: true

    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true
```

##### install to web3 server

```
ansible-playbook docker_install_playbook.yml -i  /home/USER1/.ansible/ansible.cfg -l web3
```

