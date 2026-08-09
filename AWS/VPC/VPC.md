# Networking
ISP : Internet Service Provider

Public IP: use below webistes to check public ip of pur machine
1. https://checkip.amazonaws.com/
2. https://whatismyipaddress.com/

Private IP:
1. ipconfig (192.168.1.18) --> windows
2. ipcionfig /all

ncpa.cpl --> network inter --> IP Info


Public IP : Glabally unique IP allocated to a device. In our Home network ISPs allocate one Unique IP..  

Private IP : Used within an internal network and it should be unique (within the network)

IPv4 : 32 bit Address space : 32 empty slots to fill with 0 & 1 : 8.8.8.8 : 2^32 --> 4.3 Billion..
IPv6 : 128 bit address space : 128 empty slots to fill with 0 & 1 : 2^128 --> 340 Undecillion IPs..

--

IPv4 has 5 classes

Class A : 0.0.0.0 - 126.255.255.255 : 
Class B : 128.0.0.0 - 191.255.255.255 : 
Class C : 192.0.0.0 - 223.255.255.255 : 

Class D : 224.0.0.0 - 239.255.255.255		: Multicasting
Class E : 240.0.0.0 - 255.255.255.255		: Reserved for R&D

127 --> loopback Address


DHCP : Dynamic Host Configuration protocol : It allocates IP address to the device that added to the network.
 
====
### private ip's in different ranges free of use
if you see the below range ip's these are private ip's we use this ip ranges for free of cost for internal purpouse if you use 11 series(like 11.0.0.0) we need to pay money(for internal purpouse also). that's why in home routers we mostly see the below range ip's addresses(mostly uses 192 series.)
Class A : 10.0.0.0 - 10.255.255.255
Class B : 172.16.0.0 - 172.31.255.255
Class C : 192.168.0.0 - 192.168.255.255


Network : Group of inter connected devices.. All these devices can communicate each other.. 
Host : A device with in a network, that consumes an IP.. 

Class A : N.H.H.H
Class B : N.N.H.H 
Class C : N.N.N.H

Class A : 0.0.0.0 - 126.255.255.255
Class A : N.H.H.H
How many networks we can create.? Ans: 127 Networks
How many hosts we can add/create in a single network : Ans: 16 Million Hosts

10.0.0.0
10.(0-255).(0-255).(0-255)

====

Class B : 128.0.0.0 - 191.255.255.255
Class B : N.N.H.H
How many networks we can create.? Ans: 16,000 networks
How many hosts we can add/create in a single network : Ans: 65k Hosts

172.16.0.0
172.16.0.1 / 0.2 / 0.3 ... 0.255.. 1.1...1.255.. 2.0..2.555... 255.255

====

Class C : 192.0.0.0 - 223.255.255.255
How many networks we can create.? Ans: 2 million networks
How many hosts we can add/create in a single network : Ans: 256 Hosts
192.168.0.0
192.168.0.1 
192.168.0.2
192.168.0.3
...
...
192.168.0.255

===========

Subnetting: The process of dividing a larger network into smaller sub neworks

CIDR : CLassless Inter Domain Routing :
 
IPv4 : 32 Bit
 
/32 --> 32 - 32 = 0 = 2^0 = 1
/31 --> 32 - 31 = 1 = 2^1 = 2
/30 --> 32 - 30 = 2 = 2^2 = 4
/29 --> 32 - 29 = 3 = 2^3 = 8 
/28 --> 32 - 28= 4 = 2^4 = 16							--> --> Min A VPC Supports is /28 Only
/24 --> 256
/20 --> 4096
/16 --> 32 - 16 = 16 = 2^16 = 65536						--> Max A VPC Supports is /16 Only
/0 --> 32 - 0 = 32 = 2^32 = 4M


# AWS VPC
You can take any network, First IP and Last IP will be reserved.. 
First IP : Network ID
Last IP : Broadcast ID

Along with above 2.. We cant use 3 more IPs in AWS.. 
AWS DNS Server : Second IP
AWS VPC Route : Third IP
Future Use : 1 IP reserve

Note:
1. we can't add more than 60 entries in security group(60 entries for inbound and 60 entries for outbound)
2. For Nacl we can't add more than 20 entries(20 for inbound and 20 for outbound)

