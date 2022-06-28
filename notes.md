# EX457 Red Hat Ansible Network Automaiton Exam Notes:

# DOWNLOAD THE CODE:
https:bit.ly/3ERJcCJ
https://github.com/IPvZero/CodeSamples/tree/main/Ansible/

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

# Accessing Data using Variables

https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#referencing-nested-variables

Many registered variables (and facts) are nested YAML or JSON data structures. You cannot access values from these nested data structures with the simple {{ foo }} syntax. You must use either bracket notation or dot notation.

For example, to reference an IP address from your facts using the bracket notation:

{{ ansible_facts["eth0"]["ipv4"]["address"] }}

To reference an IP address from your facts using the dot notation:

{{ ansible_facts.eth0.ipv4.address }}

# Defining Playbook Variables

Define variables directly in ansible playbook.

https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#defining-variables-in-a-play

You can define variables directly in a playbook play:

- hosts: webservers
  vars:
    http_port: 80

When you define variables in a play, they are only visible to tasks executed in that play.

Defining variables in included files and roles
You can define variables in reusable variables files and/or in reusable roles. When you define variables in reusable variable files, the sensitive variables are separated from playbooks. This separation enables you to store your playbooks in a source control software and even share the playbooks, without the risk of exposing passwords or other sensitive and personal data. For information about creating reusable files and roles, see Re-using Ansible artifacts.

This example shows how you can include variables defined in an external file:

---

- hosts: all
  remote_user: root
  vars:
    favcolor: blue
  vars_files:
    - /vars/external_vars.yml

  tasks:

  - name: This is just a placeholder
    ansible.builtin.command: /bin/echo foo

# Understanding Loops

https://github.com/IPvZero/CodeSamples/tree/main/Ansible/Vars_Loops

Used Perform repetitive tasks.

Examples:  

Configuration needed on multiple interfaces
ip ospf 1 area 0 

Configuring multiple NTP servers
ntp-server [ variable ip ]

Two types of loops

For loop 
While loop

Ansible Loop module documentation

https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html

Ansible offers the loop, with_<lookup>, and until keywords to execute a task multiple times.

Example:  Loop over a list using item special key word

---

- name: "NTP CONFIG PLAYBOOK"
  hosts: cisco
  gather_facts: false
  connection: network_cli

  tasks:
    - name: "Loop through NTP servers and configure"
      vars:
        myservers:
          - "1.1.1.1"
          - "2.2.2.2"
          - "3.3.3.3"
          - "4.4.4.4"
      cisco.ios.ios_ntp_global:
        config:
          peers:
              - peer: "{{ item }}"
                version: 2
      loop: "{{ myservers }}"

Looping through configured interfaces

---

- name: "Loop test"
  hosts: usa
  gather_facts: true
  connection: network_cli

  tasks:
    - name: "Loop through IP info"
      debug:
        msg: "{{ ansible_net_hostname }} has an IP address {{ item }} configured"
      loop: "{{ ansible_net_all_ipv4_addresses }}"

# Conditional Logic & Filtering

https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html

What is Conditional Logic?

"If" a condition is met "then" something can happen.

"If" a condition is not met "Else" can happen.

The When Keywork:

Basic conditionals with when keyword

The when clause is a raw Jinja2 expression without double curly braces (see group_by_module).

Logical Operators:

"And" logic both conditions have to be true

"Or" logic in at least one of the statements must be true

Example using multiple conditions using Logical Operators with Parantheses:

tasks:
  - name: Shut down CentOS 6 and Debian 7 systems
    ansible.builtin.command: /sbin/shutdown -t now
    when: (ansible_facts['distribution'] == "CentOS" and ansible_facts['distribution_major_version'] == "6") or
          (ansible_facts['distribution'] == "Debian" and ansible_facts['distribution_major_version'] == "7")


Using a list a Logical "And" Operator is implied:  All conditions must be met to true.

tasks:
  - name: Shut down CentOS 6 systems
    ansible.builtin.command: /sbin/shutdown -t now
    when:
      - ansible_facts['distribution'] == "CentOS"
      - ansible_facts['distribution_major_version'] == "6"

Writing a Conditional Playbook:

Transforming dictionaries into lists | dict2items }}"

https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#transforming-dictionaries-into-lists

---

