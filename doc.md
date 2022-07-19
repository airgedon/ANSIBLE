### Ansible Set Up
---
```
cat /etc/os-release
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
ansible client01 -i hosts -m ping
```
