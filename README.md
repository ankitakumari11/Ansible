![image](https://github.com/user-attachments/assets/9299dc82-fb4b-40cf-b66a-14cf1813067c)![image](https://github.com/user-attachments/assets/a080b81b-915f-4402-993c-aec05e8a82b4)![image](https://github.com/user-attachments/assets/733db3a1-096c-48e5-af50-a1a24240016a)# Ansible

### Installing Ansible in Oracle linux 8.10
1. Ans-server
2. Node1
3. Node2
{Login to Ansible server}:-

`sudo su -`  
[root@ans-server ~]# `dnf install oracle-epel-release-el8`  
[root@ans-server ~]# `dnf install ansible`   
[root@ans-server ~]# `dnf install git python3.12 python3-devel python3-pip openssl -y`  


{Go to host file inside Ansible and write the group name and Private IPs of node servers which you want to connect with Ansible server}
{Ansible server}:-
[root@ans-server ~]# `vi /etc/ansible/hosts`  
eg: 
```
[demo]
10.0.0.5
10.0.0.187
```

![image](https://github.com/user-attachments/assets/c5b53982-f693-4d2d-a7a1-2ce36dcddd5b)  

*This file will only work after updating ansible.cfg file*
*{Ansible server}:-*
```
[root@ans-server ~]# vi /etc/ansible/ansible.cfg
->uncomment : inventory      = /etc/ansible/hosts
->uncomment : sudo_user      = root
```
![image](https://github.com/user-attachments/assets/00da1dab-b1f8-4e28-be07-f8a51610f3f2)  

*{Create a user to connect Control node (Ansible server) with  managed nodes, create this user on all the managed nodes. You can create different user also but for that you need to specify in the commands that from which user you are connecting to the servers(nodes)}*
*{Ansible server}:-*  
`[root@ans-server ~]# adduser ansible`  
`[root@ans-server ~]# passwd ansible`  

*{node1 server}*  
`[root@node1 ~]# adduser ansible`  
`[root@node1 ~]# passwd ansible`

*{node2 server}*  
`[root@node2 ~]# adduser ansible`  
`[root@node2 ~]# passwd ansible`  
![image](https://github.com/user-attachments/assets/d2bf69b9-462d-4a54-aaf9-15e4edbcf432)  

*{Give sudo permissions to the "ansible" user in control server(ansible server) as well as nodes} ->{Ansible server,node1,node2,etc}*  
```
[root@ans-server ~]# visudo
ansible ALL=(ALL:ALL) NOPASSWD: ALL
```  
![image](https://github.com/user-attachments/assets/1c97b54e-2c2f-412a-a5fc-1c87d34110b1)  


*To establish connection btw control node and managed nodes, edit sshd_config file in all the 3 servers(ansible+nodes)*  
*{Ansible server,node1,node2,etc}*  
```
[root@ans-server ~]# vi /etc/ssh/sshd_config
->uncomment and comment "no" section: PasswordAuthentication yes
->uncomment or put yes :PermitRootLogin yes
[root@ans-server ~]# systemctl restart sshd

```

![image](https://github.com/user-attachments/assets/b3cccc95-bb86-4fb7-9a1a-27c1d4a7deca)  
![image](https://github.com/user-attachments/assets/615ea0c9-968f-49ed-bd74-20f449a7195e)  

```
[root@ans-server ~]# su -ansible
[ansible@ans-server ~]$ ssh <private ip of any managed node>
```

*{it will ask for password of "ansible" user, now here it is automatically login with user "ansible" becoz we are switched to it in our controlling node but if suppose on managed nodes you have created another user of some other name then you need to specify by which user you want to connect with managed nodes}*  
*{To avoid giving password everytime when you are connecting to the managed nodes, create public and private key at control node and paste private key in managed nodes , this will prevent password insertion everytime}*  

```
[ansible@ans-server ~]$ ssh-keygen
[ansible@ans-server ~]$ ls -a
o/p-> .ssh
[ansible@ans-server ~]$ cd .ssh
[ansible@ans-server .ssh]$ ls
o/p-> id_rsa  id_rsa.pub
```
![image](https://github.com/user-attachments/assets/13f0e35a-8521-4cd4-b25e-0182ab3cf3dc)

```
[ansible@ans-server .ssh] ssh-copy-id ansible@<private-ip-of-node>
{do for all the nodes}

[ansible@ans-server ~]$ ssh 141.148.197.104{private ip of node}
[ansible@node1 ~]$      --->now you are inside node1
```

- If your control node and managed nodes are in different subnets, SSH connection may not work. To fix this, apply a **Security List rule** at the **managed node subnet**:
  - **Ingress rule configuration:**
    - **Source CIDR:**
      - If the control node has a fixed public IP: `203.0.113.5/32`
      - If the control node is in the same VCN: use the subnet CIDR, e.g., `10.0.1.0/24`
    - **IP Protocol:** TCP  
    - **Destination Port Range:** 22

> ⚠️ If you want this ingress rule only on specific managed nodes (not the whole subnet), use **Network Security Groups (NSGs)** instead.

*{if you want to see all the hosts or nodes of the ansible , group specific too}*
```
[ansible@ans-server ~]$ ansible all --list-hosts
[ansible@ans-server ~]$ ansible demo[0] --list-hosts
[ansible@ans-server ~]$ ansible demo[1] --list-hosts
```
#### [AD-HOC COMMAND EXAMPLES]
```
[ansible@ans-server ~]$ ansible demo -a "ls"
[ansible@ans-server ~]$ ansible demo -a "ls -la"
[ansible@ans-server ~]$ ansible demo -a "ls -al"
[ansible@ans-server ~]$ ansible demo -a "sudo yum install httpd -y"
[ansible@ans-server ~]$ ansible demo -ba "yum install httpd -y" {if u don’t want to use sudo then use -ba}
[ansible@ans-server ~]$ ansible demo[0] -a "sudo yum remove  httpd -y"
```
![image](https://github.com/user-attachments/assets/514f2377-9810-48fb-9459-e43dfbf92cb5)  
![image](https://github.com/user-attachments/assets/97979478-c6b4-47d8-b6a2-a66a1d311308)  

#### [MODULES EXAMPLES]
```
[ansible@ans-server ~]$ ansible demo -b -m yum -a "pkg=httpd state=absent"
[ansible@ans-server ~]$ ansible demo -b -m user -a "name=Ankita"
[ansible@ans-server ~]$ touch filez  {if you want to copy a file from your ansible server to the nodes}
[ansible@ans-server ~]$ ls
[ansible@ans-server ~]$ ansible demo[0] -b -m copy -a "src=filez dest=/tmp"
```
![image](https://github.com/user-attachments/assets/a4c187d8-485e-416b-b722-06f0faf6dbe3)  
<br>
![image](https://github.com/user-attachments/assets/bf3e33ee-5a1e-45fc-931f-176155a59dbd)  
<br>
![image](https://github.com/user-attachments/assets/77b4a0c4-65b4-467e-b484-c6c25ea5a8dd)    
<br>

`[ansible@ans-server ~]$  ansible demo[0] -b -m copy -a "src=ansible_file dest=/home"`  
![image](https://github.com/user-attachments/assets/9ed8841e-3fba-4647-830c-7682fbcde4a2)  
<br>
![image](https://github.com/user-attachments/assets/a9d441f9-11b4-4b89-864a-94eefb787993)  
<br>

### PLAYBOOKS IN ANSIBLE:   
```
[ansible@ans-server ~]$ vi target.yml

--- # My First testing YAML Playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  gather_facts: yes
```

![image](https://github.com/user-attachments/assets/8cd69df4-ee82-4620-b3ce-3fa136b75d20)
<br>
```
[ansible@ans-server ~]$ vi task.yml

--- # My second playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
          - name: Install HTTPD  on linux
            action: yum name=httpd state=installed
```
<br>

![image](https://github.com/user-attachments/assets/fec9b5a0-c7a0-4350-be6f-aa115974582f)
<br>

```
[ansible@ans-server ~]$ vi vars.yml

--- # My Variable Playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  vars:
          pkgname: httpd
  tasks:
          - name: Install HTTPD server
            action: yum name='{{pkgname}}' state=installed
```
<br>

```
--- # My Variable Playbook {new version}
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  vars:
    pkgname: httpd
  tasks:
    - name: Install HTTPD server
      yum:
        name: "{{ pkgname }}"
        state: present

[ansible@ans-server ~]$ ansible-playbook vars.yml
```
![image](https://github.com/user-attachments/assets/a603a9b2-368a-46ee-acc4-29009df6ad51)
<br>

```
[ansible@ans-server ~]$ vi handlers.yml

--- # Handlers playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
          - name: install httpd servers on oracle
            action: yum name=httpd state=installed
            notify: restart httpd
  handlers:
          - name: restart httpd
            action: service name=httpd state=restarted
```

```
--- # Handlers playbook {new version}
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
    - name: Install HTTPD server
      yum:
        name: httpd
        state: present
      notify: restart httpd

  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted
```
`[ansible@ans-server ~]$ ansible-playbook handlers.yml --check {command to dry-run the scrip}`  
![image](https://github.com/user-attachments/assets/dacb98c6-f427-4d98-8cc7-6de1b8740cac) 
<br>

```
[PLAYBOOK]
--- #Loops playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
          - name: Add a list of users
            user:
               name: "{{ item }}"
               state: present
            loop:
                    - sunny
                    - hunny
[ansible@ans-server ~]$ ansible-playbook loops.yml
```
#### [ROLES IN ANSIBLE]
```
├── playbook
│   ├── master.yml
│   └── roles
│       └── webserver
│           └── tasks
│               └── main.yml
```

```
[ansible@ans-server ~]$ cat playbook/master.yml
--- #my master playbook for roles
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  roles:
          - webserver
```


```
[ansible@ans-server ~]$ cat playbook/roles/webserver/tasks/main.yml
- name: insall apache on redhat
  yum:
          pkg: httpd
          state: present
```

#### [TO DYNAMICALLY GIVE IPS TO THE INVENTORY]

` vi subnet_inventory.py`
```
[ansible@instance-1123 ~]$ cat subnet_inventory.py
#!/usr/bin/env python

import json
import subprocess
import re
import socket

# Define the subnet to scan
subnet = "10.0.2.0/24"

# Function to get the private IP address of the machine
def get_private_ip():
    try:
        hostname = socket.gethostname()
        private_ip = socket.gethostbyname(hostname)
        return private_ip
    except Exception:
        return None

# Get the private IP address
private_ip = get_private_ip()

# Run nmap to discover active hosts
try:
    nmap_output = subprocess.check_output(
        ["nmap", "-sn", "-PS22,80", subnet], stderr=subprocess.STDOUT
    ).decode('utf-8')
except subprocess.CalledProcessError as e:
    print(json.dumps({"_meta": {"hostvars": {}}}))  # Empty inventory in case of error
    exit(1)

# Regular expression to match IP addresses
ip_pattern = re.compile(r'\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b')

# Parse the nmap output to extract IP addresses
hosts = []
for line in nmap_output.splitlines():
    if 'Nmap scan report for' in line:
        ips = ip_pattern.findall(line)
        if ips:
            hosts.extend(ips)

# Remove the private IP address from the list of hosts
if private_ip in hosts:
    hosts.remove(private_ip)

# Define the inventory structure
inventory = {
    "demo1": {
        "hosts": hosts
    },
    "_meta": {
        "hostvars": {}
    }
}
```


**Print JSON output for Ansible**
```
print(json.dumps(inventory, indent=2))
sudo yum install -y nmap
chmod +x subnet_inventory.py
[ansible@ans-server ~]$  ./subnet_inventory.py
[ansible@ans-server ~]$ ansible-playbook -i subnet_inventory.py subnet_playbook.yml
[ansible@ans-server ~]$ cat subnet_playbook.yml
```
```
--- # My second playbook
- hosts: demo1
  user: ansible
  become: yes
  connection: ssh
  tasks:
          - name: Install HTTPD  on linux
            action: yum name=httpd state=installed
```

![image](https://github.com/user-attachments/assets/ff6e47ee-f275-4e2c-ab67-466dbf7a3922)

### User Management in Ansible – Control Node vs. Managed Nodes

1. Do we need to create a user on managed nodes?
Yes, a user must exist on the managed nodes for Ansible to connect and execute commands. This user should:
✅ Have SSH access from the control node.
✅ Have sudo privileges (if required) to perform administrative tasks.
✅ Be configured for passwordless authentication (using SSH keys) for smooth automation.
2. Do we need to create the same user on both the control node and managed nodes?
🔹 No, it is not mandatory.
🔹 The control node only needs Ansible installed and does not require a user that mirrors the one on managed nodes.
🔹 The managed nodes need a dedicated user (e.g., ansible) that Ansible can connect to and execute tasks under.
🔹 The user on managed nodes can be different from the one on the control node, as long as the Ansible control node can SSH into the managed nodes with the correct credentials.
3. Can we use the root user instead of creating a separate user?
🚫 Using the root user directly is not recommended. It poses serious security risks.
✅ Instead, create a dedicated user (e.g., ansible) and grant it passwordless sudo access to perform privileged tasks when needed.
✅ If you must run tasks as root, use become: yes in the playbook, which allows privilege escalation without logging in as root.
4. How to define a user in an Ansible playbook?
If you are using a specific user (e.g., ansible), define it in the playbook:
	- hosts: demo
  user: ansible  # Specify the user that will execute tasks
  become: yes    # Enable privilege escalation (sudo)
  connection: ssh  # Ensure SSH is used for communication
	If different managed nodes require different users, define them in the inventory file:
	
	[web_servers]
server1 ansible_user=ansible
server2 ansible_user=deployuser
	5. Best Practices for User Management in Ansible
	✅ Create a dedicated user on managed nodes (e.g., ansible).
	✅ Set up SSH key-based authentication for secure, passwordless connections.
	✅ Grant passwordless sudo access to the Ansible user to avoid password prompts.
	✅ Avoid running Ansible as root directly.
	✅ Define user access properly in inventory files for better control over different nodes.
	By following these best practices, you ensure a secure, scalable, and efficient Ansible automation environment. 🚀
	You said:
	what about root user?
	ChatGPT said:
	I've included information about the root user in the previous notes, but here's a more detailed breakdown focusing on the root user in Ansible:
	
	Using Root User in Ansible – Best Practices
	1. Can we use the root user in Ansible?
	✅ Yes, but it's not recommended.
	🔹 The root user has unrestricted privileges, which increases security risks.
	🔹 If compromised, the root user can allow complete control over all managed nodes.
	🔹 Best practice is to use a dedicated user with sudo access instead.
	2. How to run Ansible as the root user?
	There are two ways to execute Ansible tasks as root:
	a) Directly SSH as root (Not Recommended ❌)
		○ This means setting ansible_user=root in your inventory file and connecting as root.
		○ Security risks include exposing the root password and enabling direct root SSH login.
		○ Example inventory file:

i[servers]
server1 ansible_host=192.168.1.10 ansible_user=root
	b) Use become: yes for Privilege Escalation (Recommended ✅)
		○ Instead of logging in as root, use a non-root user and escalate privileges when required.
		○ Example playbook:

yaml
- hosts: all
  user: ansible   # Login as the ansible user
  become: yes     # Switch to root when needed
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
		○ This way, the playbook runs as ansible but executes privileged commands as root.
	3. What if the managed node does not allow root SSH login?
		○ Many systems disable root SSH login (PermitRootLogin no in /etc/ssh/sshd_config).
		○ In such cases, you must connect as a non-root user and use become: yes for administrative tasks.
	4. How to avoid entering the sudo password manually?
		○ If become: yes requires a password, you can configure passwordless sudo:
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
		○ This allows the ansible user to use sudo without being prompted for a password.
	5. When should we use the root user?
	✔️ When provisioning a new server where no user accounts exist yet.
	✔️ If you’re running commands that require root without sudo.
	✔️ For quick one-time tasks where security is not a concern.
	6. Final Recommendation
	🚫 Avoid using root directly for regular automation.
	✅ Instead, use a non-root user with sudo privileges and become: yes.
	✅ Ensure the user has passwordless sudo for seamless automation.
	By following these practices, you maintain security, flexibility, and efficiency in your Ansible environment. 🔒🚀

