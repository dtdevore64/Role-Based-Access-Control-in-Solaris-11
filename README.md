# **Role Based Access Control in Solaris 11**

<br><br>

What is RBAC(Role Based Access Control)? RBAC provides a more secure alternative to the all-or-nothing superuser model. With RBAC, you can enforce security policy at a more fine-grained level. RBAC uses the security principle of least privilege. Least privilege means that a user has precisely the amount of privilege that is necessary to perform a job. Regular users have enough privilege to use their applications, check the status of their jobs, print files, create new files, and so on. Capabilities beyond regular user capabilities are grouped into rights profiles. Users who are expected to do jobs that require some of the capabilities of superuser assume a role that includes the appropriate rights profile.
<br><br>

In this example I will create a new user and create a new role that can only shutdown the machine and that is the only privilege they will have
<br><br>


***Step 1.*** Log into Solaris 11 and log into the root role.

<br><br>


***Step 2.*** Create a new user named ```bob``` and create a password for ```bob```

```
   useradd -m bob
   passwd bob
```

<br><br><br>


***Step 3.*** Assign the ```root``` role to the newly created user ```bob```

```
  usermod -R +root bob
```
<br><br><br>



***Step 4.*** Let's look at what Profiles the reboot command is under. Profiles in Solaris 11 RBAC allow authorizations and privileged commands to be grouped together and can be assigned to users or to roles.

```
  profiles -a | less
```
<br><br>

Scroll down until you fine ```Maintenance and Repair``` then run the following command:

```
  profiles —p "Maintenance and Repair" info
  
  name=Maintenance and Repair 
  desc=Maintain and repair a system 
  auths=solaris.admin.edit/etc/nologin,solaris.admin.edit/etc/syslog.conf, 
  solaris.label.range,solaris.smf.manage.coreadm,solaris.smf.manage.system—log, solaris.smf.value.coreadm,solaris.smf.value.power_config,solaris.smf.value.power_control,solaris.system.maintenance 
  profiles=Hotplug Management, Service Configuration 
  cmd=/usr/sbin/ubiosconfig 
  cmd=/usr/sbin/biosconfig 
  cmd=/usr/sbin/itpconfig 
  cmd=/usr/sbin/ilomconfig 
  cmd=/usr/bin/mdb 
  cmd=/usr/bin/coreadm 
  cmd=/usr/sbin/croinfo 
  cmd=/usr/sbin/eeprom 
  cmd=/usr/sbin/halt 
  cmd=/usr/sbin/init 
  cmd=/usr/sbin/pcitool 
  cmd=/usr/sbin/poweroff 
  cmd=/usr/sbin/reboot 
  cmd=/usr/sbin/shutdown 
  cmd=/usr/sbin/syslogd 
  cmd=/usr/sbin/bootadm 
  cmd=/usr/sbin/ucodeadm 
  cmd=/usr/sbin/cpustat 
  cmd=/usr/sbin/sysadm 
  cmd=/usr/lib/rad/module/mod_sysmgr.so.1 
  cmd=/usr/sbin/hwmgmtc 
  cmd=/usr/sbin/fwupdate
```
<br><br>

As you can see it has the ```reboot``` command among other commands that are grouped together in the profile ```Maintenance and Repair```. You can also run this command to see where the exact profile is located:
<br><br>

```

  cat /etc/security/exec_attr.d/core—os | grep —i "Maintenance and Repair
  
  Maintenance and Repair:solaris:cmd:RO::/usr/bin/mdb:privs—all 
  Maintenance and Repair:solaris:cmd:RO::/usr/bin/coreadm:eui;privs=proc_owner 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/croinfo: euid=0 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/eeprom: eui 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/halt: euid=0 
  Maintenance and Repair:solaris:cmd:R0::/usr/sbin/init: uid=0 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/pci tool : pr i vs—all 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/poweroff: u i 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/reboot: uid=0 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/shutdown: u i 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/sys logd: euid=0 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/bootadm: eui 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/ucodeadm: pr i vs—all 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/cpustat:pr i vs—basic, cpc_cpu 
  Maintenance and Repair:solaris:cmd:RO::/usr/sbin/sysadm: euid=0 
  Maintenance and Repair:solaris:cmd:RO::/usr/l i b/rad/module/mod_sysmgr.so.1:euid=0
```

