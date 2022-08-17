# How to use multiple virtual-sites

A Namespace can have multiple Virtual-sites. For example, pref-tokyo and pref-osaka are created, and multiple actual Sites exist within the Virtual-site.
When you create Vk8s, you can choose the Virtual-site it belongs to.
The figure shows a hierarchical structure as shown below.

![vsite_object](./pics/vsite_object.svg)

When you create a Manifest such as a Deployment or Service, it will be reflected in all Virtual Sites in Vk8s. Use Annotation `ves.io/sites` if the manifest is reflected only in a specific Virtual-site.

For example, if there are two virtual-sites: pref-tokyo and pref-osaka, and only pref-tokyo is reflected, the annotation is

```metadata:
  annotations:
    ves.io/virtual-sites: namespace/pref-tokyo
```

becomes.

When specifying multiple

```metadata:
  annotations:
    ves.io/virtual-sites: namespace/pref-tokyo,namespace/pref-osaka
```
becomes.

## Create a workload with multiple virtual sites

Create namespace:`multi-sites` and set 2 following 2 virtual sites in vk8s.

Virtual site1
- Name: `pref-tokyo`
- Site type: `CE`
- Site Selecter Expression: `pref:tokyo`

Virtual site2
- Name: `pref-osaka`
- Site type: `CE`
- Site Selecter Expression: `pref:osaka`

- If you are a Free user, please delete the existing Namespace first and then create it.

![v8s_multi_vsite](./pics/v8s_multi_vsite.png)

Create a Deployment in two Virtual-sites `pref-tokyo` and `pref-osaka` in vk8s.

pref-tokyo

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tokyo-app
  annotations:
    ves.io/virtual-sites: multi-sites/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tokyo-app
  template:
    metadata:
      labels:
        app: tokyo-app
    spec:
      containers:
        - name: tokyo-app
          image: dnakajima/inbound-app:1.0
          ports:
            - containerPort: 8080
```

pref-osaka

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: osaka-app
  annotations:
    ves.io/virtual-sites: multi-sites/pref-osaka
spec:
  replicas: 1
  selector:
    matchLabels:
      app: osaka-app
  template:
    metadata:
      labels:
        app: osaka-app
    spec:
      containers:
        - name: osaka-app
          image: dnakajima/inbound-app:1.0
          ports:
            - containerPort: 8080
```


A Deployment is created in each Virtual Site.

![deployment_multi_vsite](./pics/deployment_multi_vsite.png)

Create Service in two Virtual-sites `pref-tokyo` and `pref-osaka` in vk8s


pref-tokyo

```
apiVersion: v1
kind: Service
metadata:
  name: tokyo-app
  labels:
    app: tokyo-app
  annotations:
    ves.io/virtual-sites: multi-sites/pref-tokyo
spec:
  ports:
  - port: 8080
    targetport: 8080
    protocol: TCP
  selector:
    app: tokyo-app
```

pref-osaka

```
apiVersion: v1
kind: Service
metadata:
  name: osaka-app
  labels:
    app: osaka-app
  annotations:
    ves.io/virtual-sites: multi-sites/pref-osaka
spec:
  ports:
  - port: 8080
    targetport: 8080
    protocol: TCP
  selector:
    app: osaka-app
```

## Configure Ingress Gateway

Load balance and expose the two created services.

![ingress_multi_vsite](./pics/ingress_multi_vsite.svg)

### Creating an origin pool

Register the created workloads to the origin-pool as `tokyo-app` and `osaka-app` respectively.

Go to Manage -> Origin Pools and select "Add Origin Pool"

- Name: `tokyo-app`
-Origin Servers
  - Select Type of Origin Server: `k8sService Name of Origin Server on given Sites`
  - Service Name: Enter `tokyo-app.multi-sites`. (`kubernetes service name.namespace` format)
  - Select Site or Virtual Site: `Virtual Site` -> `multi-sites/pref-tokyo`
  - Select Network on the Site: `Vk8s Networks on Site`
  -Port: `8080`

- Name: `osaka-app`
-Origin Servers
  - Select Type of Origin Server: `k8sService Name of Origin Server on given Sites`
  - Service Name: Enter `osaka-app.multi-sites`. (`kubernetes service name.namespace` format)
  - Select Site or Virtual Site: `Virtual Site` -> `multi-sites/pref-osaka`
  - Select Network on the Site: `Vk8s Networks on Site`
  -Port: `8080`

![origin_multi_vsite1](./pics/origin_multi_vsite1.png)

### Creating an HTTP loadbalancer

Go to Manage -> HTTP Load Balancers and select “Add HTTP load balancer”.

- Name: `multi-vsite-lb`
- Domains: `dummy.domain-name` (When set, the domain name will be issued from DCS to DNS info. Please set the issued domain name after setting.)
- Select Type of Load Balancer: `HTTP`
- Default Origin servers: set the two origin pools created above

Weight is set to 100,100, but you can adjust the low balance ratio by changing the ratio.

![http_lb_multi_vsite1](./pics/http_lb_multi_vsite1.png)


If you check with Curl, etc., you can see that the load is balanced between tokyo-app and osaka-app.

```
curl http://ves-io-3b89b61f-b82b-4140-915a-96f56818fd56.ac.vh.ves.io/
<html>
<body>
This pod is running on tokyo-app-767948955-jpbnx
</body>
</html>

curl http://ves-io-3b89b61f-b82b-4140-915a-96f56818fd56.ac.vh.ves.io/
<html>
<body>
This pod is running on osaka-app-7cc7958f77-2x4br
</body>
</html>
```
