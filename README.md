#Docker and Cgroup documentation to create custom and adaptative containers

## Limit Network bandwidth using docker

-------



[Source](http://serverfault.com/questions/382547/how-to-limit-network-usage-forhttps://www.google.fr/search?q=f&client=ubuntu&hs=iyn&channel=fs&source=lnms&tbm=isch&sa=X&ei=GLFsVPn_B8ffaJy1gZAO&ved=0CAoQ_AUoAw-concrete-application-in-linux-that-is-running-in) 

[Network Control with C group](  http://vger.kernel.org/netconf2009_slides/Network%20Control%20Group%20Whitepaper.odt) 

[lxc user](https://lists.linuxcontainers.org/pipermail/lxc-users/2011-February/001452.html) 

* Container creation :

```
sudo docker run --cap-add=NET_ADMIN  --lxc-conf="lxc.cgroup.net_cls.classid = 0x00100001"  -i -t ubuntu:14.04 /bin/bash
```

After the container creation a virtual network interface is created : 

```
vethfaeed83 Link encap:Ethernet  HWaddr 46:dc:df:35:a4:18  
          adr inet6: fe80::44dc:dfff:fe35:a418/64 Scope:Lien
          UP BROADCAST RUNNING  MTU:1500  Metric:1
          Packets reçus:7 erreurs:0 :0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 lg file transmission:1000 
          Octets reçus:578 (578.0 B) Octets transmis:578 (578.0 B)
```
Apply the following rules on this interface, it will limit the incoming traffic of the container :
```
tc qdisc add dev veth8f4c531 handle 1: root htb default 11
tc class add dev veth8f4c531 parent 1: classid 1:1 htb rate 1Mbps
tc class add dev veth8f4c531 parent 1: classid 1:11 htb rate 512kbit
```

In the docker container appy these rules, theyitwill limit the outgoing traffic of the container

```
 tc qdisc add dev eth0 root handle 1: htb default 30
  tc class add dev eth0 parent 1: classid 1:2 htb rate 1mbit
  tc filter add dev eth0 protocol ip parent 1:0 prio 1 handle 1: cgroup
```
###Test :

*Container creation
```
sudo docker run --cap-add=NET_ADMIN  --lxc-conf="lxc.cgroup.net_cls.classid = 0x00100001"  -i -t ubuntu:14.04 /bin/bash
apt-get update
apt-get install wget
```
* Download a file without any rate limitation :

```
time(wget http://www.obeo.fr/dnload/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip)
```

Output :
```
root@b14758fbf91c:/# time(wget http://www.obeo.fr/download/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip)
--2014-11-20 09:09:14--  http://www.obeo.fr/download/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip
Resolving www.obeo.fr (www.obeo.fr)... 91.121.50.185
Connecting to www.obeo.fr (www.obeo.fr)|91.121.50.185|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 376860906 (359M) [application/zip]
Saving to: 'ObeoDesigner-6.2-linux.gtk.x86_64.zip'

100%[======================================================================================================================>] 376,860,906 5.83MB/s   in 61s    

2014-11-20 09:10:16 (5.88 MB/s) - 'ObeoDesigner-6.2-linux.gtk.x86_64.zip' saved [376860906/376860906]


real	1m1.128s
user	0m0.902s
sys	0m8.835s
```

* Bandwith reduction :

On the host use ifconfig to find the virtual network interface :
```
vethfaeed83 Link encap:Ethernet  HWaddr 46:dc:df:35:a4:18  
          adr inet6: fe80::44dc:dfff:fe35:a418/64 Scope:Lien
          UP BROADCAST RUNNING  MTU:1500  Metric:1
          Packets reçus:137774 erreurs:0 :0 overruns:0 frame:0
          TX packets:238764 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 lg file transmission:1000 
          Octets reçus:9099838 (9.0 MB) Octets transmis:414269121 (414.2 MB)
```
To limit incoming traffic at 512 kbit: 
```
sudo tc qdisc add dev vethfaeed83 handle 1: root htb default 11
sudo tc class add dev vethfaeed83 parent 1: classid 1:11 htb rate 512kbit
```

In the docker node :
```
 tc qdisc add dev eth0 root handle 1: htb default 30
tc class add dev eth0 parent 1: classid 1: htb rate 512kbit
```














sudo docker run -t -i --cap-add=NET_ADMIN --lxc-conf="lxc.cgroup.net_cls.classid = 0x10002" ubuntu:14.04 /bin/bash


sudo docker run --cap-add=NET_ADMIN --lxc-conf="lxc.cgroup.cpuset.cpus = 1" --lxc-conf="lxc.cgroup.net_cls.classid = 0x00100001" --lxc-conf="lxc.cgroup.blkio.throttle.write_bps_device = 8:0 1048576" -i -t ubuntu:14.04 /bin/bash


docker run --lxc-conf="lxc.cgroup.blkio.throttle.write_bps_device = 8:1 8389000" --lxc-conf="lxc.cgroup.blkio.throttle.read_bps_device = 8:1 8389000" 


 



