## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.
	(Images/ELK server arch.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above.
---
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: RedAdmin
  become: true
  tasks:
    # Use apt module
  - name: Install docker.io
    apt:
      force_apt_get: yes
      name: "docker.io"
      state: present

    # Use apt module
  - name: Install python-pip
    apt:
      force_apt_get: yes
      name: "python-pip" 
      state: present 

    # Use pip module
  - name: Install Docker module
    pip:
      name: "docker" 
      state: present

    # Use command module
  - name: Increase virtual memory
    command: sysctl -w vm.max_map_count=262144

    # Use docker_container module
  - name: download and launch a docker elk container
    docker_container:
      name: elk
      image: sebp/elk
      state: started
    # Please list the ports that ELK runs on
      published_ports:
        - 5601:5601
        - 9200:9200
        - 5044:5044

---
- name: installing filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

  - name: install filebeat deb
    command: sudo dpkg -i filebeat-7.6.1-amd64.deb

  - name: copy filebeat.yml
    copy:
      src: ./filebeat.yml
      dest: /etc/filebeat/

  - name: enable, setup, and start system for filebeat ##modular (&&) does not work
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly stable and redudant, in addition to restricting traffic into the network.

	Load balancers protect in case the one of the VMs goes down. Jump box allows us to perform automated tasks instead of setting up each VM one by
        one

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.

Filebeat checks logs and centralizes it into one format for you to look at. It is visualized in Kibana

Metricbeat monitors CPU usage, RAM, file systems, Disk IO, etc to check on the health and usage of the computer. It is visualized in Kibana 

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name        | Function | IP Address  | Operating System |
|-------------|----------|-------------|------------------|
| Jump Box    | Gateway  | 10.0.0.5    | Linux            |
| My Computer | Client   | 70.44.37.22 | Windows          |
| DVWA1       | Server   | 10.0.0.6    | Linux            |
| DVWA2       | Server   | 10.0.0.7    | Linux            |
| Elk         | Server   | 10.1.0.4    | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the RedTeam VM machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: 70.44.37.22 (my public IP)

Machines within the network can only be accessed by Jumpbox
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_
	I allowed RedTeamVM access to the ELKVM, I used RedTeamVM's public IP to do so. 

A summary of the access policies in place can be found in the table below.

| Name      | Publicly Accessible |  Allowed IP Address |
|-----------|---------------------|---------------------|
| Jump Box  | Yes                 | 70.44.37.22         |
| DVWA1     | No                  | 10.0.0.5            |
| DVWA2     | No                  | 10.0.0.5            |
| Elk       | No                  | 10.0.0.5            |
| RedTeamLB | Yes                 | 70.44.37.22         |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
  In the example we have several servers, ansible could replicate all configs for us. 

The playbook implements the following tasks:
	Install docker.io, 
		python, 
		docker module, 
	increase memory of Elk VM, 
	launch Elk server, 
		allow published ports 5601, 9200 and 5044 for ELK to communicate
	ElasticSearch, Logstash, and Kibana

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

(Images/elk_capture.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

		10.0.0.6
		10.0.0.7

We have installed the following Beats on these machines:

		MetricBeat
		FileBeat

These Beats allow us to collect the following information from each machine:
		E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._
	FileBeat - Collects all logs from computers in resource pool, making it easily accessible. 
	MetricBeat - Collecting RAM, CPU usage, and other metric data from the computers 

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the ./filebeat.yml file to /etc/filebeat/

- Update the filebeat.yml file to include IP address of your ELK server
- Run the playbook, and navigate to VM1 or VM2 to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
	the config file is the playbook, you copy it into the directory of the installed module
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
	hosts file, using the either webservers for VM1 and VM2, or elkserver for elkserver 
- _Which URL do you navigate to in order to check that the ELK server is running?
 http://[your.VM.IP]:5601. Using the public IP address of the ELK server that you created
BUG FILE:
 - Noting that you must update IP address on hosts, and security groups once restarting VM's to keep active network
 - Opening port 5601 to allow communication with kibana
 - DVWA must not be attached, rather just started on VM1 and VM2 when accessing through jumpbox. 
	