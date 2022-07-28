# Ansible Set Up
---
```
cat /etc/os-release **Warning!**
```
```
update-alternatives --install /usr/bin/python python /usr/bin/python3 2
```
> Install Ansible
```
pip3 install ansible
```
> go to root folder
```
cd /
```
> Create cfg file
```
sudo nano ansible.cfg
```
> write
```
[defaults]

inventory = /root/ansible/hosts
host_key_checknig = false 
```
___
## ANSIBLE HOSTS

```
mkdir ansible
```
```
cd ansible
```
```
> hosts
```
```
sudo nano hosts
```
```
client1 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__
client2 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__
client3 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__
````
> for ssh_key access
```
client1 ansible_host=__IP_adress___   ansible_user=root    ansible_private_key_file=/directory/to/key
```
---
> where to paste ssh public key 
```
   42  cd
   43  pwd
   44  ll
   45  cd .ssh/
   46  ls
   47  cd authorized_keys 
   48  sudo nano authorized_keys 
   49  history
```
[source](https://www.youtube.com/watch?v=YYjCwLs-1hA)
---
> checking inventory file
```
ansible all -m ping
```
### errors i guess so move to root directory where ansible.cfg is located
```
cd
```
> Oops, error again
```
client1 | FAILED! => {
    "msg": "to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program"
}
client2 | FAILED! => {
    "msg": "to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program"
}
client3 | FAILED! => {
    "msg": "to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program"
}
```
>To fix that run:
```
sudo apt install sshpass
```
> run again:
```
ansible all -m ping
```
> you can move ansible.cfg wherever you want
```
ansible all -i hosts -m ping
```
____
## INVENTORY GROUPS
### for one of them 
```
ansible client01 -i hosts -m ping
```
> edit hosts file
```
sudo nano hosts
```
```
[firewall]
client1 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__

[gunicorn]
client2 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__
client3 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__
```
> then...
```
ansible firewall -i hosts -m ping
```
#### How to ping 2 and more groups once ?

> in hosts file
```
[firewall]
client1 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__

[gunicorn]
client2 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__

[nginx]
client3 ansible_host=__IP_adress___   ansible_user=root    ansible_password=__Password__

[all_groups:children]
firewall
nginx
```
> then run
```
ansible all_groups -i hosts -m ping
```
___
## MODULES
> to see all information about client , type:
```
ansible all -i hosts -m setup:
```
> to create file in client servers :
```
ansible all -i hosts -m file "path=/root/ansible-file.txt state=touch"
```
> Error 
```
client01 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Error, could not touch target: [Errno 13] Permission denied: b'/root/ansible-file2.txt'",
    "path": "/root/ansible-file2.txt"
}
```
> to fix that (-b become, -a argument ):
```
ansible all -i hosts -m file -a "path=/root/ansible-file2.txt state=touch" -b
```
>Error
```
client01 | FAILED! => {
    "msg": "Missing sudo password"
}
```
>to fix in client server:
```
visudo
```
```
utest  ALL=(ALL:ALL) NOPASSWD:ALL
```

>to copy file:
```
ansible all -i hosts -m copy -a "src=file435 dest=/home mode=777" -b
```
#VARIABLS
```
mkdir group_vars
```
```
cd group_vars/
```
```
touch firewall
```
```
touch all_groups
```
```
sudo nano firewall 
```
```
ansible_host: __IP_adress___
ansible_user: root
ansible_password: __Password__

```
```
sudo nano all_groups
```
```
ansible_user: root
ansible_password: __Password__
```
```
cd ..
```
```
sudo nano hosts
```
```
[firewall]
client1 

[gunicorn]
client2 ansible_host=__IP_adress___   

[nginx]
client3 ansible_host=__IP_adress___   

[all_groups:children]
firewall
nginx
```
```
ansible all -i hosts -m ping
```
> Done
```
ansible all -i /root/ansible/hosts -m debug -a "var=ansible_host"
```
# Playbook
> in ansible folder 
```
touch ping.yml
```
```
sudo nano ping.yml
```
> ping all servers
```
- name: Ping Servers
  hosts: all
  become: yes

  tasks:
  
  - name: Task ping
    ping:

```
```
ansible-playbook -i hosts ping.yml 
```
___
> Update and upgrade
```
- name: Ping Servers
  hosts: all
  become: yes

  tasks:
  
  - name: Task ping
    ping:

  - name: Update cache
    apt:
      update_cache: yes
  - name: Upgrade
    apt:
      upgrade: yes
```
> install Apache2
```
- name: Ping Servers
  hosts: all
  become: yes

  tasks:
  
  - name: Task ping
    ping:

  - name: Update cache
    apt:
      update_cache: yes
  - name: Upgrade
    apt:
      upgrade: yes
  - name: Install apache2
    apt:
      pkg: apache2
      state: present
```
> copy file
```
- name: Ping Servers
  hosts: all
  become: yes

  tasks:
  
  - name: Task ping
    ping:

  - name: Install apache2
    apt:
      pkg: apache2
      state: present

  - name: copy File
    copy:
      src: ./file111
      dest: /home/
      mode: 0777
```
[Docs](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/)

## Playbook Vars
> install several packages
```
- name: Ping Servers
  hosts: all
  become: yes

  tasks:
  
  - name: Task ping
    ping:

  - name: Install apache2
    apt:
      pkg: 
        - apache2
        - htop
        - tree
      state: present

  - name: copy File
    copy:
      src: ./file111
      dest: /home/
      mode: 777
