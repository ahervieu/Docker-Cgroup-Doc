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

*Result
```
root@b14758fbf91c:/# time(wget http://www.obeo.fr/download/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip)
--2014-11-20 09:17:11--  http://www.obeo.fr/download/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip
Resolving www.obeo.fr (www.obeo.fr)... 91.121.50.185
Connecting to www.obeo.fr (www.obeo.fr)|91.121.50.185|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 376860906 (359M) [application/zip]
Saving to: 'ObeoDesigner-6.2-linux.gtk.x86_64.zip.1'

18% [====================>                                                                                                  ] 68,745,524  59.6KB/s  eta 84m 13s
```



## Limit Reading and Writing speed with Docker


 - First start docker deamon with the option -e lxc. 
 - start a container
```
sudo docker run  -i -t ubuntu:14.04 /bin/bash
```

- get it's id :
```
aymeric@cirrus2:~$ sudo docker inspect -f '{{ .Id }}' suspicious_wozniak
6767ba4a0051fa948dfcfdc41c1a18a28701d899bc1ca6e71cae668947847d60

```
- go in /sys/fs/cgroup/blkio/docker/ID/
```
cd /sys/fs/cgroup/blkio/docker/6767ba4a0051fa948dfcfdc41c1a18a28701d899bc1ca6e71cae668947847d60/
```
- Change the right of the file
```
sudo chmod u+w blkio.throttle.write_bps_device
```

Tune the righting speed :
```
echo "8:0 20485760" > blkio.throttle.write_bps_device 
```

#Test
  - First test with this speed : 
```
echo "8:0 20485760" > blkio.throttle.write_bps_device 
```
- In the container :
```
root@6767ba4a0051:/# time $(dd if=/dev/zero of=testfile0 bs=1000 count=100000 && sync)
100000+0 records in
100000+0 records out
100000000 bytes (100 MB) copied, 0.290006 s, 345 MB/s

real	0m5.305s
user	0m0.016s
sys	0m0.326s
```

-Let's divide the speed by two 
```
echo "8:0 10485760" > blkio.throttle.write_bps_device 
```
- In the same container :
```
time $(dd if=/dev/zero of=testfile0 bs=1000 count=100000 && sync)
100000+0 records in
100000+0 records out
100000000 bytes (100 MB) copied, 0.314431 s, 318 MB/s

real	0m9.907s
user	0m0.004s
sys	0m0.364s
```








