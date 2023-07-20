

## The IDE (Integrated Development Environment)

The Arista lab environment runs what's known as an IDE (Integrated Development Environment) called code-server. It's based on Microsoft's Visual Studio Code (VSCode) which is currently one of the most often used IDEs, especially with network automation. The code-server project takes VSCode and runs it as a web application server. Both VSCode and code-server are open source and free. 

## Exploring code-server

In your lab environment, click on Programmability IDE link.

<img src=lab1-images/1.png width="50%" height="50%" border=1>

It will ask for a password. It's the same password used for CVP, and can be found in the lower right hand corner of your inital lab page. 

<img src=lab1-images/2.png border=1>

This will bring up code-server. In this environment, you can create files and directories, sync up with GitHub/GitLab repositories, make your own repos and so forth. 

You may see some pop-ups asking to trust authors, activate themes, etc. Select yes/activate on them. 


<img src=lab1-images/3.png width="50%" height="50%" border=1>

<img src=lab1-images/4.png border=1>


You can X the windows for "Get Started" and "Welcome to GitLens" (and any others) and dismiss any notifications. 

<img src=lab1-images/5.png width="75%" height="75%" border=1>

When you're done you'll have a screen that looks like this: 

<img src=lab1-images/6.png width="75%" height="75%" border=1>

In the upper left you'll see the "hamburger" icon. It's called the hamburger icon as it resembles a hamburger with its three layers. Under that icon, select "Terminal" and then "New Terminal". 

<img src=lab1-images/7.png width="75%" height="75%" border=1>

This will open up a terminal section at the bottom. This is where most of the work will be done. You can open up multiple terminals with the "+" icon in the upper right, as well as resize the terminal to take up the entire screen or a small portion of the screen. Feel free to play around with the interface. 

<img src=lab1-images/8.png width="75%" height="75%" border=1>

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

To simplify creation of the various files, there is a Github repository that you will clone. Use the command <tt><b>git clone https://github.com/tonybourkesdnpros/TD_AVD_L5.git</b></tt>

<pre>
➜  persist <b>git clone https://github.com/tonybourkesdnpros/TD_AVD_L5.git</b>
Cloning into 'TD_AVD_L5'...
remote: Enumerating objects: 67, done.
remote: Counting objects: 100% (67/67), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 67 (delta 14), reused 67 (delta 14), pack-reused 0
Receiving objects: 100% (67/67), 1.74 MiB | 10.89 MiB/s, done.
Resolving deltas: 100% (14/14), done.
➜  persist 
</pre>

This will clone the git repository onto your lab environment. In the upper left corner click on the "perist" directory which will open up the directory to show you the new TD_AVD_L5 directory. 

<img src=lab1-images/9.png width="75%" height="75%" border=1>


<img src=lab1-images/10.png width="75%" height="75%" border=1>

## Configure Hosts

