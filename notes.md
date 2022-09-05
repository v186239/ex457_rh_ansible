# EX457 Red Hat Ansible Network Automaiton Exam Notes:

# DOWNLOAD THE CODE:
https:bit.ly/3ERJcCJ
https://github.com/IPvZero/CodeSamples
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

------------------------------------------------------------------------------------------------------

Using loop control labels in debug module to only show output from commands example

---
- name: "Play 1 - Configure syslog and target Arista Devices"
  hosts: arista
  connection: network_cli

  tasks:
    - name: Task 1 configure host logging when no port key is defined
      arista.eos.eos_logging:
        dest: host
        name: "{{ item.ip }}"
        state: present
      loop: "{{ syslog.hosts }}"
      when: item.port is not defined
      register: task1_output

    - name: "Print Task1 output"
      debug:
        msg: "{{ item.commands }}"
      loop: "{{ task1_output.results }}"
      loop_control:
        label: none   <-------------------------------Nothing special about the world none is just label with the key.  If key is not present it will not output>
      when: item.commands is defined

    - name: Task 2 configure host logging when port key is defined
      arista.eos.eos_logging:
        dest: host
        name: "{{ item.ip }}"
        provider:
          port: "{{ item.port }}"
        state: present
      loop: "{{ syslog.hosts }}"
      when: item.port is defined
      register: task2_output

    - name: "Print Task2 output"
      debug:
        msg: "{{ item.commands }}"
      loop: "{{ task2_output.results }}"
      loop_control:
        label: none
      when: item.commands is defined



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

# Work with Jinja2 Templates

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



Dynamic Lookups will lookup a file depending on a particular values within the inventory host vars. 

For example when dealing with multiple vendors with different os environments.

Create two j2 files for each vendor (normal.j2 or named.j2)

In the hostvar for each device add the key:value for the style of vendor (example: eigrp_style: named or normal)

In the playbook call the src file with the variable example: eigrp_style

---

- name: "Play to test some variable substitution"
  hosts: "R1"
  gather_facts: false
  connection: network_cli

  tasks:
    - name: "Task 1"
      arista.eos.eos_config:
        src: "eigrp/{{ eigrp_style }}.j2"    



Jinja2 Loops with BGP logic

{% for item in my_list %}

Example:  Repetition think loops
-------------------------------------
Host vars file:
bgp_style: bgp

BGP:
  ASN: "65001"
  peers: 
    - neighbor: 10.199.199.12
      peer_asn: "65002"
---------------------------------------

router bgp {{ BGP.ASN }}

{% for nbor in BGP.peers %}
neighbor {{ nbor.neighbor }} remote-as {{ nbor.peer_asn}}
{% endfor %}  # this will end the for loop on nbor

--------------OUTPUT FROM LOOP--------------------
router bgp 65001
neighboer 10.199.199.12 remote-as 65001
---------------------------------------------------


Jinja2 Conditions with BGP logic

In the lab clear BGP config on all devices.

Some configurations on some devices based on a conditional statement.

if: if something is true

elif: if something else is true

else: if none of the conditions above are true use this else fail safe statement

endif:  will end the conditional statement

Include the conditional statements within the for loop.

--------------------------------------------------------
router bgp {{ BGP.ASN }}

{% for nbor in BGP.peers %}
{% if nbor.peer_asn == "65002" %}
neighbor {{ nbor.neighbor }} remote-as {{ nbor.peer_asn}}
neighbor {{ nbor.neighbor}} update-source loopback0
{% else %}
neighbor {{ nbor.neighbor }} remote-as {{ nbor.peer_asn}}
{% endif %}
{% endfor %}

---------------------------------------------------



# Registers, Handlers & Vault Secrets

 - Registering Variables 

When we push configs to a devices how do we see what commands have taken?

Capture the output from a particular task using the "register" module in ansible. 
The register module will capture the output into a variable which can be used with a debug module to print output.

---

- name: "REGISTER PLAYBOOK"
  hosts: switch
  gather_facts: false
  connection: network_cli

  tasks:
   - name: "Push NTP config"
     arista.eos.eos_config:
       src: "ntp.j2"
     register: ntp_result

   - name: "Print result"
     debug:
       msg:  "{{ ntp_result.commands }}"

---------------------------------------------------

 - Handlers

https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html

Handlers running operation on change of state like a conditional.

Notify allows us to run a specific task on the conditional basis.

Example 1:
---

- name: "REGISTER PLAYBOOK"
  hosts: switch
  gather_facts: false
  connection: network_cli

  tasks:
   - name: "Push NTP config"
     arista.eos.eos_config:
       src: "ntp.j2"
     notify: ntp_handler_example
     register: ntp_result

  handlers:
   - name: ntp_handler_example
     debug:
       msg: "{{ ntp_result.commands }}"

Example 2:  With the listen key to listen for the notify name.   

---

- name: "REGISTER PLAYBOOK"
  hosts: switch
  gather_facts: false
  connection: network_cli

  tasks:
   - name: "Push NTP config"
     arista.eos.eos_config:
       src: "ntp.j2"
     notify: ntp_handler_example
     register: ntp_result

  handlers:
   - name: "HANDLER 1 - This will check for NTP Updates"
     listen: ntp_handler_example
     debug:
       msg: "{{ ntp_result.commands }}"

What to Encrypt?

Credentials for devices need to be encrypted using ansible vault.

Safely store username and passwords and avoid clear text transfer using ansible vault.

Store credentials in variables and vault them before pushing them to Git (data at rest).

------------------------------   Ansible Vault  ----------------------------------------------------

https://docs.ansible.com/ansible/latest/user_guide/vault.html

Ansible Vault encrypts variables and files so you can protect sensitive content such as passwords or keys rather than leaving it visible as plaintext in playbooks or roles. To use Ansible Vault you need one or more passwords to encrypt and decrypt content. If you store your vault passwords in a third-party tool such as a secret manager, you need a script to access them. Use the passwords with the ansible-vault command-line tool to create and view encrypted variables, create encrypted files, encrypt existing files, or edit, re-key, or decrypt files. You can then place encrypted content under source control and share it more safely.


Managing vault passwords
Managing your encrypted content is easier if you develop a strategy for managing your vault passwords. A vault password can be any string you choose. There is no special command to create a vault password. However, you need to keep track of your vault passwords. Each time you encrypt a variable or file with Ansible Vault, you must provide a password. When you use an encrypted variable or file in a command or playbook, you must provide the same password that was used to encrypt it. To develop a strategy for managing vault passwords, start with two questions:

Do you want to encrypt all your content with the same password, or use different passwords for different needs?

Where do you want to store your password or passwords?

Choosing between a single password and multiple passwords
If you have a small team or few sensitive values, you can use a single password for everything you encrypt with Ansible Vault. Store your vault password securely in a file or a secret manager as described below.

If you have a larger team or many sensitive values, you can use multiple passwords. For example, you can use different passwords for different users or different levels of access. Depending on your needs, you might want a different password for each encrypted file, for each directory, or for each environment. For example, you might have a playbook that includes two vars files, one for the dev environment and one for the production environment, encrypted with two different passwords. When you run the playbook, select the correct vault password for the environment you are targeting, using a vault ID.

Managing multiple passwords with vault IDs
If you use multiple vault passwords, you can differentiate one password from another with vault IDs. You use the vault ID in three ways:

Pass it with --vault-id to the ansible-vault command when you create encrypted content

Include it wherever you store the password for that vault ID (see Storing and accessing vault passwords)

Pass it with --vault-id to the ansible-playbook command when you run a playbook that uses content you encrypted with that vault ID

When you pass a vault ID as an option to the ansible-vault command, you add a label (a hint or nickname) to the encrypted content. This label documents which password you used to encrypt it. The encrypted variable or file includes the vault ID label in plain text in the header. The vault ID is the last element before the encrypted content.

------------------------------   Ansible Vault  ----------------------------------------------------

Using Ansible Vault 

ansible-vault --help
positional arguments:
  {create,decrypt,edit,view,encrypt,encrypt_string,rekey}
    create              Create new vault encrypted file
    decrypt             Decrypt vault encrypted file
    edit                Edit vault encrypted file
    view                View vault encrypted file
    encrypt             Encrypt YAML file
    encrypt_string      Encrypt a string
    rekey               Re-key a vault encrypted file

To encrypt an already existing file use 

ansible-vault encrypt {{ filename }}

