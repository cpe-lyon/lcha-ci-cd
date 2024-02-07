# TP part 03 - Ansible

## Inventory

```yml
all:
 vars:
   ansible_user: centos #User to connect to the server
   ansible_ssh_private_key_file: ~/Downloads/id_rsa #Path to the private key
 children:
   prod:
     hosts: louis.charnay.takima.cloud #Address of the server
```

## Playbook

```yml
- hosts: all
  gather_facts: false
  become: true

  tasks:

  - name: Create network
    docker_network:
      name: tp03-network
      state: present
      roles:
        - name: create-network

  - name: Run Database
    docker_container:
      name: database
      image: leondumestre/tp-devops-database:1.0
      env:
        POSTGRES_USER: usr
        POSTGRES_PASSWORD: pwd
      networks:
        - name: tp03-network

  - name: Run App
    docker_container:
      name: backend
      image: leondumestre/tp-devops-backend:1.0
      env:
        DATABASE_URL: jdbc:postgresql://database:5432/db
        DATABASE_USER: usr
        DATABASE_PASSWORD: pwd
      networks:
        - name: tp03-network
      ports:
        - "8080:8080"

  - name: Run HTTPD
    docker_container:
      name: httpd
      image: leondumestre/tp-devops-httpd:1.0
      networks:
        - name: tp03-network
      ports:
        - "80:80"
```

## Base commands

`ansible all -i inventories/setup.yml -m ping`

This command will ping all the hosts in the inventory file `inventories/setup.yml`.

`ansible-playbook -i inventories/setup.yml playbook.yml`

This command will execute the playbook `playbook.yml` on all the hosts in the inventory file `inventories/setup.yml`.