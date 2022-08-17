# App to App connection

Within the same Site, Pods can access other Pods via Services in the same way as normal Kubernetes.
HTTP/TCP Loadbalancer settings are required when accessing services on different sites.

![app_app1](./pics/app_app1.svg)

## create vk8s manifest

Create namespace:`app-app` and set the following two virtual sites on vk8s.

Name: `pref-tokyo`
Site type: `CE`
Site Selector Expression: `pref:tokyo`

Name: `pref-osaka`
Site type: `CE`
Site Selector Expression: `pref:osaka`

- If you are a Free user, please delete the existing Namespace before creating it.

![v8s_multi_vsite](./pics/v8s_multi_vsite.png)

Create a Deployment in two Virtual-sites `pref-tokyo` and `pref-osaka` in vk8s.

pref-tokyo

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-client
  annotations:
    ves.io/virtual-sites: app-app/pref-tokyo
specifications:
  replicas: 1
  selector:
    matchLabels:
      app: app-client
  templates:
    metadata:
      labels:
        app: app-client
    specifications:
      containers:
        - name: app-client
          image: dnakajima/netutils:1.3
```

pref-osaka

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: osaka-app
  annotations:
    ves.io/virtual-sites: app-app/pref-osaka
specifications:
  replicas: 1
  selector:
    matchLabels:
      app: osaka-app
  templates:
    metadata:
      labels:
        app: osaka-app
    specifications:
      containers:
        - name: osaka-app
          image: dnakajima/inbound-app:1.0
          ports:
            - containerPort: 8080
```

A Deployment is created in each Virtual Site

![app_app_deployment](./pics/app_app_deployment.png)

Virtual-site in vk8s: create service in `pref-osaka`

```
apiVersion: v1
kind: Service
metadata:
  name: osaka-app
  labels:
    app: osaka-app
  annotations:
    ves.io/virtual-sites: app-app/pref-osaka
specifications:
  ports:
  - port: 8080
    targetport: 8080
    protocol: TCP
  selector:
    app: osaka-app
```

![app_app_service](./pics/app_app_service.png)
## Create Ingress gateway

### Create origin pool

Register the created osaka-app workload to the origin-pool.

- Name: `osaka-app`
-Origin Servers
  - Select Type of Origin Server: `k8sService Name of Origin Server on given Sites`
  - Service Name: Enter `osaka-app.multi-sites`. (`kubernetes service name.namespace` format)
  - Select Site or Virtual Site: `Virtual Site` -> `multi-sites/pref-osaka`
  - Select Network on the Site: `Vk8s Networks on Site`
  -Port: `8080`

![app_app_origin](./pics/app_app_origin.png)

### Creating an HTTP loadbalancer

Create an HTTP loadbalancer and configure the origin pool.
The domain name (osaka-app-1) set here will be registered in the pod on k8s and used by the pod to connect to the remote service.

- Name: `osaka-app-lb`
- Domains: `osaka-app-1`
- Select Type of Load Balancer: `http`
- Default Origin Servers: `app-app/osaka-app`
- VIP Configuration: Enable VIP Configuration, select `Advertise Custom` and select Configure
- Site Network: `vK8s Service Network`
- Virtual Site Reference: `app-app/pref-tokyo`
- Select Where to Advertise: `virtual-site`


![app_app_http_lb1](./pics/app_app_http_lb1.png)
![app_app_http_lb2](./pics/app_app_http_lb2.png)

### Access confirmation

Access to the pod can be accessed from the vk8s pod to the console of the container.
Select `Exec to Container` for app-client

![app_app_exec](./pics/app_app_exec.png)

Select tokyo-app and enter bash to connect to the console from Connect. By entering `dig osaka-app-1` or `curl osaka-app-1`, you can check the IP address for connection and the actual connection to osaka-app.

![app_app_pod1](./pics/app_app_pod1.png)
![app_app_pod2](./pics/app_app_pod2.png)
 
