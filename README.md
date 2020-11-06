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

* 2.3, copy its property to new instance template, and name it, then change its subnet.

       Now create another instance template for subnet-b by copying us-east1-template.
       
       For Network interfaces, select default (europe-west1) as the Subnet.

from step 3:

> create GCE instance Group.

* 3.1, in cloud console, navigate to GCE >> instance group.

            Property	      Value (type value or select option as specified)
            Name	      us-east1-mig[group name]
            Location	      Multiple zones
            Region	      us-east1
            Instance template	us-east1-template [gce instance template name]
            Autoscaling > Autoscaling metrics > Metric type	CPU utilization
            Target CPU utilization	           80
            Minimum number of instances	1
            Maximum number of instances	5 (auto scale up depends on traffic performance)
            Cool-down period	            45

* tips & attentions:

Managed instance groups offer Autoscaling capabilities that allow you to automatically add or remove instances from a managed instance group based on increases or decreases in load. 

Autoscaling helps your app gracefully handle increases in traffic and reduces cost when the need for resources is lower. You just define the autoscaling policy and the autoscaler performs automatic scaling based on the measured load.

* 3.3, create another instance group, and name it europe-west1-mig, and use europe-west1-template.


start from step 4:

> create GCE instance using GCE instance template, and explore its External IP addr.

* 4.1, in cloud console, navigate to GCE >> VM instance.

        Notice the instances that start with us-east1-mig and europe-west1-mig.

        These instances are part of the managed instance groups.
        
        And click on the External IP of the instance created.
        
        [output is Client IP and Hostname.Location as InsatanceGroupName.Zone]
        
     ![client ip](https://cdn.qwiklabs.com/cB4rkhddQchP1iTAc7xNeF5Bly34SwjtieR406NQM9w%3D)

start from step 5:

> LB config

* 5.1, in cloud console, navigate to Network >> LB
