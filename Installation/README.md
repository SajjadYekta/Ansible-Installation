# HOW TO INSTALL ANSIBLE

The Ansible package is in the default repository of Linux

```bash
apt install ansible -y
ansible --version
```

Tip:

the controller server must have Linux OS, but remote servers can be other OS like Windows and Mac even it can be Switch or Rotter, but the remote servers should have Python,

ansible is agentless because of that it doesn't have load on the server when you don't use it,

Best practice:

It is better that you make Ansible user on all of the remote servers and make it sudoers, and then use from Public-key for safe SSH connection,

make it in the controller and remote servers:

```bash
useradd -m -s /bin/bash -c “Ansible User” ansible
passwd ansible 
```

make the Ansible user, 

add this line: “ %ansible ALL=(ALL:ALL) NOPASSWD: ALL ”

```bash
sudo vi /etc/sudoers
```

Then, start to send the Controller’s Public Key to remote machines:

Best practice for many servers:

```bash
apt install sshpass
```

```bash
sshpass -f password.txt ssh-copy-id -o StrictHostKeyChecking=no USERNAME@IP
```

MAKE INVENTORY:

For one project with one inventory file :

```bash
	touch /etc/ansible/hosts
```

For several projects with several inventories, we have to use eq this command to run a playbook,

```bash
#e.g
ansible-playbook playbook.yaml -i inventory-company-name.txt
```

In host inventory:

You can use domain name or IP remote address,

```bash
	cat /etc/ansible/hosts
example.com
192.168.10.10

#e.g
[webservers] # the name of categury of the inventory, that you call by the categury name
www.example[001:009].com #it means www.example1 ... 9.com
192.168.10.11

[db-servers]
db01.intranet.mydomain.net
192.168.10.12

[all_servers] # this categury define all of the categury for simply using
webservers
db-servers
```

INVENTORY PARAMETERS

ansible_host=192.168.10.10                   #IP or DNS Name of the Host.

ansible_port=2022 SSH                          #Default port for connecting to. if  you don't use this parameter normally it uses 22

ansible_connection=ssh                          #Ansible Connection Type [ssh/winrm/localhost]. normally it uses ssh protocol 

ansible_user=ansibleSSH                       #User, if no user specified current user will used.

ansible_ssh_pass=Aa@123                    #SSH Password. the best practice is to use pub.key and sshpass package,

```bash
vim /home/ansible/inventory.txt
# Sample Inventory parameters
ubuntu-1 ansible_host=web1.example.com ansible_user=root
ubuntu-2 ansible_host=192.168.10.10 ansible_user=ansible
ubuntu-3 ansible_host=192.168.10.11 ansible_ssh_pass=Aa@123
```

HOW TO CHECK THE INVENTORY:

```bash
ansible --list-hosts all
```

CHECK THE REMOTE SERVERS:

use ansible Ad-Hoc command:

```bash
#with same user
ansible all -m ping #use biult in module 
```

Some of the Ad-Hoc command:

By default, ansible uses the Ad-Hoc command and you can don't write “-m command”

-b or —become—user (use user) —become (use root ), this option force command to switch user and then run the command,

By default use the root user,

-K means askpass 

```bash
#e.g
ansible all -m command -a "whoami" -b    # -b means, use user root  
ansible all -m command -a "whoami" --become-user sajjad #use sajjad user
ansible all -m command -a "whoami" -bK # -k means, ask password,
```

what is difference between SHELL and COMMAND:

command compile your order as Python and in the destination server again run with Python order,

but in SHELL, it is not like this, SHELL uses  /bin/$SHELL and runs commands with all of the regexes, 

```bash
#e.g
ansible all -m shell -a 'echo "shell vs command" > ~/test.txt'      # True
ansible all -m command -a 'echo "shell vs command" > ~/test.txt'    # False
```

SHELL VS RAW:

Raw is used for a minimal system without Python like Sisco Switch, but the shell needs a shell to run the order.

APT MODULE:

For install packages:

Tip:

state=present  (check it)

state=latest   (make update)

state=absent   (for remove package)

you can catch more information by “ ansible-doc <your module> ”

```bash
ansible all -m apt -a "name=apache2 state=present purge=yes" -bK # compeletly remove
#e.g2
ansible all -m apt -a "name=apache2 state=latest update_cache=yes" -bK # update_cache = apt update
```

