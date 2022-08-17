# Install and register a DCS node

## Installing DCS nodes

DCS Node can be installed in various locations such as:

1. Installation on ESXi
2. Installation on KVM
3. Hardware installation
4. Installation on AWS
5. Installation on Azure

Images can be obtained from the link. <https://docs.cloud.f5.com/docs/images>

### Installation on ESXi

Install DCS Node from OVA file.
At least one port should be able to connect to the internet via NAT.
The default is a single NIC, so if you need multiple NICs, add NICs after installing the OVA file.

See link for installation details. <https://docs.cloud.f5.com/docs/how-to/site-management/create-vmw-site>

![esxi_ova1](./pics/esxi_ova1.png)
![esxi_ova2](./pics/esxi_ova2.png)

### Installation on KVM

Install DCS Node from ISO file.
If you need multiple NICs, specify 2 NICs with Virt-Manager, etc.
At least one port should be able to connect to the internet via NAT.
Also, KVM requires a Huge page setting.
See link for installation details <https://docs.cloud.f5.com/docs/how-to/site-management/create-kvm-libvirt-site>

Set the NIC's Device Model to `virtio` during installation.
Also, allocate 4 vCPU or more for CPU and 8 GB or more for memory.

![kvm_vm1](./pics/kvm_vm1.png)
![kvm_vm2](./pics/kvm_vm2.png)

When executing with virsh, execute as follows.

```virt-install --name vm1 --ram 8192 --vcpus 4 --disk path=/home/lab/kvm/vm1.qcow2,format=qcow2,size=20 --network bridge=bridge1,model= virtio --cdrom /home/lab/vsb-ves-ce-certifiedhw-generic-production-centos-7.2003.14-202006271045.1593259578.iso --noreboot --autostart --graphics vnc,listen=0.0.0.0,port=6951 --cpu host-passthrough```

### Hardware installation

Install DCS Node from ISO file. Confirmed hardware is Intel NUC etc.
Hardware that is too new may not work properly with Linux due to lack of drivers.
Also, depending on the NIC, etc., it may not work properly.

Please check the website for currently confirmed hardware.
<https://docs.cloud.f5.com/docs/how-to/site-management/create-baremetal-site>

For HPE and Dell hardware, the interface name may appear as em1, p1p1,eno1.
DCS images can only handle eth0/eth1, so they cannot be used until Dell or HPE images are registered.

Copy the downloaded ISO to USB etc. and boot from NUC etc. as a bootable disk.

![flash](./pics/flash.png)

![flashinstall](./pics/flashinstall.png)

### Installation on AWS

### Installation on Azure

## Token settings

Setting up a DCS Node requires a Token.
Please issue a token from Cloud and Edge Sites > Site Management > Site Tokens in the Console.
Enter “Name” and click “Add Site Token” to create a token.

![create_token1](./pics/create_token1.png)
![create_token2](./pics/create_token2.png)

## DCS Node initial setup (CLI)

### Settings such as token

Connect to DCS Node via console or SSH. user/password = admin/Volterra123
The initial password must be changed after login. After logging in, set `Configure`. (If static IP is required, the settings on the next page are required first.)

1. For Token, enter the Token set in the Console.
2. For Site Name, enter the Site (cluster) name of the DCS Node. You can change it later.
3. hostname is optional. master-0 is set
4. Latitude/Longtitude enter valid numbers. You can change it later.
5. Certified hardware varies depending on the image, but select xxx-voltstack-combo for single NIC and xxx-multi-nic-voltstack-combo for multi-NIC.

![dcs_cli](./pics/dcs_cli.png)

### Interface configuration

If you need a static IP address, configure the NIC with “configura-network”.
Check OUTSIDE for Single NIC and INSIDE for Multi NIC (select with space).

SiteLocal = OUTSIDE
SiteLocalInside = INSIDE.

SiteLocalInside GW and DNS2 address are optional and can be left blank.

It is also possible to use Wifi as OUTSIDE. In that case, set SSID and PSK

![dcs_interface](./pics/dcs_interface.png)

## Initial setting of DCS Node (WebUI)

Connect to the DCS Node with a browser. `https://node-ip:65500` user/password = admin/Volterra123
The initial password must be changed after login. After logging in, set `Configure`. (If static IP is required, the settings on the next page are required first.)

![dcs_local_web](./pics/dcs_local_web.png)

### Settings such as Token

For Token, enter the Token set in the Console.
For Site Name, enter the Site (cluster) name of the DCS Node. You can change it later.
Enter master-0 for the hostname.
Enter valid numbers for Latitude/Longtitude. You can change it later.

![dcs_token_web](./pics/dcs_token_web.png)

Select xxx-voltstack-combo or xxx-multi-nic-voltstack-combo for Certified hardware depending on the image.

### Registering DCS Nodes

Once you have made the initial settings and connected to the Internet, the DCS Node will be displayed as a Site in the Console's System Namespace.
Select a site and click Accept to start setting up your DCS Node.

For Multi Master node, select `3` for `cluster size`.
At this time, `Cluster name` sets the same name for all 3 nodes, and the Hostname at the initial setting of DCS Node is `master-0`, `master-1`, `master-2` and all 3 nodes are unique hosts Please enter your first name.
**It takes about 20 minutes depending on the line speed.

![registration](./pics/registration.png)
![registration_accept](./pics/registration_accept.png)

You can see the created dcs Node in Sites -> Site List. When the SW version becomes “Successful”, provisioning is completed, and after a while the Health Score will become 100.

![registration_finish](./pics/registration_finish.png)

Here we create two DCS nodes `site1` and `site2`.

### Creating Labels

Labels set for DCS Nodes, pods, etc. can be set as `shared labels` or individually set manually. Create a shared label from the shared Configuration.
Labels created in Shared Configuration can be used in all Namespaces.

It is used to set labels that should be used in common. For example, create a Key called `site-setting`, create a Label with a value such as `kvm` or `esxi`, and set the Label to the Site.

You can create shared Labels by `Add known key` from Manage -> Labels > Known Keys.

![shared_label1](./pics/shared_label1.png)

Set the following labels here.

- Label key: pref
  - Label values: `tokyo`, `osaka`

![shared_label2](./pics/shared_label2.png)

## Label settings

Set the created label to the DCS Node. From Sites -> Site list of Cloud and Edge Sites, add the label created from Manage Configuration of DCS Node.
Set `pref:tokyo` for site1 and `pref:osaka` for site2.

![site_label1](./pics/site_labels1.png)
![site_label2](./pics/site_labels2.png)
