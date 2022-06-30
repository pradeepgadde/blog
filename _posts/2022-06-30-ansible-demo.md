---
layout: single
title:  "Ansible Demo "
date:   2022-06-30 04:55:04 +0530
categories: Automation
tags: Ansible
classes: wide
show_date: true
header:
  overlay_image: /assets/images/ansible.png
  og_image: /assets/images/ansiblepng
  teaser: /assets/images/ansible.png
  caption: "Photo credit: [**Ansible**](https://www.ansible.com)"
  actions:
    - label: "Learn more"
      url: "https://www.ansible.com"

author:
  name     : "Ansible"
  avatar   : "/assets/images/ansible.png"

sidebar:
  - title: "Blog"

    text: "Checkout other topics"
    nav: my-sidebar
---

A simple Ansible Demo

## Vagrantfile

```ruby
Vagrant.configure("2") do |config|
 config.vm.define "test1.example.com" do |test1|
 test1.vm.box="centos/7"
 test1.vm.hostname="test1.example.com"
 test1.vm.network "forwarded_port",guest:80,host:8000
 test1.vm.network :private_network, ip:"10.0.0.10"
 end
 config.vm.define "test2.example.com" do |test2|
 test2.vm.box="centos/7"
 test2.vm.hostname="test2.example.com"
 test2.vm.network "forwarded_port",guest:80,host:8001
 test2.vm.network :private_network, ip:"10.0.0.11"
 end
 config.vm.provision "ansible" do |ansible|
  ansible.playbook='playbook.yml'
 end
end
```
## Ansible.cfg

```sh
[defaults]
remote_user = vagrant
host_key_checking = false
inventory = .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory

[privilege_escalation] 
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

## HomePage.j2

```jinja2
+++++++++++++++ Welcome to {{ ansible_hostname }} ++++++++++++++++++
<br>
This system  is managed by Ansible.<br>
Today it is {{ ansible_date_time.date }}.

<br>
Here are the Properties of this sytem 
<br>
{{ ansible_nodename }}<br>
{{ ansible_processor }}<br>
{{ ansible_distribution }}<br>
{{ ansible_default_ipv4 }}<br>

 --- Pradeep ---
 <br>
 
```

## MOTD.j2

```jinja2
This is the  {{ ansible_hostname }} system managed by Ansible.
Today it is {{ ansible_date_time.date }}.
Only use this system if {{ system_owner }} has granted you permission !

+++++++++++++++ Welcome ++++++++++++++++++
```

## Playbook.yaml

```yaml
---
- hosts: all
  become: yes
  become_user: root
  tasks:
  - name: latest httpd version installed
    yum:
      name: httpd
      state: latest
  - name: latest firewalld version installed
    yum:
      name: firewalld
      state: latest
  - name: httpd enabled and running
    service:
      name: httpd
      enabled: true
      state: started
  - name: firewalld enabled and running
    service: 
      name: firewalld
      enabled: true
      state: started
  - name: firewlld permits http service
    firewalld:
      service: http
      permanent: true
      state: enabled
    # notify:
    #   - restart firewalld
  - name: restart firewalld
    service:  
      name: firewalld
      state: restarted

#- name: set message of the day
- hosts: all
  user: root
  become: true
  vars:
    system_owner: pradeep@example.com
  tasks:
   - template: 
       src: motd.j2
       dest: /etc/motd
       owner: root
       group: root
       mode: 0644

- hosts: all
  user: root
  become: true
  tasks:
   - template: 
       src: homepage.j2
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644     
```