SERVICE MODULE (systemctl)

state=started

state=restarted

state=reloaded

state=stopped

enabled=yes (enable service when machine turn on )

```bash
#e.g
ansible web-srv -b -m service -a "name=apache2 state=reloaded"
```

COPY MODULE 

Tip:

what is the checksum: Checksum is a feature  that shows you the operation like copy or zip, etc did it or not

mode=777  (change chmod)

owner=ansible (change owner)

group=root (change group)

force=yes (replace)

backup=yes (keep modifying time and I nod)

checksum=yes (check run successfully)

```bash
#e.g
ansible all -m copy -a "src=/home/sajjad/copy.sh dest=/home/ansible mode=777 owner=ansible group=root force=yes =backup=yes checksum=yes" -bK
```

FILE MODULE

make a dir in the remote servers

state=directory (make empty dir)

state=file (make empty file)

state=absent (delete file or dir)

```bash
#e.g
ansible all -m file -a "dest/tmp/new-dir mode=777 owner=ansible group=ansible state=directory"
```

SETUP MODULE (gathering fact)

```bash
#e.g
ansible all -m setup
#you cant fetch the specific parameter
ansible all -m setup -a "filter=*family*"
```

HANDLERS 

It works like a function in programming that you can call several times in your tasks)

```bash
#sample of handlers playbook
  hosts: all
  become: yes
  handlers:
    - name: start vsftpd
      service: name=vsftpd enabled=yes state=started
      
  tasks:
    - name: install vsftpd on ubuntu
      apt: name=vsftpd update_cache=yes state=latest
      ignore_errors: yes
      notify: start vsftpd

    - name: install vsftpd on centos
      yum: name=vsftpd state=latest
      ignore_errors: yes
      notify: start vsftpd
```

VARIABLES IN ANSIBLE

Host Scope:

It is used in the inventory forward to remote servers line inventory 

```bash
#e.g
#in inventory file in front of the remote line
centos http_port=8080 snmp_port=161-162 internal_ip_range=192.168.100.0

#on controller server
vi firewall-playbook.yaml

  name: Set Firewall Configurations
  hosts: centos
  become: true
  tasks:
    -  firewalld:
         service: https
         permanent: true
         state: enabled

    -  firewalld:
         port: "{{ http_port }}/tcp"
         permanent: true
         state: disabled

```

Playbook Scope:

It is used in the scope of playbook just in the playbook, it is used before the task on the top of the Tasks, 

```bash
#e.g
#on controller server
vi firewall-playbook.yaml
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars:
    http_port: 8080
    snmp_port: 161-162
    internal_ip_range: 192.168.100.0
  tasks:
    -  firewalld:
         service: https
         permanent: true
```

Sample to the set variable on the File:

```bash
#e.g
#on controller server
vi vars_file.yaml
http_port: 8080
snmp_port: 161-162
internal_ip_range: 192.168.100.0

#on controller server
vi firewall-playbook.yaml
  name: Set Firewall Configurations
  hosts: centos
  become: true
  vars_files:
    - vars_file.yaml
  tasks:
    -  firewalld:
         service: https
         permanent: true
```

Global Scope:

It is used for all of the remote servers, in the last line of the inventory file

ANSIBLE FACT

DEBUG MODULE:

This  makes your message or your description after the tack successfully run 

```yaml
#e.g
- name: test debug module
	hosts: all
	becoem: yes
	vars:
		- first_var= "DEBUG" 
	tasks:
		- name: result
			debug: msg=" test of the variable - {{ first_var }} "
```

REGISTER MODULE

Push all of the results to the a variable and then you can call with the DEBUG module,

this is an example and you can separate fields and show specific your Syntax  

```yaml
#e.g
- name: register with debug
  hosts: all

  vars:
    - var_thing: "sajjad"
  tasks:
    - name: print something
      command: echo -e "{{ var_thing }}\nTrust your progress\nI like Linux with {{ var_thing }}"
      register: my_result

    - name: show_result
      debug: msg={{ my_result.stdout_lines }}
      #or use this syntax
      #debug:
      # - var: my_result.stdout_lines
```

LOCAL_ACTION MODULE

This module do some module in the local servers for e.q you want copy from /etc/config to /home/sajjad/ , 