root@eveng-2:~/ex457_rh_ansible/group_vars# ansible-vault encrypt usa.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
root@eveng-2:~/ex457_rh_ansible/group_vars# cat usa.yml 
$ANSIBLE_VAULT;1.1;AES256
61396333346162656162363735643363333766323230623261316466323965656532313831386265
3130326638383732303633393738333337626139613466390a656165343335656561663338396138
65313865663735363234306335633061333166326666353762363031326632623665363666666666
6534376264366262350a393330343966353330323637653634386563663665626461346338613835
30663035376631393637323764343935613165316561613535313837346263613737623662656561
34333333343065343832336263363338396130363239343733393136303665306138386466333463
64366631616163376462356233376231376330353736653636363066353161303737393339366665
65386366653337323461653764613261663331326638623739333463313261346537303763343335
6531

---

- name: " Print some encrypted stuff"
  hosts: usa
  gather_facts: no

  tasks:
    - name: "Task1 - decrypt and print"
      debug:
        msg: "{{ super_secret_bgp }}"

root@eveng-2:~/ex457_rh_ansible# ansible-playbook encrypt-test.yml --ask-vault-pass

To view information in the encrypted file use

root@eveng-2:~/ex457_rh_ansible# ansible-vault view group_vars/usa.yml 
Vault password: 
---

ntp: "10.10.10.10"

ospf:
  rid: "3.3.3.3"

super_secret_bgp: 5432
ninja_ospf: "5"
root@eveng-2:~/ex457_rh_ansible# 

To edit encrypted file

root@eveng-2:~/ex457_rh_ansible# ansible-vault edit group_vars/usa.yml 


How to create an encrypted file

root@eveng-2:~/ex457_rh_ansible/group_vars# ansible-vault create uk.yml

---

- name: " Print some encrypted stuff"
  hosts: uk
  gather_facts: no

  tasks:
    - name: "Task1 - decrypt and print"
      debug:
        msg: "{{ snmp_stuff }}"

root@eveng-2:~/ex457_rh_ansible# ansible-playbook encrypt-test.yml --ask-vault-pass

TASK [Task1 - decrypt and print] ********************************************************************************************************************************************************************************
MSG:

{'community': 'cbtnuggets', 'server': '7.7.1.2', 'mode': 'RW', 'location': 'Oregon', 'contact': 'John McGovern'}

To Decryt a file use
root@eveng-2:~/ex457_rh_ansible# ansible-vault decrypt group_vars/uk.yml 
Vault password: 
Decryption successful

---

snmp_stuff:
  community: "cbtnuggets"
  server: 7.7.1.2
  mode: "RW"
  location: "Oregon"
  contact: "John McGovern"

To encrypt the file again use

root@eveng-2:~/ex457_rh_ansible# ansible-vault encrypt group_vars/uk.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful

To rekey or change the password of an already encrypted file use

root@eveng-2:~/ex457_rh_ansible# ansible-vault rekey group_vars/uk.yml 
Vault password: 
New Vault password: 
Confirm New Vault password: 
Rekey successful

# Ansible Galaxy and Roles

Roles make our jobs way easier.

Role gives you a reusable, portable, bite-sized playbook to allow modifications and collaboration with other team memembers.

Similar to a python function which can be easily shared.

You can import other playbooks into a role.

When we create a role it builds a core structure.  The name of the role defines the TOP level directory.

ntp-config
 - tasks
 - handlers

You can manually create a role structure but it is much easier to use the auto-generate command below.

Ansible Galaxy can be used to create roles using auto-generate command "ansible-galaxy init {{ name of the role}}"

--------- ROLE DIRECTORY STRUCTURE -----------

root@eveng-2:~/ex457_rh_ansible# ansible-galaxy init nuggets

root@eveng-2:~/ex457_rh_ansible# tree nuggets/
nuggets/
├── README.md
├── defaults  <---Low order of preference for variables
│   └── main.yml
├── files     <---Static files used by the roles tasks
├── handlers  <---Used to define handlers with the notify key
│   └── main.yml
├── meta      <---metadata about the role itself.  basic info of the role, like the author of the role, licensing, platforms, dependencies for the role to run
│   └── main.yml
├── tasks
│   └── main.yml <--- Ansible playbook tasks that the role is going to take
├── templates <---Templates directory used to store Jinja2 Template files
├── tests  <--- Designed for testing lab environments using the inventory of lab devices with the test.yml file.  
│   ├── inventory
│   └── test.yml
└── vars <---Used to store variables for the playbook tasks that has precedence over the defaults main.yml variables file.
    └── main.yml

--------- ROLE DIRECTORY STRUCTURE -----------

Roles are supposed to be portable.  You do not want to hard code credentials within that role. 

Pass in credentials as a parameter.  

Creating a Role

ansible-galaxy init ospf_role

root@eveng-2:~/ex457_rh_ansible# tree ospf_role/
ospf_role/
├── README.md
├── defaults
│   └── main.yml <--Has the variables used in the tasks
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml  <--Has the playbook tasks to configure OSPF
├── templates
│   └── ospf_role_example.j2
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml


Create a playbook which uses the role

---

- name: "Playbook to configure OSPF using a role"
  hosts: switch
  connection: network_cli

  roles:
    - ospf_role

Then run the playbook

root@eveng-2:~/ex457_rh_ansible# ansible-playbook execute_ospf_role.yml -i hosts


----------- EXPLORING ANSIBLE GALAXY ------------------------------------------

You can use other peoples roles hosted on the Ansible Galaxy website. 

Type of Market place for ansible and roles which you can download and have full access to use.

https://galaxy.ansible.com/


You can click community to see collections, download, install or go straight to GIT repo.

Example search for genie collection

https://galaxy.ansible.com/clay584/genie

Next we will download genie and use it as an example.

This will show you everything you installed from ansible-galaxy 

ansible-galaxy list

Search ansible-galaxy from command line

ansible-galaxy search netbox

Command to install genie ansible collection

ansible-galaxy collection install clay584.genie

apt-get install python3-venv

root@eveng-2:~/ex457_rh_ansible# python3 -m venv .venv
root@eveng-2:~/ex457_rh_ansible# cd .venv/
root@eveng-2:~/ex457_rh_ansible/.venv# source bin/activate
(.venv) root@eveng-2:~/ex457_rh_ansible/.venv# cd ..
(.venv) root@eveng-2:~/ex457_rh_ansible# 
(.venv) root@eveng-2:~/ex457_rh_ansible# pip3 --version
(.venv) root@eveng-2:~/ex457_rh_ansible# pip3 install pyats
(.venv) root@eveng-2:~/ex457_rh_ansible# pip3 install genie

Create an example playbook to parse data

---

- name: "Playbook1: Testing Genie"
  hosts: switch
  conneciton: network_cli

  tasks:
    - name: "Task 1 - send a show command"
      arista.eos.eos_command:
        commands: "show version"
      register: show_version_output

    - name: "Task 2 - Parse show command"
      set_fact: #  This will create a variable from the register show_version_output
        parsed_data: >-
          {{ show_version_output.stdout[0] }}

    - name: "Task 3- print show command"
      ansible.builtin.debug:
        msg: "{{ parsed_data }}"

Create and example playbook to parse data using genie | filter within the varaible

---

- name: "Playbook1: Testing Genie"
  hosts: switch
  connection: network_cli

  tasks:
    - name: "Task 1 - send a show command"
      arista.eos.eos_command:
        commands: "show version"
      register: show_version_output

    - name: "Task 2 - Parse show command"
      set_fact: 
        parsed_data: >-
          {{ show_version_output.stdout[0] | clay584.genie.parse_genie(command='show version', os='eos') }}

    - name: "Task 3- print show command"
      ansible.builtin.debug:
        msg: "This device has been up for {{ parsed_data.version.uptime }}"

# VYOS

VYOS is an Open Source Router and Firewall.

GNU / LINUX WITH NETWORKING FUNCTIONS 

ANSIBLE MODULES FOR VYOS ARE AVAILABLE.

https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_config_module.html

https://docs.ansible.com/ansible/latest/collections/vyos/vyos/index.html

https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_command_module.html#ansible-collections-vyos-vyos-vyos-command-module

https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_config_module.html#ansible-collections-vyos-vyos-vyos-config-module

VYOS uses "set" command to configure things.

Examples Ansible Playbook Tasks:

- name: configure the remote device
  vyos.vyos.vyos_config:
    lines:
    - set system host-name {{ inventory_hostname }}
    - set service lldp
    - delete service dhcp-server

- name: backup and load from file
  vyos.vyos.vyos_config:
    src: vyos.cfg
    backup: yes

- name: render a Jinja2 template onto the VyOS router
  vyos.vyos.vyos_config:
    src: vyos_template.j2

