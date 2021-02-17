## Automated ELK Stack Deployment

The files in this repository were used to configure the network shown below.

[Network Diagram](Network-Azure.drawio)

After beeing tested, these files were used to generate a live ELK deployment on an Microsoft Azure network, through a Jumpbox provisioner with an Ansible container. These files can be used to recreate the entire deployment pictured above or to install certain pieces of it, such as Filebeat.

[Docker Configuration file](Ansible/Config-Elk-w-Docker)

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored example of D*mn Vulnerable Web Application (DVWA). A load balancer was used to distribute web traffic across different servers, which logs can be monitored through an ELK monitoring stack within a separate virtual network (VNet).

Load balancing ensures that the application will be highly available, in addition to restricting inbond access to the network.
    An LB provides an external IP address (accessed by the internet) to a website. After receiving traffic, the LB distributes it across multiple servers, which can be expanded as the need arises, helping distribute traffic evenly among the servers to minimize the effects of possible Denial of Service (DoS) attacks. An LB has also a "health probe" that regularly checks the web servers' functionality, before sending traffic to them - issues with a server will be reported and traffic to it will be halted. Although the system could still become overwhelmed with traffic, it is much more effective than a single server running the website.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file names and watch system metrics through monitors like Filebeat and Metricbeat. These monitors automate functions that tends to be too tedius and innefective, if done mannually by a system administrator. 
    The Filebeat monitors the logs in the specified location, detects changes to the filesystem, collects logs, and forwards them to Elastisearch or Logstash for indexing.
    The Metricbeat detects changes in the system, such as CPU usage and SSH login attempts.

The configuration details of each machine is as follows:


| Name                 | Function                | Internal IP | Operating System |
|----------------------|-------------------------|-------------|------------------|
| Jump-Box-Provisioner | Gateway (Ansible)       | 10.0.0.4    | Linux            |
| Web-1                | Server                  | 10.0.0.5    | Linux            |
| Web-2                | Server                  | 10.0.0.6    | Linux            |
| Web-3                | Server                  | 10.0.0.8    | Linux            |
| VM-4                 | Server (ELK Monitoring) | 10.1.0.4    | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box-Provisioner Gateway machine can accept connections from the Internet. Access to this machine is only allowed from the pre-designated public IP address (CLIENT/User) on a SSH connection (using key pair) through port 22.

Machines within the network can only be accessed by peer servers. Only the Jump-Box-Provisioner (IP 104.45.223.53) can connect through SSH to the website servers (Web-1, Web-2, Web-3) and the Elk server (VM-4). 

All accesses are configured through separate Security Groups (Purple-Team-Net for Jump-Box, Load Balancer, Web servers; and VM-4-nsg for the Elk server VM-4)

A summary of the access policies in place can be found in the table below.

| Name                 | Function                | Public  Access? | IP Address Allowed |
|----------------------|-------------------------|-----------------|--------------------|
| Jump-Box-Provisioner | Gateway (Ansible)       | YES             | Pre-designated     |
| Web-1                | Server                  | NO              | 10.0.0.4           |
| Web-2                | Server                  | NO              | 10.0.0.4           |
| Web-3                | Server                  | NO              | 10.0.0.4           |
| VM-4                 | Server (ELK Monitoring) | NO              | 10.0.0.4           |

### Elk Configuration

A Docker-managed container running Ansible was used to automate configuration of the ELK machine (VM-4). No configuration was performed manually, which is advantageous because using Ansible allowed installation, updates, and additions of programs to the network, using "playbooks" created once instead of manually doing those tasks for each machine. Another advantage is that if one container run into problems or attacks, the system administrator can kill and regenerate it as needed, without downtime for the active servers. Containers also reduce operational costs by eliminating a large amount of file and CPU overhead. 

The playbook implements the following tasks:
  * Installing Docker on all network machines to receive and install containers
  Installing Ansible on the Jump-Box-Provisioner to distribute containers to other machines in the network
  Using Ansible playbooks to install ELK stack container on the VM-4 (ELK Server) and "beats" containers on the webservers (Web-1 through -3).

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

[Screenshot after ELK configuration](Images/Launching ELK.png)

### Target Machines & Beats
The VM-4 (ELK server) is configured to monitor the following machines:
| Name  | IP Address |
|-------|------------|
| Web-1 | 10.0.0.5   |
| Web-2 | 10.0.0.6   |
| Web-3 | 10.0.0.8   |

The following "beats" were installed on the webserver machines:
  *Filebeat
  Metricbeat
  
These Beats allow us to collect the following information from each machine:
  * Filebeat is designed to ship log files. It helps keep things simple by offering a lightweight way (low memory footprint) to forward and centralize logs and files, making the use of SSH unnecessary when you have a number of servers, virtual machines, and containers that generate logs. Other benefits of Filebeat are the ability to handle large bulks of data, the support of encryption, and deal efficiently with backpressure. In this case, Filebeat is a logging agent installed on VM-4 (ELK server) to monitor activities in three servers (Web-1 through -3), monitoring its logs, tailing them, and forwarding the data to VM-4 (ELK server) that can be accessed through the monitoring page, Kibana dashboard (specified in the Filebeat configuration file). It tracks system logs, sudo commands, SSH logins, new users and groups, etc.
  Metricbeat is a lightweight shipper installed on our servers to periodically collect metrics from the operating system and from services running on the servers (Web-1 through -3). It detects changes in system metrics, like CPU usage and memory, loads, inbound and outbound traffic, cache, API segment requests, average response, requests rates, etc.

### Configuring Files and Using the Playbooks
You will need to configure files to set Filebeat and Metricbeat services.
1. Open the terminal and access your jump-box. In this case, we SSH to our Jump-Box-Provisioner.
2. Start your Ansible container. In this case, we run `sudo docker start elastic_blackburn && sudo docker attach elastic_blackburn`
In order to use the playbooks, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned, SSH into the control node and follow the steps below.
1. Make sure the ELK server is up and running. 
2. SSH into the control node (in this case, the Jump-Box-Provisioner, then into the Ansible container, elastic_blackburn)

3. Copy the [Filebeat Playbook] /Filebeat/Filebeat-Playbook.yml file to /etc/ansible/roles directory. Note that in this case, we created a "roles" subfolder to hold playbooks.
4. Update the "hosts" file to include the internal/private IP addresses of servers. This will make Ansible run the playbook on the specified machines. In this case, we assigned each server to their server groups as follows:
```
   [webservers]
    10.0.0.5
    10.0.0.6
    10.0.0.8
   [elkservers]
    10.1.0.4 
    ```
5. Run the playbook, and navigate to http://IP-address:5601/app/kibana to check that the installation worked as expected. In this case, the public IP address of our VM-4 ELK server.
6. Click on "Add Log Data", "Systems Log", scroll down and click on "Check Data" to pull data from the servers, then click on "System Logs Dashboard" to visualize the logs.
 
Continue on to install Metricbeat, changing the last steps as follows:
3. Copy the [Metricbeat Playbook]/Metricbeat/Metricbeat-Playbook.yml file to /etc/ansible/roles directory. 
Skip step 4 (because you already made that change prior to installing Filebeat
5. Navigate to http://IP-address:5601/app/kibana using the public IP address of the Elk Server (in our case, VM-4).
6. Click on "Add Metric Data", "Docker Metrics", scrolls down and click on "Check Data" to pull data from server, then click on "Docker Metric Dashboard" to visualize information. 

