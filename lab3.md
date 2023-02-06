# Upload and Apply Configlets

In this lab you will utilize Ansible and the arista.cvp modules to upload configlets from a file and apply them to various devices. 

In the playbooks directory, open the file upload_configlets.yml. 

<img src=lab3-images/1.png width="50%" height="50%" border=1>

This is a simple playbook to upload a few configlets listed in the CVP_model.yml data model. 
<pre>
---
- name: Playbook for uploading configlets to CloudVision
  hosts: cvp1

  vars_files:
    - ../vars/CVP_model.yml

  tasks:
    - name: Upload configlets
      arista.cvp.cv_configlet_v3:
        configlets: "{{ cvp_configlet_test }}"
        state: present
</pre>

Open the CVP_model.yml data model to see the configlets that are to be uploaded. 

<pre>
path: "{{lookup('env','PWD')}}"

cvp_configlets_test:
 Alias_test: “alias ship show ip interface brief”
 MLAG-Left: "{{ lookup('file','{{path}}/configlets/MLAG-Left.cfg') }}"
</pre>

This playbook will upload two configlets: One called "Alias_test" and one called "MLAG-left". When using the data model, you can specify a configlet directly as a string or reference a file with the <tt>lookup()</tt> function.

Run the playbook with the command <tt>ansible-playbook playbooks/upload_configlet.yml</tt>

<pre>
➜  Ansible-CVP git:(main) <b>ansible-playbook playbooks/upload_configlets.yml</b>

PLAY [Playbook for uploading configlets to CloudVision] ******************************************************************************************

TASK [Upload configlets] ******************************************************************************************
changed: [cvp1]

PLAY RECAP ******************************************************************************************
cvp1                       : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
</pre>

Verify that the configlets were uploaded by searching for the configlets in CloudVision. 

<img src=lab3-images/2.png width="50%" height="50%" border=1>

If you look in the configlets directory, you'll see there's also an MLAG-right.cfg file. Modify the data model (CVP_model.yml) to include this file as well. 

<pre>
path: "{{lookup('env','PWD')}}"

cvp_configlets_test:
 Alias_test: “alias ship show ip interface brief”
 MLAG-Left: "{{ lookup('file','{{path}}/configlets/MLAG-Left.cfg') }}"
 <b>MLAG-Right: "{{ lookup('file','{{path}}/configlets/MLAG-Right.cfg') }}</b>
</pre>

Add the line in bold to your CVP_model.yml file. Then run the playbook again. 

<pre>
➜  Ansible-CVP git:(main) ✗ ansible-playbook playbooks/upload_configlets.yml

PLAY [Playbook for uploading configlets to CloudVision] *********************************************************************************

TASK [Upload configlets] *********************************************************************************
ok: [cvp1]

PLAY RECAP **********************************************************************
cvp1                       : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

</pre>

Verify that MLAG-right was also added in CloudVision. 

<img src=lab3-images/3.png border=1>

Note that under the "notes" section in the right the notes section says "Managed by Ansible". 

## Add configlets to devices

Open up the CVP_model.yml file in the editor again. Add this section at the bottom: 

<pre>
cvp_devices_test:
  - fqdn: 'leaf1'
    parentContainerName: 'Leaf'
    configlets:
      - 'ATD-INFRA'
      - 'leaf1-base'
      - 'MLAG-Left'
  - fqdn: 'leaf2'
    parentContainerName: 'Leaf'
    configlets:
      - 'ATD-INFRA'
      - 'leaf2-base'
      - 'MLAG-Right'
</pre>

This data model has leaf1 and leaf2 and specifies which container they will be located in as well as which configlets are to be applied. This adds "MLAG-Left" to leaf1 and "MLAG-Right" to leaf2. 

Create a new file under the playbooks directory called "apply_configlets.yml". Do this by selecting the playbooks directory and clicking the "+" icon above it. 

<img src=lab3-images/4.png border=1>

Paste the following into the new apply_configlets.yml file: 

<pre>
---
- name: Playbook for applying configlets to devices in CloudVision
  hosts: cvp1

  vars_files:
    - ../vars/CVP_model.yml

  tasks:
    - name: Apply configlets to devices
      arista.cvp.cv_device_v3:
        devices: "{{ cvp_devices_test }}"
        state: present
        apply_mode: strict

</pre>