There is also an Ansible Galaxy for VYOS

ansible-galaxy collection install vyos.vyos

Example VyOS playbooks:

---

- name: "Vyos Playbook"
  hosts: vyos
  connection: network_ci

  tasks:
    - name: "Push some banner configuration to Vyos"
      vyos.vyos.vyos_banner:
        banner: pre-login
        text: "CBG NUGGETS BANNER for VyOS"
        state: present

---

- name: "Multivendor Fact-Gathering Playbook"
  hosts: all
  connection: network_cli

  tasks:
    - name: "Pull Arista facts"
      arista.eos.eos_facts:
        gather_subset: config
      register: eos_facts
      when: "ansible_network_os == 'arista.eos.eos'"

    - name: "Pull VyOS facts"
      vyos.vyos.vyos_facts:
        gather_subset: config
      register: vyos_facts
      when: "ansible_network_os == 'vyos.vyos.vyos'"

    - name: "Print Arista Output"
      debug:
        msg: "{{ eos_facts.ansible_facts.ansible_net_config }}"
      when: "ansible_network_os == 'arista.eos.eos'"
  
    - name: "Print VyOS Output"
      debug:
        msg: "{{ vyos_facts.ansible_facts.ansible_net_config[0] }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"

----- NOTE FOR ANSIBLE.CFG FILE YOU CAN SKIP DISPLAY OF SKIP DEVICES WHEN CONDITIONS ARE USED  ----------

edit ansible.cfg and add this line

display_skipped_hosts = false

-----------------------------------------------------------------------------------------------------------------

Interface Configuration

Arista Ansible Interface module

https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_interfaces_module.html

VyOS Ansible Interface module

https://docs.ansible.com/ansible/latest/collections/vyos/vyos/index.html


To move data in VS Code hold down Ctrl and push the right or left square brackets [ or ]

---

- name: "Multivendor Fact-Gathering Playbook"
  hosts: all
  connection: network_cli

  tasks:
    - name: Merge provided configuration with device configuration
      arista.eos.eos_interfaces:
        config:
        - name: Ethernet1
          description: Configured by Will Ansible
          enabled: true
        - name: Ethernet2
          description: Configured by Will Ansible
          enabled: true
        state: merged
      register: eos_facts
      when: "ansible_network_os == 'arista.eos.eos'"

    - name: Merge provided configuration with device configuration
      vyos.vyos.vyos_interfaces:
        config:
        - name: eth2
          description: Configured by Will using Ansible
          enabled: true
          vifs:
          - vlan_id: 200
            description: VIF 200 - ETH2

        - name: eth3
          description: Configured by Will using Ansible
          mtu: 1500
        state: merged
      register: vyos_facts
      when: "ansible_network_os == 'vyos.vyos.vyos'"

    - name: "Print Arista Output"
      debug:
        # msg: "{{ eos_facts.ansible_facts.ansible_net_config.split('\n') }}"
        msg: "{{ eos_facts.commands }}"
      when: "ansible_network_os == 'arista.eos.eos'"
  
    - name: "Print VyOS Output"
      debug:
        # msg: "{{ vyos_facts.ansible_facts.ansible_net_config[0].split('\n')  }}"
        msg: "{{ vyos_facts.commands }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"

# Configuring OSPF

Before you start automating OSPF, always make sure you know what it is that you are trying to automation. 

The blast radius for automation is huge and you can cause outages by pushing out automaiton.  

Dymestify OSPF (OPEN SHORTEST PATH FIRST) ROUTING PROTOCOL

Devices share routing information using OSPF dynamic routing protocol.

OSPF is open standard protocol used by any vendor.  Unlike Cisco EIGRP.

OSPF is a link state routing protocol.  Devices will share routing information across the area.

Network AS managed by ansible.   

Area 0 backbone area will connect to other areas.    

All devices in the same area are internal and share routing links state information. 

Multiarea OSPF allows us to partiion devices in different areas so link states are not impacted if automation tools break the network.

OSPF devices form a neighbor relationship using hello packets then form an adjacency.  Once they devices are adjacent they can share routing information.

Needed for OSPF adjacency. 
1 - Area
2 - Subnet
3 - Authentication
4-  Matching Timers 
5-  Router ID - must be unique, using IP address.  Try to avoid duplicate Router ID on different devices.

OSPF Stub areas - cut down amount of information coming into an OSPF area you can use a stub flag, which must match on both sides of the link.  

PASSIVE INTERFACE - OSPF has a concept know as PASSIVE INTERFACE - used to block OSPF packet advertisement out of an interface, like to none OSPF speaking devices.

BE CAREFUL, DONT BE LAZY AND ASSUME OSPF CONFIGURATION WON'T BREAK YOUR NETWORK.  UNDERSTAND WHERE YOU WILL BE CONFIGURING OSPF AND INTERFACES USED FOR OSPF!

--------------------------- OSPF TOPOLOGY SETUP -----------------------------

Using EVE-NG we built a topology with R1 connected to R2,  R2 connected to R3 and R4.

Updated hosts inventory with R3 and R4.

--------------------------- OSPF TOPOLOGY SETUP -----------------------------

--------------------------- OSPF AUTOMATION -----------------------------

Let's start OSPF automation using an Ansible Playbook.

First we will try to configure OSPF using OSPF Ansible modules with a simple configuration.

Create a playbook called ospf1.yml and configure OSPF using config lines

---

- name: "OSPF PLAYBOOK"
  hosts: all
  connection: network_cli

  tasks:
    - name: "Push Arista OSPF Config"
      arista.eos.eos_config:
        lines: 
          - router ospf 1
          - router-id {{ ospf.rid }}
          - network 0.0.0.0 255.255.255.255 area 0
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"

    - name: "Print Arista OSPF Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"

Now configure OSPF using the arista.eos.eos_ospfv2 module for OSPF instead.

Arista OSPF ansible module
https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_ospfv2_module.html#ansible-collections-arista-eos-eos-ospfv2-module

VyOS OSPF ansible module
https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_config_module.html

---

- name: "OSPF PLAYBOOK"
  hosts: all
  connection: network_cli

  tasks:
    - name: "Push Arista OSPF Config"
      arista.eos.eos_ospfv2:
        config:
          processes:
            - process_id: 1
              router_id: "{{ ospf.rid }}"
              networks:
                - area: 13
                  prefix: 13.14.15.0/24
                - area: 0
                  prefix: 10.0.0.0/24
        state: merged
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"

    - name: "Print Arista OSPF Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"

    - name: "Push VyOS OSPF Config"
      vyos.vyos.vyos_ospfv2:
        config:
          areas:
            - area_id: '0'
              network:
                - address: "10.2.0.0/16"
                - address: "12.13.0.0/16"

        state: replaced  # <---- Will clean up and remove rogue configs.  Prevents configuration drifts.
      register: vyos_output
      when: "ansible_network_os == 'vyos.vyos.vyos'"

    - name: "Print VyOS OSPF Config"
      debug:
        msg: "{{ vyos_output.commands }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"


OSPF Jinja2 Templates

First configure host vars for R1, R2, R3, R4 with key values for ospf.networks

Then build Jinja2.

VYOS OSPF Jinja2 Template
set protocols ospf parameters router-id {{ ospf.rid }}
{% for ntwk in ospf.networks %}
set protocols ospf area {{ ntwk.area }} network {{ ntwk.network }}
{% endfor%}

Arista OSPF Jinja2 Template
router ospf 1
   router-id {{ ospf.rid }}
{% for ntwk in ospf.networks %}
network {{ ntwk.prefix }} area {{ ntwk.area }}
{% endfor %}
{% if ospf.default_originate == true %}
default-information originate
{% endif %}


---

- name: "OSPF CONFIGS via JINJA"
  hosts: all
  connection: network_cli

  tasks:
    - name: "Configure VyOS OSPF"
      vyos.vyos.vyos_config:
        src: "{{ ansible_network_os }}-ospf.j2"
      register: vyos_output
      when: "ansible_network_os == 'vyos.vyos.vyos'"
    
    - name: "Print VyOS OSPF Config"
      debug:
        msg: "{{ vyos_output.commands }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"

    - name: "Configure Arista OSPF"
      arista.eos.eos_config:
        src: "{{ ansible_network_os }}-ospf.j2"
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"
    
    - name: "Print Arista OSPF Config"
      debug:
        msg: "{{ arista_output.commands }}"
      when: "ansible_network_os == 'arista.eos.eos'"

--------------------- BGP OVERVIEW -----------------------------------------

BGP Border Gateway Protocol

