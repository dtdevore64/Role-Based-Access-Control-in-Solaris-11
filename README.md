# **RBAC-in-Solaris-11**

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


***Step 4.*** Exit root and let's see if our user bob can run the ```reboot``` command just to make sure that he doesn't have access to it already

```
  reboot
  reboot: permission denied
```

And as you can see he does not have access to reboot

<br><br><br>


***Step 5.*** Let's look at what Profiles the reboot command is under. Profiles in Solaris 11 RBAC allow authorizations and privileged commands to be grouped together and can be assigned to users or to roles.

```
  profiles -a | less
```

Scroll down until you fine ```Maintenance and Repair``` then run the following command:

```
  profiles —p "Maintenance and Repair" info
  
  name=Maintenance and Repair 
  desc=Maintain and repair a system 
  auths=sol aris. admin. edi t/ etc/ no login, sol aris. admin. edi t/ etc/ sys log. conf , 
  sol aris. label . range, sol aris. smf . manage. coreadm, sol aris. smf . manage. system—log, sol 
  aris. smf . val ue. coreadm, sol aris. smf . val ue. power _ conf i g, sol aris. smf . val ue. power_co 
  ntrol , sol aris. system. maintenance 
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

As you can see it has the ```reboot``` command among other commands that are grouped together in the profile ```Maintenance and Repair```. You can also run this command to see where the exact profile is located:

```

  cat / etc/ security/exec_attr.d/core—os | grep —i "Maintenance and Repair
  
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

  

