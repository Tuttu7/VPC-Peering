
# - Setting up VPC Peering between VPCs in different AWS Regions

Following are the steps involved:

* Create two VPCs, one in each region
* Create subnets in each VPC
* Create and attach IGW to each VPC
* Create and update the routing table
* Provision Ec2 instances in each VPC's subnet and test network connectivity
* Configure VPC Peering between the VPCs in different region
* Verify Network connectivity using ping


## - Creating an VPC in ohio region

> Tag : vpc-ohio-1
> IPV4 CIDR block : 10.200.0.0/16

## - Creating subnet 

> Tag : subnet01-vpc-ohio-1
> VPC : vpc-ohio-1
> Availability zone : 2a
> IPV4 CIDR : 10.200.1.0/24

## - Creating Internet gateway

> Tag : igw-vpc-ohio-1
> Attach gateway to VPC

## - Creating route table

> Tag : public-routetable-vpc-ohio-1
> Route tables > Edit routes > Add route 
```
Destination   Target            Status  Propagated

10.200.0.0/16  local            active  No

0.0.0.0/0 igw-0415e4e592d6d6cf2	active  No
```
> Subnet assosiate (Main route)
Route table > Subnet assossiation > Edit subnet assosiation > Select Subnet ID > Save

## - Creating an EC2 instance in ohio
> Services > ec2 > Lauanch instance > Select the vpc and subnet we have created 

## - Creating an VPC in oregon region
> Tag : vpc-oregon-1
> CIDR : 10.201.0.0/16

## - Creating subnet 

> Tag : subnet01-vpc-oregon-1
> VPC : vpc-oregon-1
> Availability zone : 2a
> IPV4 CIDR : 10.201.1.0/24

## - Creating Internet gateway

> Tag : igw-vpc-oregon-1
> Attach gateway to VPC

## - Creating route table

> Tag : public-routetable-vpc-ohio-1
> Route tables > Edit routes > Add route 
```
Destination     Target                 Status    Propagated

10.201.0.0/16  local                   active    No
		
0.0.0.0/0     igw-06f448e6b76cd7a70    active    NO

```
> Subnet assosiate (Main route)
Route table > Subnet assossiation > Edit subnet assosiation > Select Subnet ID > Save

##### - Then go to the instance created in ohio region and then try to ping the private IP of the instance created in the oregon region. You will not able to ping the instance.

```
ping 10.201.1.221

PING 10.200.1.221 (10.200.1.221) 56(84) bytes of data.
From 10.200.1.221 icmp_seq=1 Destination Host Unreachable
From 10.200.1.221 icmp_seq=2 Destination Host Unreachable
From 10.200.1.221 icmp_seq=3 Destination Host Unreachable
```

## - Creating the Peering connection in ohio region
> Peering connection > Create pearing connection 
> Tag : vpc-peer-ohio-oregon-1
> VPC requested : vpc-ohio
> Select another VPC to peer with > Region > Another > region > us-west-2
> VPC accepter > Give the oregon VPC ID 

## - Editing routing tables in oregon region
> Route tables > Edit routes > Add route > Select the peering as the target
```
Destination    Target                  Status  Propagated

10.200.0.0/16   pcx-01e0813de49b4261f  active  No
```
## - Editing routing tables in ohio region
> Route tables > Edit routes > Add route > Select the peering as the target
```
Destination    Target                  Status  Propagated

10.201.0.0/16  pcx-01e0813de49b4261f   active  No
```
## - Editing security group in ohio region 
> Services > EC2 > Network & Security > Security groups > Select the security group > Inbound > Add rule
```
Type             Protocol   Port Range      Source         Description

SSH              TCP        22              0.0.0.0/0

All ICMP - IPv4  All        N/A             10.201.1.0/24
````
#### - Finally go to ohio instance and try to ping the private IP of the instance we have created in the oregon region 
````
[root@ip-10-200-1-37 ~]# ping 10.201.1.221
PING 10.201.1.221 (10.201.1.221) 56(84) bytes of data.
64 bytes from 10.201.1.221: icmp_seq=1 ttl=255 time=71.8 ms
64 bytes from 10.201.1.221: icmp_seq=2 ttl=255 time=71.8 ms
64 bytes from 10.201.1.221: icmp_seq=3 ttl=255 time=71.8 ms
64 bytes from 10.201.1.221: icmp_seq=4 ttl=255 time=71.8 ms
64 bytes from 10.201.1.221: icmp_seq=5 ttl=255 time=71.9 ms
64 bytes from 10.201.1.221: icmp_seq=6 ttl=255 time=71.9 ms
64 bytes from 10.201.1.221: icmp_seq=7 ttl=255 time=71.9 ms
64 bytes from 10.201.1.221: icmp_seq=8 ttl=255 time=71.8 ms
64 bytes from 10.201.1.221: icmp_seq=9 ttl=255 time=71.9 ms
64 bytes from 10.201.1.221: icmp_seq=10 ttl=255 time=71.9 ms
64 bytes from 10.201.1.221: icmp_seq=11 ttl=255 time=71.8 ms
64 bytes from 10.201.1.221: icmp_seq=12 ttl=255 time=71.9 ms

[root@ip-10-200-1-37 ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 10.200.1.37  netmask 255.255.255.0  broadcast 10.200.1.255
        inet6 fe80::a9:52ff:fef6:9068  prefixlen 64  scopeid 0x20<link>
        ether 02:a9:52:f6:90:68  txqueuelen 1000  (Ethernet)
        RX packets 27612  bytes 37538517 (35.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3553  bytes 259979 (253.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 29  bytes 2832 (2.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 29  bytes 2832 (2.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
````

 









