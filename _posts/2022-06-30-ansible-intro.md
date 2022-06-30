---
layout: single
title:  "Getting Started with Ansible "
date:   2022-06-30 03:55:04 +0530
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
      url: "https://kubernetes.io"

author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
 
sidebar:
  - title: "Blog"
 
    text: "Checkout other topics"
    nav: my-sidebar
---

# Ansible

```yaml
---

# Ansible Certification: Red Hat EX407 
- Notes from O'Reilly Course by Sander van Vugt 

- tasks :
    - name: Ansible Fundamentals  # Space after - is mandatory !
   
    - name: Lession 1 Taking an Ansible Test-drive
        - ansible.cfg
        - inventory 
        - anisble modules location (in macOS) : /usr/local/lib/python3.7/site-packages/ansible/modules/
        - ansible localhost -m ping
        - ansible localhost -a "whoami"
        - anisble all -a "id"  # label all is implicit meaning all hosts in the inventory
        - ansible localhost -m command -a "id"
        - ansible localhost -m command -a "date"
        - ansible localhost -m shell -a "date"
        - privilege escalation : become become_user become_method become_ask_pass
        - pradeep ALL=(ALL) NOPASSWD: ALL (for sudo permissions) # /etc/sudoers.d/pradeep
        - ssh-keygen
        - ssh-copy-id user@remotehost
        - yum search epel
        - yum install epel-release
        - yum search ansible
        # ansible-lint package for syntax check

       

    - name: Lession 2 Understanding Ansible Architecture
      notes:
        - ansible-doc -l
        - ansible-doc -s ping
        - ansible-doc -s slack
        - ansible-doc -s copy
        - ansible-doc -s template
        - ansible-doc -s command
        - ansible-doc -s yum
        - ansible-doc -s file
        - ansible-doc -s junos_facts

   
    - name: Lession 3 Working with Playbooks
        - ansible-playbook --syntax-check pb_collect_facts.yml 
        - ansible-playbook pb_collect_facts.yml --list-hosts
        - ansible-playbook -C pb_collect_facts.yml # for Dry-Run
        - ansible-playbook --step pb_collect_facts.yml
        - lineinfile module # to modify an existing line in a file
        - firewalld module 


    - name: Lession 4 Working wiht Variables,Inclusions and Task Control, Facts
        - Highest level scope wins  Global, Play, Host
        - Global: command line or ansible.cfg file
        - Host: individula hosts or groups via inventory file
        - Playbook:
          - hosts: all
            vars: # keyword to define variables
              user: pradeep
              home: /home/pradeep
        - Variable files:
          - hosts: all
            vars_files: # keyword to specify exterbal variable files 
              - vars/users.yml
            
              # cat var/users.yml
              # user: pradeep
              # home: /home/pradeep
              # user: suma
              # home: /home/suma
        - Using variables 
        - tasks:
            - name : Creates the user {{ user }} 
              user:
                name: "{{ user }}" # if the variable is used as the first element , double quotes mandatory !

        - Importance of Project Directory wiht various files 
        - Host variables and Group variables
        # These may be defined in the inventory file, but that is deprecated
        # Recommended method to use group_vars and host_vars directories ...

        # [webservers]
        # server1.example.com

        # [webservers:vars] ## To define variables inside inventory file 
        # user=pradeep


        # Within the Project Directory , which contains the inventoru file create directories group_vars and host_vars

        # If we have a host group "webservers" defined in the inventory, 
        # create file with the same name "group_vars/webservers" and in this file, define the variable 

        # Similarly for individul hosts, create a file wiht the name of the host and put it in "host_vars"
        # for example, "host_vars/server1"

        # Variables can be overwritten from command line with -e "key=value" option to the ansible-playbook command 


        # (base) pradeep-mbp:Desktop pradeep$ tree vardemo/  #Project Directory
        # vardemo/
        # |-- group_vars
        # |   |-- ftpservers
        # |   `-- webservers
        # `-- inventory

        # 1 directory, 3 files

        # (base) pradeep-mbp:Desktop pradeep$ cat vardemo/inventory 
        # [webservers]
        # server1.exmaple.com
        # server2.example.com

        # [ftpservers]
        # server3.example.com
        # server4.example.com

        # A variable named "package" is defined which could be used in the respective playbooks as {{ package }}
        # (base) pradeep-mbp:Desktop pradeep$ cat vardemo/group_vars/ftpservers 
        # package: vsftp
        # (base) pradeep-mbp:Desktop pradeep$ cat vardemo/group_vars/webservers 
        # package: http
        # (base) pradeep-mbp:Desktop pradeep$ 



        # Arrays : Variables that define multiple values 
        - example array 
          users:
            pradeep:
              first_name: pradeep
              last_name: gadde
              home_dir: /home/pradeep
            suma:
              first_name: suma
              last_name: gadde
              home_dir: /home/suma

        # we may refer to these using the following syntax -  users.pradeep.first_name , users.suma.home_dir etc ..


        - Using Facts  # discovered information about a host, can be used as conditions to run specific tasks only when necessary
        # The "setup" module is used to gather facts
        - ansible server1.example.com -m setup

        # We can filter facts with -a 'filter=..'  Level 1 info
        - ansible server1.example.com -m setup -a 'filter=ansible_kernel'
        - ansible localhost -m setup -a 'filter=ansible_kernel'
          # [WARNING]: No inventory was parsed, only implicit localhost is available
          # localhost | SUCCESS => {
          #     "ansible_facts": {  
          #         "ansible_kernel": "19.4.0"
          #     },
          #     "changed": false
          # }

        - Custom Facts : manually created by Admins
        # INI or JSON format wiht .fact extension stored in /etc/ansible/facts.d 
        # will be shown as "ansible_local"
        

        # Inclusions both for tasks and variables
        - include_vars
        - include 

        # Variable Precedence  include_vars > Global Scope (command line -e option or ansible.cfg) > Playbook > Host Level
        tasks:
         - name: Read the tasks.yml to find what to do 
           include: tasks.yml  # Main task definition  included here 
           vars: # vaiables will be defined in the main file
             package: samba
             service: smb
             state: started
           register: output


          # cat tasks.yml 
          # - name: install the {{ package }}
          #   yum:
          #     name: "{{ package }}"
          #     state: latest
          
          # - name: start the {{ service }}
          #   service:
          #     name: "{{ service }}"
          #     state: "{{ state }}"

        # Include_vars Example
          - name: Install some packages
            hosts: all
            tasks:
              - name: include packages
                include_vars: packages.yml ## variables are defined here in this file 
              
              - name: installs {{ packages.my_pkg }}
                yum:
                  name:"{{ packages.my_pkg }}"
                  state: latest
        
        # cat packages.yml
        # ---
        # packages:
        # my_pkg: httpd

       - name: Lession 5 Using Flow Control, Conditionals, Jinja Templates
        # A loop is used to process a series of values in an array
        # simple loop  "with_items" statement

         - yum:
              name: "{{ item }}"  # the item variable follows from the with_items loop type
              state: latest
           with_items:
              - nmap
              - net-tools

        # - name: create users
        #   hosts: all
        #   tasks:
        #     - user:
        #         name: "{{ item.name }}"
        #         state: present
        #         groups: "{{ item.group }}
        #       with_items:
        #         - { name: 'pradeep',groups: 'wheel'}
        #         - { name: 'suma',groups: 'root'}


        # Nested Loops - a loop inside a loop 
        # Ansible iterates over the first array and applies the values in the second array to each item in the first array

        # with_nested is the keyword for this type of loop
        - name: give users access to multiple databases
          mysql_user:
            name: "{{ item[0] }}"
            priv: "{{ item[1] }}.*:ALL"
            append_privs: yes
            password: 'foo'
          with_nested:
            - [ 'pradeep','suma']
            - [ 'clientdb','employeedb','providerdb']


        # Other loop types are with_file, with_fileglob,with_sequence, with_random_choice ..

        # Working with Conditionals
        # To run tasks only on hosts that meet specific conditions

        # == < > <= != isdefined is not defined  etc 

        # When statement is used to implement a condition

        # when statement must be indented outside a module, at the top level of the task

        - name: Install the mariadb package
          package:
            name: mariadb
          when: inventory_hostname in groups["databases"]

        # multiple conditions can be combined with and or 

        # we can use "register" to save the information about the result of a task into a variable
        # result.rc  could be used in when conditions 


        # Jinja2 templates

        # cat motd.j2
        # This is the system {{ ansible_hostname }}.
        # Today it is {{ ansible_date_time.date }}.
        # Only use this system if {{ system.owner }} has granted you permission !

        # cat motd.yml
        # - hosts: all
            user: user
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
                  