--> In AWS Environment, Every VPC reserves 5 IPs.. 

/28 --> 32 - 28= 4 = 2^4 = 16 - 5 = 11 Usable IPs.. 
/24 --> 256 - 5 = 251 Usable IPs
/20 --> 4096 - 5 = 4091 Usable IPs
/16 --> 32 - 16 = 16 = 2^16 = 65536 - 5 = 65531 Usable IPs

---	

CIDR Planning : 
--> How big VPC we required.? Ans : 100.. (/25 or /24)
--> How many Subnets we required.? Ans : 8 subnets
--> How many Public SUbnets.? Ans: 2 subnets
--> How many private Subnets.? Ans: 6 subnets
--> Do you have any future plans to extend this network.? Ans: yes

Public Subnet : Internet facing
Private Subnet : No Direct Exposure to outer world to the resources running here.
	--> Private Subnet with NAT : No Direct exposure to internet, but it can access internet.
	--> Private Subnet without Internet


100 IPs --> /25 --> 128 - 5 = 123 Usable IPs
/24 --> 256 IPs - 5 = 251 Usable IPs..

251 IPs --> Make it 8 parts

192.168.0.0 - 192.168.255.255 --> Network.? : 
Ans: 192.168.100.0/24

2 --> Web
WEB-CVPC-1A: 192.168.100.0/28
WEB-CVPC-1B: 192.168.100.16/28

2 --> App
APP-CVPC-1A: 192.168.100.32/27
APP-CVPC-1B: 192.168.100.64/27

2 --> Db
DB-CVPC-1A: 192.168.100.96/27
DB-CVPC-1B: 192.168.100.128/27

2 --> Lambda / caching / future
SL-CVPC-1A: 192.168.100.160/27
SL-CVPC-1B: 192.168.100.192/27

Future use: 192.168.100.224/27

IPAM : IP Address manager : tool used to track ip's usage(enterprises uses solarwinds tools to manage ip's it is less cost.)
 
Use visual subnet calculator to divide the 192.168.100.0/24 into subnets.

When you create a new vpc, what are the components you get defualtly? 

Ans: RouteTable, NACL, DefaultSecurityGroup



1. reate a VPC with CIDR range we selected. goto vpc console click on create VPC --> select VPC only give VPC name --> in ipv4 CIDR section select *IPv4 CIDR manual input* give VPC range 192.168.100.0/24(if we are using IPAM tool we will select *IPAM-allocated IPv4 CIDR block*) --> go with Default option click create VPC.
![alt text](../.images/VPC.png)

2. Creating Subnets on left pane click on subnets click on create subnet --> select our VPC --> in Subnet section give the name for subnet select availability zone and give subnet CIDR block(like 192.168.100.0/27). --> click on add subnet to create multiple subnets.
![alt text](../.images/subnet.png)

WEB-CVPC-1A: 192.168.100.0/28
WEB-CVPC-1B: 192.168.100.16/28
APP-CVPC-1A: 192.168.100.32/27
APP-CVPC-1B: 192.168.100.64/27
DB-CVPC-1A: 192.168.100.96/27
DB-CVPC-1B: 192.168.100.128/27

**Note:** all these subnets automatically(implicitly) associated to main route table if we explictly add to other route table they will automatically removed from main route table.
![alt text](../.images/RT.png)

3. creating custom route tables for routing traffic for public and private subnets. creating 3 route tables
 
	1. **public route table:** creating route table for public subnets(WEB-CVPC-1A, WEB-CVPC-1B) follow the route table creation in step-2 then click on route table add the public subnets.
	2. **private route table:** repeat the above step and add the private DB subets(DB-CVPC-1A, DB-CVPC-1B).
	![alt text](../.images/RT1.png)
	3. **private route table with internet:** repeat the above step and add the private APP subets(APP-CVPC-1A, APP-CVPC-1B).
4. then we will attach internet gateway to public subnet then only it can able access the internet. goto route table select the public route table below select the edge associations tab click on edit edge association select the internet gateway click save changes.
![alt text](../.images/IGW1.png)
![alt text](../.images/IGW2.png)

