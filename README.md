## mdns-systemd-resolved

This is a small package to install necessary files to enable & broadcast mdns multicast dns on local network with
`systemd-resolved (>=244)` & `Networkmanager (>=1.22)` without avahi. Once enable target system can be reachable with 
`hostname.local` and can be resolved with resolvectl. It needs to be enabled on both hosts and target systems. 

```sh
$ resolvectl query -4 heimdeil.local
heimdeil.local: 192.168.19.164           -- link: wlo1

-- Information acquired via protocol mDNS/IPv4 in 421.3ms.
-- Data is authenticated: no
```

systemd-resolved provides resolver services for Domain Name System (DNS) (including DNSSEC and DNS over TLS), 
Multicast DNS (mDNS) and Link-Local Multicast Name Resolution (LLMNR). 
The resolver can be configured by editing `/etc/systemd/resolved.conf` and/or drop-in . conf files in 
`/etc/systemd/resolved.conf.d`.

Multicast DNS will be enabled on a link only if the per-link and the global setting is on. And it also requires to be enabled
from NetworkManager or systemd-network. This repository only contains configuration file for NetworkManager.

### /etc/systemd/resolved.conf.d/mdns-systemd-resolved.conf
```matlab
[Resolve]
MulticastDNS=yes
LLMNR=no
```

### /etc/NetworkManager/conf.d/mdns.conf

```matlab
[connection]
connection.mdns=2
connection.llmnr=0
```

### Why disable llmnr ?

LLMNR is an mDNS alternative, proposed by Microsoft.

Microsoft is [abandoning LLMNR itself](https://techcommunity.microsoft.com/t5/networking-blog/aligning-on-mdns-ramping-down-netbios-name-resolution-and-llmnr/ba-p/3290816), and Windows 10 has long supported mDNS. 
Even so, there may be some use scenarios where mDNS fails while LLMNR succeeds. 
For instance, a Windows 7 computer that was never upgraded to Windows 10. 
However, such devices have the choice to include mDNS support via [Bonjour print services for Windows](https://support.apple.com/kb/DL999?locale=en_US).

In such cases just remove llmnr from configuration files. 

## Broadcasting services with systemd-dnssd

It is also possible to broadcast [services](https://android.googlesource.com/platform/external/avahi/+/refs/heads/master/service-type-database/service-types) such as nfs, sftp and make them automatically appear in netwotk section 
of file managers like nautilus and nemo. Nautilus uses `gvfs-dnssd` for that. 

The .dnssd files are read from the files located in the system network directories /usr/lib/systemd/dnssd and 
/usr/local/lib/systemd/dnssd, the volatile runtime network directory /run/systemd/dnssd and the local administration 
network directory /etc/systemd/dnssd. 

### /etc/systemd/dnssd/sftp.dnssd

```matlab
[Service]
Name=%H
Type=_sftp-ssh._tcp
Port=22
```

### /etc/systemd/dnssd/ssh.dnssd
```matlab
[Service]
Name=%H
Type=_ssh._tcp
Port=22
```

Now restart systemd-resolved on both system (sudo systemctl restart systemd-resolved.service) and then all broadcasted services
can be discovered with

```sh
$ resolvectl query -p mdns --type=PTR _services._dns-sd._udp.local
_services._dns-sd._udp.local IN PTR _ssh._tcp.local         -- link: wlo1
_services._dns-sd._udp.local IN PTR _sftp-ssh._tcp.local    -- link: wlo1

-- Information acquired via protocol mDNS/IPv4 in 433.5ms.
-- Data is authenticated: no
```

![image](https://user-images.githubusercontent.com/1333217/241902080-fa1747f3-f41e-468d-927a-8c15d243df0b.png)


Note: For `systemd (<=244)` it is required to restart systemd-resolved  before using `resolvectl query` as it 
caches the PTR records. For `systemd >= 248`, there is `--cache=no` option which can be supplied.
