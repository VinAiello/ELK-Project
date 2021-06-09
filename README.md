# ELK-Project
## The diagram below is a depiction of the network we created in this repository.

(playbook file)
* Included in this YAML file is the topology of the ELK stack we have set up
* In addition it describes the access policies established, and the overall configuration of our ELK stack.

## Topology of our Network
In this project, we created a cloud hosted network via Microsoft Azure, where also established a load balancer between VM's, and a DVWA that was powered by these VM's, along with containers delivered through Docker.
* Our load balancer helps ensure the availability of our DVWA and the flow of traffic through our VM's.
* We also set up a Jump Box, which adds more security by being the central point of entry into the cloud network we created. 


Integrating the ELK server into our network provides very useful monitoring and analystics for the infrastructure we have put in place.
* We installed Filebeat for its capability to help log data and send it to Logstash and Elasticsearch, the E and the L in our ELK Stack.
* Metricbeat was installed next for it's ability to gather and log metrics from the services running and is able to provide Logstash and Elasticseaech with the statistics it's collected.

| Name  | Function   | IP Address  | Operating System  |
|---|---|---|---|
|  JumpBox Provisioner | Gateway   | 10.0.0.6  |  Linux |
| Web 1  |  Container | 10.0.0.7  | Linux  |
| Web 2  |  Container | 10.0.0.8  | Linux |
|Elk Server | ELK Stack | 10.1.0.4 | Linux |

## Access Policies
We set up our virtual machines in a way in which only the Jumpbox would be allowed connections to the internet, so our Web 1 and Web 2 machines are more secure.
* Our containers within our Web 1 and Web 2 machines are only accessible by our Jumpbox host machine.
* Our ELK Server is only accessible by our containers.

| Name  | Publicly Accessible | Allowed IP Addresses  |
|---|---|---|
| Jumpbox Provisioner  | Yes  | 10.0.0.6  |
| Web 1  | No  | 10.0.0.7  |
| Web 2 | No  | 10.0.0.8  |
| ELK Server | No | 10.1.0.4 |

## Starting Our Virtual Network
We began our Virtual Networks by creating a Resource Group within Azure, in my example I named it RedTeamRG.
* This acts as the space in which we will establish our Virtual Networks, and deploy the rest of our resources within Azure.

Next we created our actual Virtual Network within our RedTeamRG group, which I named RedTeamNet.
* Placing a Virtual Network within our resource group, allows us the option to either make another separate network within the same resource group, or having another resource group completely to establish for a different purpose or department within an organization.
* When we are setting up RedTeamNet, we want to make sure we are choosing the option to assign it an IPv4 address space, in our case it was 10.0.0.0/16.
* We also opted to keep the Security options disabled for this project, to ensure we did not deplete too much of our funds, those these options will usually be important settings to enable in many regular networks.
* To finish creating our virtual network, we need to make sure we are clicking "Review + Create" and then wait for the network to be deployed within our resource group.
* You should be able to see RedTeamNet within our RedTeamRG now.

## Establishing a Security Group
Security groups within cloud based networks, are similar to firewalls within physical networks. They will allow us to set rules and regulate what kind of traffic makes it through to our network and resources.
* In Azure, we will look up the option Securty Groups and create a new one which we will name RedTeam-SG.
* Within this group, like mentioned, we will be able to start setting rules to specify how we want traffic to be able to transfer in and out.

Next we want to create our first rule, which will be used to block all traffic to our virtual network. This keep our netowrk secure until we are finished properly configuring it to our standards.
* We want to select our new RedTeam-SG and select the option to create an inbound rule. 
* We can select "Any" for the source to block all traffic, and then we want to keep the wildcard in the "Source Port Ranges" portion. The port will be random so there will be no need to change it.
* In the "Destination" option, we will select "Any" again and it will block all destination ports. We want to keep all of our destination ports blocked as well, so we can keep the wildcard in that portion as well.
* For the "Action" portion, we can select "Block" because we want this rule to block traffic. Azure follows rules from lowest number to highest, with the highest priority being 4096 for custom rules. We will set this rule as 4096 for priority, as we want our following rules that we create to be able supersede this rule. This will be named our Default-Deny rule.

## Our Virtual Machines
Our first VM that we will set up will be our JumpBox and it will be our gateway to the other virtual machines we will set up later.
* First we want to make sure we are using an SSH key to access these machines, as it will be a more secure way to access them than it would be just using a password.
###### If we don't already have a key pair created, we can run the command "ssh-keygen" in our terminal.
###### We want to specify this folder to have the keys placed in, "/.ssh/id_rsa/".
* Now in Azure, we want to search for the "Virtual Machine" option and click "Add" so that we can start our first machine.
* We want to make sure we are placing this VM in our previously made resource group, so in that option make sure we specify our RedTeamRG. 
* We want to name our first VM something that will make it easy to identify, so I have named mine "Jump-Box-Provisioner". We also want to set the region of this VM to the same as we did for our resource group.
* For the "Image" selection, we will select the Ubuntu Server 18.04 option, and for the "Size" we can use the "Standard - B1s" which will have 1 CPU and 1 RAM, enough power for what we need in this VM.
* Next, down where it says "Authentication Type" we want o make sure we select "SSH Key" since this is how we will validating the connection and accessing our cloud server. We can create a username to use to login in with, in my case I used RedAdmin, and then in the window we paste our public key we generated before.
 ###### We get this by going back to our terminal and running the command, cat ~/.ssh./id_rsa.pub, which should display our public key.
