# Using Arista AVD (arista.avd)

Arista AVD is an advanced set of playbooks, data models, templates, roles, and modules that allow for the generation of various types of fabric configurations, as well as deployment options, automatic documentation, and post-deployment validation. 

With just a few data models, AVD can generate complex EVPN/VXLAN configurations for dozens (or even hundreds) of switches.

In this lab, you will create an L3LS+EVPN/VXLAN configuration with just a few files. 

## Prepare Data Models

Select the AVD directory.

<img src=lab4-images/1.png border=1>

You'll see several directories and files:

* atd-inventory
* atd-reset
* docs
* playbooks
* roles
ansible.cfg

## Cleanup

Chand to the AVD directory.

<pre>
➜  project <b>cd ~/project/persist/Advanced-CVP/AVD</b>
➜  AVD git:(main) 
</pre>

Run the cleanup playbook. This again will reset the lab to the default configlets. 

<pre>
➜  AVD <b>ansible-playbook playbooks/atd-reset-labs.yml</b>
[DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is being removed. This feature will be removed from ansible-core in version 2.14. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [Assign configlets to devices (via group_vars/CVP.yml)] *************************************************************************************

TASK [Build default container topology] *************************************************************************************
ok: [cvp1]

TASK [Apply configlets for default config] *************************************************************************************

PLAY RECAP **************************************************************************
cvp1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
</pre>

Run any tasks through a change control. 

## Build and Deploy

The process of deploying an L3LS+EVPN configuration will be done in two steps: Build and deploy. 

The build process will take the three data models (ATD_FABRIC.yml, ATD_SERVERS.yml, and ATD_TENANTS_NETWORKS.yml) and use them to build a complete configuration in the form of configlets written in EOS syntax. 

The deploy process will take those configlets, upload them to CloudVision, and attach them to the various devices. This will generate tasks that the operator can run through a change control. 

### Build AVD Configuration

Run the atd-build.yml playbook. The output below has been truncated. (Note: You may see several warning in magenta, they can be ignored.)

<pre>
  AVD git:(main) ✗ <b>ansible-playbook playbooks/atd-build.yml</b>
...
PLAY RECAP ******************************************************************************************************************************************************************
leaf1                      : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf2                      : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf3                      : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
leaf4                      : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine1                     : ok=13   changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine2                     : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
spine3                     : ok=5    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
</pre>

This will create a directory called "intended". Open that configuration and you will see leaf1, leaf2, leaf3, leaf4, and spine1, spine2, and spine3 configurations. 

<img src=lab4-images/2.png border=1>

You can click on them to explore the configurations generated. leaf1.cfg, for example, is over 200 lines of newly generated configuration. 

Now deploy that configuration to CloudVision: 

<pre>
  AVD git:(main) ✗ <b>ansible-playbook playbooks/atd-deploy.yml</b>
[DEPRECATION WARNING]: COMMAND_WARNINGS option, the command warnings feature is being removed. This feature will be removed from ansible-core in version 2.14. Deprecation 
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [Deploy created AVD Configuration] *************************************************************************************************************************************

TASK [arista.avd.eos_config_deploy_cvp : Create required output directories if not present] *********************************************************************************
ok: [cvp1 -> localhost] => (item=/home/coder/project/persist/Advanced-CVP/AVD/atd-inventory/intended/structured_configs/cvp)
...
PLAY RECAP ******************************************************************************************************************************************************************
cvp1                       : ok=10   changed=1    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   

</pre>

This will create 7 tasks for leaf1-4 and spine1-3 (the borderleafs have not been included).

<img src=lab4-images/3.png border=1>

Run the 7 tasks through change control. When completed, do a spot-verify that EVPN/VXLAN is configured by logging into leaf1 and using the command <tt>show bgp evpn summary</tt>. 

<pre>
leaf1#<b>show bgp evpn summary</b>
BGP summary information for VRF default
Router identifier 192.168.101.1, local AS number 65100
Neighbor Status Codes: m - Under maintenance
  Description              Neighbor       V AS           MsgRcvd   MsgSent  InQ OutQ  Up/Down State   PfxRcd PfxAcc
  spine1                   192.168.101.11 4 65001             23        28    0    0 00:03:20 Estab   8      8
  spine2                   192.168.101.12 4 65001             25        26    0    0 00:03:22 Estab   8      8
  spine3                   192.168.101.13 4 65001             24        27    0    0 00:03:21 Estab   8      8
</pre>

As you can see, the EVPN overlay is functining (Estab state). However, these types of spot checks have limited use, since leaf1 being operational does not mean the other leafs are. 

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


<img src=lab4-images/4.png border=1>

Click on the ATD_FABRIC-state.md file. This will display the source of the markdown. To see it in a web browser display style, click on the book-with-magnifying glass icon in the upper right. 

<img src=lab4-images/5.png border=1>

This will bring up a browser-style view. You can move the window slider to see more of the Validate State Report.

<img src=lab4-images/6.png border=1>

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

<img src=lab4-images/7.png border=1>

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

<img src=lab4-images/8.png width="50%" height="50%" border=1>

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

<img src=lab4-images/9.png width="50%" height="50%" border=1>

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

<img src=lab4-images/10.png width="50%" height="50%" border=1>


Run those tasks through a change control. 

<img src=lab4-images/11.png border=1>


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

