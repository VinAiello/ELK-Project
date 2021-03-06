# ELK-Project
## The diagram below is a depiction of the network we created in this repository.

![ELK Project Diagram](https://user-images.githubusercontent.com/78758609/121825707-1b84f080-cc71-11eb-890e-6eca00663a03.PNG)

## Topology of our Network
In this project, we created a cloud hosted network via Microsoft Azure, where also established a load balancer between VM's, and a DVWA that was powered by these VM's, along with containers delivered through Docker.
* Our load balancer helps ensure the availability of our DVWA and the flow of traffic through our VM's.
* We also set up a Jump Box, which adds more security by being the central point of entry into the cloud network we created. 


Integrating the ELK server into our network provides very useful monitoring and analystics for the infrastructure we have put in place.
* We installed Filebeat for its capability to help log data and send it to Logstash and Elasticsearch, the E and the L in our ELK Stack.
* Metricbeat was installed next for it's ability to gather and log metrics from the services running and is able to provide Logstash and Elasticseaech with the statistics it's collected.

| Name  | Function   | IP Address  | Operating System  |
|---|---|---|---|
|  JumpBox | Gateway VM   | 10.0.0.6  |  Linux |
| Web 1  |  Container | 10.0.0.7  | Linux  |
| Web 2  |  Container | 10.0.0.8  | Linux |
|Elk Server | ELK Stack | 10.1.0.4 | Linux |

## Access Policies
We set up our virtual machines in a way in which only the Jumpbox would be allowed connections to the internet, so our Web 1 and Web 2 machines are more secure.
* Our containers hosted on our Web 1 and Web 2 machines are only accessible by our Jumpbox host machine.
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
* We want to name our first VM something that will make it easy to identify, so I have named mine "Jump-Box". We also want to set the region of this VM to the same as we did for our resource group.
* For the "Image" selection, we will select the Ubuntu Server 18.04 option, and for the "Size" we can use the "Standard - B1s" which will have 1 CPU and 1 RAM, enough power for what we need in this VM. This is basically configuring the virtual VM server using a copy of a certain version type of server.
* Next, down where it says "Authentication Type" we want to make sure we select "SSH Key" since this is how we will validating the connection and accessing our cloud server. We can create a username to use to login in with, in my case I used RedAdmin, and then in the window we paste our public key we generated before.
 ###### We get this by going back to our terminal and running the command, cat ~/.ssh./id_rsa.pub, which should display our public key.
* We can ignore the port settings, as they will be configured by our security group.
* In the Networking tab, we want to select the virtual network we created, RedTeamNet, which will assign our VM a Private IP address, and we will keep the Public IP as default.
* We want to make sure we are selecting the "Advanced" option in the area that asks for "NIC network security group" and select the security group we created, RedTeam-SG, for our VM to be a part of.
* We can keep the rest of the settings as default, and finish with Review and Create again to finalize the VM. 
* For our other 2 VM's we will make, named Web-1 & Web-2, we will be doing the same steps for each except for a couple variations:
1. In the drop down for "Availability Options" we want to select "Availability Set" this time, then click "Create New". We will keep the default options and then name is RedTeamAS.
2. For the "Image" option, we will be using a different selection, this time it should be **Standard-B1ms** which will be 1 CPU and 2 RAM for these VM's.
3. We want to make sure we do not select for these 2 new VM's to have public IP addresses.

## Configuring the JumpBox
We want to configure our Jumpbox now by adding some rules to the security group we created. 1st we wanted to make sure that we allowed our IP address to be able to SSH to our Jumpbox VM.
* We created an inbound rule selecting IP Addresses as a source, the source IP being our public IP, any source port ranges, selecting IP as the destination, destination port range should be port 22 for SSH connection, and the destination IP being the private IP of our Jumpbox.
* Reminder, we need to set the priority lower than our default deny rule, lower than 4096, and this rule should allow all traffic from our source IP.
After you complete this rule, you can test the rule by powering up you Jumpbox VM, and in your terminal running the command **ssh admin-username@VM-public-IP**
###### The VM's public IP should be listed in Azure when you select the VM.
###### Our public IP might change depending on wi-fi settings or what wi-fi networks we are connected to, so if you have issues down the line connecting, you may want to double check this and see if your rule may need editing.

## Setting Up Containers
We want to configure our Jumpbox further by allowing it to run containers from Docker, which is a container tool that will provide images or copies of certain containers and allow us to install these containers on our VM's connected to the Jumpbox. 
###### Make sure to install Docker in your terminal if it is not already, run "*sudo apt install docker.io*". Then verify it is running with "*sudo systemctl status docker*". If it is not running, use "*sudo systemctl start docker*".
* Now we want to use the Ansible provisioner tool so we will run **sudo docker pull cyberxsecurity/ansible** to locate and pull it from Docker.
###### Ansible is a provisioner tool which will allowus to use different IaC, such as the containers we will pull from Docker. It will allow us to quickly configure our containers so they are ready for use and specific functions.
* Now to launch Ansible we use **docker run -ti cyberxsecurity/ansible:latest bash**. We can exit after we run this command.
We now want to make sure that our Jumpbox can SSH in our virtual network we created, so we will create a new inbound rule in our virtual network.
* The source should be IP Addresses again, and we will input the private IP of our Jumpbox, we can once again find this by selecting on our Jumpbox VM in Azure.
* Source port ranges will be Any again, the destination will VirtualNetwork this time, and destination port will be 22 for SSH again.
* Protocol will be any, action will be allow to allow traffic, make priority another lower number than our Default Deny rule, and create another name for this rule to make it easy to tell what it is doing for our cloud network.

## Setting Up Provisioners
We will be jumping back into our terminal to start setting up our provisioners in the form of YAML files. This will allow us to do configuration and installations much more efficiently down the line, but allowing us to just run a file to do so. 
* While in our Jumpbox shell, we want to run **sudo docker container list -a** to list our our Ansible containers. We will run **docker run -it cyberxsecurity/ansible /bin/bash** next to start it up and connect to it. This should drop us into another shell, our container shell within our Web-1 or Web-2 server.
* We want to next create another SSH key pair to allow secure access and essentially let our containers run off of Web-1 and Web-2. In our new container shell run the **ssh-keygen** command again, and then **ls .ssh/** to list them out.
* We will cat out our public key again using **cat .ssh/id_rsa.pub** and we want to copy this key again.
* Back in Azure, we will select our Web-1 VM and we want to make sure we are using these keys to verify access when we SSH. In the Reset password tab, we select Reset SSH public key instead, use our username, RedAdmin in our case, and paste the public key.
* We can test the connection by pinging Web-1's private IP, and then SSH to the private IP. Make sure to exit afterwards.
* Repeat these steps for Web-2 as well.
Next we need to do some further configuring to our Ansible containers.
* Navigate to the Ansible file with **ls /etc/ansible**. We will open the hosts file with nano, and Web-1's private IP to the file headers.
###### Uncomment the "*webservers*" header and add it there, along with this python line, "*ansible_python_interpreter=/usr/bin/python3*"
We now want to make sure our admin account will be used for SSH connections.
* Navigate to the ansible.cfg file and add your admin account name to the **remote_user =** portion. 
* You should now be able to run **ansible-playbook [name of your playbook YAML file]**.

## DVWA
We now want to establish a DVWA web app on our VM's.
* If we are not already connected to our Ansible container, make sure we are connected to our Jumpbox and then use **docker container list -a** to list out our containers. We will connect to the ansible container by running **docker start [name of container]** to start it up, and then **docker attach [name of container]** to connect. 
* We will create a playbook within the Ansible file, to configure within, and it will be a YAML file.
* We want our full playbook to look like the image below.

 ![image](https://user-images.githubusercontent.com/78758609/121428185-c1afbe00-c932-11eb-9ed8-0cb662c8ffc7.png)

###### This will make sure it downloads python3-pip, docker.io, the cyberxsecurity/dvwa docker container, and it will make sure that when you restart your VM that the container will restart too.

* We can test everything is running correctly by trying to SSH to container and running the command **curl localhost/setup.php**. 

## Load Balancers
The next step is getting our load balancers set up to help support our DVWA.
* In Azure there will be an option to create Load Balancers, and we want to make sure to connect it to our same resource group, RedTeamRG, and we want it in the same region again.
* For "**Type**" we'll select Static, and for "**SKU**" we will select Basic.
* We want our load balancer to have it's own IP address so we will select create new, and we can give the IP address name the same name we have given the load balancer to keep it simple.
* "**Assignment**" will be Static and we will not need an IPv6 address for it.
* Atfer the load balancer has been reviewed and created, we will want to help it monitor that our VM's are able to receive traffic, so we will add a health probe to it.
* Next we will add a backend pool to connect our VM servers to the load balancer, specifying that they will have the traffic balanced between them. We want to make sure it is connected to the virtual network we created previously and that we are specifying the private IP's of our Web-1 and Web-2 VM's.

## Further Configuration
We want to make a few more security measures to ensure proper access is allowed now, while still ensuring the security of our network.
* We will create a load balancer rule to allow TCP between the laod balancer and our security group, this will be through port 80.
* In addition, we want to add a new security group rule for traffic from the internet to be able to reach our virtual network through port 80. We want to make sure it is from our public IP to the VirtualNetwork when we are creating the rule.
* We can now delete our **Default Deny** rule now that we have put in enough rules and configuration to provide sufficient access and control. 
* We can now test that our DVWA is accessible on the internet by running "**http://[load balancer public IP]/setup.php**" in a browser.
* We can then test that our load balancer and backened pool are working by shutting down our Web-1 VM and trying to load the link, and then again having Web-1 back on and shutting down our Web-2 VM.
###### When one shuts down, the other VM should be able to pick up the slack since our DVWA is hosted on both machines. With both being placed in our backend pool, it should prompt the load balancer to delegate traffic between the 2 machines evenly or through 1 entirely if the other is down or experiencing some sort of issues.

## Starting Our ELK Stack
We want to implement the ELK stack into our cloud network now so that we can have access to logging of data, analytics, and monitoring of our network. We will start by creating another virtual network to host the stack on.
###### ELK stack includes Elasticsearch, Logstash, and Kibana as the tools used.
* This new virtual network will be within our same resource group, and this time we are going to use a different region to place it in, to help keep it separate from our other resouces.
* If we check in the "**IP Addresses**" tab, we should double check that Azure made a different network space for us with a different assigned IP range than for our other network.
* After the ELK virtual network has been created, we will set up a peer connection between our virtual networks in the "**Peerings**" tab, which will allow traffic to pass between both networks.

## Our ELK VM
After setting up the peering between the networks, we want to create a new VM that will host our ELK tools.
* Back in our terminal, we want to SSH into our Jumpbox, then start and connect to our Ansible container.
* We then want to get our public key again and copy it for later.
* In Azure we will create a new VM for our ELK Stack, and make sure to add this key in the configuration like before.
* This VM should be in the same resource group again, should have a size of at least 4gb of RAM when selecting the Image we want to use, should have a public IP, and be connected to our new ELK virtual network.
* We will also be configuring this VM with SSH key authenticating, so once again paste the public key we generated and place it in the required area, and we will be allowing port 22 for SSH again.

## Our ELK Container
Now we want to add our VM to our "**Hosts**" file like before, adding the line "**[ELK VM private IP] ansible_python_interpreter=/usr/bin/python3**" and create a new provisioner playbook to configure the VM for us.
* When creating our new YAML playbook file, we want to use the same confirguation we did for our last playbook, downloading docker, docker.io, and python3-pip but with a couple additions. 

![Elk docker header](https://user-images.githubusercontent.com/78758609/121700423-824db280-ca95-11eb-892a-f566aad99a5e.png)
![Docker increase size snippet](https://user-images.githubusercontent.com/78758609/121700445-88dc2a00-ca95-11eb-98b4-361ea9e6fa91.png)
* In the first snippet above, you want to make sure to substitute the remote username with whatever username you have been using up to this point. The second snippet is used to help increase the memory since our ELK container will require more.
* We also want to make sure we making an edit in the Ansible config file, where it specifies remote user we want to add the username we've been using again.

![Ansible config fiel snippet](https://user-images.githubusercontent.com/78758609/121706226-09515980-ca9b-11eb-8c5e-ead8a26849ff.png)
* After we run the playbook, we can verify everything is working by SSHing from our container to the ELK VM.

## ELK Access
We now want to make some changes to control the access to our ELK server.
* After the creation of our ELK VM, there should have a security group auto-generated for it. This is where we will be creating access rules.
###### It should be *nameofyourVM-nsg*. 
* Our ELK stack will run over port 5601 so we want to the incoming rule that we are making to allow TCP traffic from our public IP through that port.
* Source should be from our public IP address to the VirtualNetwork, with destination port being 5601.
* We can now verify our access by going to the website for our ELK stack, the Kibana tool specifically as it is the visualization portion of the stack, http://[your.ELK-VM.External.IP]:5601/app/kibana.

## Filebeat
We want to install and add Filebeat to help us with filtering and logging the data that we collect in our ELK Stack. We will be downloading this on our DVWA container.
* In our browser, at our ELK VM's IP address, we will go to the "**Add Log Data**" option in Kibana and then click on the "**System Logs**" option. We want to choose the "**DEB**" which will guide you to correctly install Filebeat on Linux.
After following the guide in Kibana, we want to now create a Filebeat configuration file, along with a playbook for Filebeat.
* We will SSH from our Ansible container into our ELK server VM and we can actually run this command to get a template for the config file, "**curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat >> /etc/ansible/filebeat-config.yml**".
* We will go into this config file now and go specifically to line #1106, which should say "**hosts**", and we want to type in, in brackets, **["ELK VM's Private IP address:9200"]**.
* At line #1806, similarly after hosts, input **["ELK VM's PrivateIP address:5601"]**.
We now want to create another playbook, which we can name **filebeat-playbook** which will be used to instal FIlebeat and also copy the config file we just made.
* We can now download the .deb file from this link, https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb , and run it using this command "**dpkg -i filebeat-7.4.0-amd64.deb**". We want to now make a copy of the config file and make sure it is placed in our Filebeat directory.
* Our new filebeat playbook YAML should look like below.

![filebeat config](https://user-images.githubusercontent.com/78758609/121739856-d7072280-cac1-11eb-8b3b-1e1f6e53f02d.png)
* We can now run "**ansible-playbook filebeat-playbook.yml**" to run the playbook, and then you can verify in Kibana that Filebeat is working correctly by clicking the "**Check Data**" button at the bottom of the DEB page.

## Metricbeat
Metricbeat will hep us collect metrics and analytics for our network and communicates them with the Elasticsearch and Logstash parts of our ELK Stack, and also allow us to parse through specific information gathered.
###### The steps for this will actually be identical to the steps for installing Filebeat with a couple variations listed below.
* In our Kibana webpage, instead of going to "Add Log Data". we will go to where it says "**Add Metric Data**" and then navigate to "**Docker Metrics**".
###### This will bring us to a set of steps similar to the Filebeat steps.
* When you get to the portion to download and edit the Filebeat configuration file, we want to use this link instead **https://gist.githubusercontent.com/slape/58541585cc1886d2e26cd8be557ce04c/raw/0ce2c7e744c54513616966affb5e9d96f5e12f73/metricbeat**.

## Conclusion
in this project we successfully established the following:
1. Creating a cloud hosted network.
2. Creating a resource group that we were able to establish the rest of our resources within.
3. Creating a virtual network within out resource group.
4. Create a security group where we established access policies for the virtual machines we later created.
5. Establishing a Jumpbox virtual machine that we used to funnel traffic and access through.
6. Establish 2 additional VM's, later used for our DVWA and containers.
7. Set up load balancers to help with accessibility and traffic distribution.
8. Creating a series of provisioners in the form of YAML files, used for efficient configuration.
9. Establishing another virtual network for the ELK stack we would implement next.
10. Establish another VM used as a server for our Elk stack.
11. Successfully install and configure our ELK stack for proper implementation within our Azure network.
12. Install and implemented Filebeat and Metricbeat to be used for analytics, logging, and monitoring.
