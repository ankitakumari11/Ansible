| Server Name     | Public IP        | Private IP   | OS               |
|-----------------|------------------|--------------|------------------|
| Master-Server   | 140.245.7.135    | 10.0.0.184   | Ubuntu 22.04     |
| Client-Server1  | 80.225.230.138   | 10.0.0.160   | Ubuntu 20.04     |
| Client-Server2  | 141.148.222.111  | 10.0.0.187   | Ubuntu 22.04     |
| Client-Server3  | 129.154.242.34   | 10.0.0.12    | Oracle Linux 8   |
| Client-Server4  | 80.225.235.74    | 10.0.0.92    | Oracle Linux 7.9 |

### Client nodes(all):
```
adduser ansible
passwd ansible
visudo:
  ansible ALL=(ALL:ALL) NOPASSWD: ALL
vi /etc/ssh/sshd_conf
  Passwordauthentication yes
```

### Run on Master node:


```
adduser ansible
visudo :
  ansible ALL=(ALL:ALL) NOPASSWD: ALL
vi /etc/ssh/sshd_config {password authentication: yes}
su - ansible
sudo apt install -y software-properties-common git python3 python3-pip
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
sudo apt install ansible-core
python3 -m pip install ansible-navigator --user
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/profile
source ~/profile
vi inventory
----
[dev-node1]
10.0.0.160
[test-node1]
10.0.0.187
[prod-node1]
10.0.0.12 ansible_python_interpreter=/usr/bin/python3.8
[lb-node1]
10.0.0.92 ansible_python_interpreter=/usr/bin/python3.8
[web-node1:children]
prod-node1

[demo]
10.0.0.160
10.0.0.187
10.0.0.12 ansible_python_interpreter=/usr/bin/python3.8
10.0.0.92 ansible_python_interpreter=/usr/bin/python3.8
----

vi ansible.cfg
-----------
[defaults]
remote_user= ansible
inventory=/home/ansible/inventory
role_path=/home/ansible/roles
collection_path=/home/ansible/mycollection

[privilege_escalation]
become=true
----------
```

`ssh 10.0.0.160`
> here you are required to put password so to avoid it , create a key on ansible server inside ansible user and paste public key to all the nodes

```
ssh-keygen
cd .ssh
ls
o/p-> id_rsa  id_rsa.pub
ssh-copy-id ansible@<private-ip-of-node>
```

Now rather than writing each ip , we can do:  
`ansible NodesGroup -m authorized_key   -a "user=ansible key={{ lookup('file', '~/.ssh/id_rsa.pub') }}"   --ask-pass`

> 	In my case - I had 4 servers (2 servers - ubuntu on which python version >8 available but for rest two servers (7.9 and 8.4)->it was python 3.6 with which ansible modules are not compatable so need to upgrade them first.  
```
	sudo apt install sshpass -y
	ansible@masterserver:/etc/ansible$ ansible all -m raw -a "yum install -y python38" --ask-pass --become --ask-become-pass
```
On version oracle linux 8 , it installed python 3.8 but didn’t on version 7.9  

![image](https://github.com/user-attachments/assets/71f237da-c4c9-474a-a447-dec1943edfd2)

`ansible@masterserver:/etc/ansible$ ansible 10.0.0.12 -m raw -a "ls /usr/bin/python3.8" --ask-pass`

![image](https://github.com/user-attachments/assets/5e524baa-01df-4703-98b0-adbf42747739)

For 7.9 , I did this:  
```
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "yum install -y oracle-softwarecollection-release-el7" --ask-pass --become --ask-become-pass
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "yum install -y rh-python38" --ask-pass --become --ask-become-pass
```

![image](https://github.com/user-attachments/assets/d18d44a7-541d-408b-9ab3-5a8fd9a4d934)

```
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "ls /usr/bin/python3.8" --ask-pass
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "python3 --version" --ask-pass
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "scl enable rh-python38 -- python --version" --ask-pass
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "python3 --version" --ask-pass
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "ln -sf /opt/rh/rh-python38/root/usr/bin/python3.8 /usr/bin/python3.8" --ask-pass --become --ask-become-pass
ansible@masterserver:/etc/ansible$ ansible 10.0.0.92 -m raw -a "scl enable rh-python38 -- python --version" --ask-pass
```

![image](https://github.com/user-attachments/assets/73e73abb-1271-4c9b-b72a-495e6bf92bfb)

Since I installed ansible using root so the ansible hosts file had root privilages so I changed the ownership to ansible user:  

![image](https://github.com/user-attachments/assets/c350ee4b-73b8-47d7-bf9a-376a26303e48)

I did this entry inside the inventory (commented the other 2 become , I had already copied the public key to them):  

![image](https://github.com/user-attachments/assets/9a49715d-dd15-4543-b188-b7ebe0f64bbc)

Again ran the command and how it copied the keys:
`ansible@masterserver:/etc/ansible$ ansible NodesGroup -m authorized_key   -a "user=ansible key={{ lookup('file', '~/.ssh/id_rsa.pub') }}"   --ask-pass`


![image](https://github.com/user-attachments/assets/e060139e-0986-46f9-9160-9b35487d9dd1)






