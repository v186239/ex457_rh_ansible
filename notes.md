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















