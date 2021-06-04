# ELK-Project
## The diagram below is a depiction of the network we created in this repository.

(playbook file)
* Included in this YAML file is the topology of the ELK stack we have set up
* In addition it describes the access policies established, and the overall configuration of our ELK stack.

# Topology of our Network
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

# Access Policies
We set up our virtual machines in a way in which only the Jumpbox would be allowed connections to the internet, so our Web 1 and Web 2 machines are more secure.
* Our containers within our Web 1 and Web 2 machines are only accessible by our Jumpbox host machine.
* Our ELK Server is only accessible by our containers.

| Name  | Publicly Accessible | Allowed IP Addresses  |
|---|---|---|
| Jumpbox Provisioner  | Yes  | 10.0.0.6  |
| Web 1  | No  | 10.0.0.7  |
| Web 2 | No  | 10.0.0.8  |
