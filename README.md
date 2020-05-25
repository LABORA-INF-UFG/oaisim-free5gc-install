# oaisim-free5gc-install
OpenAirInterface + Free5GCore deployment environment

This project aims to build a playbook for implementing the elements that make up the [OpenAirInterface System Emulation](https://gitlab.eurecom.fr/oai/openairinterface5g/wikis/OpenAirLTEEmulation). To use the playbook you need the following elements:

1. A machine called 'operator's machine', running Linux and with a properly installed version of [Ansible](https://docs.ansible.com/). The next sections will present the steps for installing Ansible.
2. Other's machine's for installing the Free5G and OpenAirSIM elements.


The figure below shows more details about the deployment environment:
![](images/environment_description.png)

We assume that the <b>all machines are connected to the internet</b> and <i>see each other</i>.
# Installation Guide
The first thing to do, is configure the <i>operator machine</i>.

## 1 - Ansible Installation / Configuration (Operator Machine)
Ansible's installation procedures depend on the inclusion of some repositories on the operator's machine. Depending on the distribution uses the commands for the inclusion of these repositories they can change, for more information see [this page](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-the-control-node) . The next steps works to <b>linux Ubuntu 16.04.x LTS</b>. To add a new repository, run:
```
sudo apt-add-repository -y ppa:ansible/ansible-2.7
```
then, update the dependencies tree:
```
sudo apt-get update
```
and finally install Ansible with the following command:

```
sudo apt-get install ansible
```
After installation check if the installed version is 2.7 or higher using the following command:
```
ansible --version
```
the expected result should be equivalent to that shown in the image below:
![](images/ansible_result_installation.PNG)

### Access Settings (Operator Machine / enB Machine)
After installing ansible on the operator's machine, the next step is to configure the connection between the operator's machine and the other machines involved in the OpenAir deployment process. For the correct operation, Ansible needs to have full access to the other machines involved, this is done through the exchange of <i>SSHKeys</i> process:

Generate an ssh key from the operator's machine using the following command:
```
ssh-keygen -t ecdsa -b 521
```
I recommend that you use  <i>empty passphrase</i>, the result should be equivalent to that shown in the image below:
![](images/ssh_keys_gen.PNG)

This key will be used by <i>Ansible</i> when running the deployment playbooks, so we must copy that key to the other machines involved in the process and ensure that it stays in the **root directory of the respective machines**. To copy the operator's machine key to the machine where OAISim+free5gc will be deployed, use the following command:
```
ssh-copy-id -i ~/.ssh/id_ecdsa.pub <user>@<enB-host>
```
the result should be equivalent to that shown in the image below:
![](images/ssh_copy_keys.PNG)

after copy ssh key, access the deployment machine ``` ssh <user>@<enB-host> ``` and run the following commands:
```
 apt install python-minimal -y
```
the last command install **python minimal**. This package contains the interpreter and some essential modules. It is used by Ansible for same basic tasks.

After install <i>python minimal</i>, we need get some information about **physical network interface** of the machine. To do this, run ```ifconfig``` and take note the **_physical network interface name_** display in the next figure.
![](images/if_config.PNG)
this information will be necessary when executing the deployment playbook.

#### Test Ansible Connection (Operator Machine / enB Machine)

On the <i>operator's machine</i> it will be necessary to clone this project to test the connection throught <i>Ansible</i>. To be possible, it is necessary to have **GIT** properly installed. You can check this with the following command:
```
git --version
```
the expected result should be something similar to:
```
git version x.x.x
```  
if GIT is not installed, just run the following command:
```
sudo apt-get install git
```
 
 Then choose a directory and clone the **oaisim-with-ansible project**:
```
git clone https://github.com/ciromacedo/oaisim-with-ansible.git
```
after clone, access the project folder and open the **hosts** file with a text editor (Nano, Vi). The file content is similar to:
```
[OAISim-with-free5gc]
<deployment-environment-IP-address>
```
replace the ```<deployment-environment-IP-address>``` for the IP address of the <i>deployment environment machine</i>. Save and close the file, and inside the project base directory run the next command:
```
ansible -i ./hosts -m ping all -u root
```
the expected result should be equivalent to that shown in the image below:
![](images/ansible_test_connection.PNG)

this means that everything is fine and that <i>Ansible</i> has full access to the <i>deployment machine</i>.
 
## 2 - Run Ansible Playbook (Free5G + OpenAirSIM Install)
 After configuration steps, just run the next command.
```
ansible-playbook   -vvvv   Deploy5GC.yml  -i  hosts -e "physical_network_interface=<< physical network interface name>>"
```
it will be start the process of deployment the elements of **enB/Ue's + free5GC**. The ```-vvvv``` parameter controls the **verbosity level of log** and can be adjusted (```-v```, ```-vv```, ```-vvv``` or ```-vvvv```) or omitted.

## 3 - Running and testing
After finish installation for default, **MongoDB** and **Web User Interface** is initialized. You can check this in your browser ```http://<deployment-environment-IP-address>:3000```. If you access the deployment machine and type ```docker ps```, you can see that all the elements ar running, the result should be equivalent to that shown in the next figure:
![](images/docker_ps.png) 

Now, we will __run__ the other 5GC elements, for this, access the deployment machine with 7 different terminal's and in each terminal run the follow steps.

### Running AMF
Access the _first terminal_ and and run the following commands:
```
docker exec -ti amf bash
/root/free5gc-stage-1/install/bin/free5gc-amfd 
```
the result should be equivalent to that shown in the next figure:
![](images/amf_start.png) 

### Running UPF
Access the _secound terminal_ and and run the following commands:
```
docker exec -ti upf bash
/root/free5gc-stage-1/install/bin/free5gc-upfd
```
the result should be equivalent to that shown in the next figure:
![](images/upf_start.png) 

### Running SMF
Access the _third terminal_ and and run the following commands:
```
docker exec -ti smf bash
/root/free5gc-stage-1/install/bin/free5gc-smfd
```
the result should be equivalent to that shown in the next figure:
![](images/smf_start.png) 

the green mark in the figure, represents the _UPF Association Response_ container reaction when SMF is initialized.

### Running HSS
Access the _fourth terminal_ and and run the following commands:
```
docker exec -ti hss bash
/root/free5gc-stage-1/install/bin/nextepc-hssd 
```
the result should be equivalent to that shown in the next figure:
![](images/hss_start.png) 

the green mark in the figure, represents the _AMF Connection_ container reaction when HSS is initialized.

### Running PCRF
Access the _fifth terminal_ and and run the following commands:
```
docker exec -ti pcrf bash
/root/free5gc-stage-1/install/bin/nextepc-pcrfd 
```
the result should be equivalent to that shown in the next figure:
![](images/pcrf_start.png) 

the green mark in the figure, represents the _SMF Connection_ container reaction when PCRF is initialized.

So far all the elements of the _free5gC_ have been initialized, the next will be elements of the _OpenAirSIM_.

### Running enB
Access the _sixth terminal_ and and run the following commands:
```
docker exec -ti enb bash
cd /root/enb/cmake_targets/ran_build/build
sudo -E ./lte-softmodem -O /root/enb/ci-scripts/conf_files/rcc.band7.tm1.nfapi.conf 
```
the result should be equivalent to that shown in the next figure:
![](images/enb_start.png) 
The _enB_ terminal will be in constant loop displaying the next message ``` Waiting fo PHY_config_req```. The green mark in the figure, represents the _AMF enB Registration_ container reaction when enB is initialized.

### Running UE
Access the _seventh terminal_ and and run the following commands:
```
docker exec -ti ue bash
cd /root/ue/cmake_targets/ran_build/build
./lte-uesoftmodem -O /root/ue/ci-scripts/conf_files/ue.nfapi.conf --L2-emul 3 --num-ues 1 --nums_ue_thread 1 --nokrnmod 1
```
the result should be equivalent to that shown in the next figure:
![](images/ue_start.png) 
The alert messages are not relevant.

## 4 - Testing User Equipments (UE) Internet Connection
Now we can test the UE internet connection. For this, access the deployment machine and type ``` docker exec -ti ue bash ``` to access de UE Container. Inside the container type ```ifconfig``` to check network interface. The result should be equivalent to that shown in the next figure:
![](images/ue_network_interfce.png) 