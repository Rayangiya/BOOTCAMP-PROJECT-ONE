# BOOTCAMP-REPO

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.


![Elk network diagram](https://user-images.githubusercontent.com/67084375/143054564-6df63444-a584-4adb-8b18-a0c3fc900580.png)


These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Ansible and YML file may be used to install only certain pieces of it, such as Filebeat.

<p>
<details>
	<summary>Elk Installation Playbook</summary>
	
<pre><code>---
---
- name: Configure ELK
  hosts: ELK
  remote_user: sysadmin
  become: True
  tasks:
  - name: use more memory
    sysctl:
      name: vm.max_map_count
      value: '262144'
      state: present
      reload: yes

  - name: docker.io
    apt:
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
     force_apt_get: yes
     name: python3-pip
     state: present

  - name: install python module
    pip:
      name: docker
      state: present

  - name: elk container
    docker_container:
      name: elk
      image: sebp/elk:761
      state: started
      restart_policy: always
      published_ports:
        - 5601:5601
        - 9200:9200
        - 5044:5044
        
  - name: Enable Service docker on boot
    systemd:
      name: docker
      enabled: yes</code></pre> </details> </p>
	   
<p>
<details>
	<summary>Docker</summary>
	
<pre><code>---
 ---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes</code></pre> </details> </p>	   
	   

<p>
<details>
	<summaryFilebeat playbook</summary>
	
<pre><code>---
 ---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system

    # Use command module
  - name: Setup filebeat
    command: filebeat setup

    # Use command module
  - name: Start filebeat service
    command: service filebeat start

    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes</code></pre> </details> </p>

<p>
<details>
	<summary>Metricbeat playbook</summary>
	
<pre><code>---
 ---
- name: Install Metricbeat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start

    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes</code></pre> </details> </p>


This document contains the following details:
•	Description of the Topology
•	Access Policies
•	ELK Configuration
      o	Beats in Use
      o	Machines Being Monitored
•	How to Use the Ansible Build



### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available,in addition to restricting inbound access to the network.

What aspect of security do load balancers protect? 

The load balancer ensures that work to process incoming traffic will be shared by all vulnerable web servers. Access controls will ensure that only authorized users will be able to connect in the first place.

What is the advantage of a jump box?
The advantage of a jump box is that it allows automation, improves security, audtis traffic of segmented networks, and allows for ease of access control by using a single point to be secured and monitored.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file systems of the VMs on the network, and system metrics.

What does Filebeat watch for?

Filebeat monitors the log files or locations that you specify, collects log events, and forwards them either to Elasticsearch or Logstash for indexing.

What does Metricbeat record?
Metricbeat takes the metrics and statistics that it collects and ships them to the output that you specify, such as Elasticsearch or Logstash. Metricbeat helps you monitor your servers by collecting metrics from the system and services running on the server, such as: Apache.

The configuration details of each machine may be found below.

| Name            | Function   | IP Address | Operating System |
|-------------------|---------------|----------------|--------------------------|
| Jump Box      | Gateway   | 10.1.0.4     | Linux                     |
| web-1            | Server       | 10.1.0.5     | Linux                     |
| web-2            | Server       | 10.1.0.6     | Linux                     |
| web-3            | Server       | 10.1.0.9     | Linux                     |
| elk                 | Monitoring | 10.0.0.4     | Linux                     |
| load balancer|LB               | Static IP    | Linux                     |


### Access Policies

The machines on the internal network are not exposed to the public Internet.
In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:
* Availability Zone 1: web-1 + web-2 + web-3
* Availability Zone 2: ELK



Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:13.92.82.5. 

Machines within the network can only be accessed by each other.
 Which machine did you allow to access your ELK VM? What was its IP address?_
jump box IP=13.92.82.5
A summary of the access policies in place can be found in the table below.

| Name         | Publicly Accessible      | Allowed IP Addresses             |
|--------------|--------------------------|----------------------------------|
| Jump Box     | Yes                      |   13.92.82.5                     |
| Web-1        | No                       |    10.1.0.5                      |
| Web-2        | No                       |    10.1.0.6                      |
| Web-3        | No                       |    10.1.0.9                      |
| elk          | No                       |    10.0.0.4                      |
|load balancer | No                       |user public IP via HTTP port 80   |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because…

Ansible automation helps considerably with the representation of Infrastructure as Code (IAC). IAC involves provisioning and management of computing infrastructure and related configuration through machine-processable definition files.

The playbook implements the following tasks:

•	Machine groups and remote user specifications:

- name: Configure Elk VM with Docker
  	  hosts: elk
  	  remote_user: sysadmin
 	  become: true
  	  tasks:

•	Increase system memory:

- name: Use more memory
      	  sysctl:
       	  name: vm.max_map_count
              value: "262144"
              state: present
              reload: yes

•	Install the following packages:

      o	Docker.io
      o	Python3-pip
      o	Docker elk container

•	Launch docker container with these ports:

      o	5601:5601
      o	9200:9200
      o	5044:5044


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.
ELK-SERVER

azadmin@Elk-Server:~$ sudo docker ps

CONTAINER ID   IMAGE          COMMAND                  CREATED      STATUS        PORTS                                                                              NAMES
c6cee5528647   sebp/elk:761   "/usr/local/bin/star…"   3 days ago   Up 45 hours   0.0.0.0:5044->5044/tcp, 0.0.0.0:5601->5601/tcp, 0.0.0.0:9200->9200/tcp, 9300/tcp   elk
azadmin@Elk-Server:~$

WEB-1
```
azadmin@Web-1:~$ sudo docker ps

CONTAINER ID   IMAGE                 COMMAND      CREATED       STATUS        PORTS                NAMES
26e1101ff4f6   cyberxsecurity/dvwa   "/main.sh"   12 days ago   Up 45 hours   0.0.0.0:80->80/tcp   dvwa
azadmin@Web-1:~$1`
```
WEB-2
```
azadmin@Web-2:~$ sudo docker ps

CONTAINER ID   IMAGE                 COMMAND      CREATED       STATUS        PORTS                NAMES
5d8495319190   cyberxsecurity/dvwa   "/main.sh"   12 days ago   Up 45 hours   0.0.0.0:80->80/tcp   dvwa
azadmin@Web-2:~$
```
WEB-3
```
azadmin@Web-3:~$ sudo docker ps

CONTAINER ID   IMAGE                 COMMAND      CREATED       STATUS        PORTS                NAMES
187bfa738369   cyberxsecurity/dvwa   "/main.sh"   12 days ago   Up 45 hours   0.0.0.0:80->80/tcp   dvwa
azadmin@Web-3:~$
```
### Target Machines & Beats

This ELK server is configured to monitor the following machines:
- Web-1 :10.1.0.5
- Web-2 :10.1.0.6
- Web-3 :10.1.0.9

We have installed the following Beats on these machines:
- Elk Server, Web-1, Web-2 and Web-3
Specifying Beats successfully installed: filebeat and metricbeat

These Beats allow us to collect the following information from each machine:

•	Filebeat allows us to collect log events.
      o	Ex: Files generated by Apache or MS Azure tools.

•	Metricbeat allows us to collect the metrics and statistics.
      o	Ex: CPU usage.

### Using the Playbook


In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:

• Copy the Filebeat Playbook file to etc/ansible.
```
root@df9a706dfefe:~# cd /etc/ansible
root@df9a706dfefe:/etc/ansible# ls
ansible.cfg  filebeat-config.yml  files  hosts  metricbeat-config.yml  playbooks
root@df9a706dfefe:/etc/ansible#
```

•	Update the /etc/ansible/hosts file to include Web-1, Web-2, Web-3 and Elk-server.


  GNU nano 4.8  hosts                                                                   
  
 - This is the default ansible 'hosts' file.

- It should live in /etc/ansible/hosts
- Comments begin with the '#' character
- Blank lines are ignored
- Groups of hosts are delimited by [header] elements
- You can enter hostnames or ip addresses
- A hostname/ip can be a member of multiple groups

#Ex 1: Ungrouped hosts, specify before any group headers.

#green.example.com
#blue.example.com
#192.168.100.1
#192.168.100.10

#Ex 2: A collection of hosts belonging to the 'webservers' group
```
[webservers]

10.1.0.5 ansible_python_interpreter=/usr/bin/python3
#10.1.0.5 = Web-1
10.1.0.6 ansible_python_interpreter=/usr/bin/python3
#10.1.0.6 = Web-2
10.1.0.9 ansible_python_interpreter=/usr/bin/python3
#10.1.0.9 = Web-3

#alpha.example.org
#beta.example.org
#192.168.1.100
#192.168.1.110

[elk]
10.0.0.4 ansible_python_interpreter=/usr/bin/python3
#10.0.0.4 = ELK Server
```
•	Run the playbook, and navigate to the Elk VM via Kibana URL to check that the filebeat installation worked as expected.

	
![filebeat data verification](https://user-images.githubusercontent.com/67084375/143051087-dc3ef42f-7195-4b66-9742-5800f37a558b.jpg)
![filebeat data checking](https://user-images.githubusercontent.com/67084375/143051117-1f539325-cca8-472b-a012-05e73c40289c.jpg)



•	Run the playbook, and navigate to the Elk VM via Kibana URL to check that the metricbeat installation worked as expected.


![metricbeat datachecking success](https://user-images.githubusercontent.com/67084375/143051728-f1e995a1-c937-4ce9-b653-b17153d29ebc.jpg)
![metricbeat data check](https://user-images.githubusercontent.com/67084375/143051675-4ac879fd-ff4c-4e5b-8c2c-a70c46f3f44c.jpg)



Which file is the playbook?   Filebeat-playbook.yml
Where do you copy it?  Into the /etc/ansbile/files

Which file do you update to make Ansible run the playbook on a specific machine?
/etc/ansible/hosts file and add the IP address of the VM's under [webserver].

 How do I specify which machine to install the ELK server on versus which to install Filebeat on?
By specifying two groups in the /etc/ansible/hosts file, labled [webservers] for filebeat, and [ELK] for Elk installation.

- _Which URL do you navigate to in order to check that the ELK server is running?
http:[Elk.VM.IP]:5601/app/kibana

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._


