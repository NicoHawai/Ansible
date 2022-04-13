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

> nano /etc/ansible/ansible.cfg  


> nano /etc/ansible/hosts  


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

### Module user

--------------

### Module apt

--------------

### Installation server Apache

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
# .ssh/config (backward compatible with old key exchange)

```
KexAlgorithms diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1,diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
Ciphers aes256-ctr,aes128-ctr,aes256-cbc,aes128-cbc,3des-cbc
```
### group_vars

> group_vars > all.yml  

```
---
ansible_connection= network_cli
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
