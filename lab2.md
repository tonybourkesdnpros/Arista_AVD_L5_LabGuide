# Lab 2 Reset Labs Using Ansible (arista.cvp)

In this lab you will explore utilizing the Ansible arista.cvp collection to reset the lab environment to its default state.  

## Preparing for Ansible

Click on the Ansible-CVP directory. In this directory you will see configlets, playbooks, and vars as well as the ansible.cfg and inventory.yml file. 

<img src=lab2-images/1.png width="50%" height="50%" border=1>

Look at the ansible.cfg file. 

<pre>
[defaults]
inventory = inventory.yml
gathering = explicit
</pre>

It's not complicated and doesn't need editing. Some Ansible environments may require more parameters, but these will be sufficient for your labs. 

Look at the inventory.yml file. 

<pre>
---
all:
  children:
    CVP_cluster:
      hosts:
        cvp1:
          ansible_httpapi_host: 192.168.0.5
          ansible_host: 192.168.0.5
          ansible_user: arista
          ansible_password: <b>arista123</b>
          ansible_connection: httpapi
          ansible_httpapi_use_ssl: True
          ansible_httpapi_validate_certs: False
          ansible_network_os: eos
          ansible_httpapi_port: 443

</pre>

Note the bold text for ansible_password. This will need to be changed to whatever your password is. That password can be found in the lower right hand corner of your initial lab page. It'll be the password that starts with "arista" and ends with four alphanumeric characters. 

<img src=lab2-images/2.png width="50%" height="50%" border=1>

Note: When you edit a file using code-server, the file is saved immediately. Files are changed in near real-time when you edit them, so there's no need to save files. 

##

With ansible.cfg and the inventory.ini sorted out, you can start to work with Ansible playbooks and CloudVision. 

Click on the file "reset_lab.yml", under the playbooks directory. This is a playbook that will reset the lab to the default configuration of ATD-INFRA for every device (required) as well as each device's base configlet (also required). 

<img src=lab2-images/4.png width="50%" height="50%" border=1>

Notice in the file that it's calling on two Ansible modules: cv_container_v3 and cv_device_v3 (they're prepended with arista.cvp). It also loads up a variable file called CVP_base.yml, which contains the data models needed for both modules to use. 

Look at the CVP_base.yml under the vars directory and familiarize yourself with it. 

<pre>
---
# This file is a data model for configlets, containers, and devices that resets the ATD environment to default

path: "{{lookup('env','PWD')}}"


cvp_containers_default:
  DC1:
      parentContainerName: Tenant
  Spine:
      parentContainerName: Tenant
  Leaf:
      parentContainerName: Tenant
  Borderleaf:
      parentContainerName: Tenant
  Hosts:
      parentContainerName: Tenant
 
cvp_devices_default:
  - fqdn: 'spine1'
    parentContainerName: 'Spine'
    configlets:
      - 'ATD-INFRA'
      - 'spine1-base'
  - fqdn: 'spine2'
    parentContainerName: 'Spine'
    configlets:
      - 'ATD-INFRA'
      - 'spine2-base'
  - fqdn: 'spine3'
    parentContainerName: 'Spine'
    configlets:
      - 'ATD-INFRA'
      - 'spine3-base'
  - fqdn: 'leaf1'
    parentContainerName: 'Leaf'
    configlets:
      - 'ATD-INFRA'
      - 'leaf1-base'
  - fqdn: 'leaf2'
    parentContainerName: 'Leaf'
    configlets:
      - 'ATD-INFRA'
      - 'leaf2-base'
  - fqdn: 'leaf3'
    parentContainerName: 'Leaf'
    configlets:
      - 'ATD-INFRA'
      - 'leaf3-base'
  - fqdn: 'leaf4'
    parentContainerName: 'Leaf'
    configlets:
      - 'ATD-INFRA'
      - 'leaf4-base'
  - fqdn: 'borderleaf1'
    parentContainerName: 'Borderleaf'
    configlets:
      - 'ATD-INFRA'
      - 'borderleaf1-base'
  - fqdn: 'borderleaf2'
    parentContainerName: 'Borderleaf'
    configlets:
      - 'ATD-INFRA'
      - 'borderleaf2-base'
</pre>

This data model has two sections: cvp_containers_default cvp_devices_default. The playbook will make sure the containers specified in cvp_containers_default are there, and the playbook will make sure that the configlets listed under cvp_devices_default are attached to the devices (and only those configlets are attached). 

In the terminal window, change the directory to the Advanced-CVP directory and then the Ansible-CVP directory. 

<pre>
➜  persist <b>cd Advanced-CVP/Ansible-CVP </b>
➜  Ansible-CVP git:(main) 
</pre>

Then run the Ansible playbook called reset_lab.yml. 

<pre>
➜  Ansible-CVP> <b>ansible-playbook playbooks/reset_lab.yml</b>

PLAY [Reset lab environment]******************************************************************************************

TASK [Build default container topology] ******************************************************************************************
changed: [cvp1]

TASK [Apply configlets for default config] ******************************************************************************************
changed: [cvp1]

PLAY RECAP ******************************************************************************************
cvp1                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
</pre>

Check CloudVision to see if any tasks were created (if the labs were already at the same state of the data models, no tasks will be created). The playbook was not setup to automatically create and execute a change control. If there are tasks, put them into a change control and execute the change control as you have done for previous labs. 


