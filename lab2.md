# Setting up Arista AVD

Arista AVD is an advanced set of playbooks, data models, templates, roles, and modules that allow for the generation of various types of fabric configurations, as well as deployment options, automatic documentation, and post-deployment validation. 

With just a few data models, AVD can generate complex EVPN/VXLAN configurations for dozens (or even hundreds) of switches.

In this lab, you will create an L3LS+EVPN/VXLAN configuration with just a few files. 

## Prepare Data Models

Select the TD_AVD_L5 labs directory.

<img src=lab2-images/1.png border=1>

You'll see a few directories and files:

* group_vars
* playbooks
* ansible.cfg
* inventory.yml

Group_vars will have the data models used to build the fabric configuration files. 

Playbooks will have the various playbooks being used (build, deploy, test).

The ansible.cfg file has the minimum parameters for Ansible to work with AVD. 

The inventory will have the individual devices, grouped according to their roles and the network topology. 

Feel free to explore the files. 

## Inventory File

For Ansible to run, there must be an inventory file. This file will have the login information for both CloudVision Portal and the individual leafs and spines. The former is used to upload and apply the generated configurations, and the later is used to know which devices AVD should generate configurations for. 

Open the inventory file in the editor by clicking on inventory.yml. 

<img src=lab2-images/2.png border=1>


There is the "all" group, which is all of the groups. "Children" signifies that there are more groups (as opposed to hosts). 

There is a group called <b>CVP_cluster</b> which is where you would put all of the CVP hosts. In the lab environment, there is only one CVP host, named cvp1. 

Change the <tt>ansible_password</tt>: field (highlighted in red) to your environment's password. It will be "arista" followed by four alphanumeric characters. 

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
          ansible_password: <span style="color:red;"><b>aristaXXXX</b></span>
          ansible_connection: httpapi
          ansible_httpapi_use_ssl: True
          ansible_httpapi_validate_certs: False
          ansible_network_os: eos
          ansible_httpapi_port: 443
</pre>

* <i>Note: While the passwords are located in the inventory file for convenience sake, there are methods to encrypt the password using mechanisms like Ansible Vault to ensure that no password is show in plaintext</i>

You can explore the rest of the inventory file, which will have a FABRIC group, as well as sub groups. You do not need to modify anything at this time. 

* FABRIC
  * DC1
    * SPINES_DC1
    * LEAFS_DC1
  * EVPN_SERVICES
    * LEAFS_DC1
  * ENDPOINT_CONNECT
    * LEAFS_DC1


## Build EVPN/VXLAN 

In AVD, there will be three playbooks used. 

* build_fabric.yml
* deploy_fabric.yml
* test_fabric.yml

The build_fabric.yml playbook is the first one we will use. It will both build configlets for the fabric, as well as build documentation for that build. 

The build process will take the three data models. You'll find them in the group_vars directory: 
* FABRIC.yml: This file describes the overall fabric (for a single DC environment, this includes all the leafs and spines)
* EVPN_SERVICES.yml: This file describes the VXLAN segments and anycast gateways to be created
* ENDPOINT_CONNECT.yml: This file describes how the hosts will be connected to the network through the leafs

AVD will use Ansible to take these data models and convert them into individual configlets for the leafs and spines. 

The deploy process will take those configlets, upload them to CloudVision, and attach them to the various devices. This will generate tasks that the operator can run through a change control. 

After the change control process has been completed, then the test playbook will run a series of tests on all of the devices, such as checking each device's routing table to make sure the loopbacks are present. 

### Build AVD Configuration

Run the build_fabric.yml playbook. The output below has been truncated. (Note: You may see several warning in magenta, they can be ignored.)

<pre>
  AVD git:(main) ✗ <b>ansible-playbook playbooks/atd-build.yml</b>
...
PLAY RECAP ******************************************************************************************************************************************************************************************
leaf1-DC1                  : ok=13   changed=8    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2-DC1                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3-DC1                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4-DC1                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1-DC1                 : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine2-DC1                 : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
</pre>