* We can ignore the port settings, as they will be configured by our security group.
* In the Networking tab, we want to select the virtual network we created, RedTeamNet, which will assign our VM a Private IP address, and we will keep the Public IP as default.
* We want to make sure we are selecting the "Advanced" option in the area that asks for "NIC ntwork security group" and select the security group we created, RedTeam-SG, for our VM to be a part of.
* We can keep the rest of the settings as default, and finish with Review and Create again to finalize the VM. 
* For our other 2 VM's we will make, named Web-1 & Web-2, we will be doing the same steps for each except for a couple variations:
1. In the drop down for "Availability Options" we want to select "Availability Set" this time, then click "Create New". We will keep the default options and then name is RedTeamAS.
2. For the "Image" option, we will be using a different selection, this time it should be **Standard-B1ms** which will be 1 CPU and 2 RAM for these VM's.
3. We want to make sure we do not select for these 2 new VM's to have public IP addresses.

## Configuring the JumpBox
We want to configure our Jumpbox now by adding some rules to the security group we created. 1st we wanted to make sure that we allowed our IP address to be able to SSH to our Jumpbox VM.
* We created an inbound rule selecting IP Addresses as a source, the source IP being our public IP, any source port ranges, selecting IP as the destination, destination port range should be port 22 for SSH connection, and the destination IP being the private IP of our Jumpbox.
* Reminder, we need to set the priority lower than our default deny rule, lower than 4096, and this rule should allow all traffic from our source IP.
After you complete this rule, you can test the rule by powering up you Jumpbox VM, and in your terminal running the command *ssh admin-username@VM-public-IP*
###### The VM's public IP should be listed in Azure when you select the VM.
###### Our public IP might change depending on wi-fi settings or what wi-fi networks we are connected to, so if you have issues down the line connecting, you may want to double check this and see if your rule may need editing.

## Setting Up Containers
We want to configure our Jumpbox further by allowing it to run containers from Docker and install these containers. 
###### Make sure to install docker in your terminal if it is not already, run *sudo apt install docker.io*. Then verify it is running with *sudo systemctl status docker*. If it is not running, use *sudo systemctl start docker*.
* Now we want to use the Ansible container so we will run *sudo docker pull cyberxsecurity/ansible* to locate and pull that container from docker.
* Now to launch Ansible we use *docker run -ti cyberxsecurity/ansible:latest bash*. We can exit after we run this command.
We now want to make sure that our Jumpbox can SSH in our virtual network we created, so we will create a new inbound rule in our virtual network.
* The source should be IP Addresses again, and we will input the private IP of our Jumpbox, we can once again find this by selecting on our Jumpbox VM in Azure.
* Source port ranges will be Any again, the destination will VirtualNetwork this time, and destination port will be 22 for SSH again.
* Protocol will be any, action will be allow to allow traffic, make priority another lower number than our Default Deny rule, and create another name for this rule to make it easy to tell what it is doing for our cloud network.

## Setting Up Provisioners
We will be jumping back into our terminal to start setting up our provisioners.
* We want to run *sudo docker container list -a* to list our our ansible containers. We will run *docker run -it cyberxsecurity/ansible /bin/bash* next to start it up and connect to it. This should drop us into another shell.
* We want to next create another SSH key pair to allow secure access. In our new container shell run the *ssh-keygen* command again, and then *ls .ssh/* to list them out.
* We will cat out our public key again using *cat .ssh/id_rsa.pub* and we want to copy this key again.
* Back in Azure, we will select our Web-1 VM and we want to make sure we are using these keys to verify access when we SSH. In the Reset password tab, we select Reset SSH public key instead, use our username, RedAdmin in our case, and paste the public key.
* We can test the connection by pinging Web-1's private IP, and then SSH to the private IP. Make sure to exit afterwards.
Next we need to do some further configuring to our Ansible containers.
* Navigate to the ansible file with *ls /etc/ansible*. We will open the hosts file with nano, and Web-1's private IP to the file headers.
###### Uncomment the *webservers* header and add it there, along with this python line, *ansible_python_interpreter=/usr/bin/python3*
We now want to make sure our admin account will be used for SSH connections.
* Navigate to the ansible.cfg file and add your admin account name to the *remote_user =* portion. 

## DVWA
We now wanto establish DVWA web app on our VM's.
* If we are not already connected to our Ansible container, make sure we are connected to our Jumpbox and then use *docker container list -a* to list out our containers. We will connect to the ansible container by running *docker start [name of container]* to start it up, and then *docker attach [name of container]* to connect. 
* We will create a playbook within the ansible file, to configure within, and it will be a YAML file.
---
- name: Config Web VM with Docker
    hosts: web
    become: true
    tasks:
