# Ansible

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