This will create a directory called "intended" and "documentation". The "intended" directory contains both the intermediary EOS config (in YAML) in a directory called "structured_configs" as well as "configs" which contain the configlets that will be uploaded. 

Open the configuration directory and then configs and you will see leaf1-DC1, leaf2-DC1, leaf3-DC1, leaf4-DC1, and spine1-DC1, and spine2-DC1 configurations. 

<img src=lab2-images/3.png border=1>

You can click on them to explore the configurations generated. leaf1-DC1.cfg, for example, is over 200 lines of newly generated configuration based off the data models.

## Deploy Configurations

Now deploy that configuration to CloudVision:

<pre>
  AVD git:(main) ✗ <b>ansible-playbook playbooks/deploy_fabric.yml</b>
[DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is being removed. This feature will be removed from ansible-core in version 2.14. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [Deploy created AVD Configuration] *************************************************************************************************************************************

TASK [arista.avd.eos_config_deploy_cvp : Create required output directories if not present] *********************************************************************************
ok: [cvp1 -> localhost] => (item=/home/coder/project/persist/Advanced-CVP/AVD/atd-inventory/intended/structured_configs/cvp)
...
PLAY RECAP ******************************************************************************************************************************************************************
cvp1                       : ok=10   changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   

</pre>

This will create 6 tasks for leaf1-4-DC1 and spine1-2-DC1 (the borderleafs have not been included).

On the top bar, select "Provisioning". 

<img src=lab2-images/4.png border=1>

Click on the "Tasks" section on the sidebar 

<img src=lab2-images/5.png border=1>

The tasks represent a potential change in configuration reflecting the new configlets applied to each device. The change isn't implemented until a change control has been run. 

### Run Change Control

Under Provisioning and Tasks, select all the tasks by clicking the button at the top, and then click "+ Create Change Control". 

<img src=lab2-images/6.png border=1>

Change the arrangement to "Parallel", and click "Create Change Control with 6 Tasks"

<img src=lab2-images/7.png border=1>

This will bring you to a new change control. Click on "Review and Approve" in the upper right. 

<img src=lab2-images/8.png border=1>

You can scroll through the various proposed changes. The change control can only be executed when it has been approved. Click on "Approve" in the lower right hand corner. 

<img src=lab2-images/9.png border=1>

This will bring you back to the change control page. Click on the "Execute Change Control" button in the upper right hand corner. 

<img src=lab2-images/10.png border=1>

Click the "Execute" button in the confirmation window. 

<img src=lab2-images/11.png border=1>

The change control process will start. It usually will complete in less than 30 seconds, though it may take longer depending on how busy the server is. 

When it's completed, you'll see green checks for each device, and the status will show a green "Completed". 

<img src=lab2-images/12.png border=1>

### Verify Change on Leaf1-DC1

From the command line, SSH into host1-DC1 with the command <tt>ssh arista@host1-DC1</tt>

<pre>
  TD_AVD_L5
 git:(main) ✗ <b>ssh arista@host1-DC1</b>
  host1-DC1#
</pre>

Run the command <tt>ping 10.1.10.1</tt>. This is the anycast gateway for VLAN 10, configured on leaf1-DC1 and leaf2-DC1. 

<pre>
TD_AVD_L5 git:(main) ✗ <b>ssh arista@host1-dc1</b>                        
host1-DC1#<b>ping 10.1.10.1</b>
PING 10.1.10.1 (10.1.10.1) 72(100) bytes of data.
80 bytes from 10.1.10.1: icmp_seq=1 ttl=64 time=5.71 ms
80 bytes from 10.1.10.1: icmp_seq=2 ttl=64 time=5.97 ms
80 bytes from 10.1.10.1: icmp_seq=3 ttl=64 time=4.51 ms
80 bytes from 10.1.10.1: icmp_seq=4 ttl=64 time=3.10 ms
80 bytes from 10.1.10.1: icmp_seq=5 ttl=64 time=3.28 ms

--- 10.1.10.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 23ms
rtt min/avg/max/mdev = 3.107/4.519/5.974/1.189 ms, ipg/ewma 5.766/5.031 ms
</pre>

Logout of host1-DC1 and log into host2-DC1. 

<pre>
host1-DC1#exit
Connection to host1-dc1 closed.
➜  TD_AVD_L5 git:(main) ✗ ssh arista@host2-dc1
The authenticity of host 'host2-dc1 (192.168.0.52)' can't be established.
ECDSA key fingerprint is SHA256:/pZTDJtSUEGuo2ylRRenu715kqJZK7qdnrddgiG4lj0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? <b>yes</b>
</pre>



</pre>
Warning: Permanently added 'host2-dc1,192.168.0.52' (ECDSA) to the list of known hosts.
host2-DC1#
</pre>


<pre>
leaf1#<b>show bgp evpn summary</b>
BGP summary information for VRF default
Router identifier 192.168.101.1, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Description              Neighbor       V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  spine1-DC1                   192.168.101.11 4 65001             23        28    0    0 00:03:20 Estab   8      8
  spine2-DC1                   192.168.101.12 4 65001             25        26    0    0 00:03:22 Estab   8      8

</pre>

### Add spine3-DC1

The environment has a third spine, spine1-DC1. You will add this spine by modifying just two files. 

First, select inventory.yml and scroll to the section where you see spine1-DC1 and spine2-DC1, which will be in the SPINES_DC1 group. 


<img src=lab2-images/13.png border=1>

Change add spine3-DC1 under SPINES_DC1 like below. Be careful to preserve the original indentation. 

<pre>
           SPINES_DC1:
              vars:
                type: spine
              hosts:
                spine1-DC1:
                  ansible_host: 192.168.0.11
                spine2-DC1:
                  ansible_host: 192.168.0.12
                <span style="color:red;">spine3-DC1:
                  ansible_host: 192.168.0.13</span>
</pre>

Next, open the FABRIC.yml file and scroll the the "nodes" section under the "spine" section. 

<img src=lab2-images/14.png border=1>

Add spine3-DC1 as shown bellow. Again, be careful to preserve indentation.

<pre>
# Spine Switches
spine:
  defaults:
    bgp_as: 65001
    loopback_ipv4_pool: 192.168.101.0/24
    bgp_defaults:
      - 'no bgp default ipv4-unicast'
      - 'distance bgp 20 200 200'
    mlag: false
  nodes:
    spine1-DC1:
      id: 11
      mgmt_ip: 192.168.0.11/24
    spine2-DC1:
      id: 12
      mgmt_ip: 192.168.0.12/24
    <span style="color:red;">spine3-DC1:
      id: 13
      mgmt_ip: 192.168.0.13/24</span>

</pre>

Run the build playbook again to generate the configurations: <tt>ansible-playbook playbooks/build_fabric.yml</tt>



<pre>
TD_AVD_L5 git:(main) ✗ <b>ansible-playbook playbooks/build_fabric.yml</b>
. . .
PLAY RECAP ******************************************************************************************************************************************************************************************
leaf1-DC1                  : ok=13   changed=6    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2-DC1                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3-DC1                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4-DC1                  : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1-DC1                 : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine2-DC1                 : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine3-DC1                 : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0 
</pre>

A new configuration will be generated for spine3-DC1, as well as new versions of the configurations for the leafs as they now have new uplinks. 

Run the deploy playbook again: ansible-playbook playbooks/deploy_fabric.yml

<pre>
➜  TD_AVD_L5 git:(main) ✗ <b>ansible-playbook playbooks/deploy_fabric.yml</b>

PLAY [Deploy fabric to CVP] *************************************************************************************************************************************************************************
...
PLAY RECAP ******************************************************************************************************************************************************************************************
cvp1                       : ok=10   changed=3    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
</pre>

### Test via Ping




As you can see, the EVPN overlay is functioning (Estab state). However, these types of spot checks have limited use, since leaf1 being operational does not mean the other leafs are. 

AVD has a built-in validation function that can verify the fabric by performing a series of tests on the devices outlined in the YAML files. 

To run this validation, run the atd-validation.yml playbook: 

<pre>
➜  AVD git:(main) <b>ansible-playbook playbooks/atd-validate-states.yml</b>

.... (output truncated)

PLAY RECAP ********************************************************************
leaf1                      : ok=38   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=1   
leaf2                      : ok=38   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=1   
leaf3                      : ok=38   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=1   
leaf4                      : ok=38   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=1   
spine1                     : ok=34   changed=3    unreachable=0    failed=0    skipped=13   rescued=0    ignored=1   
spine2                     : ok=26   changed=0    unreachable=0    failed=0    skipped=13   rescued=0    ignored=1   
spine3                     : ok=26   changed=0    unreachable=0    failed=0    skipped=13   rescued=0    ignored=1   
</pre>

The reuslts are placed into a report markdown file (as well as a CSV). You can find that in the reports directory under the AVD/atd-inventory directory:


<img src=lab2-images/4.png border=1>

Click on the ATD_FABRIC-state.md file. This will display the source of the markdown. To see it in a web browser display style, click on the book-with-magnifying glass icon in the upper right. 

<img src=lab2-images/5.png border=1>

This will bring up a browser-style view. You can move the window slider to see more of the Validate State Report.

<img src=lab2-images/6.png border=1>

Note how many tests were run. The total number may vary depending on the version of the lab environment, but generally over 200. They should all have passed. 

Log into leaf1 and <tt>shutdown</tt> interfaces e3-e5, which are leaf1's connection to the spines. 

<pre>
➜  AVD git:(main) <b>ssh arista@leaf1</b>
Last login: Wed Feb  8 18:47:32 2023 from 192.168.0.1
leaf1#<b>conf</b>
leaf1(config)#<b>int e3-5</b>
leaf1(config-if-Et3-5)#<b>shutdown</b>
</pre>

Then run the tests again. You'll see several falures this time that the validate states module caught. 

<img src=lab2-images/7.png border=1>

Re-enable the interfaces (they'll be neaded for later parts of the lab). Re-run the validate states again 

### Network Services

View the file "<tt>ATD_TENANTS_NETWORKS.yml</tt>"

<pre>
tenants:
  # Tenant A Specific Information - VRFs / VLANs
  Tenant_A:
    mac_vrf_vni_base: 10000
    vrfs:
      VRF_A:
        vrf_vni: 10
        svis:
          10:
            name: VLAN_10
            tags: [leaf_group1]
            enabled: true
            ip_address_virtual: 10.1.10.1/24
</pre>

This file details the tenants, VRFs, VLAN/SVIs of the network services provided to hosts. 

Now look at the file "ATD_SERVERS.yml". This file details how the hosts are connected to the fabric. This file does not configure anything on the hosts, only the interfaces on the leafs that connect to the hosts. 

<pre>
---
port_profiles:
  VLAN_10:
    mode: access
    vlans: "10"

servers:
  host1:
    rack: mlag1
    adapters:
      - type: nic
        switch_ports: [Ethernet6, Ethernet7, Ethernet6, Ethernet7]
        switches: [leaf1,leaf1, leaf2, leaf2]
        profile: VLAN_10
        port_channel:
          state: present
          description: PortChannel_to_host1
          mode: active
  host2:
    rack: mlag2
    adapters:
      - type: nic
        switch_ports: [Ethernet6, Ethernet7, Ethernet6, Ethernet7]
        switches: [leaf3, leaf3, leaf4, leaf4]
        profile: VLAN_10
        port_channel:
          state: present
          description: PortChannel_to_host2
          mode: active
</pre>

Host1 is connected to leaf1 and leaf2 through four links (Ethernet6 and Ethernet7 on leaf1/2). Host2 is simlilarly connected to leaf3 and leaf4. They're both attached to profile VLAN_10, which is just an acceess port for VLAN 10. 

Log into the host1 and add a simple configuration for a port channel and IP address (don't forget to save the config with a <b><tt>wr</b></tt> command): 

<pre>
➜  project <b>ssh arista@host1</b>
host1#<b>conf</b>
host1(config)#<b>int e1-4</b>
host1(config-if-Et1-4)#<b>channel-group 1 mode active</b>
host1(config-if-Et1-4)#<b>int po1</b>
host1(config-if-Po1)#<b>no switchport</b>
host1(config-if-Po1)#<b>ip address 10.1.10.11/24</b>
host1(config-if-Po1)#<b>wr</b>
Copy completed successfully.
</pre>

Do the same with host2 (with IP 10.1.10.12/24)

<pre>
➜  project <b>ssh arista@host2</b>
host2#<b>conf</b>
host2(config)#<b>int e1-4</b>
host2(config-if-Et1-4)#<b>channel-group 1 mode active</b>
host2(config-if-Et1-4)#<b>int po1</b>
host2(config-if-Po1)#<b>no switchport</b>
host2(config-if-Po1)#<b>ip address 10.1.10.12/24</b>
host2(config-if-Po1)#<b>wr</b>
Copy completed successfully.
</pre>

This puts both host1 and host2 on the same subnet (10.1.10.0/24). 

<img src=lab2-images/8.png width="50%" height="50%" border=1>

Host1 and host2 should be able to ping. Ping host1 from host2: 

<pre>
host2(config-if-Po1)#<b>ping 10.1.10.11</b>
PING 10.1.10.11 (10.1.10.11) 72(100) bytes of data.
80 bytes from 10.1.10.11: icmp_seq=1 ttl=64 time=338 ms
80 bytes from 10.1.10.11: icmp_seq=2 ttl=64 time=353 ms
80 bytes from 10.1.10.11: icmp_seq=3 ttl=64 time=348 ms
80 bytes from 10.1.10.11: icmp_seq=4 ttl=64 time=341 ms
80 bytes from 10.1.10.11: icmp_seq=5 ttl=64 time=340 ms

--- 10.1.10.11 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 42ms
rtt min/avg/max/mdev = 338.525/344.535/353.641/5.627 ms, pipe 5, ipg/ewma 10.739/341.325 ms
</pre>


## Adding a Network

In this section, you will add a new Layer 2 network to the fabric, specifically VLAN 20. You will move Host2 to VLAN 20 and the fabric will provide routing between the two subnets/VLANs. 

<img src=lab2-images/9.png width="50%" height="50%" border=1>

### Host Configs

Change the configuration on host1 and host2 to put them onto different subnets. 

For host2, change the IP to 10.1.20.12/24 and ad a static route to get to 10.1.10.0/24 via 10.1.20.1. (Normally this would be done with a default route, but one already exists that's necessary for the lab functionality.)

<pre>
host2#<b>conf</b>
host2(config)#<b>int po1</b>
host2(config-if-Po1)#<b>ip address 10.1.10.12/24</b>
host2(config-if-Po1)#<b>ip route 10.1.10.0/24 10.1.20.1</b>
host2(config)#wr
Copy completed successfully.
</pre>

For host1, you just need to add the static route to get to 10.1.20.0/24. 

<pre>
host1#<b>conf</b>
host1(config)#<b>ip route 10.1.20.0/24 10.1.10.1</b>
host1(config)#wr
Copy completed successfully.
</pre>

### Fabric Configs

Change the ATD_TENANTS_NETWORKS.yml file 

<pre>
tenants:
  # Tenant A Specific Information - VRFs / VLANs
  Tenant_A:
    mac_vrf_vni_base: 10000
    vrfs:
      VRF_A:
        vrf_vni: 10
        svis:
          10:
            name: VLAN_10
            tags: [leaf_group1]
            enabled: true
            ip_address_virtual: 10.1.10.1/24
          <b>20:
            name: VLAN_20
            tags: [leaf_group1]
            enabled: true
            ip_address_virtual: 10.1.20.1/24</b>
</pre>

A second SVI, 20, will be added (additional lines are in bold). SVI IDs correspond to local VLAN IDs. 

Next, edit the ATD_SERVERS.yml file and add a port_profile for VLAN_20 and change host2 to use that profile. The changed lines are bold. 
<pre>
---
port_profiles:
  VLAN_10:
    mode: access
    vlans: "10"
  <b>VLAN_20:
    mode: access
    vlans: "20"</b>

servers:
  host1:
    rack: mlag1
    adapters:
      - type: nic
        switch_ports: [Ethernet6, Ethernet7, Ethernet6, Ethernet7]
        switches: [leaf1,leaf1, leaf2, leaf2]
        profile: VLAN_10
        port_channel:
          state: present
          description: PortChannel_to_host1
          mode: active
  host2:
    rack: mlag2
    adapters:
      - type: nic
        switch_ports: [Ethernet6, Ethernet7, Ethernet6, Ethernet7]
        switches: [leaf3, leaf3, leaf4, leaf4]
        profile: <b>VLAN_20</b>
        port_channel:
          state: present
          description: PortChannel_to_host2
          mode: active
</pre>

Run the fabric build command again.

<pre>➜  AVD git:(main) <b>ansible-playbook playbooks/atd-build.yml</b>
....

PLAY RECAP ****************************************************************
leaf1                      : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2                      : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3                      : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4                      : ok=5    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1                     : ok=13   changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine2                     : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine3                     : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0  
</pre>

Then run the deploy playbook. 
<pre>
➜  AVD git:(main)<b> ansible-playbook playbooks/atd-deploy.yml </b>
...

PLAY RECAP ******************************************************************************************************************
cvp1                       : ok=10   changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
</pre>

This will create 4 tasks in CloudVision (only the leafs are affected). (Note the lines added/removed/changed may be different for you.)

<img src=lab2-images/10.png width="50%" height="50%" border=1>


Run those tasks through a change control. 

<img src=lab2-images/11.png border=1>


Once that change control is complete, log back into host2 if you're not already logged in. 
<pre>
➜  project <b>ssh arista@host2</b>
Last login: Wed Feb  8 20:32:06 2023 from 192.168.0.1
host2#

</pre>

Ping the new first hop for host2 (10.1.20.1). Note that it may take a few attempts in the lab environment for the ARP to resolve. 
<pre>
host2#<b>ping 10.1.20.1</b>
PING 10.1.20.1 (10.1.20.1) 72(100) bytes of data.
80 bytes from 10.1.20.1: icmp_seq=1 ttl=64 time=9.02 ms
80 bytes from 10.1.20.1: icmp_seq=2 ttl=64 time=8.32 ms
80 bytes from 10.1.20.1: icmp_seq=3 ttl=64 time=5.61 ms
80 bytes from 10.1.20.1: icmp_seq=4 ttl=64 time=4.26 ms
80 bytes from 10.1.20.1: icmp_seq=5 ttl=64 time=4.74 ms

--- 10.1.20.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 38ms
rtt min/avg/max/mdev = 4.262/6.393/9.029/1.928 ms, ipg/ewma 9.669/7.585 ms
</pre>

<pre>
host2#<b>ping 10.1.10.11</b>
PING 10.1.10.11 (10.1.10.11) 72(100) bytes of data.
80 bytes from 10.1.10.11: icmp_seq=1 ttl=62 time=36.1 ms
80 bytes from 10.1.10.11: icmp_seq=2 ttl=62 time=45.8 ms
80 bytes from 10.1.10.11: icmp_seq=3 ttl=62 time=36.2 ms
80 bytes from 10.1.10.11: icmp_seq=4 ttl=62 time=30.1 ms
80 bytes from 10.1.10.11: icmp_seq=5 ttl=62 time=53.6 ms

--- 10.1.10.11 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 70ms
rtt min/avg/max/mdev = 30.129/40.412/53.690/8.328 ms, pipe 4, ipg/ewma 17.529/38.521 ms
</pre>

Ping you

