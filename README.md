# HVirgen
Repository for Hector Virgen-Marquez

## Automated ELK Stack Deployment

The following repository is designed for the purpose of fullfilling a sucessful ELK Stack Server Deployment. The purpose of an ELK Stack Server is to create a network of virtual machines that can be monitored through Kibana with the assistance of filebeat and metricbeat services. 

Below is a diagram using Draw.io, the diagram represents a visualization of how the network is set up, how each connection is established, and what components are involved.

[Project 1 Diagram](https://github.com/hvirgenmarquez/HVirgen/blob/main/Diagrams/Project1.jpg)  <-- Click Here

In order to allow for an accurate deployment of the ELK Stack Server, I utilized Ansible's feature of playbooks to allow for more robost automation of the service installation. These playbooks allow me to install a service on a specified host group, in this case being the [webservers] group. The [webservers] group contains 3 virtual machines that are used for educational purposes for this project. 

For this activity I created several playbooks, but for the sake of exmaple, I have provided a preview of this [Filebeat Playbook](https://github.com/hvirgenmarquez/HVirgen/blob/main/Ansible/Filebeat-playbook.yml): 

ex:
```
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: sudo dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/filebeat/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: sudo filebeat modules enable system

  - name: setup filebeat
    command: sudo filebeat setup

  - name: start filebeat service
    command: sudo service filebeat start

  - name: Increase virtual memory
    command: sysctl -w vm.max_map_count=262144
```

## Outline of Report
- Network Hiearchy Structure
- Network Access Policies
- ELK Stack Setup and Configuation
  - Services being utilized
  - Machines Being Monitored
- How to Use the Ansible Build
- Playbooks used for this build

### Network Hiearchy Structure

In Project 1, I created a network structure designed to allow penetration in a virtual, safe environment. This allows for educational activities as to how a network is setup, basic functions to secure the network, and monitoring tools to detect penetration attempts. 

The first layer of security in the Network Hiearchy starts with a Load Balancer and a Jumpbox Provisioner Virtual Machine (VM).
- The Load Balancer allows for a first-layer of network protection by filtering network traffic into manageable data.
- The Jumpbox Provisioner VM allows for a second-layer of network protection by creating a dedicated access point to further access the rest of the network. 
  - To make the Jumpbox Provisioner VM more secure, I created an SSH-key on my personal computer. This SSH key is needed to access the Jumpbox Provisioner VM. 
  - Further, in order to add a 3rd layer of security, I used an ansible container. 
      - The Jumpbox Provisioner has 1 purpose: to connect to an ansible container, which is named "focused_poincare", to connect further down the Network Hiearchy. 
      - focused_poincare has a generated SSH-key that is programmed into the webservers and elk server, making it the only available method for access to the rest of the network setup. 

Further down the Network Hiearchy you will find the webservers and the Elk server. The webservers are named Web-1 and Web-2 (and for the purpose of this project, I also added Web-3 for educational purposes), and the Elk Server is Web-3. 
These servers are installed with the Filebeat and Metricbeat services to allow for monitoring using the Kibana interface:
-  Filebeat is a log-collection service, it collects detailed log data of activity detected within the hosts configured. 
   - Filebeat is great to find information about what may be happening to your server, or if someone is trying to break into your server. 
      - SSH login attempts are recorded using filebeat's log data service. This allows for a defensive supervision of the network activity in your Elk Stack setup. 
-  Metricbeat is another log-collection service, however unlike Filebeat, Metricbeat provides details on CPU activity, available memory, etc.,
   - Metricbeat is useful to see if any particiular stress if affecting your servers. By monitoring the CPU, memory, network activity, and more, Metricbeat allows for a more   technical analysis of your server's activities. 

Additonally, the network structure has a split between virtual networks. There are two different virtual networks (VN): Red-Team VN and ELK VN. 
-  These two different Virtual Networks have their own Network Security Groups (NSG), each NSG allows for a different configuration to allow the ELK Stack Configuration to properly work (will explain further down). 

Below is a detailed table visualizing the Virtual Machines used in this project:
| Name                | Function             | IP Address | OS    | Virtual Net  |
|---------------------|----------------------|------------|-------|--------------|
| Jumpbox Provisioner | Primary Access Point | 10.0.0.4   | Linux | Red-Team VN  |
| Web-1               | Webserver            | 10.0.0.5   | Linux | Red-Team VN  |
| Web-2               | Webserver            | 10.0.0.6   | Linux | Red-Team VN  |
| Web-3               | ELK Server           | 10.1.0.4   | Linux | ELK VN       |


### Network Access Policies

As mentioned above, there are two seperate Virtual Networks, each with their own respective Network Security Groups. 

#### Red-Team Virtual Network:
This VN contains the following virtual machines: 
| Name                | IP Address | OS    |
|---------------------|------------|-------|
| Jumpbox Provisioner | 10.0.0.4   | Linux |
| Web-1               | 10.0.0.6   | Linux |
| Web-2               | 10.0.0.6   | Linux |

These 3 virtual machines have whitelist security policies to create specific access points. 
-  Jumpbox Provisioner has a whitelisted IP address of my personal computer (IP ommitted for privacy). 
   - By whitelisting my personal computer, I have ensured that the only possible conneciton to the Jumpbox is through my personal computer. 
-  Web-1 and Web-2 have whitelisted the IP of the Jumpbox Provisioner (10.0.0.4).
   - This allows me to use the container "focused_poincare" to connect to Web-1 and Web-2. 
Putting forward these security policies ensures that are no additional access for these machines. 

#### ELK Virtual Network:
This VN contains the following virtual machines:
| Name  | IP Address | OS    |
|-------|------------|-------|
| Web-3 | 10.1.0.4   | Linux |

The purpose of creating a seperate virtual network for Web-3 is to allow the ELK Stack Server to function properly by giving my personal computer access to Port 5601. 
-  By giving access to port 5601, I am allowing my computer to access Web-3's kibana server (connected from the ELK VM). 
  - Web-3 has an ansible container installed named sebp/elk:761. 
    - This container is provided by the sebp company, it allows for access to a Kibana dashboard.
    - Kibana is accessed through a web browser through the following url: http://<external.ip.of.web-3>:5601/app/kibana 
Creating a seperate virtual network allows for a more smooth connection to Kibana.

A summary of the access policies in place can be found in the table below.
| Name                | Publicly Accessible | IP Address   |
|---------------------|---------------------|--------------|
| Jumpbox Provisioner | NO                  | 10.0.0.4     |   
| Web-1               | NO                  | 10.0.0.5     |   
| Web-2               | NO                  | 10.0.0.6     |   
| Web-3               | NO                  | 10.1.0.4     |   

### Elk Stack Setup and Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_

The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ...
- ...

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Services being utilized & Machines monitored
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _____ file to _____.
- Update the _____ file to include...
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
