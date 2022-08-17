# Manage Kuberetes

Until now, we have created Pods using VK8S from the console, but it is also possible to use APIs that are not open to VK8S using local APIs like normal Kuberetes cluster.
In DCS, this function is set to Managed K8s(Physical K8s).https://docs.cloud.f5.com/docs/how-to/app-management/create-deploy-managed-k8s We call it " <url>".

In order for Manage k8s to work, you must create a template for the K8s cluster on the AppStack Site and provision the CE.
The templates can be Master, Worker node, IP address, and other interface settings, external storage, GPU availability, Kubevirt availability, and so on.

AppStack Site can also define and combine multiple Kuberentes Manifests to easily apply and update the same K8s Manifest to multiple sites.

For example, you can create a Pod Security Policy and apply its Manifest to 3 AppStack sites.
When you change the Pod Security Policy, the changes are automatically applied to the 3 AppStack sites (K8s clusters).

![appstack_site1](./pics/appstack_site1.svg)

## Create AppStack site

Create an AppStack site to use Manage k8s.

1. Create Cluster Role (Option)
1. Creating a Cluster Role Binding
1. Create Pod Security Policy (Option)
1. Creating a K8s Cluster
1. Creating an AppStack site
1. Provisioning of CE

### Create a Cluster Role

Under Home -> Cloud and Edge Sites -> Manage -> Manage K8s -> K8s Cluster Roles, select “Add K8s K8s Cluster Role”.

The Cluster role can be configured in the UI or Yaml.(In Yaml, Show Advanced Fields must be enabled.))
It is also possible to use the preset Cluster role, but `ves-io-admin-cluster-role` has all roles enabled, and `ves-io-psp-permissive` has only the following settings, so it is recommended to set the appropriate Role for commercial use.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: null
  name: ves-io-psp-permissive
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  verbs:
  - use
```

The following Yaml is configured here.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

![cr_config1](./pics/cr_config1.png)


### Create a Cluster Role Binding

Under Home -> Cloud and Edge Sites -> Manage -> Manage K8s -> K8s Cluster Role Bindings, select Add K8s K8s Cluster Role Binding.

The following configuration is set in the preset.This is only the `ves-io-psp-permissive` of the preset ClusterRole, so you need to create a new ClusterRoleBinding and configure the ClusterRole to allow Post, etc.

Here we set the preset Cluster role `ves-io-admin-cluster-role` as ClusterRoleBinding `admin-crb` and create the object.

- name: `admin-crb`
- K8s Cluster Role : `shared/ves-io-admin-cluster-role`
- Subject
  - Select Subject: `User`
    - User: `DCS user name`

![crb_config1](./pics/crb_config1.png)

### Create Pod Security Policy

Under Home -> Cloud and Edge Sites -> Manage -> Manage K8s -> K8s Pod Security Policies, select Add K8s Pod Security Policy.

The PSP can be configured with UI or Yaml.(In Yaml, Show Advanced Fields must be enabled.))
Setting up the PSP is optional, but for commercial use it is recommended to set up the appropriate PSP, since the default setting is `Privileged true` and you can create Pods with `root`.

![psp_config1](./pics/psp_config1.png)

Setting `Name` and 1 or more PSP objects is required in the UI.Please set the items that need to be set in the UI.
If you have an existing PSP YAML, you can configure it directly.In this case, `Name` and the `in YAML'metadata.name Make sure the '' is the same value.

The following Yaml is configured here.

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: not-allow-priv
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
    - "*"
```

![psp_config2](./pics/psp_config2.png)

### Create K8s Cluster

Apply the created ClusterRole, ClusterRoleBinding, etc. as a template in K8s cluster.

Go to Home -> Cloud and Edge Sites -> Manage -> Manage K8s -> K8s Clusters and select ”Add K8s K8s Cluster".

- name: `mk8s-cluster1`
- Site Local Access: `Enable Site Local API Access`
  - Local Domain: `mk8s.localnet' # The domain name you set here will be set as 'Site name + Local Domain name' to the server in Kubeconfig.
  - Port for K8s API Server: `Default k8s Port`