5. it is optional if we want auto assign ip's to resorces deployed in subnets we need to enable Auto Assign Public IP. goto subnets console select subnet click on actions select edit subnet settings in *Auto-assign IP settings* click enable auto assign ip click save. we can also enable auto assign ip while creating instance as well.
![alt text](../.images/Aip.png)

6. it's also optional but recommended choose VPC, Select "edit vpc settings", Enable "DNS Hostnames" and "DNS resolution".
![alt text](../.images/VPC1.png)

Note: if you see the target local in a specific subnet console means it can communicate with every subnet in that vpc.
![alt text](../.images/IGW4.png)

7. we add a route in public subnet to allow traffic flow from internet vpc and vpc to internet goto route table console select the public route in routes tab section click on edit routes and add like routes 0.0.0.0/0 to IGW.
![alt text](../.images/IGW5.png)



---


** MAKE SURE TO DELETE NAT AFTER YOUR PRACTICE. 

I need internet to my private subnet instances.!!

## NAT Gateway: 
We can control communication (Public / Private)

Public NAT gateway (Regional): 
--> Placed in a Public Subnet
--> Uses an EIP
--> This allows Private subnet instances to access internet.
--> Managed by AWS

Private NAT gateway : 
--> Placed in private Subnet
--> No EIP
--> used for Private-to-private traffic
--> No Direct access to internet.

### creating nat gateway
1. goto natgatway console click on create nat gateway give name --> in availability modes select zonal select the public subnet(WEB-CVPC-1A) --> select public in connectivity type allocate elastic ip click on create nat gateway.

![alt text](../.images/nat.png)

create a route in private with internet subnet from 0.0.0.0/0 to nat gateway.
Availability Mode of NAT: 

Regional : Scales automatically across all regional AZs, simplifying management for multi AZ deployments.
Zonal: Provides granular control within a specific availability zone, adhering to subnet level settings.

---

We can use Site-2-site VPN conenction / Direct Connect to establish connection between AWS and On-premise environment. Directly, we can access all the AWS private subnet resources using Private IPs.

---

D: 02/03/2026

NETWORK ACL / NACls : 

By defaultly, all subnets witll be part of Defualt NetworkACL. Ans, It allows all the traffic to all the network..

newly created network ACLs, wont allow any traffic to any network. It blocks everything.

One subnet can be member of one NACL at a time. 

We have to create rules increments of 100s.. (100 / 200 / 300..)

SGs will have only allow option.. SG Dont have DENY option..

NACLs have allow and deny option. If you want to deny the traffic to a particular network, you cna grab the Network IP and and deny it. 


--

We have option to enable logs at 
1. vpc level
2. subnet level
3. individual instance level

---

Endpoints: Access S3/dynamodb / other resources without internet using endpoints

---

D: 03/03/2026

VPC Peering : This helps to enable communication between 2 VPCs. 

Source VPC and Target VPC should not have same CIDR ranges. 

VPC peering is Non-Transitive peering.

				Requester VPC				Accepter VPC
Acc ID			655700896650				655700896650
VPC IP			vpc-09fa12df9e4d9e195		vpc-0cbfc0d30295f9d10
CIDR Range		192.168.100.0/24			10.0.0.0/16
Region			ap-south-1					ap-southeast-2

---

Central NW Account = Place where TGW created..

Step 1 : create a TGW/Transit Gateway in Central networking account.

Step 2 : Share it Multiple AWS Accounts, Switch to other acounts RAM service and accept the invitation.

Step 3 : In central NW account, Create a TGW attachment with local VPCs desired subnets.. 

Step 4 : In Member account, Create a transit gateway attachment with local VPC subnets.

Step 5 : Switch to Central NW VPC TGW, Choose the attachment and accept it (Accpeting Step 4).

Step 6 : Edit Member account route table with "Central NW" Account VPC CIDR. 

Step 7 : Edit Central VPC account route table with "Member account"  VPC CIDR. 

---

Site-2-Site VPN : 

Step 1 : Create customer gateway with office firewall device public ip.

Step 2 : Create virtual private gateway.

Step 3 : Create site-to-site vpn with CGW and VPGW. Donwload the configuration and provide it with Firewall team.

---

Just like NAT Gateway, If you want to provide internet to "private subnet instances" in "IPv6", we use "Egress-only Internet gateways".

---