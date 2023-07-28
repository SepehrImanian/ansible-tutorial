## important ansible var
```
ansible_os_family ==> this var for specify os type (RedHat,Ubuntu,Debian)
ansible_system ==> (windows,linux)
inventory_hostname ==> name of server
```

## Block and Rescue and Always

### block
for use same module in many task we create block , in this example use **ignore_errors** for all **command** tasks :

```yaml
- 
  name: block
  hosts: localhost
  tasks:
  - block:
        - command: "ls /home"
          register: home_out

        - command: "ls /root"
          register: root_out

        - command: "ls /temp"
          register: home_out
    ignore_errors: yes
```

### rescue and always

it's like (try,catch) in ansible is (block,rescue) that means if first task under **block** will be fail , the task under **resume** will be run and the task under **always** will always executes:

```yaml
- 
  name: block
  hosts: localhost
  tasks:
  - block:
        - command: "ls /home"
          register: home_out
    rescue:
        - debug:
            msg: "this task is gonna run if 'ls /home' be failed"
    always:
        - debug:
            msg: "this will always executes"
```

## include_* vs import_*

import_ * are static and include_* are dynamic

All import* statements are pre-processed at the time playbooks are parsed. \
All include* statements are processed as they encountered during the execution of the playbook. \
So import is static, include is dynamic.

include: ( include_playbook, include_tasks, include_vars, include_role ) \
import: ( import_playbook, import_tasks, import_vars, import_role )

```yaml
- name: import and include
  hosts: localhost
  tasks:
    - include_task: install_webserver_{{ ansible_os_family }}.yml # it's run without problem
    - import_task: install_webserver_{{ ansible_os_family }}.yml # it's going to error, beacase can't parse var during plybook execution
```

## delegate_to, local_action

**delegate_to** for run just for specific task in different server but **local_action** for localhost + all servers

```yaml
  name: delegate_to
  hosts: server1
  tasks:
    - name: ls /home
      command: "ls /home"
      register: home_out
      delegate_to: localhost # just run in localhost
      run_once: true # just run once in localhost not another server in the group
```

```yaml
  name: local_action
  hosts: server1
  tasks:
    - name: ls /home
      local_action: shell 'ls /home' # run on localhost + all servers
      register: home_out
```