# Service Policy (Ingress Gateway)

Ingress Gateway provides HTTP-based security.
Traffic coming into the DCS Node from the outside is the Client, and the Kubernetes Service is the Server.
For example, in the following cases, the external network (Any) and the Kubernetes Service will be the Service with app:web configured.

![service_policy1](./pics/service_policy1.svg)

In the following cases, Client will be the Pod with app:web and Server will be the Service with app:DB.

![service_policy2](./pics/service_policy2.svg)

## Service policy structure

Create a Clinet condition in the Service Policy Rule and apply the Service Policy Rule to the Server in the Service Policy Rule.In the Service Policy Set, apply Service Policy Rules to the Namespace.

![service_policy3](./pics/service_policy3.svg)

## Service Policy

Create 2 services, and the service with deny-server externally (HTTP loadbalancer) denies access to the Path of the url /deny/.

### Communication control from the Internet

Create 2 services, and the service with deny-server externally (HTTP loadbalancer) denies access to the Path of the url /deny/.

![service_policy4](./pics/service_policy4.svg)

#### Kubenretes Configuration

Create a known key in Shared Configuration.
If you are a Free tenant, delete the existing Label before creating it.

Label key: `app`

Label value:

- `allow-server`
- `deny-server`

Let namespace be `security` and virtual-site create `pref-tokyo'.
Create 2 Pods, app:allow-server and app:deny-server with different labels.

allow-server

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: allow-server
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: allow-server
  template:
    metadata:
      labels:
        app: allow-server
    spec:
      containers:
        - name: allow-server
          image: dnakajima/inbound-app:3.0
```

deny-server

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deny-server
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deny-server
  template:
    metadata:
      labels:
        app: deny-server
    spec:
      containers:
        - name: deny-server
          image: dnakajima/inbound-app:3.0
```

Create 2 service, corresponding to the Pod you created.

allow-server

```
apiVersion: v1
kind: Service
metadata:
  name: allow-server
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector:
    app: allow-server
  type: ClusterIP
```

deny-server

```
apiVersion: v1
kind: Service
metadata:
  name: deny-server
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
  labels:
    app: deny-server
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector:
    app: deny-server
  type: ClusterIP
```

#### Configure Origin pool

Configure the Ingress Gateway so that the service you created can be accessed externally.Register the 2 services you created as Origin pools. In Manage -> Origin Pools, select “Add Origin Pool”.

<b>Allow-server</b>

- Name: `allow-server`
- Origin Servers:
  - Select Type of Origin Server: `k8s Service Name of Origin Server on given Sites.`
  - Service Name: `allow-server.security` (”Kubernetes service name. namespace”)
  - Select Site or Virtual Site: `Virtual Site`
  - Virtual Site: `pref-tokyo`.
  - Select Network on the Site: `Vk8s Networks on Site`
- Port: `80`

<b>Deny-server</b>

- Name: `deny-server`
- Origin Servers:
  - Select Type of Origin Server: `k8s Service Name of Origin Server on given Sites.`
  - Service Name: `deny-server.security` (”Kubernetes service name. namespace”)
  - Select Site or Virtual Site: `Virtual Site`
  - Virtual Site: `pref-tokyo`.
  - Select Network on the Site: `Vk8s Networks on Site`
- Port: `80`

#### Configure HTTP Load Balancer

The Label selection for the Load Balancer in Service Policy is done with the Labels of the HTTP/TCP Load Balancer.
Under Manage -> HTTP Load Balancers, select ”Add HTTP load balancer".

- Name: `allow-server-lb`
- Labels: `app: allow-server`
- Domains: `dummy.localhost' (if set, the DCS will issue the domain name to DNS info.Please set the domain name that was paid out after setting up.)
- Select Type of Load Balancer: `HTTP`
- Default Route Origin Pools: 'security/allow-server' (Origin pool created above)

If set, the domain name will be sent out from the DCS to DNS info.Set it to the Domains of the load balancer you created, or to the CNAME record of any DNS server.
Nginx's WebUI will be displayed when you access the domain you set from the outside.

![service_policy5](./pics/service_policy5.png)

Similarly, create a load balancer for the Deny server.