BGP is the routing protocol used by the Internet

BGP is an EGP Exterior Gateway Protocol

BGP is used to communicate between different networks, externally, out of your control.

BGP is used to communicate between autonomous systems.

ASN Autonomous System Numbers
   - Private ASN range numbers 64512 to 65535 used mainly internally within an autonomous system.
   - Public ASN range numbers 1 to 64511.

BGP scales very well.   

eBGP - External BGP - BGP neighbors with different ASN numbers

iBGP - Internal BGP - BGP neighbors with the same ASN numbers

By default, eBGP runs a connected check to validated connectivity between directly connected peers.

With eBGP, if you want to peer with a loopback it does require a change to the TTL (default 1 hop) to multihop.  

With iBGP, the IGP is used and peering with loopback easier if IGP protocol provides connectivity.

vYOS uses something called BGP unnumbered (similar to Cumulus linux ) which uses the keyword external or internal to specify the remote peer ASN automatically under the hood.

--------------------- Create a BGP Topology  -----------------------------------------

R1 = ASN1, R2 = ASN2, R3 = ASN3, R4 = ASN4

R1 BGP configuration on Arista.  This configuration could be variablized using Jinja2.  Network statements can be looped with a condition in a Jinja2 template.

R1(config-router-bgp)#do show run | sec bgp
router bgp 1
   router-id 1.1.1.1
   neighbor 10.1.2.2 remote-as 2
   neighbor 10.1.2.2 maximum-routes 12000 
   network 99.99.99.99/32
   network 100.100.100.100/32

R2#show run | sec bgp
router bgp 2
   router-id 2.2.2.2
   neighbor 10.1.2.1 remote-as 1
   neighbor 10.1.2.1 maximum-routes 12000 
   neighbor 10.2.3.2 remote-as 3
   neighbor 10.2.3.2 maximum-routes 12000 
   neighbor 10.2.4.2 remote-as 4
   neighbor 10.2.4.2 maximum-routes 12000

vyos@R3-yos# show protocols bgp
vyos@R3-yos# history | grep 'set protocols bgp'
set protocols bgp local-as 3
set protocols bgp parameters router-id 3.3.3.3
set protocols bgp neighbor 10.2.3.1 address-family ipv4-unicast 
set protocols bgp neighbor 10.2.3.1 remote-as external
set protocols bgp neighbor 10.3.4.2 address-family ipv4-unicast 
set protocols bgp neighbor 10.3.4.2 remote-as external 

vyos@R4-vyos:~$ history | grep 'set protocols bgp'
set protocols bgp local-as 4
set protocols bgp parameters router-id 4.4.4.4
set protocols bgp neighbor 10.2.4.1 address-family ipv4-unicast 
set protocols bgp neighbor 10.2.4.1 remote-as external 
set protocols bgp neighbor 10.3.4.1 address-family ipv4-unicast 
set protocols bgp neighbor 10.3.4.1 remote-as external 

--------------------- BGP Configuration Templates using Jinja2 -----------------------------------------

Arista BGP Jinja2
router bgp {{ bgp.asn }}
  router-id {{ bgp.rid }}
{% for neigh in bgp.neighbors %}
  neighbor {{ neigh.neighbor }} remote-as {{ neigh.peer_asn }}
{% endfor %}
{% if bgp.networks is defined %}
{% for ntwk in bgp.networks %}
  network {{ ntwk.network }}
{% endfor %}
{% endif %}

VyOS BGP Jinja2
set protocols bgp local-as {{ bgp.asn }}
set protocols bgp parameters router-id {{ bgp.rid }}
{% for neigh in bgp.neighbors %}
set protocols bgp neighbor {{ neigh }} address-family ipv4-unicast 
set protocols bgp neighbor {{ neigh }} remote-as external
{% endfor %}

Ansible Playbook:

---

- name: "BGP CONFIGS via JINJA"
  hosts: all
  connection: network_cli

  tasks:
    - name: "Configure VyOS BGP"
      vyos.vyos.vyos_config:
        src: "{{ ansible_network_os }}-bgp.j2"
      register: vyos_output
      when: "ansible_network_os == 'vyos.vyos.vyos'"
    
    - name: "Print VyOS BGP Config"
      debug:
        msg: "{{ vyos_output }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"

    - name: "Configure Arista BGP"
      arista.eos.eos_config:
        src: "{{ ansible_network_os }}-bgp.j2"
        # match: "none"  # <---- Note that Arista and Cisco IOS have indentation in the configuration.  Using match none ignores indentation by ansible.  
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"
    
    - name: "Print Arista BGP Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"

Example R1 host vars:
bgp:
  asn: "1"
  rid: "1.1.1.1"
  neighbors:
    - neighbor: "10.1.2.2"
      peer_asn: "2"
  networks:
    - network: "99.99.99.99/32"
    - network: "100.100.100.100/32"

Example R3 host vars:
bgp:
  asn: "3"
  rid: "3.3.3.3"
  neighbors:
    - "10.2.3.1"
    - "10.3.4.2"

--------------------- BGP Configuration using Ansible Modules -----------------------------------------

BGP feature Ansible Modules

Arista eos BGP Module

https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_bgp_module.html#ansible-collections-arista-eos-eos-bgp-module

VyOS BGP Module

https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_bgp_global_module.html#ansible-collections-vyos-vyos-vyos-bgp-global-module

Example playbook:

---

- name: "BGP CONFIGS Playbook via MODULES"
  hosts: all
  connection: network_cli

  tasks:
    - name: "Configure Arista BGP"
      arista.eos.eos_bgp:
        config:
          bgp_as: "{{ bgp.asn }}"
          router_id: "{{ bgp.rid }}"
          log_neighbor_changes: true
          neighbors:
          - neighbor: "{{ item.neighbor }}"
            remote_as: "{{ item.peer_asn}}"
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"
      loop: "{{ bgp.neighbors }}"  <--loop hostvars
    
    - name: "Print Arista BGP Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"


# Configure VLANS

--------------------- VLAN OVERVIEW -----------------------------------------

VLAN = VIRTUAL LAN (LOCAL AREA NETWORK)

4 PCs---SW1-----SW2----4 PCs

Broadcast communicate to all devices on a network.

When a devices sends a broadcast packet the switches forward out every interface.

VLANS allow selective broadcast amongst a targeted group of hosts.

VLANS break up the layer 2 broadcasts in a switch.

VLANS allow switches to group interfaces together logically for different departments in a company like HR vs Finance Dept.

Imagine each switch was on a different floor in a building.  

Trunks allow multiple VLANS to traverse across links between switches.  VLANs are configured as allowed on trunks. 

Access ports are used to connect to devices like a PC.

Trunk ports are used for switch to switch communication.

Layer 3 SVIs switch virtual interfaces are used to allow communication between different VLANs.  

Access control list can be configured on SVIs to limit communication to certain devices in a VLAN.

Be carefull when configuring VLANS using automation because you can cause an outage if you wipe out VLANs on let's say a trunk.

--------------------- VLAN Create Topology -----------------------------------------

PC6 VLAN 5, PC7 VLAN 10 connected to "SW1----TRUNK-----SW2" connected to PC8 VLAN 5, PC9 VLAN 10

--------------------- VLAN Configuration Jinja2 Templates  -----------------------------------------

Ansible Playbook:
---

- name: "VLAN CONFIGS via JINJA"
  hosts: arista
  connection: network_cli

  tasks:
    - name: "Configure Arista VLAN"
      arista.eos.eos_config:
        src: "{{ ansible_network_os }}-vlans.j2"
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"
    
    - name: "Print Arista VLAN Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"

HOST VARS

vlans:
  interfaces:
    - interface: "Et4"
      mode: access
      vlan: 5
    - interface: "Et5"
      mode: access
      vlan: 10
    - interface: "Et2"
      mode: trunk
  vlan:
    - number: 5
      name: "FIVE"
      svi: "192.168.5.1 255.255.255.0"
    - number: 10
      name: "TEN"
      svi: "192.168.10.1 255.255.255.0"

JINJA2 Template

{% for vl in vlans.vlan %}
vlan {{ vl.number }}
name {{ vl.name }}
interface vlan {{ vl.number }}
ip address {{ vl.svi }}
no shut
exit
{% endfor %}

{% for intf in vlans.interfaces %}
{% if intf.mode == "access"%}
interface {{ intf.interface }}
switchport mode access
switchport access vlan {{ intf.vlan }}
{% else %}
interface {{ intf.interface }}
  switchport mode trunk
exit
exit
copy run start
{% endif %}

--------------------- VLAN Configuration using ANSIBLE MODULES  -----------------------------------------


