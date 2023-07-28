### ssh config :
```
/etc/ansible/ansible.cfg ==> ansible config file
#host_key_checking=false ==> uncomment it for not check key
```
----------------------------------
## host_vars , group_vars

dir structure for host_vars :
```
playbook.yml
host_vars/
    server1.yml
    server2.yml
```

server1.yml :
```
ansible_host=10.12.90.232
ansible_ssh_pass=qazwsx
```

dir structure for group_vars (same var for all server in this group) :
```
playbook.yml
group_vars/
    db.yml
```
----------------------------------
## Include Tasks

dir structure for separate tasks :

```yaml
- 
  name: separate tasks
  hosts: localhost
  tasks:
  - include: tasks/server.yml
  - include: tasks/db.yml
```

tasks/db.yml :
```yaml
- name: install dep for db
- name: config db
```

tasks/server.yml :

```yaml
- name: install dep for app
- name: config app
```
----------------------------------
## Ansible Galaxy

create new role :
```bash
ansible-galaxy init new-role-name
```

search in public roles :
```bash
ansible-galaxy search role-name
```
----------------------------------

## Asynchronous

**when we want not be ssh connection alive for long time during proccess**

for example this script takes at least 5 min long use **aysnc** to stay 360 sec for this script
```yaml
- name: 5 min script for web
  command: /opt/web.sh
  async: 360
```
by default ansible check status script every 10 sec but we can modify it with **poll**, in this example ansible check status every 15 sec
```yaml
- name: 5 min script for web
  command: /opt/web.sh
  async: 360
  poll: 15
```

if we have more than one moudle takes 5 min time, ansible wait 5 min for each moudle until down, beacuse it run parallel during runtime, set poll to 0
and then use async_status for check moudles work fine and retries until down

```yaml
- name: 1 min script for web
  command: /opt/web.sh
  async: 60
  poll: 0
  register: script_web

- name: 1 min script for db
  command: /opt/db.sh
  async: 60
  poll: 0
  register: script_db

- name: check status of task web
  async_status: 
    jid: '{{ script_web.ansible_job_id }}'
  register: job_result
  until: job_result.finished
  retries: 30 # ==> it's for 30 times retry until down
```
----------------------------------
## Strategy

### Linear Strategy (default ansible strategy)

it run all these servers at same time but wait to complete each task in all servers and then go to next task
```
server1 ===> task1 ===> task2    task3    end
server2 ===> task1 ===> task2    task3    end
server3 ===> task1 ===> task2    task3    end
```
``` yaml
- name: linear strategy
  hosts: server1, server2, server3
```

### Free Strategy

each server doing taks independent and not wait for another servers
```
server1 ===> task1 ===> task2 ===> task3      end
server2 ===> task1 ===> task2      task3      end
server3 ===> task1 ===> task2 ===> task3 ===> end
```
```yaml
- name: free strategy
  strategy: free
  hosts: server1, server2, server3
```

### Batch Strategy (it's base on linear strategy)

it's like linear strategy but we can specify number of servers run in same time
```yaml
- name: batch strategy
  serial: 3
  hosts: all_servers
```

ansible just talk to 5 server parallel at time if want to change number of server, we should do it in **ansible.cfg**
```
forks = 5
```
----------------------------------
## Error Handling

by default if one of server get error during runtime , just that server stoped but in this case ,even one server get error during runtime all server stoped with set this parameter **any_errors_fatal:** to **true** :
```yaml
- name: error handling
  hosts: all_servers
  any_errors_fatal: true
  tasks:
```

in this task, it's not very important , if fail during runtime , ignore error and continue with set **ignore_errors:** parameter to **yes** :
```yaml
- name: just echo 'hello wolrd'
  command: /opt/hello.sh
  ignore_errors: yes
```

if find **ERROR** word in file, value of file is **command_output.stdout** , this task gonna fail with set **failed_when:** parameter :
```yaml
- name: important script
  command: /var/log/db.log
  register: command_output
  failed_when: "'ERROR' in command_output.stdout"
```
----------------------------------
## Jinja2 Template Engine

### Filters :

#### Strings
```
my name is {{ my_name }} ==> subtitle
my name is {{ my_name | upper }} ==> for upper case of value
my name is {{ my_name | lower }} ==> for lower case of value
my name is {{ my_name | title }} ==> for title
my name is {{ my_name | replace("sepehr", "sina") }} ==> for replca "sepehr" word with "sina"
my name is {{ my_name | default("sepehr") }} {{ family_name }} ==> for default value
```

#### List and Set
```
{{ [1,2,3] | min }}  ==> 1
{{ [1,2,3] | max }}  ==> 3
{{ [1,2,3,3] | unique }} ==> 1,2,3
{{ [1,2,3] | union([4,5]) }} ==> 1,2,3,4,5
{{ [1,2,3,4] | intersect([4,5]) }} ==> 4
{{ 100 | random }} ==> random number from 0 to 100
{{ ["sepehr", "is", "too", "smart" ] | join("") }} ==> sepehr is too smart
```

### Condition
```
{% if x > y %}
  x is bigger
{% elif z == 0 %}
  negetive var value
{% else %}
  y is smaller
{% endif %}
```

### Loop
```yaml
var:
  z: [1,2,3,4,5]
```

```
{% for each in z %}
  {{ each }}
{% endfor %}
```

----------------------------------
## Lookups

credential.csv file is :
```
target1,password
target2,password
```
for get password from credential.csv file use **lookup** :
```
{{ lookup('csvfile' , 'target1 file=/tmp/credential.csv delimiter=,') }} ==> password
```

> it can use with **ini** , **dns** , **mongodb**, ... files
----------------------------------
## Ansible Vault

```
ansible-vault encrypt inventory.txt ==> encrypt inventory file
ansible-vault decrypt inventory.txt ==> decrypt inventory file
ansible-vault rekey inventory.txt ==> change password
ansible-vault view inventory.txt ==> view value of encrypted file
ansible-vault create inventory.txt ==> create encrypt file
ansible-vault edit inventory.txt ==> edit encrypted file
```

for run playbook with encrypted files :
```
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
ansible-playbook -i inventory.ini playbook.yml --vault-password-file ./pass.txt ==> read pass from text file
ansible-playbook -i inventory.ini playbook.yml --vault-password-file ./pass.py ==> read pass from py script 
```