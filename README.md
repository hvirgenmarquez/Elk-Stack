# HVirgen
Repository for Hector Virgen-Marquez

## Automated ELK Stack Deployment

The following repository is designed for the purpose of fullfilling a sucessful ELK Stack Server Deployment. The purpose of an ELK Stack Server is to create a network of virtual machines that can be monitored through Kibana with the assistance of filebeat and metricbeat services. 

Below is a diagram using Draw.io, the diagram represents a visualization of how the network is set up, how each connection is established, and what components are involved.

![Project 1 Diagram](https://github.com/hvirgenmarquez/HVirgen/blob/main/Diagrams/Project1%20Diagram.jpg?raw=true)

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
- Further down the Network Hiearchy you will find the webservers and the Elk server. The webservers are named Web-1 and Web-2 (and for the purpose of this project, I also added Web-3 for educational purposes), and the Elk Server is Web-3. 

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
| Name                | Publicly Accessible | Accessible by IP Address   |
|---------------------|---------------------|----------------------------|
| Jumpbox Provisioner | NO                  | Personal IP address        |   
| Web-1               | NO                  | 10.0.0.4                   |   
| Web-2               | NO                  | 10.0.0.4                   |   
| Web-3               | NO                  | 10.0.0.4                   |   

### Elk Stack Setup and Configuration

In order to allow for a steamless instalation of the ELK Stack Setup, I created a vareity of playbooks formating to be run through Ansible.
- Running playbooks through Ansible gives the advantage of automation, while ensuring in the event of multiple instalations to multiple VMs, each VM will have the same configuration. 

The playbook implements the following tasks:
1. Clarify What is being installed
2. Get installation from website
3. Install files downladed from Step 2
4. Copy Configurations from Host VM to Webserver VMs
5. Enable System Configuration Modules
6. Setup Service
7. Start Service

For this project I utilized a total of 5 playbooks:
- Playbook to Install [ELK Stack Container](https://github.com/hvirgenmarquez/HVirgen/blob/main/Ansible/elkplaybook.yml)
- Playbook to Install [DVWA Container](https://github.com/hvirgenmarquez/HVirgen/blob/main/Ansible/web-3playbook.yml)
- Playbook to Install [Filbeat](https://github.com/hvirgenmarquez/HVirgen/blob/main/Ansible/Filebeat-playbook.yml)
- Playbook to Install [Metricbeat](https://github.com/hvirgenmarquez/HVirgen/blob/main/Ansible/metricbeat-playbook.yml)
- Playbook to [Start Metricbeat and Filebeat](https://github.com/hvirgenmarquez/HVirgen/blob/main/Ansible/filebeat-metricbeat-playbook-start-services.yml)
  - This playbook is to start up the services and increase max memory in case these settings were not saved.

When the ELK Server Container is properly setup, you should be able to run _sudo docker container list -a_ and get a ![ELK Server Response](https://github.com/hvirgenmarquez/HVirgen/blob/main/Diagrams/Screenshots/123123123.jpg?raw=true) 


### Services being utilized & Machines monitored
For further reiteration: there are 4 total VMs and 2 containers being utilized in this project. 
- The 4 VMs are: Web-1, Web-2, Web-3 (ELK), and Jumpbox Provisioner.
- The 2 Containers are: Focused_poincare, sebp:elk/726.

In order for the ELK stack to work properly, the container Focused_poincare was used as the host machine for this project. As the host machine, Focused_poincare was given configurations for the /Ansible/Hosts file, and given directions to place Web-1, Web-2, and Web-3 in [Webservers] Group, and Web-3 in [ELK] Group.
- [Webservers]: This group is used to as the host for the playbooks. As the host, the playbooks will push forward the installations on to all three VMs, allowing for automated installation of Filebeat and Metricbeat.
- [ELK]: This group is to specify which server will be running the ELK Container. Since Web-3 is the only one in this group, in order to access Kibana, the external IP address of Web-3 is used to access the Kibana interface, while the internal IP is used to configure filebeat and metricbeat. 
  - Placing only 1 VM in the ELK group allows for seemless connection from an outside source (like my personal computer) to the kibana interface. 

Metricbeat and FIlebeat installations:
- [Filebeat] and [Metricbeat] As shown in the playbook, the installation of Filebeat has been completely automized thanks to the Ansible command. 
  - To make sure Filebeat is properly configured, I created a Filebeat-Configuration.yml file that specifies:
    - The setup.kibana IP and port, in this case: "10.1.0.4:5601"
    - The output.elasticsearch IP and port, in this case: ["10.1.0.4:9200"]
  - Using these ports gives specifications for the Filebeat and Metricbeat services, by instructing them to gather data and output them to this data.
  - For example:
    - The 5601 port is the port required to connect to the Kibana interface.
    - The 9200 port is the port required to output elasticsearch log data and metric data to Kibana.
  - Both Ports need to be specified in order for the Kibana interface to properly receive data.
  - The IP address of 10.1.0.4 is Web-3's IP, where the ELK Container is installed. Sending all data to this IP address ensures that there is 1 VM gathering all data so that Kibana can properly display the data. 

Metricbeat and FIlebeat usage:
-  [Filebeat] is a log-collection service, it collects detailed log data of activity detected within the hosts configured. 
   - Filebeat is great to find information about what may be happening to your server, or if someone is trying to break into your server. 
      - SSH login attempts are recorded using filebeat's log data service. This allows for a defensive supervision of the network activity in your Elk Stack setup. 
-  [Metricbeat] is another log-collection service, however unlike Filebeat, Metricbeat provides details on CPU activity, available memory, etc.,
   - Metricbeat is useful to see if any particiular stress if affecting your servers. By monitoring the CPU, memory, network activity, and more, Metricbeat allows for a more   technical analysis of your server's activities. 

### Using the Playbook
In order to properly use the playbook, there are a few steps you need to take.
1. SSH to the server where you will be pushing these installations out, we will call this the control node server. In my case, I used the container "Focused_poincare" as my control node server.
  - The Focused_Poincare container is located in the Jumpbox Virtual Machine: 10.0.0.4. 
2. From the control node server you should have Ansible installed. If Ansible is installed, use the command: _Anisble-PLaybook_ _<Playbook here>. So if I wanted to install Filebeat, I would use the following command: _Ansible-Playbook Filebeat-Playbook.yml_
3. Once you enter this command, assume all syntax is correct in your playbook (use _Ansible-Playbook <Playbook here> --syntax-check_ to check ssyntax), the installation will begin.
4. During installation you will see all the tasks being completed one-by-one. It is crucial to line-up your tasks in a logical chronological order. So if you are installing Filebeat, make sure the Download is the first step, followed by installation, followed by setup, etc., (refer to my playbook for an example).
5. Assuming all works, the installtion will give you a result line with either: #=Succeeded, #=Changed, #=Failed.
   - #=Succeeded is self-explanatory. Your playbook worked!
   - #=Changed implies that data was overwritten, this is OK! Your playbook worked!
   - #=Fialed implies that there was an error during the installation. Typically you will get an error message as well as a explanation of the error. Fix up your syntax, or torubleshoot the issue, and try the Ansible-Playbook command again.

 ## Final Thoughts 
This brings me to the end of my project report. Creating the ELK Stack Server was definitely an experience that I appreciate. Even though at first, I decided to manually install Filebeat and Metricbeat, which led to a myriad of issues with Kibana not receiving logs, getting 5601:connection refused errors, and more. I quickly learned that there is a reason the playbook is the most eficient method to install these files, not just because it's automated, but because it will ensure everything is properly configured accross the board. 
 
I definitely feel ready to create a network like this should the opportunity present itself in my professional career. If anyone is reading this and has never setup an ELk Stack Server, I definitely recommend you give it a shot! There are plenty of resources online to help you along the journey. 
