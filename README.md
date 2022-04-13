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
- Des milliers de modules 
- Ecrit en python
- Push / ssh / pas d'agent
- Tâches complexes ou ponctuelles
- Racheter par redhat en 2015
- Facilité d'utilisation > yaml > indentation
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

> sudo nano /etc/ansible/ansible.cfg  
```
[defaults]
inventory   = /etc/ansible/hosts
forks       = 1
```
> sudo nano /etc/ansible/hosts  
```
192.168.1.x
192.168.1.y
192.168.1.z
192.168.1.a
192.168.1.b
```
--------------
### ansible (le binaire) :
--------------

**-u**             = user  
**-b**             = become sudo  
**-k** = ask password  
**-vvv** verbose  
**-K** ask sudo password  
**--ask-vault-pass** = déchiffrer un password  
**--vault-password-file** = stock password de vault dans un file  
**-f** = forks (5 by default)  
**--one-line**  
**-e** = extra varibale  
*gather facts*  
**-m setup -a "filter="**  

--------------
### ansible-playbook (le binaire) :
--------------

*Fichier qui va déclencher des actions*

**-i** = inventory  
**-l** = limit > spécifier un serveur, ou un groupe de serveurs  
**-t** = tags  
**--step** = étape par étape, demande confirmation  
**-u**  
**-b**  
**-k**  
**-K**  
**-e**  
**-f**  
**--ask-vault**  
**--vault-password-file**  

--------------

### My first playbook

```
---
- name: Mon Premier PlayBook
  hosts: all
  gather_facts: no
  tasks:
  - name: check connection
    ping:
```
Launch *playbook-ansible myplaybook.yml*  

Try to change forks in the ansible.cfg to see the difference (forks=5 for instance) and replay the ping playbook  
Now edit:  
> sudo nano /etc/ansible/hosts  
```
[web]
192.168.1.x
192.168.1.y
[db]
192.168.1.z
[syslog]
192.168.1.a
192.168.1.b
```
Launch *playbook-ansible web myplaybook.yml*

Add :

```
  - name: add folder /tmp/hawai/...
    file:
      path: "/tmp/hawai/1/2/3/4/5"
      state: directory
      recurse: yes
      #owner: hawai
      #group: hawai
```
If you want to set/change the owner/group  
1) Make sure the user exists
2) Launch *playbook-ansible -b -K myplaybook.yml  *

Add :
```
  - name: add file
    file:
      path: "/tmp/hawai/1/2/3/file.txt"
      state: touch
```


### Module user

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
      password: "{{'12345678' | password_hash('sha512')}}"

```
--------------

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
    name: htop
    state: latest
```

--------------
### Installation server Apache

> create a custom index.html

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

--------------
# CISCO

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
## .ssh/config (backward compatible with old key exchange)  
On linux, add this file with this content:
```
KexAlgorithms diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
Ciphers aes256-ctr,aes128-ctr,aes256-cbc,aes128-cbc,3des-cbc
```
### group_vars

> group_vars > all.yml  

```
---
ansible_connection: network_cli
ansible_network_os: ios
ansible_user: hello
ansible_password: hello
ansible_become: yes
ansible_method: enable
ansible_become_password: password

```
### NETPLAN

![](https://vitux.com/wp-content/uploads/2019/05/word-image-265.png?ezimgfmt=ng:webp/ngcb10)

> sudo netplan try

puis Enter

```
---
- name: playbook cisco
  hosts: all
  gather_facts: no
  
  tasks:
    - name: show version
      ios_command:
        commands: show version
```

```


