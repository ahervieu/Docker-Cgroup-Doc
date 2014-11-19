#Docker and Cgroup documentation to create custom and adaptative containers

## Network bandwidth

-------

[Source](http://serverfault.com/questions/382547/how-to-limit-network-usage-for-concrete-application-in-linux-that-is-running-in) 

[Network Control with C group](  http://vger.kernel.org/netconf2009_slides/Network%20Control%20Group%20Whitepaper.odt) 

###Test :
Reference :

```
time(wget http://www.obeo.fr/download/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip)
```

Output :
```

--2014-11-19 10:41:19--  http://www.obeo.fr/download/release/designer/6.2/latest/juno3/bundles/ObeoDesigner-6.2-linux.gtk.x86_64.zip
Resolving www.obeo.fr (www.obeo.fr)... 91.121.50.185
Connecting to www.obeo.fr (www.obeo.fr)|91.121.50.185|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 376860906 (359M) [application/zip]
Saving to: 'ObeoDesigner-6.2-linux.gtk.x86_64.zip'

100%[======================================>] 376,860,906 5.94MB/s   in 60s    

2014-11-19 10:42:19 (6.01 MB/s) - 'ObeoDesigner-6.2-linux.gtk.x86_64.zip' saved [376860906/376860906]


real	0m59.900s
user	0m1.192s
sys	0m9.912s
```
 Command to reduce brandwidth

sudo docker run -t -i --lxc-conf="lxc.cgroup.net_cls.classid = 0x10002" ubuntu:14.04 /bin/bash


 



