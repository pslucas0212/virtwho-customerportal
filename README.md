# Configuring virt-who with vSphere to Report Hypervisor Host Information to the Red Hat Customer Portal

If you are not running Red Hat Satellite as part of your Red Hat Enterprise Linux (RHEL) managment environment, you may still have the need to run virt-who separeatly to track the deployment of RHEL VMs with RHEL or Virtual Data Center subscriptions.  If you have Simple Content Access enabled on your customer portal, you would not have the need to attach a specific subscription to the RHEL VM.  With SCA enabled as the consumer of Red Hat subscriptions you will still need to track your usage to be compliant with your Red Hat agreement, and virt-who can assist with this.

In this tutorial we will look at configuring virt-who to provide vSphere hypervisor host information when the RHEL VM is registered to the customer portal.  We can see the RHEL VM information in both the customer portal and the Insights console.

For this tutorial I created a small RHEL VM (1 vCPU wth 2 GB RAM) running on VMWare to host the virt-who daemon.

I registered the system with the activation key to Red Hat customer portal. Remember I have Simple Content Access enabled on my Red Hat customer portal.

```
# subscription-manager register --org=xxxxxxxxx --activationkey=your_key_here
```

When using SCA, I am not attaching a subscription to my RHEL VM. When you run subscription-manage status, you will see that SCA is enabled and that you can attach any required content repositories to your RHEL VM.
```
# subscription-manager status
+-------------------------------------------+
   System Status Details
+-------------------------------------------+
Overall Status: Disabled
Content Access Mode is set to Simple Content Access. This host has access to content, regardless of subscription status.

System Purpose Status: Disabled
```
 
I checked to make sure that the RHEL 8 repos are enabled on the VM.  When creaing the RHEL 8 VM from an ISO image on vSphere, the RHEL 8 repos are automatically enabled.
```
# subscription-manager repos --list-enabled
+----------------------------------------------------------+
    Available Repositories in /etc/yum.repos.d/redhat.repo
+----------------------------------------------------------+
Repo ID:   rhel-8-for-x86_64-appstream-rpms
Repo Name: Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
Repo URL:  https://cdn.redhat.com/content/dist/rhel8/$releasever/x86_64/appstream/os
Enabled:   1

Repo ID:   rhel-8-for-x86_64-baseos-rpms
Repo Name: Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)
Repo URL:  https://cdn.redhat.com/content/dist/rhel8/$releasever/x86_64/baseos/os
Enabled:   1
```

Check to see if virt-who is installed.
```
# rpm -qa virt-who
```

If virt-who is not installed, let's install it.
```
# yum -y install virt-who
...
Complete!
```

The virt-who installation creates template configuration file that we can use to setup virt-who for environment.  The template configuration file is located under the /etc/virt-who.d/ directory.  


We will create the virt-who.conf file. Before edting the file we need the organzation id under which the RHEL VM is registered. 
```
# subscription-manager identity
system identity: 2278e9a5-...-2b605607f2ba
name: virt-who.example.com
org name: #######
org ID: #######
```

The vCenter user ID needs read-only access to all objects in the vCenter.  I created a user ID named 'virt-who' on my vCenter and gave it read-only access.

Now create and edit the virt-who.conf file.  This configuration file is self-explanatory.  The hypervisor_id setting is a default setting we need in the configuration file.  
```
# vi virt-who.conf
[vmware]
type=esx
server=vsca01.example.com
username=virt-who@vsphere.local
password=password
owner=#######
hypervisor_id=hostname
```

If you want to exclude exsi hosts from virt-who reporting for a particular vCenter add the filert hosts opton to the virt-who.conf file.  Use commas to separate host names.  See the following example filter hosts line entry.
```
filter_hosts=esx02.example.com, esx04.example.com
``
If you have more than one vCenter, simply create as many sections in the configuration file to match the number of vCenters.
```
[vcenter01]
type=esx
server=vsca01.example.com
username=virt-who@vsphere.local
password=password
owner=#######
hypervisor_id=hostname

[vcenter02]
type=esx
server=vsca02.example.com
username=virt-who@vsphere.local
password=password
owner=#######
hypervisor_id=hostname
```

Before starting virt-who you manually test the configuration.
```
# virt-who --print
```

For an easier readout of the configuration test, try the following command.
```
# virt-who --print | jq
```

Let's start the virt-who daemon.
```
# systemctl start virt-who
```

Let's check the status virt-who
```
# systemctl status virt-who
```

Make virt-who startup automatically when the server is rebooted.
```
# systemctl enable virt-who
Created symlink /etc/systemd/system/multi-user.target.wants/virt-who.service → /usr/lib/systemd/system/virt-who.service.
```

Let's encrypt the password contained in the virt-who.conf file.

To do that run the virt-who-password command and supply the vCenter password.
```
virt-who-password
Password: 
Use following as value for encrypted_password key in the configuration file:
14809...98a885d4cecd
```

Change your virt-who.conf file to the following.
```
# vi virt-who.conf
[vmware]
type=esx
server=vsca01.example.com
username=virt-who@vsphere.local
encrypted_password=14809...98a885d4cecd
owner=#######
hypervisor_id=hostname
```

You can test this new configuration using the same command above.  Remember to restart the virt-who daemon.
```
# systemctl restart virt-who
```


## References
- [Configuring virt-who with Red Hat Subscription Management](https://www.youtube.com/watch?v=0KptauyDAxE) - YouTube Video
- [Why and when do I need Virt-Who?](https://access.redhat.com/articles/1300283)
- [How to configure virt-who for an ESX/ESXi host with RHSM?](https://access.redhat.com/solutions/3243861)
- [Using RHEL Virtual Data Center Subscription - Master Article](https://access.redhat.com/solutions/3243071)