Arista EOS VLANS module
https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_vlans_module.html#ansible-collections-arista-eos-eos-vlans-module

Arista EOS L2 Interfaces module used to specify mode access or trunk ports
https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_l2_interfaces_module.html#ansible-collections-arista-eos-eos-l2-interfaces-module

Cisco IOS VLANS Module
https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_vlans_module.html#ansible-collections-cisco-ios-ios-vlans-module

Cisco IOS L2 Interfaces module used to specify mode access or trunk ports
https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_l2_interfaces_module.html#ansible-collections-cisco-ios-ios-l2-interfaces-module

VyOS VLANS Module
https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_vlan_module.html#ansible-collections-vyos-vyos-vyos-vlan-module



Be careful when configuring the state of a VLAN.  The Overide option can remove vlans and cause outages.

Arista VLAN module playbook
---

- name: "vlans CONFIGS Playbook via MODULES"
  hosts: usa
  connection: network_cli

  tasks:
    - name: "Configure Arista vlans"
      arista.eos.eos_vlans:
        config:
        - name: MGMT_EVENG
          vlan_id: 1
          state: active
        - name: FIVE
          vlan_id: 5
          state: active
        - name: TEN
          vlan_id: 10
          state: active
        # state: overridden  # this statement will remove any VLANS not listed in this playbook
         state: merged
      register: arista_output
      when: "ansible_network_os == 'arista.eos.eos'"
 
    
    - name: "Print Arista vlans Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"


Arista L2 Interface Module playbook
---

- name: "vlans interface CONFIGS Playbook via MODULES"
  hosts: usa
  connection: network_cli

  tasks:
    - name: "Configure Arista ACCESS ports for vlans"
      arista.eos.eos_l2_interfaces:
        config:
        - name: "{{ item.interface }}"
          mode: access
          access:
            vlan: "{{ item.vlan }}"
        state: merged
      register: arista_output
      loop: "{{ vlans.interfaces }}"
      when: item.mode == "access" and "ansible_network_os == 'arista.eos.eos'"

    - name: "Configure Arista TRUNK ports for vlans"
      arista.eos.eos_l2_interfaces:
        config:
        - name: "{{ item.interface }}"
          mode: trunk
        state: merged
      register: arista_output
      loop: "{{ vlans.interfaces }}"
      when: item.mode == "trunk" and "ansible_network_os == 'arista.eos.eos'"
    
    - name: "Print Arista L2 Interface vlans Config"
      debug:
        msg: "{{ arista_output }}"
      when: "ansible_network_os == 'arista.eos.eos'"

VyOS VLAN Module playbook

---

- name: "vlans interface CONFIGS Playbook via MODULES"
  hosts: vyos
  connection: network_cli

  tasks:
    - name: "Configure vyos VLAN and ACCESS ports for vlans"
      vyos.vyos.vyos_vlan:
        vlan_id: 10
        name: TEN
        interfaces: eth3
        state: present
      register: vyos_output
      when: "ansible_network_os == 'vyos.vyos.vyos'"
    
    - name: "Print vyos L2 Interface vlans Config"
      debug:
        msg: "{{ vyos_output }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"

Note with VYOS you can also use the aggregate key word to provide a list of vlans and interfaces
aggregate 
list / elements=dictionary
List of VLANs definitions.

---

- name: "vlans interface CONFIGS Playbook via MODULES"
  hosts: vyos
  connection: network_cli

  tasks:
    - name: "Configure vyos VLAN and ACCESS ports for vlans"
      vyos.vyos.vyos_vlan:
        aggregate:
          - vlan_id: 10
            name: TEN
            interfaces: eth1
            state: present
          - vlan_id: 33
            name: THIRTYTHREE
            interfaces: eth2
            state: present
          - vlan_id: 42
            name: FORTYTWO
            interfaces: eth3
            state: present
      register: vyos_output
      when: "ansible_network_os == 'vyos.vyos.vyos'"
    
    - name: "Print vyos L2 Interface vlans Config"
      debug:
        msg: "{{ vyos_output }}"
      when: "ansible_network_os == 'vyos.vyos.vyos'"


# NAPALM-Ansible

--------------------- Understanding NAPALM-----------------------------------------

In previous sessions we were able to configure OSPF, BGP and VLANS in a multivendor environment using ansible playbooks.

Multivendor environments can be tricky.   Different CLI and connection methods.

Enemy of automation is uniqueness.  The more uniqueness the less repeatable they are and it makes it harder to configure with automation.

This skill will focus on a tool called NAPALM for use within Ansible.

NAPALM (Python Library) = Network Automation and Programmability Abstraction Layer with Multivendor support

https://napalm.readthedocs.io/en/latest/

NAPALM will abstract the configuration commands between multiple vendors.  Using the exact same command!  

Supported "CORE PLATFORM" Network Operating Systems:
Arista EOS
Cisco IOS  
Cisco IOS-XR
Cisco NX-OS
Juniper JunOS 

VYOS is not Supported with NAPALM.

NAPALM will normalize data when the vendor CLI commands are different.  

NAPALM can REPLACE and MERGE configuration.

NAPALM allows ROLLBACK.

NAPALM uses a GETTER to pull data from a device, translate, and return normalized data.

Example:  the get-vlan, get-interfaces will pull vlan and interface information from Cisco, Arista, Juniper and return the same data.

We want things as cookie cutter as possible and this is what NAPALM can do for us.


--------------------- Installing Arista Images on EVE-NG ----------------------------------------

Login into the Arista Website
https://www.arista.com/en/support/software-download

You will need an account to login.

Download the following

Aboot
EOS Boot-loader files
https://www.arista.com/custom_data/aws3-explorer/download-s3-file.php?f=/support/download/vEOS-lab/Aboot/Aboot-veos-serial-8.0.1.iso

vEOS
Virtual EOS vmdk files
https://www.arista.com/custom_data/aws3-explorer/download-s3-file.php?f=/support/download/vEOS-lab/4.27/vEOS-lab-4.27.5M.vmdk

Now that you have the two images you can add to your EVE-NG.
Connect to your EVE-NG machine via ssh and copy files and follow these instructions.
https://www.eve-ng.net/index.php/documentation/howtos/howto-add-arista-veos/

Add an Arista node to your EVE-NG toplogy.

Log into the Arista and run this command to disable zero touch provisioning (ZTP).

zerotouch cancel

copy run start

--------------------- Installing NAPALM ANSIBLE ----------------------------------------

To make things less clutered lets create a new directory.

Create a new directory called Network-Automation and copy important files to it.

root@eveng-2:~/ex457_rh_ansible# mkdir Network-Automation
root@eveng-2:~/ex457_rh_ansible# cd Network-Automation
root@eveng-2:~/ex457_rh_ansible/Network-Automation# cd ..
root@eveng-2:~/ex457_rh_ansible# cp -r hosts ansible.cfg host_vars/ group_vars/ n
notes.md        ntp-config.yml  nuggets/        
root@eveng-2:~/ex457_rh_ansible# cp -r hosts ansible.cfg host_vars/ group_vars/ Network-Automation/
root@eveng-2:~/ex457_rh_ansible# cd Network-Automation
root@eveng-2:~/ex457_rh_ansible/Network-Automation# ls
ansible.cfg  group_vars  host_vars  hosts

On your linux machine:

ansible-galaxy collection install arista.eos

pip install napalm

or

Create a virtual pip env

python3 -m venv .venv

cd .venv/

source bin/activate

pip3 install napalm
or 
pip3 install --ignore-installed PyYAML

Here is the important part we need to git clone napalm-ansible

Instructions
https://napalm.readthedocs.io/en/latest/tutorials/ansible-napalm.html

cd to your workspace directory
git clone https://github.com/napalm-automation/napalm-ansible.git

Now you want to point ansible to this napalm-ansible installation

Edit your ansible.cfg file and add this with your relative path
library =$HOME/ex457_rh_ansible/Network-Automation/napalm-ansible

In order to interact with our ARISTA devices NAPALM is not going to try ssh instead it's going to eAPI on Arista so we need to enable it.

On the Arista devices run the following commands

conf t

username api privilege 15 secret arista

management api http-commands

no shutdown

On the Cisco devices run the following commands

conf t

username api privilege 15 secret cisco

ip domain-name cisco.com

ip ssh v 2

crypto key gen rsa mod 1024

line vty 0 15

transport input ssh 

EDIT YOUR GROUP VARS AND ADD THIS VARIABLE IN YOUR arista and cisco group vars

napalm_platform: "eos"

napalm_platform: "ios"


