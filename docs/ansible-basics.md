## Inventory
for target system (list of servers) in yaml or ini

* file
* /etc/ansible/hosts

```ini
sever1.company.com
sever1.company.com

[group_1] # Group
sever3.company.com
sever4.company.com

[group_2]
sever5.company.com
sever6.company.com

[main:children]  # Group of Groups with ':children'
group_1
group_2

[main:vars] # apply variables to these groups of groups with ':vars'
halon_system_timeout=30
```

#### Inventory Parameters (these are variables):
* ansible_host (dns, ip)
* ansible_connection (ssh, winrm, localhost) (linux, windows, localhost)
* ansible_port (22, 5986)
* ansible_user (root, administrator) (linux, windows)
* ansible_ssh_pass ( password )

set **Alias** and **Parameters** :

```ini
web  ansible_host=sever1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=1234
db  ansible_host=sever2.company.com ansible_connection=ssh ansible_user=root
mail  ansible_host=sever3.company.com ansible_connection=winrm ansible_user=administrator
localhost  ansible_connection=localhost
```
----------------------------------
## Playbook
```yaml
- 
  name: play
  hosts: all, *, localhost, Group1, Host1  # ==> read from inventory
  tasks:
    - name: run command date
      command: date  # ==> module
    - name: run script
      script: test.sh  # ==> module
```

list of ansible modules :
```bash
ansible-doc -l
```
run ansible playbook :
```bash
ansible-playbook playbook.yml
```
**dynamic inventory**
```bash
ansible-playbook -i inventory.txt playbook.yml
                    inventory.py
```
----------------------------------
## Modules
* System (User, Group, Hostname, Iptables, Service, Systemd, ...)
* Commands (Command, Script, Raw, Expect, ...)
* Files (Archive, File, Copy, Template, ...)
* Database (Mongodb, Mysql, Postgressql)
* Cloud (Amazon, Google, Asure)
* windows
* ...
----------------------------------
## Variable

variable in main plybook file :
```yaml
- 
  name: play
  hosts: localhost
  vars:
    dns_server: 4.2.2.4
  tasks:
```
define var in inventory file for that specific host:
```ini
web dns_server=4.2.2.4 key=value
```
using define variable :
```yaml
  name: play
  hosts: localhost
  vars:
    dns_server: 4.2.2.4
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver {{ dns_server }}' # use var with '{{}}'
```
variable in separate file "var.yml" :
```yaml
var1: value1
var2: value2
```
using var file in playbook in specific dir :
```yaml
- 
  hosts: all
  vars_files:
    - /vars/var.yml
  tasks:
```
----------------------------------
## Conditionals

for using conditionals , use 'when' \
> expressions in 'when' is  **==**, **!=**, **or** , **and** , **is** , **is not** , **>** , **<**

```yaml
- 
  name: play
  hosts: localhost
  tasks:
    - service: name=mysql state=started
      when: ansible_host == sever1.company.com 
      # just run in sever1.company.com server
    
    - service: name=httpd state=started
      when: ansible_host == sever1.company.com or ansible_host == sever2.company.com 
```

store command output in variable use 'register' module :
```yaml
- 
  name: play
  hosts: localhost
  tasks:
    - command: service httpd status
      register: command_output

    - debug: # show during terminal when start playbook
        msg: '{{ command_output }}'
```
----------------------------------
## Loops
> loop is new version of with_* after ansible 2.5
```yaml
- 
  name: start service
  hosts: localhost
  tasks:
    - service: name='{{ item }}' state=started
      with_items: # or it can be 'loop:'
        - mongo
        - mysql
        - httpd
```
----------------------------------
## Roles

for include playbook in our main yml file use '- include < playbook name >' :
```yaml
- include install_dependencies.yml
- include configure_web_server.yml
- include setup_start_app.yml
```

it's sample of each playbook
* install_dependencies.yml
* configure_web_server.yml
* setup_start_app.yml
```yaml
- 
  name: role name
  hosts: localhost
  roles:
```

this is dir structure for using roles:
```
ansible-project/
    inventory.txt
    app.yml
    roles/
        mysql/
            tasks/
            handlers/
            library/
            files/
            templates/
            vars/
            defaults/
            meta/
        nodejs/
            tasks/
            defaults/
            meta/
```

for using role in our main file :
```yaml
---
- hosts: webservers
  roles:
    - mysql
    - nodejs
```

for use tasks and vars in separate file:
```yaml
- 
  name: play
  hosts: localhost
  vars_files:
    - var.yml
  tasks:
    - include: task.yml
```
