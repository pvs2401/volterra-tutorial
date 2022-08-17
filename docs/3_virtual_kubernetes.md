# Configuring Virtual Kubernetes

Virtual Kubernetes is a concept unique to DCS. DCS treats multiple Kubernetes Clusters as one virtual Kubernetes.
Therefore, Virtua Kubernetes can have multiple Virtual Sites, and one Kubernetes manifest can be distributed to multiple Kubernetes clusters.

![vk8s](./pics/vk8s.svg)

## Create User Namespace

Create a Namespace because the Virtual Kubernetes will be created in the user Namespace.
From `Personal Management` -> `My Namespaces` in Administration, open `Add namespace`, enter Namapace name and save.
Select the created Namespace to put it in User Namaspace.

![namespace1](./pics/namespace1.png)
![namespace2](./pics/namespace2.png)

## Creating Virtual Sites

Go to Distributed Apps and navigate to the Namespace you created. Select `Add Virtual site` from Manage -> Virtual sites.
Select the virtual-site name for name, select CE for Site Type, and select the label set for Site for Site Selector Expression. Select Continue to create a virtual site.

Set up the following two virtual sites.
Name: `pref-tokyo`
Site type: `CE`
Site Selector Expression: `pref:tokyo`

Name: `pref-osaka`
Site type: `CE`
Site Selector Expression: `pref:osaka`

![vsite1](./pics/vsite1.png)
![vsite2](./pics/vsite2.png)

## Create virtual kubernetes

Select `Add Virtual K8s` from Applications -> Virtual k8s. Enter a name and select `pref-tokyo` created from Select vsite ref. Click Add Virtual k8s to create Virtual kubernetes.
*It takes several tens of seconds to create.

![vk8s1](./pics/vk8s1.png)
![vk8s2](./pics/vk8s2.png)
![vk8s3](./pics/vk8s3.png)

## Create deplyoment

Select the created Virtual K8s to display the Kubernetes creation screen. Workloads are created in the actual Site when you create a Deployment or Service in the same way as a normal Kubernetes Manifest.

If you set the Deployment from Add Deployment as shown below, the container will be launched on the corresponding Site.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
specifications:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  templates:
    metadata:
      labels:
        app: nginx
    specifications:
      containers:
      -name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

![vk8s_deployment1](./pics/vk8s_deployment1.png)

## Create Service

When you select `Add service`, a screen to enter Yaml (json) opens.
If you set the Service as below, the Service will be set for the corresponding Site.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
specifications:
  ports:
  - ports: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```

![vk8s_service1](./pics/vk8s_service1.png)

Clicking on `Endpoints` will display the Pods recognized by the Service.

![vk8s_service_pod](./pics/vk8s_service_pod.png)

## Access from Kubectl

Download the kubeconfig used by kubectl from Virtual K8s in the user namespace.

Select `Kubeconfig` from `...` of the created vk8s in Applications -> Virtual k8s and set the deadline for Kubecofig.

![kubeconfig1](./pics/kubeconfig1.png)

Clicking on `Downlaod Credential` will download Kubeconfig.

![kubeconfig2](./pics/kubeconfig2.png)

Access vk8s using kubectl.

```
kubectl --kubeconfig ~/Downloads/ves_trial_vk8s.yaml get deployment
NAME READY UP-TO-DATE AVAILABLE AGE
nginx-deployment 0/1 0 0 9s
```

The maximum number of days for Kubeconfig etc. can be changed in Administration -> Tenant Settings -> Tenant Overview -> Credential Expiry Policy up to 365 days.

![expiredate](./pics/expiredate.png)
