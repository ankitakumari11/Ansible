# SERVER PROVISIONING USING ANSIBLE

🛠 Step 1: Create an Ansible Control Node on OCI  

1.1. Create an OCI Compute Instance  
- 1. Log in to OCI Console → Go to Compute → Instances.  
- 2. Click Create Instance and set:  
  		○ Name: ansible-control-node  
  		○ OS: Ubuntu 22.04 (or CentOS 8)  
  		○ Shape: VM.Standard2.1 (1 OCPU, 16GB RAM)  
  		○ Boot Volume: 50GB  
  		○ SSH Keys: Upload your public SSH key (~/.ssh/id_rsa.pub).  
- 3. Click Create and wait for the instance to be ready.  
1.2. Connect to the Instance via SSH  
Once the instance is running, copy the Public IP and run:  
`ssh -i ~/.ssh/id_rsa opc@<PUBLIC_IP>`  

🛠 Step 2: Install Ansible on the Control Node  
For Ubuntu  
`sudo apt update && sudo apt install -y ansible python3-pip`

For CentOS  
```
sudo yum install -y epel-release
sudo yum install -y ansible python3-pip
```  
Check Ansible installation:  
`ansible --version`  

🛠 Step 3: Install OCI CLI and Configure Authentication  
OCI CLI is required for authentication.  
*{in my case , I install and create everything inside home directory of ansible user so I first switched to ansible user and then installed the below things}*  

3.1 Install OCI CLI on ansible managed node  
```
curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh | bash
source ~/.bashrc
```
Verify installation:  
`oci --version`

3.2 Generate OCI API Key Pair  
Run the following commands:  
```
mkdir -p ~/.oci
openssl genrsa -out ~/.oci/oci_api_key.pem 2048
openssl rsa -in ~/.oci/oci_api_key.pem -pubout -out ~/.oci/oci_api_key_public.pem
```
This creates:  
	• oci_api_key.pem (Private key, do not share)  
	• oci_api_key_public.pem (Public key, to be uploaded to OCI)  

 
3.3 Upload API Key to OCI  
- 1. Go to OCI Console → Profile (Top-right corner) → User Settings.  
- 2. Click API Keys (Left-side menu) → Add API Key.  
- 3. Open the public key file:  
`cat ~/.oci/oci_api_key_public.pem`  
- 4. Copy the key content and paste it into OCI.  
- 5. Click Add and note the Fingerprint (used in the next step).
     
3.4 Configure OCI CLI  
Run:  
`oci setup config`
Enter:  
	• Tenancy OCID  
	• User OCID  
	• Region  
	• Private Key Path → /home/opc/.oci/oci_api_key.pem  
	• Fingerprint (from API Key in OCI Console)  
Test authentication:  
`oci os ns get`  
If it returns your namespace, OCI CLI is correctly configured. ✅  

🛠 Step 4: Install Ansible OCI Collection  
To allow Ansible to interact with OCI:  
`ansible-galaxy collection install oracle.oci`  

🛠 Step 5: Write an Ansible Playbook to Create an OCI VM  
5.1 Create a Project Directory  
`mkdir ~/ansible-oci && cd ~/ansible-oci`  

5.2 Create an Inventory File (inventory.yml)  
```
[localhost]  
127.0.0.1
```

*What does localhost mean?  
In this case, localhost refers to the Ansible Control Node itself (the machine where Ansible is installed).  
  ○ [localhost] → This is the group name (you can name it anything).  
  ○ 127.0.0.1 → This is the IP address for localhost, meaning the playbook will run on the same machine (Ansible Control Node).  
Why is this needed?  
  ○ Normally, Ansible connects to remote servers via SSH to run commands.  
  ○ But in this case, we are managing OCI resources using API calls (not SSH).  
  ○ The OCI modules in Ansible use OCI API to create VMs, so the tasks run locally on the control node without needing SSH to any server.*  

OR:  
```
[localhost]
localhost ansible_connection=local
```
This tells Ansible not to use SSH, but to execute tasks locally.  

5.3 Create an Ansible Playbook (create_oci_vm.yml)  

