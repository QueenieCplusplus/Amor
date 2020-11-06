# Amor
LB


            User <---------------- PoP, Proxy, Amor ------- FW ----------> Service (Instance Group -> Instance)


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



from step 1

> config FW for Backend

* 1.1, in cloud console, navigate to Network >> VPC >> FW

![fw](https://cdn.qwiklabs.com/o3ZzeAWb50voTy3ENkYicuiDP9Wen8Sybx83FHz9XhY%3D)

* 1.2, to create FW rule, and check rules for internal, RDP, SSH, ICMP, then config the value for the property.

            Property	       Value (type value or select option as specified)
            Name	       default-allow-health-check
            Network	       default
            Targets	       Specified target tags
            Target tags	       http-server
            Source filter	IP Ranges
            Source IP ranges	130.211.0.0/22, 35.191.0.0/16
            Protocols and ports	Specified protocols and ports, and then check tcp
            
 
 from step 2
 
 > create GCE instance templates
 

* 2.1, in cloud console, navigate to GCE >> instance template, type template's name, then click on the "Mgmt & Disk & Network", and click on "managment" tab.

                      Key	                Value
                    startup-script-url	gs://cloud-training/gcpnet/httplb/startup.sh
                    
* tips & attentions, script below is for startup of the instance template, which is called as Metadata in console.
 
            #! /bin/bash

            apt-get update 
            apt-get install -y apache2 php
            apt-get install -y wget
            cd /var/www/html
            rm index.html -f
            rm index.php -f
            wget https://storage.googleapis.com/cloud-training/gcpnet/httplb/index.php
            META_REGION_STRING=$(curl "http://metadata.google.internal/computeMetadata/v1/instance/zone" -H "Metadata-Flavor: Google")
            REGION=`echo "$META_REGION_STRING" | awk -F/ '{print $4}'`
            sed -i "s|region-here|$REGION|" index.php

* 2.2, click on "networking" tab, to config its network properties, and wait for the instance template to be created.

            Property	  Value (type value or select option as specified)
            Network	  default
            Subnet	  default (us-east1)
            Network tags  http-server

 * tips & attentions:
 
 The network tag http-server ensures that the HTTP and Health Check firewall rules apply to these instances template.
