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

## Load Balancing

```
<VirtualHost *:80>
    Header set Access-Control-Allow-Origin *
    ProxyRequests On
    ProxyPreserveHost On
    <Proxy "http://serverpool">
        BalancerMember "http://backend-1:8080"
        BalancerMember "http://backend-2:8080"
    </Proxy>
    ProxyPass "/api" "http://serverpool"
    ProxyPassReverse "/api" "http://serverpool"
    ProxyPass / http://frontend:80/
    ProxyPassReverse / http://frontend:80/
    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>
</VirtualHost>

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule lbmethod_bytraffic_module modules/mod_lbmethod_bytraffic.so
LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyness.so
LoadModule lbmethod_heartbeat_module modules/mod_lbmethod_heartbeat.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
```

The goal of this configuration is to create a load balancer that will redirect the requests to the backend servers. The `BalancerMember` directive is used to define the backend servers. The `ProxyPass` and `ProxyPassReverse` directives are used to redirect the requests to the backend servers. The `ProxyPass` directive is used to redirect the requests to the backend servers and the `ProxyPassReverse` directive is used to redirect the responses to the frontend server. The `Proxy` directive is used to define the backend servers. The `Order`, `Allow` and `Deny` directives are used to define the access control.