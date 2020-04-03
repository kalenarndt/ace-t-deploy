# Ansible for NSX-T

## Disclaimer:
I'm not great at Ansible so your mileage may vary, just fill out the variable files and let me know if you have problems

## Overview
This repository contains NSX-T Ansible Modules, which one can use with
Ansible to work with [VMware NSX-T Data Center][vmware-nsxt].

[vmware-nsxt]: https://www.vmware.com/products/nsx.html

For general information about Ansible, visit the [GitHub project page][an-github].

[an-github]: https://github.com/ansible/ansible

These modules are maintained by [VMware](https://www.vmware.com/).

Documentation on the NSX platform can be found at the [NSX-T Documentation page](https://docs.vmware.com/en/VMware-NSX-T/index.html)


## Prerequisites
We assume that ansible is already installed.
These modules support ansible version 2.8.1 and onwards.

* Python3 >= 3.5.2
* PyVmOmi - Python library for vCenter api.
* OVF Tools - Ovftool is used for ovf deployment.
* ESXi 7 Hosts 
* vCenter 7 with a VDS configured with 1600 MTU or above 
* ESXi Hosts joined to VDS
* A Management Port Group (for NSX Manager and Edges)
* 2 Trunk Portgroups (1 for fabric a (uplink-1 active) and 1 for fabric b (uplink-2 active))
* NSX-T 3.0 license

### What does this do?
* This deploys 1 NSX Manager (it defaults to medium so buckle up)
* Checks the health of the API
* Sets the VIP
* Checks the Health
* Registers a Compute Manager
* Licenses the NSX Manager
* Creates an IP Pool ONLY FOR THE EDGES (I have DHCP in my homelab so if you want more pools then you need to add another pool in the yaml)
* Creates NSX Edge Uplink Profiles (MTU 1600) with named teaming profiles and multi-tep
* Creates ESXi Uplink Profile ( No MTU ) with namted teaming profiles and multi-tep
* Creates 3 Transport Zones (1 for Edges, 1 for ESXi Hosts, 1 for VLAN all backed by 1 NVDS)
* Creates a TN Profile for the ESXi hosts with a vCenter 7 VDS as the backing switch
* Attaches the TN Profile to the cluster named in the variable 
* Deploys 1 NSX Edge (named edge1) and waits for it to check in
* Deploys 1 NSX Edge (named edge2) and waits for it to check in
* Creates an Edge cluster (named edge-cluster-01) and adds edge1 and edge2 to the cluster object
* Creates 2 VLAN backed Segments in the tz-edge transport zone (for edge peering)
* Assigns a VLAN tag (200 and 201 based on the yaml change this to match your environment)
* * Sets a named teaming policy to force connectivity out fp-eth0 for fabric-a for vlan segment 200
* * Sets a named teaming policy to force connectivity out fp-eth1 for fabric-b for vlan segment 201
Creates a Tier-0 Logical Router (t0-ecmp)
* * Configures ECMP
* * Enables BGP
    * Sets the local AS number
* * * Sets 1 BGP neighbor for fabric a
* * * Sets the source addresses to be the router ports that exist on the vlan for fabric a
* * * Configures modified BGP timers for 4, 12
* * * Configures a BFD peer for this neighbor
* * * Enables Route Redistribution [TIER0_CONNECTED, TIER0_NAT, TIER1-Connected, T1_LB_VIP]
* * Creates 4 router ports (2 per edge, 1 per vlan)
* Creates a Tier-1 Logical Router (t1-general)
* Downlinks the Tier-1 to t0-ecmp
* Configures route redistribution for TIER1_CONNECTED, TIER1_LB_VIP
* Creates 3 Logical Segments in the tz-overlay transport zone and uplinks them to the t1-general

## What do I have to modify?
So for the first playbooks (001,002,003) they reference a variable file called deploy_nsx_cluster_vars.yaml. Those variables are used to stand up everything until we need to make policy calls

For playbooks (100,101) they reference a variable file called build_topology_vars.yml and you just need to change the information in there to match what you want to deploy.

## I modified it so what do I do now chief?

So if you modified all the variables to be specific to your environment you can either run the playbooks manually by doing something like this 

* ansible-playbook 001_deploy_first_node.yml 

If you want to live dangerously and execute all the playbooks in order or just want to see what happens do the following:

* ansible-playbook build_topology.yml 

# Hey it broke

Double check the variables that you are using and if you need to kick it back off you can just rerun the last ansible play with a flag to start at the task that failed 

* ansible-playbook 003_deploy_edges.yml --start-at-task="task name in the playbook"

