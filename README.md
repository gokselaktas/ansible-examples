<h1 align="center" id="title">Ansible for Network Engineers</h1>

<p align="center"><img src="https://socialify.git.ci/gokselaktas/ansible-examples/image?description=1&amp;descriptionEditable=Ansible%20for%20Network%20Engineers&amp;font=Raleway&amp;language=1&amp;name=1&amp;owner=1&amp;pattern=Solid&amp;stargazers=1&amp;theme=Light" alt="project-image"></p>

# Ansible for Network Engineers

This repository contains the resources and code for setting up and using Ansible to automate network engineering tasks.

## Software Requirements

Before you begin, ensure you have the following software installed:

- **GNS3 and VMware Workstation**: GNS3 is a network software emulator that allows the combination of virtual and real devices, used to simulate complex networks. VMware Workstation enables you to run multiple operating systems as virtual machines on a single PC.
  - *Resources*: [GNS3](https://www.gns3.com/) | [VMware](https://www.vmware.com/products/workstation-pro.html)

- **Installing GNS3 and VMware**: Follow the installation guides provided for both GNS3 and VMware to set up your environment.

- **Importing Ubuntu Machine and Downloading VirtualBox**: You will need an Ubuntu virtual machine for certain tasks, and VirtualBox is a general-purpose full virtualizer for x86 hardware.
  - *Resources*: [Ubuntu](https://ubuntu.com/download/desktop) | [VirtualBox](https://www.virtualbox.org/)

- **Importing Cisco Images into GNS3**: To simulate network environments, you will need to import Cisco images into GNS3.


## Understanding Ansible's File Structure

In this section, we'll dive into the file structure of Ansible to understand how to organize our automation scripts and configurations effectively.

- **ansible.cfg File**: This is the main configuration file for Ansible. It defines the default settings for Ansible modules, libraries, and plugins.
```
[defaults]
inventory = inventory
host_key_checking = False
retry_files_enabled = False
```

- **Hosts File**: A hosts file consists of host groups and hosts within those groups. A super-set of hosts can be built from other host groups using the :children operator.
```
[routers]
R1    ansible_host=192.168.1.1
R2    ansible_host=192.168.1.2

[switches]
S1    ansible_host=192.168.1.3
S2    ansible_host=192.168.1.4
```

- **"ping" Module**: This module is used to test the connection to your hosts. Understanding how to use it is vital for verifying your setup.
```
sudo ansible all -m ping
```


## Ad-Hoc Commands for Ansible

Ad-Hoc commands are a quick and powerful way to run tasks against your inventory.

- **IOS Modules for Cisco**: Learn which modules are available for managing Cisco IOS devices. It's important to familiarize yourself with these to make full use of Ansible's capabilities for network automation.

- **Using localhost Setup Module**: Understand how to gather facts about your control machine using the localhost setup module. This can be a useful debugging tool.

- **Limit Ad-Hoc Command**: Discover how to restrict Ad-Hoc command execution to specific hosts in your inventory, allowing for more granular control of your network devices.

- **-u -k Ad-Hoc Command**: This command allows you to specify the user and ask for the SSH password when running Ad-Hoc commands. It's a useful option when your authentication is not set up through SSH keys.

## Running "Show" Command Playbooks

Playbooks are the heart of Ansible's configuration, deployment, and orchestration. In this section, we will focus on playbooks that execute 'show' commands on network devices.

- **Show cdp neighbors**: Learn how to create a playbook that retrieves CDP (Cisco Discovery Protocol) neighbor details. This is useful for network discovery and documentation.
```
---
- name: Get device cdp neighbors
  hosts:
    - all
  gather_facts: no
  connection: local

  vars_prompt:
    - name: username
      prompt: Username
      private: no

    - name: password
      prompt: Password
      private: yes

  tasks:
    - name: Get neighbors
      ios_command:
        commands:
          - show cdp neighbors
        provider:
          username: "{{ username }}"
          password: "{{ password }}"
      register: neighbors

    - name: display neighbors
      debug:
        var: neighbors.stdout_lines

```

- **Saving arp table to list**: This playbook will demonstrate how to save the ARP (Address Resolution Protocol) table from a device to a list for later analysis or record-keeping.
```
---
- name: Output ARP table
  hosts:
    - all
  connection: local
  gather_facts: no
  
  vars_prompt:
    - name: username
      prompt: Username
      private: no

    - name: password
      prompt: Password
      private: yes

  tasks:
    - name: Execute arp command
      ios_command:
        commands:
          - show arp
        provider:
          username: "{{ username }}"
          password: "{{ password }}"
      register: arp

    - name: copy arp-table to file
      copy:
        content: "{{ arp.stdout_lines }}"
        dest: "{{ inventory_hostname }}-arp-table.txt"
```

- **Clearing interface counters**: A playbook for clearing interface counters can be helpful in troubleshooting and performance monitoring situations.
```
---
- name: Clear interface counters
  hosts:
    - all
  gather_facts: no

  tasks:
    - name: Clear interface counters
      ios_command:
        commands:
        - command: 'clear counters'
          prompt: 'Clear "show interface" counters on all interfaces'
          answer: 'y'
```
- **Rebooting device**: Sometimes a device may need to be rebooted. This playbook will show how to safely reboot a network device via Ansible.

```
---
- name: Reboot device
  hosts:
    - all
  gather_facts: no

  tasks:
    - name: Reboot device
      ios_command:
        commands:
        - command: 'reload'
          prompt: 'Proceed with reload?'
          answer: 'y'
```

- **Backup device config**: Maintaining backups of device configurations is a best practice. This playbook will cover how to automatically back up the configurations of your network devices.

```
---
- name: Get cisco backup config 
  hosts: all
  gather_facts: no

  tasks:
    - name: Backup config
      ios_config:
          backup: yes
```


Each playbook example provides a practical application of Ansible in a network engineering context, helping you to automate routine but critical tasks.


## Configuring Network Devices

This section is dedicated to the practical aspects of configuring network devices using Ansible. Here, we focus on common network configuration tasks that can be automated.

- **Configure banner**: Setting up a banner is a standard practice for displaying a legal notice or warning message before the user logs into the device. This playbook will connect to all the Cisco IOS devices listed in Ansible inventory, remove any existing MOTD banner, and then set a new MOTD banner based on the contents of a file located at './tmp/banner.cfg'.
```
---
 - name: Configure the login banner
   hosts: all
   gather_facts: no


   tasks:
     - name: Remove old banner
       ios_banner:
          banner: motd
          state: absent

     - name: Replace banner with banner from file
       ios_banner:
          banner: motd
          text: "{{ lookup('file', './tmp/banner.cfg' )}}"
          state: present
```


- **Creating VLANs and users**: VLANs are used to segment networks into different broadcast domains, and user accounts are necessary for access control. The Ansible playbook  is designed to create user accounts on Cisco devices.
```
---
- name: Create users on cisco devices
  hosts: all
  gather_facts: no


  tasks: 
    - name: Create Users and encrypt the passwords
      ios_config:
        lines:
          - username "{{ item.username }}" password "{{ item.password }}"
          - service password-encryption
      loop: "{{ users }}"
```
users.yml should be encrypted.
```
users:
  - username: user1
    password: user1

  - username: user2
    password: user2

  - username: user3
    password: user3

  - username: user4
    password: user4
```
- **Using ansible-vault to encrypt files**: Security is paramount, especially when dealing with configuration files that contain sensitive data. 
```
sudo ansible-vault encrypt ../././users.yml 

sudo ansible-vault view ../././users.yml 


```

