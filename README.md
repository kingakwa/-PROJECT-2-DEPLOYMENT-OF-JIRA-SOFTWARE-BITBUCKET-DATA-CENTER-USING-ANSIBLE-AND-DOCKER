# -PROJECT-2-DEPLOYMENT-OF-JIRA-SOFTWARE-BITBUCKET-DATA-CENTER-USING-ANSIBLE-AND-DOCKER

# JIRA_software-deployment-in-containers-using-Ansible-and-Docker
<img width="922" alt="Image" src="https://github.com/user-attachments/assets/0cccb91e-ab24-4428-84c5-abc64293d8ae" />

**1.** Launching 3 Amazon Linux 2 instances and naming them **“Ansible_Control”**, **“Worker_Node_1”** and **“Worker_Node_2”**. (The security group of Ansible_Control is configured to allow just SSH requests from a desired IP (e.g., my IP) address. The security group of the Worker_Node_1 and Worker_Node_2 is configured to allow SSH traffic from the Ansible_Control and SSH traffic from a desired IP (e.g., my IP).

## Ansible security group inbound rule
<img width="562" alt="Image" src="https://github.com/user-attachments/assets/1367126f-7878-4185-a0ce-5f4bec2655ea" />

## Security Group inbound rules of Worker Nodes
<img width="571" alt="Image" src="https://github.com/user-attachments/assets/548a885c-bec2-4c47-ba5a-43551c07f3ec" />

## Created Instances
<img width="758" alt="Image" src="https://github.com/user-attachments/assets/dd8c09d0-441b-4825-8877-10e50c8a0e4c" />

**2.** Login to 3 three instances using Git Bash

<img width="939" alt="Image" src="https://github.com/user-attachments/assets/2196a418-626b-4cb2-bb13-338227b8c3a3" />

To determine which Git Bash belongs to which node, rename the servers.
- In each server, Switch as a root user
  ```sh
  sudo su –

- Get into the **“hostname”** file in the /etc/ directory then edit the hostname of each server
  ```sh
  nano /etc/hostname

<img width="755" alt="Image" src="https://github.com/user-attachments/assets/850e433e-6474-4425-b01b-8c6db4b452a3" />

- Save the changes and reboot the servers
  ```sh
  reboot

- Log into the instances again and you will realize that you can identify which Git bash belongs to which server

<img width="755" alt="Image" src="https://github.com/user-attachments/assets/abe8ff2b-5b62-40a0-bc7c-9a4503e3adb9" />

**3.** For all 3 servers create a common user called “ansible”, set a common password, and allow for “Password authentication”  using the following commands for all 3 servers.
  ```sh
  sudo su - # Login as Root User
  useradd ansible
  passwd ansible # pass a password
  sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config # change password authentication permission to "yes"
  ```

- Navigate to the /etc/ssh/sshd_config path, and uncomment **"PermitRootLogin""PubkeyAuthentication"** in the sshd_config file.
  ```sh
  nano /etc/ssh/sshd_config
  ```
<img width="636" alt="Image" src="https://github.com/user-attachments/assets/13faed9c-e9f1-4fc8-9583-49fb5c65513b" />  
- Restart the sshd service.
  ```sh
  sudo systemctl restart sshd
  ```
  
**4.** Add ansible to the “sudoers” group in each server.
- Navigate to the sudoers group file via the following command.
  ```sh
  nano /etc/sudoers
  ```

- Add **“ansible”** user in each of the following points in the file.

<img width="380" alt="Image" src="https://github.com/user-attachments/assets/0a1482eb-b711-4acb-9e0e-b489ccf56e61" />
<img width="326" alt="Image" src="https://github.com/user-attachments/assets/76ef3e2a-8964-4632-baac-3ca643c54ec9" />


- In the **“Ansible_Control”** Node switch to the created “ansible” user.
  ```sh
  sudo su ansible
  ```

- Try reaching the worker nodes one after the other from the Control Node. A password will be required.
  ```sh
  ssh ansible@private_ip
  ```

You will have something like the above picture

<img width="460" alt="Image" src="https://github.com/user-attachments/assets/7955ce09-4a72-4ad0-ae33-4d53784885ba" />

**5.** Create a Keypair in the Control node and copy it in the Worker Node so that it won't require a password when the Control node “SSH” into worker nodes. This will facilitate the configuration of the worker nodes or running the playbook via the control node since it won’t be hindered or blocked by asking for a password which we might not be available at every moment to pass that in.


- While in the control Node, ensure you are at the home directory of the ansible user or run the following lines to do so.
  ```sh
  sudo su ansible 
  cd ~
  ```

- Generate key pair and give required permissions to the generated keypair file(.ssh) so it can be copied to the worker nodes. After the code generates a key-pair, you can hit **“Enter”** till the end.
  ```sh
  ssh-keygen -t rsa # generate keypair
  sudo chmod 700 /home/ansible/.ssh # gives permission to .ssh file to copy it to worker_node
  ```

<img width="458" alt="Image" src="https://github.com/user-attachments/assets/ff10ee3a-afa6-4390-add4-3b66647b5d96" />

- Copy Keypair to each worker node.
  ```sh
  ssh-copy-id ansible@private_ip
  ```

<img width="451" alt="Image" src="https://github.com/user-attachments/assets/b41c6f57-27d8-4160-8506-3c0389eab4fc" />

- Once done, try to “SSH” again to your worker nodes and you won’t be prompted to pass a password.

<img width="450" alt="Image" src="https://github.com/user-attachments/assets/b1aafde5-f2fb-49a0-96b7-1774511d07f7" />

**6.** Installing Ansible and Docker packages.

- Install the ansible package in the Control Node.
  ```sh
  sudo amazon-linux-extras install ansible2 -y # install ansible in control node
  ansible --version # check if ansible is installed
  ```

<img width="499" alt="docker and ansible install" src="https://github.com/user-attachments/assets/f67918cc-53b4-46b1-be09-6b2006275854" />

- SSH into each worker node through the Control and install the Docker engine.
  ```sh
  sudo amazon-linux-extras install docker -y # install docker
  sudo service docker start # start docker
  sudo systemctl enable docker # enable docker
  sudo systemctl status docker # ensure docker runs
  sudo docker run hello-world # verify if docker is install
  ```

**7.** Update the Inventory file in Control Host so that Ansible knows identified the nodes it communicates with when the playbook is run
`ssh ansible@<control-node-ip>`, `vi /etc/ansible/hosts` 
- In the home directory of the “ansible” user, run this code.
  ```sh
  cd /etc/ansible/
  sudo vi hosts
  ```

- At example 2 section uncomment **"[webservers]"** then pass the private Ip addresses of the Worker nodes beneath.

<img width="690" alt="Image" src="https://github.com/user-attachments/assets/a38e944d-94b5-40cc-96bf-5eabafd4fee3" />

**8.** Creating directories and files in the Control Node for the docker-compose file and ansible-playbook

- To create the Docker compose file, run the following lines.
  ```sh
  mkdir ~/jira-docker # make a directory called jira-docker in home directory
  cd ~/jira-docker # navigate to the directory
  sudo nano docker-compose.yml # create a file called docker-compose.yml and paste in docker compose code
  ```

- Paste in the docker-compose code in the **docker-compose.yml** file
  

- To create an Ansible Playbook file, run the following lines.
  ```sh
  mkdir ~/ansible-playbooks # make a directory called sndible-playbook in home directory
  cd ~/ansible-playbooks # navigate to the directory
  sudo nano deploy_jira.yml # create a file called deploy_jira.yml and paste in ansible playbook file
  ```

- Paste in this ansible-playbook code in the **deploy_jira.yml** file

**9.** Run playbook

- Ensure you are in the “cd /ansible_playbooks/deploy_jira.yml”, then run the playbook.
  ```sh
  ansible-playbook deploy_jira.yml
  ```

  output:

<img width="484" alt="Image" src="https://github.com/user-attachments/assets/67e71da2-ba08-4513-ab1f-b3d467274bda" />

**10.** Creating a Load balancer to route traffic to the Worker Nodes.

- Create a Load balancer Target Group. The target group protocol should be HTTP and port 8080 since in the docker-compose file, we created containers that are open on port 8080 and are mapped on port 8080 of worker nodes. Hit Next.

<img width="713" alt="Image" src="https://github.com/user-attachments/assets/2ab517ca-664b-4006-a26e-09e27cb621ce" />

- Select the worker nodes and “include as pending below” then create the target group.

<img width="942" alt="Image" src="https://github.com/user-attachments/assets/979a2fe8-9396-4739-a08c-9dc6103b147c" />
Create the Load balancer.

- Choose Application Load balancer

<img width="596" alt="Image" src="https://github.com/user-attachments/assets/e5997153-692f-431b-ae97-749b57cd5e52" />

- Name the Load balancer and at the level of the Network setting, choose the desired VPC and all subnets.
- Create a Security group that allows HTTPS and HTTPS traffic from anywhere.

<img width="855" alt="Image" src="https://github.com/user-attachments/assets/f87112ac-59be-4e97-9f19-4a7f0d2ec77d" />

- At the level of Listeners and Routing, choose an HTTP protocol listening from port 80, and pass the created target group. Then create the load balancer.

<img width="722" alt="Image" src="https://github.com/user-attachments/assets/5856a1d3-7a8a-43ee-a4bc-b8f2eb7fa7e7" />

**11.** Testing

- Edit the inbound rule of the worker nodes security group to accept HTTP and HTTPS traffic from the Load balancer security group.

<img width="898" alt="Image" src="https://github.com/user-attachments/assets/f7bee6f6-0df2-4a34-980a-854f3876dcb8" />

- Copy the DNS of the load balancer and paste it on the Browser, you should see this.

![image-20240717-205519](https://github.com/user-attachments/assets/362693bc-66aa-4eaf-bc6c-8989ff77bc35)



  
