# Infrastructure Lab

This lab will guide you through the process of creating a baseline environment that we'll use for out kubernetes installation. I have tried this on Windows-11 Pro (version 21H2).

- [VirtualBox](#Virtual-Box)
- [GNS3](#GNS3)
- [Virtual Machines](#Virtual-Machines)
- [IP addresses for nodes](#IP-addresses-for-nodes)
- [Add your machines in GNS3](#Add-your-machines-in-GNS3)
- [Connect to master and nodes](#Connect-to-master-and-nodes)

## Virtual Box

- I am using VirtualBox, although GNS3 documentation says that vmware workstation is a better solution. This is because some features of vmware Workstation are not supported om the free version(Player).
- install (or upgrade) [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## GNS3
- Install [GNS3](https://docs.gns3.com/docs/getting-started/installation/windows)
- To upgrade first uninstall (win-x -> apps & features -> find gns3 -> uninstall)
- Here's the tricky part:
you should download the VirtualBox (or vmware workstations if you go for it), then make GNS3 use it, [as explained in the guide](https://docs.gns3.com/docs/getting-started/installation/download-gns3-vm).
- You should use the VM from gns3 (in GNS3 go to help-> setup wizard). Increase vcpu and memory when you do so
- If all goes well, when you run GNS3 you should see two green lights:
        Desktop...
        GNS VM....
so that you know that GNS3 is running the two servers (although you will probably not use the local dynamips server).

## Virtual Machines

- We are using "Stand Alone" servers for gns3, so don't look for an appliance
- Download Centos 8.4 from osboxes.org 
    link: https://www.osboxes.org/centos/
    user: osboxes pass: osboxes.org
- Unzip the downloaded file
- Create a Linux (RedHat 64G) machine
- 8192 MB of RAM
- use your downloaded file as the system disk.
- Leave networking as "Not Attached" (let GNS3 handle this)
- Clone it carefully - to create 3 workers and 1 master:
     - Clone when machine is not working
     - Rename your new machine (master, worker..etc)
     - right-click clone
     - create new MAC addresses
     - Full clone !!!
  - Make sure your new machines can work

--------------------------------------------

## IP addresses for nodes

- These are Centos nodes.
- Address range should be: 192.168.122.0/24  (I'll explain why later)
- Configure static IP addresses for master and workers:
   sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33
   change:
	BOOTPROTO=static
	IPADDR=192.168.122.x (where x is 1,2,3 for workers, 10 for master)
	NETMASK=255.255.255.0
	GATEWAY=192.168.122.100
   to restart networking:
	nmcli networking off
	nmcli networking on

## Add your machines in GNS3

- First, both "local server" and "GNS3 VM" server should run.
    They start automatocally when you run GNS3, but it may take a minute.
- Start edit/preferences, look for VirtualBox and add your machines from there.
- Add a simple Switch
- Connect all machines to the switch
- Add a "Nat Device"

## Connect to master and nodes

I'm using a local Linux (Ubuntu-20) as a host, to control master and nodes.

- Create a new ssh keypair: 
          ssh-keygen 
- copy the key to each machine (you may have problems with fingerprints..)
          ssh-copy-id osboxes@192.168.122.1
- Now connect like this:
          ssh osboxes@192.168.122.1

Use Terminator (See here: https://dev.to/xeroxism/how-to-install-terminator-a-linux-terminal-emulator-on-steroids-1m3h)
You can then login and command all nodes at once.