- name:  "Playbook to test conditional logic"
  hosts: usa
  gather_facts: true
  connection: network_cli

  tasks:
    - name: "Task 1 print out facts"
      debug:
        msg: "Interface {{ item['key'] }} has an ip address of {{ item['value']['ipv4']['address'] }}" 
      loop: "{{ ansible_net_interfaces | dict2items }}"
      when: item['value']['type'] == "routed"
      or
      when: item['value']['ipv4'] != {} <--IPv4 is not an empty dictionary

Ansible Filters

https://docs.ansible.com/ansible/2.3/playbooks_filters.html

Filters are going to allow us to filter or transform our data.

Filters for Formatting Data

https://docs.ansible.com/ansible/2.3/playbooks_filters.html#filters-for-formatting-data

The following filters will take a data structure in a template and render it in a slightly different format. These are occasionally useful for debugging:

{{ some_variable | to_json }}
{{ some_variable | to_yaml }}
For human readable output, you can use:

{{ some_variable | to_nice_json }}
{{ some_variable | to_nice_yaml }}
It’s also possible to change the indentation of both (new in version 2.2):

{{ some_variable | to_nice_json(indent=2) }}
{{ some_variable | to_nice_yaml(indent=8) }}
Alternatively, you may be reading in some already formatted data:

{{ some_variable | from_json }}
{{ some_variable | from_yaml }}
for example:

tasks:
  - shell: cat /some/path/to/file.json
    register: result

  - set_fact: myvar="{{ result.stdout | from_json }}"

List Filters
These filters all operate on list variables.

New in version 1.8.

To get the minimum value from list of numbers:

{{ list1 | min }}
To get the maximum value from a list of numbers:

{{ [3, 4, 2] | max }}

Symetric Difference filter
---

- name: "Filter Test"
  hosts: "localhost"

  tasks:
    - name: " Testing some filter stuff"
      vars:
        usa_prefixes_list:
          - "192.168.0.0/24"
          - "10.0.0.0/30"
          - "1.1.1.1/32"
        uk_prefixes_list:
          - "192.168.0.0/24"
          - "10.0.0.0/30"
      debug:
        msg: "Difference between two lists is {{ usa_prefixes_list | symmetric_difference(uk_prefixes_list) }}" 

JSON Query Filter 

https://docs.ansible.com/ansible/2.3/playbooks_filters.html#json-query-filter

IP Filters - requires a pip library called netaddr

https://docs.ansible.com/ansible/2.3/playbooks_filters.html#ip-address-filter

pip install netaddr

or 

pip3 install netaddr

---

- name: "IP Filter Test"
  hosts: "localhost"
  
  tasks:
    - name:  "IP addr filter sto validate an IP address"
      vars:
        #myip: "192.168.55.1"
        #myip: "2001:db8:abcd::1"
        myip:
          - "192.168.1.1"
          - "10.0.0.1"
          - "10.0.0.0.1"
          - "2001:db8:aaaa:bbbb::2"
          - "192.168.333.1"
          - "8.8.8.8"
      debug:
        #msg: "{{ myip | ipaddr }}"
        #msg: "{{ myip | ipv4 }}"
        #msg: "{{ myip | ipv6 }}"
        msg: "Print out only valid IP addresses in myip list {{ myip | ipaddr }}"

# Work with Templates

Jinja2

Jinja is a fast, expressive, extensible templating engine. Special placeholders in the template allow writing code similar to Python syntax. Then the template is passed data to render the final document.

Variables
https://jinja.palletsprojects.com/en/3.1.x/templates/#variables

The following lines do the same thing:

We will use these variables to print out a value.

{{ foo.bar }}
{{ foo['bar'] }}

Arista eos config module with the src parameter to take a string value using a jinja2 template. 

https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_config_module.html#ansible-collections-arista-eos-eos-config-module

mkdir templates
create a jinja2 file called bgp.j2

Because you created a templates directory ansible will automatically look in the templates directory for j2 files.

You can also specify a relative path for the j2 files.  
Example: src: /root/ex457_rh_ansible/bgp-configs

---

- name: "Play to test some variable substitution"
  hosts: "R1"
  gather_facts: false
  connection: network_cli

  tasks:
    - name: "Task 1"
      arista.eos.eos_config:
        src: bgp.j2


