- voltconsole Access: `Enable Voltconsole API Access' If this is enabled, you can access the local API via the Console.However, you must have the appropriate user permissions to have Console access.
- POD Security Policies：`Default Pod Security Policy`
- K8s Cluster Roles: `Custom K8s Cluster Roles`
  - List of Cluster Role List: `shared/ves-io-admin-cluster-role`
- K8s Cluster Role Bindings: `K8s Cluster Role Bindings`
  - List of Cluster Role Binding List: `system/admin-crb`

![k8s_cluster_config1](./pics/k8s_cluster_config1.png)

### Create AppStack site

AppStack site pre-creates a template for the Site, and when a Site with the same name as AppStack is Registered, the template is downloaded and provisioned to CE.

In this example, you can configure Multi node cluster settings, interface Bonding, etc., but in this example, you will simply configure it on a 1 node cluster.

Home -> Cloud and Edge Sites -> Manage -> Site Management
 -> In App Stack Sites, select "Add App Stack Site".

- name: `mk8s-cluster1`
- Generic Server Certified Hardware: `vmware-voltstack-combo` (depending on your environment)
- List of Master Nodes: `master-0`
- Latitude: `Latitude'
- Longitude: `Longitude'
- Advanced Configuration: Enable `Show Advanced Fields`
  - Site Local K8s API access: `Enable Site Local K8s API access`
    - Enable Site Local K8s API access: `system/mk8s-cluster1`
  - Enable/Disable VMs support: `VMs support Enabled` # When enabled, Kubevirt is available.

When created, the Site Admin State is set to `Waiting for Registration`.

![appstack_site_config1](./pics/appstack_site_config1.png)

### Provisioning of CE

As usual, configure the initial configuration of the CE and Register it with the same name as the AppStack site.

![appstack_site_reg1](./pics/appstack_site_reg1.png)

After provisioning is complete, the cluster will be displayed in Home -> Cloud and Edge Sites -> Managed k8s -> Overview so that you can see resources such as Pods.

![appstack_site_overview1](./pics/appstack_site_overview1.png)
![appstack_site_overview2](./pics/appstack_site_overview2.png)

## K8s Operation using Local API

Using the Local API, it is possible to perform the same operations as a regular K8s cluster.
You can use Kubectl to create Pods and services using Manifest.

You can check the created object from Managed K8s Overview, and you can also check the resource utilization of the Pod, etc.
You can also set CRD, etc., which are not available in vk8s.

The Kubeconfig for the connection can be obtained from the Actions of the corresponding Site.
Local Kubeconfig is required to connect directly to the Site.Global Kubeconfig connects to the Kubenetes API of the site via the Console.

![appstack_site_overview3](./pics/appstack_site_overview3.png)

mk8s uses the local API just like regular Kubernetes.This requires name resolution for the FQDN of the Server in Kubeconfig.

For example 'server: https://mk8s-cluster1.mk8s.localnet:65443 If it says '#', set the appropriate record in DNS or /etc/hosts.

``` bash
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.201.2 mk8s-cluster1.mk8s.localnet
```

You can actually connect using Kubectl.

```
kubectl --kubeconfig ves_system_mk8s-cluster1_kubeconfig_local.yaml get node
NAME       STATUS   ROLES        AGE   VERSION
master-0   Ready    ves-master   25h   v1.21.7-ves
```

### mk8s Pod/Service Creation and Ingress Gateway

Create Nginx using Kubectl like below.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx
```

```
[lab@localhost ~]$ kubectl --kubeconfig ves_system_mk8s-cluster1_kubeconfig_local.yaml create namespace my-nginx
namespace/my-nginx created

[lab@localhost ~]$ kubectl --kubeconfig ves_system_mk8s-cluster1_kubeconfig_local.yaml apply -f mk8s-sample.yaml -n my-nginx
deployment.apps/my-nginx created
service/my-nginx created
[lab@localhost ~]$ kubectl --kubeconfig ves_system_mk8s-cluster1_kubeconfig_local.yaml get po -n my-nginx
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-5b56ccd65f-4npvc   2/2     Running   0          24s
my-nginx-5b56ccd65f-t7jpj   2/2     Running   0          24s
[lab@localhost ~]$ kubectl --kubeconfig ves_system_mk8s-cluster1_kubeconfig_local.yaml get svc -n my-nginx
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   100.127.192.1   <none>        443/TCP   25h
my-nginx     ClusterIP   100.127.193.0   <none>        80/TCP    26s
```

You can also see what you created here from the UI.

![mk8s_ui_ops1](./pics/mk8s_ui_ops1.png)
![mk8s_ui_ops2](./pics/mk8s_ui_ops2.png)

The service created here can also be published in the DCS Load balancer.
The setting method is the same as vk8s.However, if there is no mk8s namespace on the Console, create the same Namespace and configure Origin pool and HTTP Loadbalancer in that Namespace.

### mk8s VM Creation and Ingress Gateway

Launch the Cirros test VM as shown below.If you apply the Manifest to raise the normal Pod, the VM will rise up as a Pod.You can also configure the Service as well as Container.

```
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: true
  template:
    metadata:
      labels:
        kubevirt.io/size: small
        kubevirt.io/domain: testvm
        run: testvm
    spec:
      domain:
        devices:
          disks:
            - name: containerdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: containerdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userDataBase64: SGkuXG4=
---
apiVersion: v1
kind: Service
metadata:
  name: testvm
spec:
  ports:
  - port: 22
    protocol: TCP
    targetPort: 22
    name: ssh
  - port: 80
    protocol: TCP
    targetPort: 80
    name: http
  selector:
    run: testvm
```

You can see that the VM is `Running` as shown below, and the VM is running as a Pod.

```
[lab@localhost ~]$ kubectl --kubeconfig ves_system_mk8s-cluster1_kubeconfig_local.yaml get vm -n my-nginx
NAME     AGE   STATUS    VOLUME
