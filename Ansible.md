# Ansible

## HOW TO INSTALL ANSIBLE:

the ansible package is in the default repository of Linux

```bash
apt install ansible -y
ansible --version
```

Tip:

the controller server must have Linux OS, but remote servers can be other OS like Windows and Mac even it can be Switch or Rotter, but the remote servers should have Python,

ansible is agentless because of that it doesn't have load on the server when you don't use it,

Best practice:

Its better you make Ansible user on all of the remote servers and make it sudoers, and then use from Public-key for safe SSH connection,

```bash
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

```

 Then, start to send the Controller’s Public Key to remote machines:

```bash

ansible@controller:~# ssh-keygen
ansible@controller:~# ssh-copy-id ansible@remote-1
```

**Ansible ad-hoc command:**

all of the Command that you can use in CLI 

**Play scope variables:**

the variables of the Playbook in the Playbook.yml

in the an

### Playbook for sync dir

---

```jsx
---
- name: Sync directories
  hosts: your_host
  become: yes
  become_user: root
  tasks:
    - name: Synchronize directories
      synchronize:
        src: /path/to/source
        dest: /path/to/destination
			delegate_to: targert_server
```

**delegate_to: target_server (**instead of the default target hosts defined in the playbook.**)**

### TEST IT ::::

Based on the provided information about your Ansible environment, here are some key paths and locations you might need to replicate to an isolated server:

1. **Ansible Python Module Location**:
    - **`/usr/lib/python3/dist-packages/ansible`**
    - This directory contains the Ansible Python modules. If you've made any modifications or added custom modules, you might need to transfer these to the same location on the isolated server.
2. **Configured Module Search Path**:
    - **`/home/ansible/.ansible/plugins/modules`**
    - **`/usr/share/ansible/plugins/modules`**
    - These paths are where Ansible looks for modules. If you've added any custom modules or plugins, they might be located in these directories.
3. **Ansible Collection Location**:
    - **`/home/ansible/.ansible/collections`**
    - **`/usr/share/ansible/collections`**
    - Ansible collections are stored in these paths. If you're using custom or additional collections, you'll need to transfer them to these locations on the isolated server.
4. **Executable Location**:
    - **`/usr/bin/ansible`**
    - The location of the Ansible executable. This might not need to be transferred, but it's important to ensure the same version of Ansible is installed on the isolated server.

When replicating your Ansible setup to an isolated server, ensure that these paths are mirrored to the corresponding locations on the isolated server. Adjust permissions and ownership if needed to ensure Ansible has proper access to these directories and files on the isolated server.