--------------------- Retrieving State informion from devices with NAPALM Getters ----------------------------------------

Let's create a playbook to get facts from devices

NAPALM MODULES CAN BE FOUND ON THEIR WEBSITE
https://napalm.readthedocs.io/en/latest/tutorials/ansible-napalm.html#modules

napalm_get_facts

napalm_install_config

napalm_validate

---

- name: "Playbook to test NAPALM ANSIBLE"
  hosts: cisco, arista
  connection: network_cli

  tasks:
    - name: "Retrieve devices facts via NAPALM"
      napalm_get_facts:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
      
      register: result

    - name: "Print Result"
      debug:
        msg: "{{ result }}"

 # Note for enabling Arista eAPI

#####################################################################################
 To ename eAPI for NAPALM on Arista you must no shutdown management api.
 
 conf t
 !
management api http-commands
   no shutdown

Also on the Ansible Control node - you must modify your /etc/hosts file to include the DNS Name of your Arista devices.
Otherwise you will get this Ansible ERROR MESSAGE.

TASK [Retrieve devices facts via NAPALM] 
fatal: [R1]: FAILED! => {
    "changed": false
}
MSG:
cannot connect to device: Socket error during eAPI connection: [Errno -3] Temporary failure in name resolution

vi /etc/hosts
R1 10.199.199.11

#####################################################################################

Napalm Getters

https://napalm.readthedocs.io/en/latest/support/


get_interfaces_ip
get_interfaces_counters
get_mac_address_table

EXAMPLE 2 PLAYBOOK WITH A FILTER
---

- name: "Playbook to test NAPALM ANSIBLE"
  hosts: cisco, arista
  connection: network_cli

  tasks:
    - name: "Retrieve devices facts via NAPALM"
      napalm_get_facts:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        # filter: ["interfaces_ip"]
        # filter: ["interfaces_counters"]
        # filter: ["mac_address_table"]
        # You can chain your napalm fiters using a comma
        filter: ["interfaces_counters", "mac_address_table"]
      
      register: result

    - name: "Print Result"
      debug:
        # msg: "{{ result.ansible_facts.napalm_mac_address_table[0].mac }}"
        # msg: "{{ result.ansible_facts.napalm_mac_address_table }}"
        msg: "{{ result.ansible_facts.napalm_interfaces_counters.Management1.tx_errors }}"
      when: "ansible_network_os == 'arista.eos.eos'"

https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#formatting-data-yaml-and-json

##########################################################################################################

# Validating Configurations and State using napalm_validate

We are going to be using NAPALM validate to validate our configs.

Introduce aliases into inventory file.
S1 ansible_host=10.199.199.11
S2 ansible_host=10.199.199.12

Create a "napalm_get_facts_playbook.yml" playbook that filters on facts

---

- name: "Playbook to test NAPALM ANSIBLE"
  hosts: cisco, arista
  connection: network_cli

  tasks:
    - name: "Retrieve devices facts via NAPALM"
      napalm_get_facts:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        filter: ["facts"]
      
      register: result

    - name: "Print Result"
      debug:
        msg: "{{ result.ansible_facts.napalm_facts | to_yaml(indent=8, width=1337) }}"
        # msg: "{{ result.ansible_facts.napalm_facts | to_nice_json  }}"

Grab the results of the napalm facts and convert it from json to yaml.  Or use the | to_yaml filter with debug msg.

Go to https://www.json2yaml.com/

Create {{ inventory_hostname }}-facts.yml files and paste the yaml output results from napalm_get_facts playbook above.

Remove any key:value pair items in the list you don't need to validate. 

The valutes in this {{ inventory_hostname }}-facts.yml will be used with the napalm_example3_playbook.yml to validate configuration compliancy.

Modify the {{ inventory_hostname }}-facts.yml to reference the entire name of get_facts NAPALM getter
https://napalm.readthedocs.io/en/latest/support/index.html#getters-support-matrix

Example:  R1-facts.yml
---
- get_facts:
    hostname: R1
    model: vEOS
    serial_number: ''
    vendor: Arista

######################################################################################################################################################
Example Playbook "napalm_example3_playbook.yml" that validates compliance of facts based on data in the {{ inventory_hostname}}-facts.yml File.

---

- name: "NAPALM PLAYBOOK TO VALIDATE DEVICES"
  hosts: arista, cisco
  connection: network_cli

  tasks:
    - name: " Task1 - use NAPALM VALIDATE "
      napalm_validate:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        validation_file: "{{ inventory_hostname}}-facts.yml"
      register: result
      ignore_errors: yes

    - name: "Task1 - print the result"
      debug:
        msg: {{ result }}

Run the ansible-playbook napalm_example3_playbook.yml 

If configuration data is compliant the task for that device will show complies: true.

