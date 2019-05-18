# Ansible

## Install

**Arch:** `sudo pacman -S ansible`

**Ubuntu:**
```  
sudo apt update
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

## Define hosts
**What:** servers will be installed
**Where:** `/etc/ansible/hosts`

*Example:*
```
[hieudeptrai]
178.128.55.12
178.128.51.14

[hieudeptrai:vars]
ansible_ssh_user=root
ansible_ssh_private_key_file=/home/hieudeptrai/.ssh/id_rsa
```
- `hieudeptrai` is group name
- Provide ssh key & user `ansible_ssh_user`, `ansible_ssh_private_key_file`

## How to run

1. Create `ansible.yml` file
2. Run `ansible-playbook ansible.yml`

## Example `ansible.yml`

Example 1: Install docker

```yml
- hosts: hieudeptrai
  remote_user: hieudeptrai
  become: yes
  become_method: sudo

  vars:
    - user: "hieudeptrai"

  tasks:
    - name: Add docker gpg key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: set up the repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ansible_distribution_release}} stable
        state: present

    - name: Install docker
      apt:
        name: "{{item}}"
        state: present
      with_items:
        - apt-transport-https
        - ca-certificates
        - curl
        - software-properties-common
        - docker-ce
        - docker-compose

    - name: add group docker
      user:
        name: "{{user}}"
        groups: docker
        append: yes
```
Example 2: Install golang
```yml
- hosts: hieudeptrai
  remote_user: hieudeptrai
  become: yes
  become_method: sudo

  vars:
    - user: "hieudeptrai"

  tasks:
    - name: Add Go repository
      apt_repository:
        repo: ppa:longsleep/golang-backports

    - name: Install golang
      apt:
        name: golang-go
        state: present
```

## Ansible Galaxy:

**What:** `Docker` has `Docker Hub`. `Ansible` has `Ansible Galaxy`.

Example, I want to setup FTP server. It has many steps. But I use https://galaxy.ansible.com/weareinteractive/vsftpd. It's called `role`

1. Download role from Ansible Galaxy: `ansible-galaxy install weareinteractive.vsftpd`
2. Create `ansible.yml`

```yml
- hosts: hieudeptrai
  remote_user: root
  roles:
    - weareinteractive.vsftpd
  vars:
    vsftpd_users:
      - username: hieudeptrai
        name: hieudeptrai
        password: "{{ 'hieudeptrai' | password_hash('sha256', 'mysecretsalt') }}"
    vsftpd_config:
      listen: NO
      listen_ipv6: YES
      anonymous_enable: NO
      local_enable: YES
      write_enable: YES
      local_umask: 022
      dirmessage_enable: YES
      use_localtime: YES
      chroot_local_user: YES
      allow_writeable_chroot: YES
```
