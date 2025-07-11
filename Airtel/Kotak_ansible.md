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