<br><br><br>


***Step 5.*** Create our new role that can only reboot the machine. You will notice the shell ```/bin/pfbash```. This shell enables a user to execute a command in a profile.
```
   roleadd -m -s /bin/pfbash reboot_r
```

<br><br>

To see where it created our new role at run the following command:

```
   grep reboot_r /etc/passwd
   reboot_r:x:103:10::/export/home/reboot_r:/usr/bin/bash
```
   
<br><br><br>
   
   
***Step 6.*** We just want our new role to be able to run the ```reboot``` command and nothing else so we will not add the ```Maintenance and Repair``` profile to our new role and user ```bob```. Instead we will create a new profile and assign only the ```reboot``` command to it. If you want to see a list of all the system defined profiles then run the following command:

```
   less /etc/security/prof_attr.d/core-os
```
<br><br>


Instead of editing or adding to the already defined system profiles in the ```/etc/security/prof_attr.d``` directory we will add our local profile into the ```/etc/security/prof_attr``` file like so(all new profiles should be written to this file).

```
   nano /etc/security/prof_attr
   
   #
   # The system provided entires are stored in different files
   #under "/etc/security/prof_attr.d". They should not be
   # copied to this file
   #
   # Only local changes should be stored in this file.
   # This line should be kept in this file or it will be
   overwritten.
   #
   
   Reboot:RO::\
   For authorized users to reboot the system:\
```

We can tell from this file that our profile name ie ```Reboot``` and the ```RO(read-only)``` characters tell us that it is not modifiable by any tool that changes this database.

<br><br><br>


***Step 7.*** Since the profile is created we have to assign one or more commands to this profile and we will start by editing the ```/etc/security/exec_attr``` file:

```
   nano /etc/security/exec_attr
   
   #
   # The system provided entires are stored in different files
   # under "/etc/security/exec_attr.d". They should not be
   # copied to this file.
   #
   # Only local changes should be stored in this file.
   # This line should be kept in this file or it will be
   overwritten.
   #
   Reboot:solaris:cmd:RO::/usr/sbin/reboot:uid=0
```

Explanation of the last line:
<br><br>

```reboot = profile name```
<br>
```solaris = security policy associated with reboot profile```
<br>
```cmd = type of object; command to be executed by a shell```
<br>
```RO = line is not modifiable by any tool that changes this file```
<br>
```/usr/sbin/reboot = command to be executed by a user when they assume the role that contains reboot profile```
<br>
```uid=0 = user runs the command and the command will be executed to run by a root user```

<br><br><br>


***Step 8.*** Map the ```Reboot``` profile to the ```reboot_r``` role

```
   rolemod -P Reboot reboot_r
```
<br><br><br>



***Step 9.*** Set a new password for our new ```reboot_r``` role

```
   passwd reboot_r
```
<br><br><br>


***Step 10.*** Assign the ```reboot_r``` role to our user ```bob```

```
   usermod -R reboot_r bob
```
<br><br>

Run the following to verify that ```bob``` has the ```reboot_r``` role assigned to him

```
   more /etc/user_attr
   
   #
   # Copyright (c) 1999, 2013, Oracle and/or its affiliates. All rights reserved.
   #
   # The system provided entires are stored in different files
   # under "/etc/user_attr.d". They should not be copied to this file.
   #
   # Only local changes should be stored in this file.
   #
   root::::type=role
   joe::::lock_after_retires=no;roles=root;clearance=ADMIN_HIGH;min_label=ADMIN_LOW
   ;auth_profiles=System Adminstrator
   bob::::roles=reboot_r
   reboot_r::::profiles=Reboot;type=role;roleauth=role
```

<br><br><br>


***Step 11.*** Log in as Bob and try to run the command ```reboot```

```
   reboot
   reboot: permission denied
```

<br><br>


It denied the permission because you have to transition to the ```reboot_r``` role like so:

```
   su - reboot_r
   Password:
   reboot_r@solaris:~$ reboot
```
<br>
As you can see it rebooted the machine just fine and everything works with no issues.
   



   


   
   








  