yaml  
```
- name: Create an OCI Compute Instance
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Create a compute instance
      oracle.oci.oci_compute_instance:
        auth_type: api_key
        compartment_id: "ocid1.compartment.oc1..xxxxxx"
        availability_domain: "EXAMPLE:US-ASHBURN-AD-1"
        shape: "VM.Standard2.1"
        create_vnic_details:
          assign_public_ip: yes
          subnet_id: "ocid1.subnet.oc1..xxxxxx"
        metadata:
          ssh_authorized_keys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        display_name: "ansible-created-instance"
        image_id: "ocid1.image.oc1..xxxxxx"
```
Replace:  
	• compartment_id → Your OCI compartment OCID  ----> oci iam compartment list --all  
	• availability_domain → Your region’s AD  
	• subnet_id → Your OCI subnet OCID  
	• image_id → The OCID of the desired OS image  

Filter for a specific OS (Example for Oracle Linux): If you want to find the image ocid :{Change 'Oracle-Linux' to 'Ubuntu', 'CentOS', etc.}  
`oci compute image list --compartment-id ocid1.tenancy.oc1..aaaaaaaacx55j4hqwc7di4ecttu6swur66owlwlofq2sea6wl3f2exw4oqkq --all --query "data[?contains(\"display-name\", 'Oracle-Linux')].{id:id, name:\"display-name\"}" --output table`
	
`oci compute image list --compartment-id ocid1.tenancy.oc1..aaaaaaaacx55j4hqwc7di4ecttu6swur66owlwlofq2sea6wl3f2exw4oqkq --all --query "data[?contains(\"display-name\", 'Ubuntu')].{id:id, name:\"display-name\"}" --output table`



🛠 Step 6: Run the Ansible Playbook  

`ansible-playbook -i inventory.yml create_oci_vm.yml`
This will provision a Compute instance on OCI.  

🛠 Step 7: Verify the Created Instance  
	1. Go to OCI Console → Compute → Instances.  
	2. Locate the instance "ansible-created-instance".  
	3. Copy the Public IP and SSH into the new instance  

`ssh -i ~/.ssh/id_rsa opc@<INSTANCE_PUBLIC_IP>`

### NOTE

`ssh_authorized_keys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"`
Explanation:
- ○ ssh_authorized_keys: This is an Ansible module that manages SSH authorized keys for a user.
- ○ lookup('file', '~/.ssh/id_rsa.pub'): This tells Ansible to read the contents of the file located at ~/.ssh/id_rsa.pub (which is the public key for SSH authentication) on the control node (the machine running Ansible).
- ○ "{{ ... }}": This is a Jinja2 expression that evaluates the result of the lookup function.
What It Does:
- ○ Reads the SSH public key (id_rsa.pub) from the control node.
- ○ Assigns the content of the public key to ssh_authorized_keys.
- ○ Typically, this is used to add the public key to a remote user’s ~/.ssh/authorized_keys file, allowing passwordless SSH login.
- ○ The task ensures that the public key (id_rsa.pub) from the Ansible control node is added to the ~/.ssh/authorized_keys file of the ansible user on the target hosts.
This allows the control node to SSH into those hosts without a password.

