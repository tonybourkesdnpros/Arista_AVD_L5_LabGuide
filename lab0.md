# 0. Lab Overview

Every student will have a set of virtual cloudEOS instances (cEOS) set in several leaf and spine topologies. There will also be a single instance of CloudVision Portal (CVP). 

<img src=lab0-images/1.png width="50%" height="50%" border=1>


## Lab Rules: 

* Do not change or remove the Management IP address.
* Do not change or remove the eAPI configuration.
* Do not change or remove pre-configured users.
* Do not change or remove the hostname.
* Do not add a password to the admin user.
* Do not add an enable secret password.

# 2. Connection to Lab Environment

## 2.1 Lab Objectives

In this activity you will learn how to: 

* Connect to your lab environment
* Ensure the connectivity to the lab environment

## 2.2 Connect To Your Lab Environment

You will receive a link from SDN Pros for your lab environment. It will contain over 20 virtualized switches (cEOS) in various interconnected Clos architectures. For the most part, everything that works in a physical EOS instance will work (including data and control plane). There are, of course, some exceptions (ASIC-based telemetry, ACLs, etc.).

When you open the link, the lab should be in the “shutdown” state. Click “START” to start the environment.

<img src=lab0-images/2.png border=1>

This will start the VM the lab environment is based in, as well as starting CloudVision Portal (CVP). The startup process will take 10-15 minutes to fully complete. The switches may start up quicker, but the various components of CVP may take 10-15 minutes to fully instantiate.

Once the lab environment is started, it will continue for 8 hours. At the end of 8 hours, the environment automatically shuts down (there is no way to stop this). You can, however, start the environment back up immediately (but it will take the same 10-15 minutes to complete the startup process).

If the lab is idle (no interaction) it will shut down after about 3 hours.

The main link will show how long until shutdown commences. It will also show when the lab environment will discontinue entirely.

<img src=lab0-images/3.png border=1>

The lab environment is stateful, as in the state of the environment (configurations, etc.) will persist across shutdowns. If cEOS switch configurations are saved, they will persist when you start the environment back up. The same is true for CloudVision, any configlets, dashboards, etc., will be there when the environment is started back up.

Warning: If you do not save a cEOS configuration (i.e., “copy run start”), when the environment shuts down, the configuration isn’t saved. If you push a configlet from CVP, however, the configuration is automatically saved.

Once the environment has started up (10-15 minutes after clicking “START”) you can click the “Click Here to Access Topology” button.

<img src=lab0-images/4.png border=1>

## 2.3: CloudVision Portal (CVP)

Click on the CVP link in the main lab link. 

<img src=lab0-images/cvp1.png border=1>

In your lab environment, click on Programmability IDE link. 

<i>Note: If the link is grayed-out, CVP has not completed its startup sequence (which can take 10-15 minutes)</i>

Log in using the credentials list in your lab interface.

<img src=lab0-images/cvp2.png border=1>

This will drop you into the device inventory page. 

<img src=lab0-images/cvp3.png border=1>

Feel free to explore this tab. After that, click on the Provisioning tab at the top. Then click the the icon indicated on the right. 

<img src=lab0-images/cvp4.png border=1>

This will show the a map of the switches and the containers they're organized under. 

<img src=lab0-images/cvp5.png border=1>

Feel free to explore the rest of the CVP environment. 

## 2.4: The IDE (Integrated Development Environment)

The Arista lab environment runs what's known as an IDE (Integrated Development Environment) called code-server. It's based on Microsoft's Visual Studio Code (VSCode) which is currently one of the most often used IDEs, especially with network automation. The code-server project takes VSCode and runs it as a web application server. Both VSCode and code-server are open source and free. 

## Exploring code-server


<img src=lab0-images/6.png width="50%" height="50%" border=1>

It will ask for a password. It's the same password used for CVP, and can be found in the lower right hand corner of your initial lab page. 

<img src=lab0-images/7.png border=1>

This will bring up code-server. In this environment, you can create files and directories, sync up with GitHub/GitLab repositories, make your own repos and so forth. 

You may see some pop-ups asking to trust authors, activate themes, etc. Select yes/activate on them. 


<img src=lab0-images/8.png width="50%" height="50%" border=1>

<img src=lab0-images/9.png border=1>


You can X the windows for "Get Started" and "Welcome to GitLens" (and any others) and dismiss any notifications. 

<img src=lab0-images/10.png width="75%" height="75%" border=1>

When you're done you'll have a screen that looks like this: 

<img src=lab0-images/11.png width="75%" height="75%" border=1>

In the upper left you'll see the "hamburger" icon. It's called the hamburger icon as it resembles a hamburger with its three layers. Under that icon, select "Terminal" and then "New Terminal". 

<img src=lab0-images/12.png width="75%" height="75%" border=1>

This will open up a terminal section at the bottom. This is where most of the work will be done. You can open up multiple terminals with the "+" icon in the upper right, as well as resize the terminal to take up the entire screen or a small portion of the screen. Feel free to play around with the interface. 

<img src=lab0-images/13.png width="75%" height="75%" border=1>

In this window, confirm that Ansible is installed with the command <b><tt>ansible --version</tt></b>

<pre>
➜  project <b>ansible --version</b>
ansible [core 2.12.8]
  config file = /home/coder/.ansible.cfg
  configured module search path = ['/home/coder/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/dist-packages/ansible
  ansible collection location = /home/coder/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
  jinja version = 3.1.2
  libyaml = True

</pre>

Use the Linux <b><tt>cd</tt></b> command to change directories into the persist directory which is already there. The perist directory will persist (as the name implies) between reboots of the lab environment. Most files and subdirectories outside of either persist or labfiles will be erased upon reboot. 

<pre>
➜  project <b>cd persist</b> 
➜  persist 
</pre>

To simplify creation of the various files, there is a Github repository that you will clone. Use the command <tt><b>git clone https://github.com/tonybourkesdnpros/Arista_AVD_L5.git</b></tt>

<pre>
➜  persist <b>git clone https://github.com/tonybourkesdnpros/Arista_AVD_L5.git</b>
Cloning into 'Arista_AVD_L5'...
remote: Enumerating objects: 67, done.
remote: Counting objects: 100% (67/67), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 67 (delta 14), reused 67 (delta 14), pack-reused 0
Receiving objects: 100% (67/67), 1.74 MiB | 10.89 MiB/s, done.
Resolving deltas: 100% (14/14), done.
➜  persist 
</pre>

This will clone the git repository onto your lab environment. In the upper left corner click on the "persist" directory which will open up the directory to show you the new Arista_TD_Labs directory. 

<img src=lab0-images/14.png width="75%" height="75%" border=1>


<img src=lab0-images/15.png width="75%" height="75%" border=1>
