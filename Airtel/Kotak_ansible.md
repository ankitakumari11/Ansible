# <ins>Ansible</ins>

![image](https://github.com/user-attachments/assets/c0ecac23-3d98-4af9-9be4-88a2fbb469b9)
  
Prerequistic to install ansible: -> Python  
### STEPS:

1. Create 3 servers (Amazon ami type os)

![image](https://github.com/user-attachments/assets/1aafeea9-abd1-4502-809c-8a0859838657)

2. Step 2:Make Authentication:
- login to master 
- switch to root user
- cd 
- `ssh - keygen`

![image](https://github.com/user-attachments/assets/1dd3ba1d-441b-4c59-92d2-ee5d39c4af3c)

- cd .ssh/
- pwd - /root/.ssh
- cat id_rsa.pub
- copy the key and paste in nodes cd /etc/.ssh/authorized-keys (paste the key on both target nodes)
  
![image](https://github.com/user-attachments/assets/5705f017-bade-4621-8ddd-2e3b29fcd5bf)

Step 3:
 - To establish the ssh connection(Both master and nodes)
   - cd /etc/ssh
   - vi sshd_config
     - PermitRootlogin yes (uncomment line set to yes)
     - PasswordAuthentication yes (uncomment line set to yes)
  
![image](https://github.com/user-attachments/assets/4902dc3f-3f01-4541-a1dd-259a23f90adc)

step 4; Establish conncetion among the servers
 - login ansible master - switch to root user
 - cd /etc/
 - vim hosts
 - add the private IPs of all ther servers including master
	 - 172.31.30.163
	 - 172.31.19.26
	 - 172.31.27.132
 - restart the sshd by systemctl restart sshd
 - Repeat same process on all the servers.
 - check the inbound rules of secrity group "All ICMP IPV4 need to be added"
 - check the ping connection with ping <private IP>
   
 - Only on Ansible master
   - login with root user
   - install ansible as per document (https://ianodad.medium.com/how-to-install-ansible-2-on-aws-linux-2-ec2-ba0ffde42792)
   - to check ansible version ansible --version
 
 - Only on ansible master( inventory)
   - cd /etc/ansible
   - vi hosts

```
 [webservers]
 add private ips of target nodes
 172.31.19.26
 172.31.27.132
```
 
`systemctl restart sshd`
`ansible -m ping <private IP of nodes>`
 
`ansible -m ping 172.31.93.231`

step5 :  
- Create a file : `mkdir playbooks`
- create 3 yaml files:
  - index.html
  - playbook1.yml

vi index.html  
```
Hello World
```

vi playbook1.html
```
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
    - name: start httpd
      service: name=httpd state=started
```

```
ansible-playbook playbook1.yml
```
<img width="1895" height="837" alt="image" src="https://github.com/user-attachments/assets/34cc00b2-2425-4612-8eec-470820c2ecb5" />

Now go to security group of your target vms and allow ingress at 80 from all.  
  
<img width="1893" height="883" alt="image" src="https://github.com/user-attachments/assets/eaf48148-456c-4978-87d2-8f60fd45d3a9" />  

Now to go browser and write : http://public-ip of target server  

<img width="797" height="154" alt="image" src="https://github.com/user-attachments/assets/4eed863c-1e52-47fc-9859-d815986abbf4" />  

## VARIABLES IN ANSIBLE

vim playbook2var.yml
```
- hosts: webservers
  remote_user: root
  become: yes
  vars:
    pkg: httpd
  tasks:
    - name: installing httpd
      yum: name={{pkg}} state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
    - name: start httpd
      service: name={{pkg}} state=started

```

`ansible-playbook playbook3varfile.yml"

#### Use of variable file:
vim var.yml  
```
pkg: httpd
```
  
vim playbook3varfile.yml  
```
---
- hosts: webservers
  remote_user: root
  become: yes
  vars_files:
   - var.yml
  tasks:
    - name: installing httpd
      yum: name={{pkg}} state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
    - name: start httpd
      service: name={{pkg}} state=started
```  

<img width="1887" height="595" alt="image" src="https://github.com/user-attachments/assets/37c881de-13c9-4c16-a372-d0eefab0f9cf" />

  
## Interactive mode: user will be asked the package name
  
vim playbook4varfile.yml  
```
---
- hosts: webservers
  remote_user: root
  become: yes
  vars_prompt:
   - name: pkg
     prompt: enter package name
     private: no
  tasks:
    - name: installing httpd
      yum: name={{pkg}} state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
    - name: start httpd
      service: name={{pkg}} state=started
```

<img width="1882" height="651" alt="image" src="https://github.com/user-attachments/assets/98f4c981-3376-483d-8291-20db43ef524f" />

Now if you wirte private: yes , then whatever u type , it wont be visible but it is taking input:
```
---
- hosts: webservers
  remote_user: root
  become: yes
  vars_prompt:
   - name: pkg
     prompt: enter package name
     private: yes
  tasks:
    - name: installing httpd
      yum: name={{pkg}} state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
    - name: start httpd
      service: name={{pkg}} state=started
```

<img width="1901" height="577" alt="image" src="https://github.com/user-attachments/assets/cac6bc44-e5fc-4b69-a6eb-3431958b19a2" />

## HANDLERS

first delete the previous index.html  

vim playbook5handlers.yml  

```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
      notify: restart httpd
    - name: start httpd
      service: name=httpd state=started
  handlers:
   - name: restart httpd
     service: name=httpd state=restarted
```

<img width="1630" height="673" alt="image" src="https://github.com/user-attachments/assets/e7f13977-7f45-4927-ac6a-2e33f5d08195" />

## IGNORE ERRORS:

vim playbook6ignore.yml
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
    - name: copy index1.html file
      copy: src=index1.html dest=/var/www/html
      ignore_errors: yes
    - name: start httpd
      service: name=httpd state=started
```
*here since u have written ignore yes so even error came , still it ignored it and completed the playbook*
<img width="1896" height="841" alt="image" src="https://github.com/user-attachments/assets/377930a7-2f9d-4b5a-8ea7-46e7e59f6c4c" />


<img width="934" height="360" alt="image" src="https://github.com/user-attachments/assets/4e48fdb2-3259-43aa-a9e6-7a2559ee210b" />  

<img width="1901" height="818" alt="image" src="https://github.com/user-attachments/assets/3b54edc0-32ab-490b-b47a-a4f09c3a4a82" />


  
## FOR LOOPS

vi playbook7 
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
   - name: To install multiple packages
     yum: name={{item}} state=installed
     with_items:
        - httpd
        - wget
        - curl
```

## VAULT MODULE

Vault module: vault module will be used to encrypt the playbook  	
commands:  
- ansible-vault encrypt playbook8vault.yml  - to encrypt the playbook
- ansible-vault edit playbook8vault.yml  - to edit the playbook
- ansible-vault decrypt playbook8vault.yml - To descrypt the playbook  

<img width="1110" height="223" alt="image" src="https://github.com/user-attachments/assets/b770e059-edcf-4bb0-a55a-4b029361a9f1" />
  

vim playbook8vault.yml  
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
    - name: copy index1.html file
      copy: src=index1.html dest=/var/www/html
    - name: start httpd
      service: name=httpd state=started

```

If u do directly vi playbook8vault.yml or cat , it will show encrypted file:  

<img width="1217" height="738" alt="image" src="https://github.com/user-attachments/assets/78eb562d-e5aa-4a58-a524-bfaea99af694" />

ansible-vault decrypt playbook8vault.yml  

<img width="1053" height="437" alt="image" src="https://github.com/user-attachments/assets/ddaa0ff0-8671-4cd9-ba66-5c3ce46a4ad9" />


vim playbook9register.yml
  
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
    - name: start httpd
      service: name=httpd state=started
      register: output
    - debug:
        msg: "{{output}}"
```

- Service module ensures the httpd service is running. (It does not enable it for boot; add enabled: yes if you want that.)
- register: output captures the module’s return dictionary in a variable called output. The dict tells you whether anything changed, exit status, etc.

     <img width="1909" height="692" alt="image" src="https://github.com/user-attachments/assets/79feebce-403a-4b61-bc02-d700b6d4e146" />
  

## Tags:   
  
By using tags we can control execution of playbook. we can execute perticular steps  
- Method1:
  - ansibe-playbook <playbook name> --tag "install,configure" : eg: ansible-playbook Playbook10tags.yml --tag "install,configure"
  - ansible-playbook <plabook name> --skip-tags "configure" : eg: ansible-playbook Playbook10tags.yml --skip-tags "configure"
- Method2:
  - ansible-playbook <playbook name> --start-at-task="copy index.html file"
                  <img width="1891" height="676" alt="image" src="https://github.com/user-attachments/assets/81138652-7306-4b2e-bc55-1baf07265405" />

- Method3:
  - ansible-playbook <Playbook name> --step (it will ask at every step if u want to perform that task or not)
                 <img width="1904" height="655" alt="image" src="https://github.com/user-attachments/assets/4e3488c1-c9e8-447a-81fd-c0be38d423b7" />

		
vim Playbook10tags.yml  
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
      tags:
       - install
    - name: copy index.html file
      copy: src=index.html dest=/var/www/html
      tags:
       - configure
    - name: start httpd
      service: name=httpd state=started
      tags:
       - service
```  	   
## Include Method:

If u want to resuse the same ymls again and again then u can use include method.  

vim install.yml
```
---
- name: httpd
  yum: name=httpd state=installed
```
 
vim service.yml
```
---
- name: start httpd
  service: name=httpd state=started
```
     
vim playbook11include.yml
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
   - include: install.yml
   - include: service.yml
 ```


## To use when condition in playbook
   
vim playbook12when.yml
```
---
- hosts: webservers
  remote_user: root
  become: yes
  tasks:
    - name: httpd
      yum: name=httpd state=installed
      when: ansible_os_family== "RedHat"
    - name: install apache2
      apt: name=apache2,state=installed
      when: ansible_os_family== "Debian"
```

   <img width="1910" height="695" alt="image" src="https://github.com/user-attachments/assets/637632e1-981d-4a04-9c7c-74e54dfb9bfe" />

## Setup module  
ansible <private IP of target node> -m setup  - It will give complete information about the node

   <img width="1920" height="529" alt="image" src="https://github.com/user-attachments/assets/cbbde40c-2ddd-4da6-a323-5d94b7a7a622" />

  
## Ansible-Roles

We didnt create these folders , at the time of ansible installation only , it created these folder like playbooks , roles etc  

<img width="1893" height="163" alt="image" src="https://github.com/user-attachments/assets/a3cd107d-0d3e-47a5-bede-144bb8f3da2d" />

`ansible-galaxy init ankita-role` -> This will create a role.

<img width="1920" height="225" alt="image" src="https://github.com/user-attachments/assets/752650be-3b53-400f-a893-ca5a93496c3f" />

- Here inside **files** , we can write .html files directly (write index.html)
```
<html>
        <body>
                <h1> Hello world </h1>
        </body>
</html>
```
  
- Inside **tasks** , u can write tasks (write main.yml)
```
---
# tasks file for ankita-role
- name: httpd
  yum: name=httpd state=installed
- name: copy index.html file
  copy: src=index.html dest=/var/www/html
- name: start httpd
  service: name=httpd state=started
```


- Inside **vars** , u can write variables
- So ansible roles is the structures directory.
- Now go inside files and tasks and write infex.html and tasks inside tasks/main.yml
      <img width="1920" height="695" alt="image" src="https://github.com/user-attachments/assets/b2f4dace-9b6f-4b52-9ecb-1bcf9db85aa7" />

- Now after writing index and main.yml inside files and tasks of roles/ankita-role , go back to /etc/ansible/playbooks and create a my_role.yml file.
  ```
  [root@ip-172-31-83-27 tasks]# cd ..`
  [root@ip-172-31-83-27 ankita-role]# cd ..
  [root@ip-172-31-83-27 roles]# cd ..
  [root@ip-172-31-83-27 ansible]# ls
    ansible.cfg  hosts  playbooks  roles
  [root@ip-172-31-83-27 ansible]# cd playbooks/
  [root@ip-172-31-83-27 playbooks]# vim my_role.yml
  ```
- vim my_role.yml
  ```
  ---
  - hosts: webservers
    remote_user: root
    become: yes
    roles:
     - ankita-role
  ```
- `ansible-playbook my_role.yml`
                    <img width="1901" height="680" alt="image" src="https://github.com/user-attachments/assets/3639baa0-1705-4674-8df5-0297a6543ce6" />

  






