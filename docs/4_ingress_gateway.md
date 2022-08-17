# Configure Ingress Gateway

Since Kubernetes Service cannot be accessed externally out of the box, configure the Ingress Gateway so that you can connect externally to the application on the AppStack that you created.You can use Mesh on the Internet, or you can use a local DCS Node.

The Ingress Gateway acts as an HTTP/TCP loadbalancer.The destination of the Loadlabancer is defined as the Origin Pool.

![ingress_gw1](./pics/ingress_gw1.svg)

## Create Origin pool

Configure the Ingress Gateway so that Nginx created in the Virtual Kubernetes configuration can be accessed externally.Register the Nginx Service you created as the Origin pool. Select `Add Origin Pool` in Manage -> Loab Balancers -> Origin Pools.

Configure the following settings

- Name: `nginx-endpoint`
- Origin Servers
  - Select Type of Origin Server: `k8s Service Name of Origin Server on given Sites.`
  - Service Name: `nginx.Enter 'namespace'. ('kubernetes service name.format of 'namespace')
  - Select Site or Virtual Site: `Virtual Site` -> `namespace/pref-tokyo`
  - Select Network on the Site: `Vk8s Networks on Site`
- Port: `80`

![origin_server1](./pics/origin_server1.png)
![origin_server2](./pics/origin_server2.png)

## Connect from the Internet

### Configure HTTP load balancer

In Home -> Load Balancers -> HTTP Load Balancers, select “Add HTTP load balancer”.

- Name: `nginx-lb`
- Domains: `dummy.domain-name' (If set, the DCS will issue the domain name to DNS info.Please set the domain name that was paid out after setting up.)
- Select Type of Load Balancer: `HTTP`
- Default Origin servers: `namespace/nginx-endpoint` (Origin pool created above)

If set, the domain name will be sent out from the DCS to DNS info.Set it to the Domains of the load balancer you created, or to the CNAME record of any DNS server.
Nginx's WebUI will be displayed when you access the domain you set from the outside.

![lb_config1](./pics/lb_config1.png)
![lb_config2](./pics/lb_config2.png)
![lb_config3](./pics/lb_config3.png)
![lb_config4](./pics/lb_config4.png)

It will appear when you enter your domain in your browser.

![nginx_ss](./pics/nginx_ss.png)
> It may take 1-2 minutes to propagate DNS and reflect the configuration.

## Access from Local Interface

### Configure HTTP Loadbalancer

Under Manage -> HTTP Load Balancers, select ”Add HTTP load balancer".

- Name: `nginx-lb`
- Domains: `nginx.domain-name`
- Select Type of Load Balancer: `HTTP`
- Default Origin Pools: `namespace/nginx-endpoint` (Origin pool created above)
- VIP Configuration: Enable `Show Advanced Fields` and specify `Advertise Custom`.
- Edit Configure
  - List of Sites to Advertise
    - Select Where to Advertise: `virtual-site`
    - Site Network: `Inside and Outside Network`
    - Virtual Site Reference: `namespace/pref-tokyo`

![lb_config5](./pics/lb_config5.png)
![lb_config6](./pics/lb_config6.png)
![lb_config7](./pics/lb_config7.png)

If you do not have local DNS, enter the domain name and IP address of the edge node that you set in /etc/hosts, or check with Curl with -H “Host: domain name”.

```
curl http://192.168.2.197 -H "Host: localhost.com"
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

** If you create 'pref-osaka' in the Virtual Site and set 'pref-osaka' in the Virtual Site Reference, when you access the IP address of the DCS Node in pref-osaka, you can dynamically tunnel between Site1 and Site2 and access the app in pref-tokyo.


## Troubleshooting

If you do not have access to Nginx, launch a container such as Ubuntu on the same Virtual-site and check if you can access Nginx via Service.

If you can access it via Service, make sure the Origin Pool is running properly.
If Global Saturs in the `Show Child Object` of Origin pool is left blank, the Service name in the configuration may be wrong or the Virtual-site may be pointing to a different site.Please open the case to support if there is no problem with the configuration.
* This may take some time, so check again after configuring the Load balancer.

Common Problems
- The Kubernetes Service Selector label is wrong and the Pod and Service are not linked
- The Service specification for Origin pool is incorrect. (For example, the namespace is wrong)
- Virtual site/Site is wrong
- The Origin pool is not specified correctly in the Load balancer.

![trouble_originpool1](./pics/trouble_originpool1.png)
![trouble_originpool2](./pics/trouble_originpool2.png)
