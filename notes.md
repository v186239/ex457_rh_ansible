# EX457 Red Hat Ansible Network Automaiton Exam Notes:

# Install Ansible

pip install ansible

or 

pip3 install ansible

pip3 install --upgrade setuptools
pip3 install --upgrade pip
pip3 install paramiko

# YAML Parser

https://jsonformatter.org/yaml-parser

# Edit Nano to indent YAML files correctly

cd ~

nano .nanorc
set autoindent
set tabsize 2
set tabstospaces

# Ansible modules

https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html

https://docs.ansible.com/ansible/2.9/modules/list_of_network_modules.html

Debugs are in ALL Modules
https://docs.ansible.com/ansible/2.9/modules/debug_module.html#debug-module

# Ansible Ad-HOC commands

An Ansible ad hoc command uses the /usr/bin/ansible command-line tool to automate a single task on one or more managed nodes:

https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html

On network devices we can use the raw module to run ad-hoc commands.

ansible usa -m raw -c paramiko -a "show ip interface brief"

# Playbooks

Cisco IOS Modules
https://docs.ansible.com/ansible/latest/collections/cisco/ios/

Module:  cisco.ios.{{name of module}}:

         cisco.ios.ios_banner:

Before you can use a module you want to make sure you have the collection by
running this command:

ansible-galaxy collection list
 
If the module is not listed in the collection you can install with:

ansible-galaxy collection install cisco.ios

# Variables

Ansible is running python under the hood.  

Concept of a variable or var is a value that can change through out the exectution of a program.

Example:  a username
name:  {{ name }}  <--Variable memory location holding data

Ansible can define host or group variables

# Host and group variable files

Must use YAML syntax

host variables are unique to the host like an ip address

group variables apply to a group of hosts in the inventory

How to build Ansible Inventory includes organizing host and group variables

https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#organizing-host-and-group-variables

Ansible loads host and group variable files by searching paths relative to the inventory file or the playbook file.

Order of operations - 
  - Host var will be selected over a group var
  - Child group 
  - Parent group
  - All group last

By default Ansible merges groups at the same parent/child level in "ASCII" order, and the last group loaded overwrites the previous groups. For example, an a_group will be merged with b_group and b_group vars that match will overwrite the ones in a_group.