{
    "changed": false,
    "compliance_report": {
        "complies": true,
        "get_facts": {
            "complies": true,
            "extra": [],
            "missing": [],
            "present": {
                "hostname": {
                    "complies": true,
                    "nested": false
                },
                "model": {
                    "complies": true,
                    "nested": false
                },
                "serial_number": {
                    "complies": true,
                    "nested": false
                },
                "vendor": {
                    "complies": true,
                    "nested": false
                }

If configuration data is NOT compliant the task for that device will show complies: false along with actual vs expected value
                "hostname": {
                    "actual_value": "R1",
                    "complies": false,
                    "expected_value": "R20",
                    "nested": false

# Validatiion Troubleshooting using napalm_validate

There are some caviates that we have to do.


Create a "napalm_example_getvlans_playbook.yml" playbook that filters on facts

---

- name: "Playbook to test NAPALM ANSIBLE"
  hosts: cisco, arista
  connection: network_cli

  tasks:
    - name: "Retrieve devices facts via NAPALM"
      napalm_get_facts:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        # filter: ["interfaces_ip"]
        # filter: ["interfaces_counters"]
        # filter: ["mac_address_table"]
        # You can chain your napalm fiters using a comma
        filter: ["vlans"]
      
      register: result

    - name: "Print Result"
      debug:
        # msg: "{{ result.ansible_facts.napalm_mac_address_table[0].mac }}"
        # msg: "{{ result.ansible_facts.napalm_mac_address_table }}"
        msg: "{{ result.ansible_facts.napalm_vlans | to_nice_json }}"
      # when: "ansible_network_os == 'arista.eos.eos'"


Grab the results of the napalm facts and convert it from json to yaml.  Or use the | to_yaml filter with debug msg.

Go to https://www.json2yaml.com/

Create {{ inventory_hostname }}-facts.yml files and paste the yaml output results from napalm_get_facts playbook above.

Remove any key:value pair items in the list you don't need to validate. 

The valutes in this {{ inventory_hostname }}-facts.yml will be used with the napalm_example3_playbook.yml to validate configuration compliancy.

Modify the {{ inventory_hostname }}-facts.yml to reference the entire name of get_facts NAPALM getter

######################################################################################################################################################
Example Playbook "napalm_example3c_vlans_playbook.yml" that validates compliance of facts based on data in the {{ inventory_hostname}}-facts.yml File.
---

- name: "NAPALM PLAYBOOK TO VALIDATE DEVICES"
  hosts: arista, cisco
  connection: network_cli

  tasks:
    - name: " Task1 - use NAPALM VALIDATE "
      napalm_validate:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        validation_file: "{{ inventory_hostname}}-vlan.yml"
      register: result
      ignore_errors: yes

    - name: "Task1 - print the result"
      debug:
        msg: "{{ result | to_nice_json }}"

Run the ansible-playbook napalm_example3c_vlans_playbook.yml

If configuration data is compliant the task for that device will show complies: true.
  
################ GOCHAS There are some caviates that we have to do #######################################

https://napalm.readthedocs.io/en/latest/validate/index.html

Lists of objects to be validated require an extra key list. You can see an example with the get_facts getter. Lists can be strict as well. In this case, we want to make sure the device has only those two interfaces.

What this means is we have to modify our validation {{ inventory_hostname }}-facts.yml file with a new "list" object
Edit the R1-vlan.yml
---
- get_vlans:
    '1':
      interfaces:
        list:
          - Ethernet2
          - Ethernet3
          - Ethernet6
          - Ethernet7
      name: default
    '5':
      interfaces:
        list:
          - Ethernet2
          - Cpu
          - Ethernet4
      name: FIVE
    '10':
      interfaces:
        list:
          - Ethernet2
          - Cpu
          - Ethernet5
      name: TEN

If configuration data is NOT compliant the task for that device will show complies: fail.
Device does not comply with policy
...ignoring
ok: [R2]

TASK [Task1 - print the result] **************************************************************************************************************************************************************************************************
ok: [R1] => {}
MSG:
{
    "changed": false,
    "compliance_report": {
        "complies": false,
        "get_vlans": {
            "complies": false,
            "extra": [],
            "missing": [],
            "present": {
                "5": {
                    "complies": false,
                    "diff": {
                        "complies": false,
                        "extra": [],
                        "missing": [],
                        "present": {
                            "interfaces": {
                                "complies": false,
                                "diff": {
                                    "complies": false,
                                    "extra": [],
                                    "missing": [
                                        "Ethernet3"
    "failed": true,
    "msg": "Device does not comply with policy"


  # Backup and Restore Configurations

--------------------- INTRODUCTION -----------------------------
  Mundane or cookie cutter tasks are repetative.

  One of the most common things to do is to backup your configuration typically done by logging into a device and running "show run" command.

  We are going to learn about backing up configuration using Ansible Automation.

  Which can be done on a scheduled basis like at 7am every morning.

  If something goes wrong during a change how do you restore the configuration?  

  We are going to backup and restore configuration using ansible modules.

  It's also important to have a log, date and time of the device.
-----------------------------------------------------------------

--------------------- USING THE SETUP MODULE -----------------------------

ANSIBLE SETUP MODULE
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html

This module is automatically called by playbooks to gather useful variables about remote hosts that can be used in playbooks. It can also be executed directly by /usr/bin/ansible to check what variables are available to a host. Ansible provides many facts about the system, automatically.

The first thing you want to do is to get the date and time from the local ansible host and register it as an output variable.

---
- name: "TEST SETUP MODULE PLAYBOOK"
  hosts: localhost

  tasks:
    - name: "Collect facts aobut local host"
      ansible.builtin.setup:
        gather_subset:
          - "min"
      register: output

    - name: "Print facts from local host min"
      debug:
        # msg: "{{ output.ansible_facts.ansible_date_time.iso8601 | to_nice_json }}"
        msg: "{{ output.ansible_facts.ansible_date_time.date | to_nice_json }}"

You can also use the filter ansible_date_time to get the local date and time information and then set a fact to save this varialbe for use with another task.

---
- name: "TEST SETUP MODULE PLAYBOOK"
  hosts: localhost

  tasks:
    - name: "Collect facts aobut local host"
      ansible.builtin.setup:
        filter:
          - "ansible_date_time"
      register: output

    - name: "Task2: Recording Variable as a fact"
      set_fact:
        datetime: "{{ ansible_date_time }}"

    - name: "Print facts from local host min"
      debug:
        # msg: "{{ datetime.date | to_nice_json }}"
        msg: "{{ datetime.iso8601 | to_nice_json }}"
        
------------------------------------------------------------------------------


--------------------- CREATING DIRECTORIES WITH THE FILE MODULE -----------------------------

Ansible to be able to create a new directory based on today's date.

Within that new directory store backups.

Ansible File module.
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html

path will create directory.

state (default file) if you change to directory all intermediate subdirectories will be created.

run_once is a Boolean true or false force the task to attempt to execute on the first host available and afterwards apply any results and facts to all active hosts in the same batch.

Example Playbook:

---
- name: "TEST SETUP MODULE PLAYBOOK"
  hosts: localhost

  tasks:
    - name: "Collect facts aobut local host"
      ansible.builtin.setup:
        filter:
          - "ansible_date_time"
      register: output

    - name: "Task2: Recording Variable as a fact"
      set_fact:
        datetime: "{{ ansible_date_time.date }}"

    - name: "Task3 - Creating directories with file module"
      file:
        path:  "Backups/{{ datetime }}"
        state: directory
      run_once: true

--------------------- Startup and Running Configurations -----------------------------

Storing running configurations using ansible and NAPALM.

NAPALM get_config getters will get running and startup configs for multiple vendors.
https://napalm.readthedocs.io/en/latest/support/

Example Playbook with multiple plays:

---
- name: "Play 1: TEST SETUP MODULE PLAYBOOK"
  hosts: localhost

  tasks:
    - name: "Task1 of Play 1: Collect facts aobut local host"
      ansible.builtin.setup:
        filter:
          - "ansible_date_time"
      register: output

    - name: "Task2 of Play 1: Recording Variable as a fact"
      set_fact:
        datetime: "{{ ansible_date_time.date }}"

    - name: "Task3 of Play 1: Creating directories with file module"
      file:
        path:  "Backups/{{ datetime }}"
        state: directory
      run_once: true

- name: "Play 2: Backing up Configurations"
  hosts: arista
  connection: network_cli

  tasks:
    - name: "Task 1 of Play2: Pull Configurations from Remote Devices"
      napalm_get_facts:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        filter: ["config"]
      register: result
    
    - name: "Task 2 of Play2: Create Startup Config Subdirectory"
      file:
        path: "Backups/{{ hostvars.localhost.datetime}}/startup-configs"
        state: directory
      run_once: true

    - name: "Task 3 of Play2: Create Running Config Subdirectory"
      file:
        path: "Backups/{{ hostvars.localhost.datetime}}/running-configs"
        state: directory
      run_once: true

    - name: "Print result"
      debug:
        msg: "{{ result | to_nice_json }}"
    

--------------------- The COPY Module -----------------------------

Writing files into directories using the ansible COPY module

Copy module can be used to copy to local or remote machines
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html

content is a string value parameter to define content to copy

dest is the destination path using absolut or relative path

Example playbook using the COPY module

---
- name: "Play 1: TEST SETUP MODULE PLAYBOOK"
  hosts: localhost

  tasks:
    - name: "Task1 of Play 1: Collect facts aobut local host"
      ansible.builtin.setup:
        filter:
          - "ansible_date_time"
      register: output

    - name: "Task2 of Play 1: Recording Variable as a fact"
      set_fact:
        datetime: "{{ ansible_date_time.date }}"

    - name: "Task3 of Play 1: Creating directories with file module"
      file:
        path:  "Backups/{{ datetime }}"
        state: directory
      run_once: true

- name: "Play 2: Backing up Configurations"
  hosts: arista
  connection: network_cli

  tasks:
    - name: "Task 1 of Play2: Pull Configurations from Remote Devices"
      napalm_get_facts:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        filter: ["config"]
      register: result
    
    - name: "Task 2 of Play2: Create Startup Config Subdirectory"
      file:
        path: "Backups/{{ hostvars.localhost.datetime}}/startup-configs"
        state: directory
      run_once: true

    - name: "Task 3 of Play2: Create Running Config Subdirectory"
      file:
        path: "Backups/{{ hostvars.localhost.datetime}}/running-configs"
        state: directory
      run_once: true

    - name: "Task 4 of Play2: Copy Startup configs to disk"
      copy:
        content: "{{ result.ansible_facts.napalm_config.startup }}"
        dest: "Backups/{{ hostvars.localhost.datetime}}/startup-configs/{{ inventory_hostname }}-startup-config.txt"

    - name: "Task 5 of Play2: Copy Startup configs to disk"
      copy:
        content: "{{ result.ansible_facts.napalm_config.running }}"
        dest: "Backups/{{ hostvars.localhost.datetime}}/running-configs/{{ inventory_hostname }}-running-config.txt"

    - name: "Print result"
      debug:
        msg: "{{ result | to_nice_json }}"

sudo timedatectl set-time "2022-08-29" to change date of local machine then run the same playbook again to create a new Backup subdirectory

ansible-playbook ansible_date_time_filter_backup.yml 


--------------------- RESTORING CONFIGS -----------------------------

Restoring our network devices from Backup configurations.

Using the NAPALM install_config module to load a configuration file on the network device.

https://napalm.readthedocs.io/en/latest/tutorials/ansible-napalm.html

Note for Cisco devices to use NAPALM you must configure the following on Cisco

conf t

ip scp server enable

archive

path flash:archive

write-memory

NAPALM uses netmiko python and causes timeout error

Add this to the playbook to increase the factor of time by 2 or double time

       optional_args:
          global_delay_factor: 2

Also note with NAPALM requires a ETX character for Banners

You can create a converter.py python script to replace the Banner special character

touch converter.py

import sys

target = sys.argv[1]
etx = chr(3)

def get_config():
    with open(target, "r") as f:
        my_data = f.read()
        final_config = my_data.replace("^C", etx)
    return final_config

def update_config(result):
    with open(target, "w") as q:
        q.write(result)

result = get_config()
update_config(result)

python3 converter.py R1-Running-config.txt       will replace special character with etx

Also note with NAPALM the Cisco IOS Configuration file must start with a version to restore


Example Playbook to restore configs:

---
- name: "Play 1 get date and time from localhost"
  hosts: localhost

  tasks:
    - name: "Task1 of Play 1: Collect date on local host"
      ansible.builtin.setup:
        filter:
          - "ansible_date_time"
      register: output

    - name: "Task2 of Play 1: Recording Variable as a fact"
      set_fact:
        datetime: "{{ ansible_date_time.date }}"

- name: "Play 2 restore configs"
  hosts: R1
  connection: network_cli

  tasks:
    - name: "Task1 of Play 2: Create a Diff directory"
      file:
        path: "Backups/diffs/{{ hostvars.localhost.datetime }}/{{ inventory_hostname }}"
        state: directory
      run_once: false
    
    - name: "Task2 of Play2: Use NAPALM to Restore Backup Configs"
      napalm_install_config:
        hostname: "{{ inventory_hostname }}"
        username: "{{ ansible_user }}"
        password: "{{ ansible_password }}"
        dev_os: "{{ napalm_platform }}"
        config_file: Backups/2022-08-28/startup-configs/{{ inventory_hostname }}-startup-config.txt
        commit_changes: true
        replace_config: true
        get_diffs: true
        diff_file: Backups/diffs/{{ hostvars.localhost.datetime }}/{{ inventory_hostname }}/{{ inventory_hostname }}-diffs.txt
        optional_args:
          global_delay_factor: 2
      register: result

    - name: "Print Result"
      debug:
        msg: "{{ result | to_nice_json }}"


# CONFIGURE SYSLOG AND SNMP

--------------------- Syslog Overview ----------------------------------------------------------

Logging levels 0 - 7

*0 Emergencies - System is unusable

*1 Alerts - Immediate action needed

*2 Critical - Critical conditions

*3 Errors - Error conditions

*4 Warnings - Warning conditions

*5 Notifications - Informational messages

*6 Informational - Normal but significant conditions

*7 Debugging - Debugging messages

Logging levels are inclusive meaning:

If you select logging at level 3 you get level 3 and above logging levels.   Don't worry about anything below it.

If you select logging at leve 7 it will capture everything which can overwelm your devices.

Console Logging 

The actual device itselfs logs.
        
Buffered Logging

Allow device to save all the logs in a buffer cache

Terminal Logging

Logging sent over your VTY connection ssh or telnet

Server Logging

Is the best option which offloads logging and sends it to an actual Server!  

----------------------------------------------------------------------------------------------

--------------------- Cisco Syslog Configuration using ANSIBLE----------------------------------------------------------

Cisco Ansible IOS Global logging module for Ansible
https://docs.ansible.com/ansible/latest/collections/cisco/ios/ios_logging_global_module.html#ansible-collections-cisco-ios-ios-logging-global-module

Example Syslog playbook with no variables
---
- name: "Play 1 - target Cisco Devices"
  hosts: cisco
  connection: network_cli

  tasks:
    - name: Apply the provided configuration
      cisco.ios.ios_logging_global:
          config:
            buffered:
              severity: notifications
              size: 5099
              xml: True
            console:
              severity: critical
              xml: True
            facility: local5
            hosts:
              - hostname: 172.16.1.12
              - hostname: 172.16.1.11
                xml: True
              - hostname: 172.16.1.10
                filtered: True
                stream: 10
              - hostname: 172.16.1.13
                transport:
                  tcp:
                    port: 514
            monitor:
              severity: warnings
            message_counter: log
            snmp_trap:
              - errors
            trap: errors
            userinfo: True
            logging_on: enable
            exception: 4099
            dmvpn:
              rate_limit: 10
            cns_events: warnings
          state: merged
      register: cisco_output

    - name: "Cisco Output"
      debug:
        msg: "{{ cisco_output | to_nice_json  }}"

Example Syslog playbook with variables

First add this to the host_vars
---
syslog: 
  hosts:
    - ip: 1.1.1.1
    - ip: 2.2.2.2
      port: 1234
    - ip: 3.3.3.3
    - ip: 4.4.4.4
      port: 5678

---
- name: "Play 1 - target Cisco Devices"
  hosts: cisco
  connection: network_cli

  tasks:
    - name: Task1 Apply the Syslog configuration with no port
      cisco.ios.ios_logging_global:
          config:
            hosts:
              - hostname: "{{ item.ip }}"
          state: merged
      loop: "{{ syslog.hosts }}"
      when: item.port is not defined
      register: task1_output

    - name: "Task1 Output"
      debug:
        msg: "{{ item.commands | to_nice_json  }}"
      loop: "{{ task1_output.results }}"
      loop_control:
        label: none
      when: item.commands is defined

    - name: Task2 Apply the Syslog configuration with port
      cisco.ios.ios_logging_global:
          config:
            hosts:
              - hostname: "{{ item.ip }}"
                transport:
                  tcp:
                    port: "{{ item.port }}"
          state: merged
      loop: "{{ syslog.hosts }}"
      when: item.port is defined
      register: task2_output
    
    - name: "Task2 Output"
      debug:
        msg: "{{ item.commands | to_nice_json  }}"
      loop: "{{ task2_output.results }}"
      loop_control:
        label: none
      when: item.commands is defined

    - name: Display inventory hostname
      ansible.builtin.debug:
        msg:
          - "-------------------"
          - "{{ inventory_hostname }}"
        
-----------------------------------------------------------------------------------------        
------------------------- VYOS SYSLOG CONFIGURATION -------------------------------------

VYOS LOGGING ANSIBLE MODULE
https://docs.ansible.com/ansible/latest/collections/vyos/vyos/vyos_logging_global_module.html#ansible-collections-vyos-vyos-vyos-logging-global-module

VYOS CONFIGURATION GUIDE - 

SYSLOG FACILITIES
https://docs.vyos.io/en/equuleus/configuration/system/syslog.html#facilities

SEVERITY ALERTS
https://docs.vyos.io/en/equuleus/configuration/system/syslog.html#severity-level

VYOS SYSLOG EXAMPLE PLAYBOOK

- name: "Play 2 - Target VYOS"
  hosts: vyos
  connection: network_cli
   
  tasks:
    - name: Apply the provided configuration
      vyos.vyos.vyos_logging_global:
        config:
          console:
            facilities:
              - facility: local7
                severity: err
          files:
            - path: logFile
              archive:
                file_num: 2
              facilities:
                - facility: local6
                  severity: emerg
          hosts:
            - hostname: 172.16.0.1
              facilities:
                - facility: local7
                  severity: all
                - facility: all
                  protocol: udp
              port: 223
          users:
            - username: vyos
              facilities:
              - facility: local7
                severity: debug
          # global_params:
          #   archive:
          #     file_num: 2
          #     size: 111
          #   facilities:
          #   - facility: cron
          #     severity: debug
          #   marker_interval: 111
          #   preserve_fqdn: true
        state: merged
      register: vyos_output

    - name: "Print VYOS OUTPUT"
      debug:
        msg: "{{ vyos_output | to_nice_json }}"

-------------------------------------------------------------------------------

------------------- SNMP OVERVIEW ---------------------------------------------

SNMP SIMPLE NETWORK MANAGEMENT PROTOCOL

APPLICATION LAYER PROTOCOL THAT ALLOWS MANAGERS TO TALK TO AGENTS AND GATHER INFORMATION

AGENTS IS THE DEVICE BEING MONITORED

RETRIEVE STATE DATA

MAKE CONFIGURATION CHANGES

NETCONF NEW PROTOCOL THAT REPLACES SNMP TO FILL IN A WHOLE THAT SNMP LEFT

SNMP WORKS OFF A DATABASE CALLED MIB MANAGEMENT INFORMATION BASE

EXAMPLE CRC errors has a specific MIB 3.1.2.4.6 NUMBER EXAMPLE

NMS NETWORK MANAGEMENT SYSTEM CAN "GET" OR "SET" information.

SNMP TRAP MESSAGE

SNMP INFORM MESSAGE

POLL DATA - NMS CAN CONTINUALLY POLL AGENTS "POLL MODEL"

PUSH DATA - TRAP INFORM WILL BE SENT BY THE DEVICE TO INFORM THE NMS

TRAP IS ONE WAY

INFORM IS BIDIRECTIONAL 

NMS MUST ACK,  AGENT WILL WAIT AND LOOK AT IT'S CLOCK AND RESEND TO NMS LIKE BY THE WAY....

------------------------------------------------------------------------------------------------------------

-------------------------  SNMP ANSIBLE MODULES ------------------------------------------------------------














   






