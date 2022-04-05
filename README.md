# Configuring virtwho to work with the Red Hat Customer Portal

In this tutorial we will look at configuring virtwho to provide hypervisor host information when the RHEL VM is registered to the customer portal.

For this tutorial I created a small RHEL VM (1 vCPU wth 2 GB RAM) running on VMWare to host virtwho.

I setup an activation key in the Red Hat customer portal with RHEL standard support subscription attached.  I then registered the system with the activation key to Red Hat customer portal.  **Note:** I have Simple Content Access enabled on my Red Hat customer portal.

```
 # subscription-manager register --org=xxxxxxxxx --activationkey=your_key_here
 ```
 
 I checked to make sure that the RHEL 8 repos are enabled on the VM.
 ```
 # subscription-manager repos --list-enabled
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
system identity: 2278e9a5-90d2-41d3-ba51-2b605607f2ba
name: virt-who.example.com
org name: #######
org ID: #######
```

Now create and edit the virt-who.conf file.
```
# vi virt-who.conf
[vmware]
type=esx
server=vsca01.example.com
username=user
password=password
owner=#######
env=Library
hpervisor_id=hostname
```



## References
- [Configuring virt-who with Red Hat Subscription Management](Configuring virt-who with Red Hat Subscription Management) - YouTube Video
- [Why and when do I need Virt-Who?](https://access.redhat.com/articles/1300283)
- [How to configure virt-who for an ESX/ESXi host with RHSM?](https://access.redhat.com/solutions/3243861)
- [Using RHEL Virtual Data Center Subscription - Master Article]](https://access.redhat.com/solutions/3243071)

