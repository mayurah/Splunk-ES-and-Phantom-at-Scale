# Deploying Splunk ES and Splunk Phantom At Scale

### Description about the Splunk Products
##### Enterprise Security
Splunk Enterprise Security uses the Splunk platform's searching and reporting capabilities to provide the security practitioner with an overall view of their organization's security posture. Enterprise Security uses __correlation searches__ to provide visibility into security-relevant threats and generate __notable events__ for tracking identified threats. You can capture, monitor, and report on data from devices, systems, and applications across your environment.

> SEC2233 - Splunk ES and Phantoma at Scale


##### Phantom
Harness the full power of your existing security investments with __security orchestration, automation and response__. With Splunk Phantom, execute actions in seconds not hours.


#### Overview

![High Level Overview](./images/High&#32;Level&#32;Overview.png)

![Clustered Phantom Components](./images/Clustered&#32;Setup&#32;-&#32;Phantom&#32;nodes&#32;turned&#32;into&#32;dedicated&#32;handler.png)


### Clustering / Scaling
In this scenario, we are going to scale the products horizontally, i.e., creating a cluster of nodes that would process the results in a distributed manner => Clustering.

One of the first questions that would pop in your head would be: _Why clustering?_
The answer to that is pretty simple, __High Availability and Higher Efficiency.__
There's one disadvantage though: _Higher costs_ for maintaining the resources.
Due to the resources needed for a cluster configuration are expensive, we really want the benefit of the need to meet the cost.

### __Scaling__ the Splunk products
We are going to scale the Enterprise Security and Phantom in this case.

##### __Scaling__ Splunk Phantom
Phantom requires several external shared resources that each Phantom node uses simultaneously. Those resources are:
- __HAProxy__ for load balancing
- __GlusterFS server__ for shared file storage 
- External __Splunk Enterprise__ instance
- External __PostgreSQL Database__ server

A simple cluster design would look as follows:
<insert image here>

##### Example scenario of setting up a simple cluster on AWS
Requirements:
- 7 Running AWS Phantom Standalone instances
- AWS AMI information (Custom image with Phantom 4.1 standalone installed)
- 1 Virtual Private cloud - 10.0.0.0/16
- 1 Subnet with Public Access - 10.0.0.0/24
- Security Groups:
```
Inbound
- HTTP - TCP 80 - 0.0.0.0/0
- All traffic - All All - 10.0.0.0/24
- SSH - TCP 22 - 0.0.0.0/0
- Custom TCP Rule - TCP 8089 - 0.0.0.0/0
- Custom TCP Rule - TCP 8088 - 0.0.0.0/0
- HTTPS - TCP 443 - 0.0.0.0/0

Outbound
- All Traffic - All All - 0.0.0.0/0
```
###### Installing HAProxy Server
If you have your own internal load balancers, you should be able to use them as long as they provide reverse proxy load balancing capabilities.
If not, you can concert your server running to an HAProxy instance using Phantom's `make_server_node.pyc` script.
Please make sure you are logged in as root (`sudo su`).

To use the script, make use of the following command:
`/opt/phantom/bin/phenv python /opt/phantom/bin/make_server_node.pyc proxy`
If you are facing any problems/issues while installing HAProxy, please check the logs generated according to your current timestamp.

###### Installing glusterFS Server
The glusterFS server instance will be installed using Phantom's `make_server_node.pyc` script. 
Please make sure you are logged in as root (`sudo su`).

To use the script, make use of the following command:
`/opt/phantom/bin/phenv python /opt/phantom/bin/make_server_node.pyc fs`.
If you are facing any problems/issues while installing glusterFS, please check the logs generated according to your current timestamp.

##### Installing PostgreSQL server
Splunk doesn't support customer builds for clustered PostgreSQL servers as part of the default configuration. We will use a standalone Postgres server.
The PostGreSQL server instance will be installed using the `make_server_node.pyc` script.
Please make sure you are logged in as root (`sudo su`).

To use the script, make use of the following command:
`/opt/phantom/bin/phenv python /opt/phantom/bin/make_server_node.pyc db`.
If you are facing any problems/issues while installing PostgreSQL, please check the logs generated according to your current timestamp.

##### Installing the Splunk server (optional, necessary for this demo)
The Splunk server instance will be installed using the `make_server_node.pyc` script.
Please make sure you are logged in as root (`sudo su`).

To use the script, make use of the following command:
`/opt/phantom/bin/phenv python /opt/phantom/bin/make_server_node.pyc db`.
If you are facing any problems/issues while installing PostgreSQL, please check the logs generated according to your current timestamp.

##### Installing Phantom Cluster Node(s)
The first instance is considered the __“master”__ node. This is not a true master (all nodes are basically the same), but just the point where you can start adding nodes. After the second node is added, you are considered to be in an Active/Active HA architecture. 
The Phantom Cluster node server instance will be installed using the `make_server_node.pyc` script.
Please make sure you are logged in as root (`sudo su`).

To use the script, make use of the following commands:
```
sudo phenv python2.7 /opt/phantom/bin/initialize.py --init-guid -- force
/opt/phantom/bin/phenv python /opt/phantom/bin/make_cluster_node.pyc --record
```
If you are facing any problems/issues while installing Phantom Cluster node, please check the logs generated according to your current timestamp.

##### Clustered Enterprise Security on Search Head Cluster

  - Installing Splunk ES on SHC environment: https://docs.splunk.com/Documentation/ES/5.3.1/Install/InstallEnterpriseSecuritySHC

  - SHC Architecture: https://docs.splunk.com/Documentation/Splunk/8.0.0/DistSearch/SHCarchitecture

  - Pre-req. & Arch consideration: https://docs.splunk.com/Documentation/Splunk/8.0.0/DistSearch/SHCsystemrequirements

## Optional Ansible Deployment Scripts

## Reference Troubleshooting

```
# Fix DNS Issues if any
echo -e "search us-west-1.compute.internal\nnameserver 1.1.1.1\nnameserver 8.8.8.8\nnameserver 10.0.0.2" > /etc/resolv.conf


# Ensure  `hostname` assigned to the cluster is resolvable.

cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.45   blah ip-10-0-0-45.us-west-1.compute.internal

```

##### Credit

 - Ankit Bhagat
 - Mayur Pipaliya