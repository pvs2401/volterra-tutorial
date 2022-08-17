# Basic knowledge of DCS

DCS is provided as a SaaS service and consists of managed Kubernetes AppStack, integrated security service Mesh, and controller Console. AppStack and Mesh run on DCS Node, and it is also possible to use only their respective functions.
DCS Node runs on cloud, x86 hardware and virtual machines.

![overview](./pics/overview.svg)

-Mesh
With DCS's global network infrastructure and distributed application gateways, we securely connect applications in the cloud and at the edge and provide a monitoring infrastructure.
  -Routing (BGP)
  - L3-L4 Firewall
  - Load balancer
  -GSLB

-AppStacks
A distributed application management platform for deploying, securing and operating applications and heterogeneous infrastructure across multi-cloud and edge.

![DCS node](./pics/dcs_node.svg)

## Sites and Virtual-sites

DCS manages DCS Node clusters as Sites, and Virtual Sites are groups of multiple Sites. Virtual Sites are grouped based on the label set for the Site, and when setting application distribution, security policies, etc., information propagates to the entire target group.

![site_vsite1](./pics/site_vsite1.svg)

For example, if there are 4 sites (Kuberentes cluster), create `minami-kanto` as a virtual site.
If you set a Kubernetes Manifest or security policy for this virtual site, the settings will be distributed to all sites within the virtual site.
If some sites are down at this time, the latest configuration will be automatically applied after the site is restored. The user does not need to reset the config.

In the case of Kubernetes objects, it can be handled by simply entering the virtual site name created on DCS as Annotation, so there is no need to resize the existing manifest.

![site_vsite2](./pics/site_vsite2.svg)

## Origin pools and Load Balancers

When communicating with different DCS Node clusters, set the Origin Pool and Load Balancer
Origin Pool sets destination services and IP addresses as endpoints. The Origin Pool will load balance the Endpoints when there are multiple destinations.
Load Balancer sets the source DCS Node and Load Balancer. Load Balancer can also be used when applications on AppStack communicate with remote sites, and when terminals in user sites access remote sites. If there are multiple Origin Pools, Load Balancer will act as a GSLB.

![origin_lb](./pics/origin_lb.svg)