```
### Yaml format doesn`t like tabs
> after
```
- name: Ping Servers
  hosts: all
  become: yes

  vars:
    packages:
             - htop
             - tree
             - zsh

  tasks:
  
  - name: Task ping
    ping:

  - name: Install apache2
    apt:
      pkg: "{{packages}}"
      state: present
```
```
- name: Ping Servers
  hosts: all
  become: yes

  vars:
    packages:
             - htop
             - tree
             - zsh
    file_src: file111
    file_dest: /srv

  tasks:
  
  - name: Task ping
    ping:

  - name: Install apache2
    apt:
      pkg: "{{packages}}"
      state: present

  - name: Copy File
    copy:
      src: "{{file_src}}"
      dest: "{{file_dest}}"
```
```
- name: Loops
  hosts: all
  become: yes


  tasks:

  - name: Create Folder
    file: 
      path: "/home/{{item}}"
      state: directory
    loop:
      - dir1
      - dir2
```
```
ansible-playbook -i hosts play1.yml
```
___
```
- name: Loops
  hosts: all
  become: yes


  tasks:

  - name: Create Folder
    file: 
      path: "/home/{{item}}"
      state: directory
    loop:
      - dir1
      - dir2
```
```
id test1
```
___
> at first edit inventories (hosts file)
```
[firewall]
client1 user_client=client01

[gunicorn]
client2 ansible_host=__IP_adress___   user_client=client02

[nginx]
client3 ansible_host=__IP_adress___   user_client=client03

[all_groups:children]
firewall
nginx
```
```
- name: Create User
  hosts: all
  become: yes


  tasks:

  - name: Create Groups
    group:
      name: "{{item}}"
      state: present
    loop:
      - dev
      - test

  - name: Create users
    user:
      name: "{{user_client}}"
      shell: /bin/bash
      groups: dev, test
      append: yes
      home: /home/test1

```
> to create users and groups
___
> to create different users for each server 
```
- name: Create User
  hosts: all
  become: yes


  tasks:

  - name: Create Groups
    group:
      name: "{{item}}"
      state: present
    loop:
      - dev
      - test

  - name: Create users
    user:
      name: "{{item.clientname}}"
      shell: /bin/bash
      groups: dev, test
      append: yes
      home: "/home/{{item.homedir}}" 
    with_items:
       - {clientname: client1, homedir: client1}
       - {clientname: client2, homedir: client2}
```
## DEBUG & MESSAGE
> display messages
```
- name: Messages
  hosts: firewall
  become: yes

  vars:
    slovo1: HOME
    slovo2: in
    mesto: USA

  tasks:

  - name: Print Vars
    debug: 
      var: slovo1

  - debug:
      msg: "MY home is in {{mesto}}"
```
```
- name: Messages
  hosts: firewall
  become: yes

  vars:
    slovo1: HOME
    slovo2: in
    mesto: USA

  tasks:

  - name: Print Vars
    debug: 
      var: slovo1

  - debug:
      msg: "MY home is in {{mesto}}"	

  - debug:
      msg: "MY {{slovo1}} is {{slovo2}} {{mesto}}"

  - set_fact:
       message:  "MY {{slovo1}} is {{slovo2}} {{mesto}}"

  - debug:
      var: message

  - debug:
      var: ansible_distribution_version

  - debug:
      msg: "Linux: {{ansible_distribution}} Version: {{ansible_distribution_version}}"

```
```
ansible firewall -i hosts -m setup
```
```
- name: Messages
  hosts: firewall
  become: yes

  vars:
    slovo1: HOME
    slovo2: in
    mesto: USA

  tasks:

  - name: Print Vars
    debug: 
      var: slovo1

  - debug:
      msg: "MY home is in {{mesto}}"	

  - debug:
      msg: "MY {{slovo1}} is {{slovo2}} {{mesto}}"

  - set_fact:
       message:  "MY {{slovo1}} is {{slovo2}} {{mesto}}"

  - debug:
      var: message

  - debug:
      var: ansible_distribution_version

  - debug:
      msg: "Linux: {{ansible_distribution}} Version: {{ansible_distribution_version}}"

  - shell: id client1
    register: client_groups
    
  - debug:
      msg: "Client1 in Groups: {{client_groups}}"
```
## BLOCKS & WHEN
```
- name: Test Blocks
  hosts: all
  become: yes

  vars:


  tasks:

  - block:

    - name: Install Packages
      apt: 
        pkg:
          - zsh
          - nmon
          - htop
        state: present


    - name: Create Folder
      file: 
        path: /srv/folder21
        state: directory

    when: user_client == "client01"


  - name: Copy File
    copy:
      src: file822
      dest: /srv/
    when: user_client == "client03" 
```
## TEMPLATE
```
sudo mv file822 file822.j2 
```
```
Host: {{user_client}}
User: {{inv_user}}
Position: {{position}}
```
>hosts file
```
[firewall]
client1 user_client=client01 inv_user=cli1

[gunicorn]
client2 ansible_host=__IP_adress___   user_client=client02 inv_user=cli2

[nginx]
client3 ansible_host=__IP_adress___   user_client=client03 inv_user=cli3

[all_groups:children]
firewall
nginx
```
```
sudo nano templates.yml
```
```
- name: Test Blocks
  hosts: all
  become: yes

  vars:

  - position: boss

  tasks:

  - name: Copy File
    template:
      src: ./file822.j2
      dest: /home/config
      mode: 0777

```
## ROLES

```
ansible-galaxy init first_setup
```
```
ls
```
```
cd first_setup
```
```
sudo nano side.yml
```
```
- name: First Install + Web-Project
  hosts: client01
  gather_facts: yes
  roles:
          - first_setup
          - web-project
```
[Galaxy Ansinble](https://galaxy.ansible.com/)
