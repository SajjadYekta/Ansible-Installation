# Ansible

## HOW TO INSTALL ANSIBLE:
the ansible package is in the default repository of Linux

apt install ansible -y
ansible --version

Tip:

the controller server must have Linux OS, but remote servers can be other OS like Windows and Mac even it can be Switch or Rotter, but the remote servers should have Python,
ansible is agentless because of that it doesn't have load on the server when you don't use it,

Best practice:
Its better you make Ansible user on all of the remote servers and make it sudoers, and then use from Public-key for safe SSH connection,

#in controller side
root@controller:~# useradd -m -s /bin/bash -c “Ansible User” ansible
root@controller:~# passwd ansible

#in remote side
root@remote-1:~# useradd -m -s /bin/bash -c “Ansible User” ansible
root@remote-1:~# passwd ansible

#in controller side
root@controller:~# usermod -aG sudo ansible
root@controller:~# vi /etc/sudoers
%ansible ALL=(ALL:ALL) NOPASSWD: ALL

#in remote side 
root@remote-1:~# usermod -aG sudo ansible
root@remote-1:~# vi /etc/sudoers
%ansible ALL=(ALL:ALL) NOPASSWD: ALL


 Then, start to send the Controller’s Public Key to remote machines:

ansible@controller:~# ssh-keygen
ansible@controller:~# ssh-copy-id ansible@remote-1