![image](https://github.com/user-attachments/assets/1f767a5d-e591-47ec-891c-6c19f9b47eda)  

I had already created the public and private key pair for the ansible user in the ansible user's home directory above earlier:
![image](https://github.com/user-attachments/assets/881f3826-8216-4511-a46d-6f1148541dca)  

When I specified:  `ssh_authorized_keys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"`
Since I was inside the ansible user, this command copied the public key (id_rsa.pub) of the ansible user into the newly created instance’s authorized_keys file.
However, if you want to use the same private key that you use for logging into the Ansible control node, you need to copy and use the public key of the opc user, which is already present in the authorized_keys file of the opc user's home directory.
1. opc user:
	* ○ The authorized_keys file in the opc user's .ssh directory contains the public key(s) of users who are allowed to authenticate to the instance as the opc user (the default user for Oracle Linux instances).
	* ○ This key is used to authenticate anyone who connects via SSH using the corresponding private key. In your case, it allows users with the correct private key to log in to the server as the opc user.
2. ansible user:
	* ○ The id_rsa.pub file in the ansible user's .ssh directory is the public key corresponding to the private key (id_rsa) in the same directory.
	* ○ The private key (id_rsa) is typically used for SSH authentication. The public key (id_rsa.pub) is what you'd add to authorized_keys on remote servers to allow the ansible user to authenticate.

The Flow:
- opc user and authorized_keys:
	* ○ The public key stored in the authorized_keys file of the opc user allows anyone with the matching private key to log in as opc.
	*	○ If you're logging in as opc, you're using a private key that matches the public key in authorized_keys.
- ansible user and id_rsa.pub:
	*	○ The ansible user has its own private and public keys (id_rsa and id_rsa.pub).
	*	○ If the ansible user needs to SSH into another system, it will use the private key (id_rsa), and the corresponding public key (id_rsa.pub) must be added to the authorized_keys file on the remote system to allow access.

Note: so initially I used this command sh_authorized_keys: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}" due to which I was not able to login to new server via putty using the same private key of ansible control node server so I just copied the content of authorized_keys of opc and pasted inside new built server's authorized keys

Ansible control node server:  
![image](https://github.com/user-attachments/assets/5b37a15a-913b-4f79-9bba-da8a9353c547)


New Server:  
![image](https://github.com/user-attachments/assets/57141002-ee01-4aac-a30c-66320f161c6e)


2nd method:
- • Copy the authorized_keys content to id_rsa.pub:
`cp /home/opc/.ssh/authorized_keys /home/opc/.ssh/id_rsa.pub`
- • Ensure proper permissions:
`chmod 644 /home/opc/.ssh/id_rsa.pub`
	• Update your Ansible playbook:

yaml
`ssh_authorized_keys: "{{ lookup('file', '/home/opc/.ssh/id_rsa.pub') }}"`

when we create id_rsa id_rsa.pub inside .ssh , it has nothing to do with authentication right ? it just stores a newly generated private/public key. but actual authentication occurs only through authorized key content matching with private key used right?
Yes, exactly! 🔥

How SSH Authentication Works:  
- ✅ id_rsa (private key) – Stored on the client (your machine). Used for authentication.
- ✅ id_rsa.pub (public key) – Stored on the client. Can be copied to remote servers but not needed for authentication.
- ✅ authorized_keys (on the server) – This is what SSH actually checks to see if a connecting private key is authorized.

Key Takeaways:  
-	• id_rsa & id_rsa.pub are only needed on the client side.
-	• authorized_keys is what the SSH server checks to allow access.
-	• The actual authentication happens when the private key (id_rsa) matches a public key inside authorized_keys.
-	• You can delete id_rsa.pub on the server, and SSH will still work fine!
💡 So when you generate SSH keys, it’s just creating a new key pair—authentication happens through authorized_keys.

![image](https://github.com/user-attachments/assets/7e5186ec-9a26-4183-8ea7-5577708cd8ec)  

now for this server , i am having authorized_key only for opc , why not for ansible?

1️⃣ Why does opc have authorized_keys but ansible does not?  
✅ When OCI creates a new instance, by default, it sets up SSH access for the opc user using the public key provided during instance creation.  
✅ The public key gets added to /home/opc/.ssh/authorized_keys.  
✅ If you switch to ansible, it does not have an authorized_keys file because no public key was added for it.  

2️⃣ Why does ansible have id_rsa and id_rsa.pub?  
✅ The ansible user likely generated an SSH key pair (id_rsa + id_rsa.pub) to SSH into other servers, but its own authorized_keys file was never created.  
✅ This means the ansible user can SSH out to other servers, but you cannot SSH into ansible directly unless you add a key to its authorized_keys.  

3️⃣ How to allow SSH into the ansible user?  
If you want to SSH into ansible, you need to:  
📌 Manually create authorized_keys for ansible and add a public key.  

Run this as ansible user:
```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
Now, you should be able to SSH into the ansible user using its private key.  

Final Summary  
	• ✅ opc has authorized_keys because OCI adds it by default during instance creation.  
	• ❌ ansible does not have authorized_keys, so it cannot be accessed via SSH yet.  
🛠️ To enable SSH access to ansible, copy its public key to authorized_keys.  

> image ocid:
> ocid1.image.oc1.ap-mumbai-1.aaaaaaaath2k2vruzikodsndascrjxkarsbt7sy64to7bpjvcgyndzrtbh7q

> compartment ocid:
> ocid1.compartment.oc1..aaaaaaaa62uwziqusioyk6kax7qrrmutmgangc6f77aaowec4ltcdc2jokra
> subnet ocid:
> ocid1.subnet.oc1.ap-mumbai-1.aaaaaaaauubdj4qeqxsntqzm5oiewxcct7xemuerep5yc56v76i7kt7ejwtq

> AD:
> ZBQE:AP-MUMBAI-1-AD-1

> Tenancy id:
> ocid1.tenancy.oc1..aaaaaaaacx55j4hqwc7di4ecttu6swur66owlwlofq2sea6wl3f2exw4oqkq
