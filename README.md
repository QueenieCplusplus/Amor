# Amor
LB


![amor](https://cdn.qwiklabs.com/7wJtCqbfTFLwKCpOMzUSyPjVKBjUouWHbduOqMpfRiM%3D)

Google Cloud HTTP(S) load balancing is implemented at the edge of Google's network in Google's points of presence (POP) around the world. User traffic directed to an HTTP(S) load balancer enters the POP closest to the user and is then load balanced over Google's global network to the closest backend that has sufficient capacity available.

Cloud Armor IP allowlist/denylist enable you to restrict or allow access to your HTTP(S) load balancer at the edge of the Google Cloud, as close as possible to the user and to malicious traffic. This prevents malicious users or traffic from consuming resources or entering your virtual private cloud (VPC) networks.

In this lab, you configure an HTTP Load Balancer with global backends, as shown in the diagram below. Then, you stress test the Load Balancer and denylist the stress test IP with Cloud Armor.


# Core Steps:

(1) setup FW rules (connection to Backend)

(2) create GCE instance Templates

(3) create GCE instance Group

(4) check/explore External IP of the VM instance (part of the instance group in step 3)

(5) naviagate to Network in console, select LB feature to do config 

(6) note the IPv4/IPv6 of the LB_IP (IP will be client IP, and Host.Location as InstanceGroupName.Zone)

(7) test and pressure test to the LB_IP

(8) config Amor List (Deny/Allow) to the LB
