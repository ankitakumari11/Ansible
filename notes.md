# 🔹 1. Built-in modules

- These are modules that come bundled with Ansible itself (no need to install extra content).
- Example: copy, file, yum, apt, service.
- Execution:
  - They are executed on the managed node (target server), unless the module is specifically a "local" one (like add_host or meta).
  
# 🔹 2. Collections  
  
- Collections are just a packaging format in Ansible.
- They can contain:
  - Roles
  - Modules
  - Plugins
  - Playbooks
  - Docs/tests
- Collections can be installed from Ansible Galaxy or private repos.
- Example:
  - amazon.aws (for AWS modules/plugins/roles)
  - community.general (tons of community modules)
- Execution:
  - Modules inside collections (e.g., amazon.aws.ec2) often interact with cloud APIs.
  - When you run them:
    - The API calls happen from the control node (because AWS, Azure, GCP APIs aren’t running on the managed node).
    - Some other modules in collections (like Linux hardening modules) still run on the managed node.

# 🔹 Inventory Structure

```
inventory/
├── group_vars/
│   ├── apc_south_az2_linux.yml
│   └── apc_south_az2_windows.yml
├── hosts.ini
└── host_vars/
```
a. hosts.ini
- This file lists all your servers that Ansible can manage.
- Servers are grouped so you can target multiple servers at once.
```
[apc_south_az2_linux]
linux-server1.example.com
linux-server2.example.com

[apc_south_az2_windows]
win-server1.example.com
win-server2.example.com
```

b. group_vars/
- This folder stores variables that apply to an entire group of hosts.
- Each file is named after a group in your inventory (like apc_south_az2_linux.yml).
- Any variable defined here will automatically apply to all hosts in that group.
Example: apc_south_az2_linux.yml  
```
ansible_user: cloud-user
```
*This means all Linux servers in the apc_south_az2_linux group will be accessed using the SSH user cloud-user.*  

c. host_vars/
- This folder is similar to group_vars, but variables here are specific to one host.
- `host_vars/linux-server1.yml`


**How Ansible Uses These**  
When you run a playbook:
- Ansible checks which group(s) the host belongs to.
- It loads group variables automatically.
- It then overrides with host variables if they exist.
- It uses these variables to connect and execute tasks.

Quick visual:  
```
hosts.ini
├─ group: apc_south_az2_linux
│   ├─ linux-server1
│   └─ linux-server2
├─ group: apc_south_az2_windows
│   ├─ win-server1
│   └─ win-server2

group_vars/
├─ apc_south_az2_linux.yml   -> applies to all Linux servers
└─ apc_south_az2_windows.yml -> applies to all Windows servers

host_vars/
├─ linux-server1.yml -> overrides variables for linux-server1
```

- hosts.ini → Who are your servers and which groups do they belong to.
- group_vars/ → Variables that apply to all hosts in a group.
- host_vars/ → Variables for a specific server.

# 🔹 Jinja2

- Jinja2 is a templating engine used in Ansible to make files, variables, and outputs dynamic.
- It is mainly used with the template module to copy configuration files dynamically from source (.j2 file) to destination.
- You can insert variables with {{ variable }} (declared in defaults/main.yml, vars, inventory, or facts).
- You can also use logic like conditionals (if/else) and loops (for) inside templates.
- Apart from files, Jinja2 is also used everywhere in Ansible (tasks, debug messages, conditions, when statements, etc.) — not only for configuration files.

🔹 Example in context:  
  
defaults/main.yml  
```
my_timezone: "Asia/Kolkata"
```

templates/timezone.conf.j2  
```
# Timezone configuration
ZONE="{{ my_timezone }}"
```

tasks/main.yml  
```
- name: Deploy timezone config
  template:
    src: timezone.conf.j2
    dest: /etc/timezone.conf
```

👉 Result on target machine (/etc/timezone.conf):  
```
# Timezone configuration
ZONE="Asia/Kolkata"
```

### There are two ways you use Jinja2 in Ansible:  

1️⃣ Inside Templates (.j2 files)  
  
👉 Best for generating configuration files (like sshd_config, nginx.conf, etc.).
Example we just saw.  

2️⃣ Directly Inside Playbooks / Tasks (main.yml)  
  
👉 You can use Jinja2 inline with variables or even conditionals.  
```
- name: Create user with dynamic home directory
  user:
    name: "{{ new_user }}"
    home: "/home/{{ new_user }}"
    shell: "{{ default_shell }}"
```

Where variable values can come from:  
1. defaults/main.yml (inside a role)
```
# roles/myrole/defaults/main.yml
new_user: ankita
default_shell: /bin/bash
```
  
2. vars/main.yml (inside a role)
```
# roles/myrole/vars/main.yml
new_user: adminuser
default_shell: /bin/zsh
```
  
3. group_vars/ (for groups of hosts in inventory)
```
inventory/
├── group_vars/
│   ├── all.yml
│   └── webservers.yml
```
Example group_vars/all.yml:  
```
new_user: devops
default_shell: /bin/bash
```
4. host_vars/ (specific to one host) : host_vars/server1.yml  
```
new_user: ubuntu
default_shell: /bin/sh
```

5. host_vars/ (specific to one host)
```
- hosts: all
  vars:
    new_user: testuser
    default_shell: /bin/bash
  tasks:
    - name: Create user
      user:
        name: "{{ new_user }}"
        shell: "{{ default_shell }}"
````
6. Extra vars (command line -e)
```
ansible-playbook site.yml -e "new_user=cliuser default_shell=/bin/zsh"
```
   