- Name: `deny-server-lb`
- Labels: `app: deny-server`
- Domains: `dummy.localhost' (if set, the DCS will issue the domain name to DNS info.Please set the domain name that was paid out after setting up.)
- Select Type of Load Balancer: `HTTP`
- Default Route Origin Pools: `security/deny-server' (Origin pool created above)

#### Confirm connection to Service

Verify that you have access to the service that you created.
<http://url />, <http://url/allow />, <http://url/deny Make sure you have access to >.

![adc_deny_1](./pics/adc_deny_1.png)
![adc_deny_2](./pics/adc_deny_2.png)
![adc_deny_3](./pics/adc_deny_3.png)

#### Create Service policy

Create a Service Policy in `Manage` -> `Servive Policies' -> 'Service Policy'.

deny- Denies access to `/deny` in the created deny-server.
The creation procedure is as follows.
1. Create a rule to allow all services
2. deny-Create a rule to deny '/deny' in server
3. Applying Rules

<b>1.Create rules to allow all services</b>

Because there is an implicit Deny, create a policy to allow everything

- name: `allow-any`
  - Server Selection: `Any Server`
  - Select Policy Rules: `Allow All Requests`

<b>2. deny-Create a rule to deny `/deny' in server</b>

- name: `deny-server`
  - Server Selection: `Group of Servers by Label Selector`
    - Selector Expression: `app: in (deny-server)`
  - Select Policy Rules: `Custom Rule List`
    - Rules
      - Name: deny-rule1 (Rule1)
        - Rule Specification
          - Action: `Deny`
          - Client Selection: `Any Client`
          - HTTP Method: `ANY`
          - HTTP Path: `Prefix Values : /deny`
      - Name: allow-others (Rule2)
        - Rule Specification
          - Action: `Allow`
          - Client Selection: `Any Client`

![adc_deny_4](./pics/adc_deny_4.png)
![adc_deny_5](./pics/adc_deny_5.png)

<b>3. Application of rules</b>

Service Policy Add Service Policy to Active Service Policies in `Manage` -> `Servive Policies' -> 'Active Service Policies'

- service-policy-set1
  - Policies: Select policy: `[1: deny-server, 2:allow-server]`

![adc_deny_6](./pics/adc_deny_6.png)

#### Confirm Settings

Verify that you have access to the service that you created.
deny-web-server<http://url />,<http://url/allow /> appears fine, but <http://url/deny > will check to return a 403 error.

![adc_deny_1](./pics/adc_deny_1.png)
![adc_deny_2](./pics/adc_deny_2.png)
![adc_deny_7](./pics/adc_deny_7.png)

Verify that you have access to the service that you created.
allow-web-server<http://url />,<http://url/allow />,<http://url/deny >, is accessible.

From `Load Balancers' -> 'NTTP Load Balancers' you can see the log that hit the filter. The logs show the Pod name of the source, the IP address and protocol of the destination, and the policies that were hit.

![adc_deny_8](./pics/adc_deny_9.png)

#### Apply Service Policy to Kubernetes Service

In DCS, there are 3 types of Kubernetes Service types: `HTTP_PROXY`, `TCP_PROXY`, and `TCP_PROXY_WITH_SNI`, and the default is `TCP_PROXY`.
Because TCP Proxy does not have an HTTP-based filter, Kubernetes ServiceのマニフェストにHTTP_PROXYを有効にするannotationves.io/proxy-typeを設定します i'm not.
Since the Free account cannot build 'app-client', remove the allow-server and then add the manifest.

Change the Manifest created above to the following:

```
apiVersion: v1
kind: Service
metadata:
  name: deny-server
  annotations:
    ves.io/proxy-type: HTTP_PROXY
    ves.io/virtual-sites: security/pref-tokyo
  labels:
    app: deny-server
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector:
    app: deny-server
  type: ClusterIP
```

It also launches a client that connects to Serivce.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-client
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-client
  template:
    metadata:
      labels:
        app: app-client
    spec:
      containers:
        - name: app-client
          image: dnakajima/netutils:1.3
```

You can see that running `curl -v deny-server/deny/' from app-client returns a 403 error.

![adc_deny_7](./pics/adc_deny_8.png)
