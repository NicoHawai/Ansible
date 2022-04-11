# Config Management tools

## Challenges and solutions

- Configuration drift
- Centralized Configuration & Version Control
- Configuration Monitoring & Enforcement
- Configuration Provisionning
    - Configuration Templates and Variables
    - Control Configuration Automation
- Ansible
    - Playbooks
    - Inventory
    - Templates
    - Variables
 - Puppet
    - Manifest
    - Resource, Class, Module
    - Templates
 - Chef
    - Resource
    - Recipe
    - Cookbooks
    - Runlist

## Pros & Cons    

![](https://www.veritis.com/wp-content/uploads/2020/03/infographic-chef-vs-puppet-vs-ansible-what-are-the-differences.png)

# Ansible

## Intro

- Orchestrateur polyvalant 
- Ecrit en python
- Push / ssh / pas d'agent
- Tâches complexes ou ponctuelles
- Racheter par redhat en 2015
- Facilité d'utilisation > yaml > indentation
- Des milliers de modules 
- Idempotent

## Définitions

**Control node** : dispose de ansible pour le deploiement, acces ssh > on doit le sécuriser. Machine sensible. Normalement ça ne devrait pas être un laptop  

**Managed node** : serveur cible, un utilisateur avec elevation de privilège, ssh, python  

**Inventory** : l'inventaire des machines (ip, dns), soit en ini (plat), soit en yaml. Mais c'est aussi des variables qui sont attachés à ces machine (host_vars, group_vars), patterns  

**Groupes**: dans un inventaire on peut regrouper des machines (groupe des serveurs web, groupe des serveur sql, etc...) homogéiniser. Groupe racine : all  

**Tasks** : actions variées (copier fichier, créer user, commandes, modules etc...)  

**modules** : ensemble d'actions sur un environement particulier, fournit par ansible (mais possible de créer ses propres modules), prend des options et peuvent fournir des retours  

**Roles** : ensemble d'actions coordonnées  

**Playbooks** : un fichier, son role est centré entre l'inventaire et les roles. Va faire jouer des roles sur un inventaire specifique  

*2 executables* :  

ansible (tests, tâches sommaires) & ansible-playbook (joue un playbook)  

---

## Mise en place des VMs

- Create a debian machine / **bridge mode**  
- 5x clones / **new mac address**  
- hostname srv1/2/3/4/5  
- master > ssh config
- master > install ansible  

--------------

## Configuration de Ansible

nano /etc/ansible/ansible.cfg
```
[defaults]
inventory     = /etc/ansible/hosts
gather_facts  = no
forks         = 1
```
nano /etc/ansible/hosts > 5 x ip

> ansible all -m ping

srv1  ansible_host=192xx
[web]
[db]

change forks

-u = user
-b = become sudo
-k = ask password
-vvv verbose
-K ask sudo password
--ask-vault-pass = déchiffrer un password
--vault-password-file = stock password de vault dans un file
-f = forks (5 by default)
--one-line
-e = extra varibale

ansible -i 'ip," all -u -k -m ping

ansible srv1 -e "var=12345" -m debug -a "msg={{ var }}"

-m command -a "cat /etc/motd"
uptime
uname -r
reboot
apt install -y git

-m shell = for complex commands

-m raw = no python


-m apt -a "name=nginx"
systemctl status nginx

-m service -a "name=nginx state=stopped"

-m copy -a "src= dest="

-m fetch -a "src= dest= flat=yes"

### gather facts
-m setup
-a "filter="

--------------------------

## Premier playbook

Fichier qui va déclencher des actions

commande : ansible-playbook

-i = inventory
-l = limit > spécifier un serveur, ou un groupe de serveurs
-t = tags
--step = étape par étape, demande confirmation
-u
-b
-k
-K
-e
-f

--ask-vault 
--vault-password-file

```
---
- name: Mon Premier PlayBook
  hosts: all
  tasks:
  - name: check connection
    ping:
```

---> gather facts

```
---
- name: Mon Premier PlayBook
  hosts: all
  gather_facts: no
  tasks:
  - name: check connection
    ping:
```

FILE : https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html

```
---
- name: Mon Premier PlayBook
  hosts: all
  gather_facts: no
  tasks:
  - name: check connection
    ping:
  - name: add folder /tmp/hawai
    file:
      path: "/tmp/hawai"
      state: directory
```

--> ls -l --> owner/group appartiennent au user utilisé

```
---
- name: Mon Premier PlayBook
  hosts: all
  #remote_user: hawai
  gather_facts: no
  tasks:
  - name: check connection
    ping:
  - name: add folder /tmp/hawai
    file:
      path: "/tmp/hawai"
      state: directory
      owner: seb
      group: seb
```

ERROR --> become: yes + -K


```
---
- name: Mon Premier PlayBook
  hosts: all
  #remote_user: hawai
  gather_facts: no
  tasks:
  - name: check connection
    ping:
  - name: add folder /tmp/hawai
    file:
      path: "/tmp/hawai/1/2/3/4/5"
      state: directory
      recurse: yes
      owner: seb
      group: seb
```
```
  - name: add file
    file:
      path: "/tmp/hawai/1/2/3/file.txt"
      state: touch
```

---> check with tree
---> try : file & absent

NOTION de idempotence (comparer la situation qui est decreite dans mon code avec la situation d'arrivée)

Refaire un "absent"
----> pas de "changed"


### MODULE USER

https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html

```
---
- name: mon playbook users
  hosts: all
  become: yes
  tasks:
    - name: ajout de l'utilisateur c3po
    user:
      name: c3po
      state: present
      password: "{{ '12345678' | password_hash('sha512') }}"

```

--> groups: sudo
--> uid: 1030

---> state absent


### APT module

```
---
- name: my playbook with APT
  hosts: web
  become: yes
  tasks:
  - name: update cache & installation htop
  apt:
    update_cache: yes
    cache_valid_time: 3600
```
Try it, idempotence

```
---
- name: my playbook with APT
  hosts: web
  become: yes
  tasks:
  - name: update cache & installation htop
  apt:
    update_cache: yes
    cache_valid_time: 3600
    name: htop
    state: latest
```
try it

---> absent, purge: yes, autoremove: yes


### Installation server Apache

```
---
- name: My playbook Apache
  hosts: web
  become: yes
  tasks:
  - name: update cache & installation htop
    apt:
      update_cache: yes
      cache_valid_time: 3600
  - name: install apache2  
    apt:
      name: apache2
      state: latest
  - name: install php
    apt:
      name: php
      state: latest
  - name: ajout de notre page index.html
    copy:
      src: "index.html"
      dest: "/var/www/html/index.html"
      owner: root
      group: root
      mode: 0644
```

<?php
phpinfo();
?>

delete index.html


## Get a lease (or fix it)
```
switch>en
switch#conf t
switch(conf)#hostname alpha
alpha(conf)#int vlan 1
alpha(conf-if)#no shut
alpha(conf-if)#ip address dhcp
alpha(conf-if)#end
alpha#wr mem
alpha#sh ip int br
```

## Enable SSH
```
alpha#conf t
alpha(conf)#ip domain-name nunux.lan
alpha(conf)#crypto key generate rsa modulus 1024
alpha(conf)#ip ssh logging events
alpha(conf)#ip ssh version 2
alpha(conf)#end
alpha#wr mem
alpha#show ip ssh
```

## Create user
```
alpha#conf t
alpha(config)#username hello secret hello
alpha(config)#line vty 0 15
alpha(config-line)#transport input ssh
alpha(config-line)#login local
alpha(config-line)#end
alpha#wr mem
```