```yaml
- name: register with debug
  hosts: all	
    - name: Initialize Kubernetes Cluster on Master-1
      shell: "kubeadm init --upload-certs --apiserver-advertise-address {{ ansible_default_ipv4.address }} --control-plane-endpoint {{ ansible_default_ipv4.address }}:6443 --pod-network-cidr 10.244.0.0/16"
      register: results

    - name: Save the Result of the command
      local_action:
        module: copy
        content: "{{ results.stdout }}"
        dest: /root/kubeadm_init_result_kmaster1.log
        force: yes
```

CONDITIONS WHEN OR FAILED_WHEN

in this sample I said go and run the command and WHEN the parameters is true

```yaml
- hosts: ubuntu,centos,suse
  become: yes

  tasks:
    - name: Remove Apache on Ubuntu Server
      apt: name=apache2 state=absent
      when: ansible_os_family == "Debian"

    - name: Remove Apache on CentOS  Server
      yum: name=httpd  state=absent
      when: ansible_os_family == "RedHat"
```

CONDITIONS LOOP

with_item: 

this condition is going to deprecated, but the syntact is like this:

```yaml
#e.g 
- hosts: ubuntu
  become: yes

  tasks:
 #(depricated)
  # - name: install packages
  #   apt: name={{ item }} update_cache=yes state=latest
  #   with_items:
  #    - vim
  #    - nano
  #    - apache2
 #e.g (it sugessted)
   - name: install packages
     apt: 
       name: ["vim", "nano", "apache2"]
 #or
   #    name:
   #     - vim
   #     - nano
   #     - apache2      
       update_cache: yes
       state: latest
```

with_file:

It makes copy from several src file, src: {{ item }} is fixed,

```yaml
- name: copy serverall file
  with_file:
    - file.txt
    - file2.txt
  copy:
    src: {{ item }}
    dest: /home/ansible/
    
```

with_sequence:

Another loop, using a different sequence orders:

```yaml
  tasks:
   - name: show file(s) contents
     debug: msg={{ item }}
     with_sequence: start=1 end=5
```

ANSIBLE-VAULT

this feature is for securing your playbook.yaml

```yaml
#run this command to make a password for securing playbook
ansible-vault encrypt apache-playbook.yaml
New Vault password:
Confirm New Vault password:

#editing
ansible-vault edit apache-playbook.yaml

#runing
ansible-vault apache-playbook.yaml --ask-vault-pass # is going depricated
ansible-vault apache-playbook.yaml --vault-password file password.txt # is going depricated
ansible-vault apache-playbook.yml --vault-id @prompt # it sugessted

#regnerate password
ansible-vault rekey apache-playbook.yaml

#encript-string # use the output as a value of the your-variable
ansible-vault encrypt-string 'your-pass' --name 'your-variable'
```

TEMPLATE MODULE

Sometimes we need to call dynamic variables and set them as a value in our file, 

this example shows us how is it used:

```yaml
#make sample file that is used from dynamic and static variable 
vi index.html.j2
<html>
<center>
<h1> This machine's hostname is {{ ansible_hostname }}</h1>
<h2> os family is {{ ansible_os_family }}</h2>
<small>file version is {{ file_version }}</small> 
{# This is comment, it will not appear in final output #}
</center>
</html>
```

make a playbook for using top file:

```yaml
vi template-playbook.yaml

- hosts: all
  become: yes
  vars:
   file_version: 1.0

  tasks:
   - name: install default web page
     template:
       src: index.html.j2
       dest: /var/www/html/index.html #its better to identify your file name,
       mode: 0777
```

INCLUDE STATEMENT

by this feature we can call the other playbook in a specific playbook,

```yaml
vi include-playbook.yaml

- include: pre-installation-playbook.yaml

- hosts: all
  become: yes
  tasks:
  - include: installation-playbook.yaml
  - include: post-installation-playbook.yaml
```

ANSIBLE ROLLS

this feature separate and organize your big playbook to the small YAML file for simply using 

```yaml
#make a dir for name of the company and run it this command 
#it sugesst to cp your inventury to the this folder
nsible-galaxy init <your name of the project>

```

you can download the many ROLLS and COLLECTION from galaxy.ansible.com

ANSIBLE COLLECTION

the collection is the assembled  modules and plugins in one collection, one plugin made several modules

```yaml
#for e.q you can download the collection in the galaxy.ansible.com
ansible-galaxy collection install kubernetes.core
```