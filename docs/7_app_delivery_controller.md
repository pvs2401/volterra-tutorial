# Application Delivery Controller

Mesh alone provides WAF and API gateway functionality to virtual machines and existing Kubernetes Clusters.It is also possible to use Mesh on the Internet, and it is also possible to use it locally.
[4.Ingress Gateway](4_ingress_gateway.md There is no difference in the configuration and functionality of the Origin pool, only the Endpoint specification in the Origin pool is different.

![app_delivery_cntl1](./pics/app_delivery_cntl1.svg)

Nginx provides website on ESXi and KVM to publish the service via the Internet from Mesh's Inside interface.
The configuration can be outside-only one-arm or Outside/Inside routing、

![app_delivery_cntl2](./pics/app_delivery_cntl2.svg)

## Create Origin pool

Let namespace be `security` and virtual-site create `vsite-adc'.
Register the IP address of the VM you want to communicate with via Mesh in Origin-pool.

- origin pool
  - name: `nginx-vm`
  - Select Type of Orivin Server: Select `Private IP of Origin Server on given Stes`.

  - IP: `IP address of actual Nginx server'
  - Select site or Virtual site: `virtual-site`
  - virsual-site: `vsite-adc`
  - Slect Network on the site: `Outside Network`
  - port: `80`
  
    (Inside for Multiic, Outside for Single NIC)

 You can add multiple servers with > Add item.

![app_delivery_cntl_origin.png](./pics/app_delivery_cntl_origin.png)

## Configure HTTP load balancer (from Internet)

Under Manage -> HTTP Load Balancers, select ”Add HTTP load balancer".

- HTTP load balancer
  - Name: `nginx-ingress`
  - Basic Configuration
    - Domains: `dummy.domain`

    If you do not have an external DNS server, set the issued domain for CNAME to the domain name of the HTTP load Balancer, and you can access it via a web browser, for example.dummiy.Please overwrite the domain with the domain that was paid out.
    - Select Type of Load Balancer: `HTTP`
    - Default Route Origin Pools: `nginx-vm`

![app_delivery_cntl_httplb1.png](./pics/app_delivery_cntl_httplb1.png)
![app_delivery_cntl_httplb2.png](./pics/app_delivery_cntl_httplb2.png)

## Configure HTTP load balancer (from Local)

If you want direct access to the edge node, configure Custom Adcertise VIP.
Under Manage -> HTTP Load Balancers, select ”Add HTTP load balancer".

- HTTP load balancer
  - Basic Configuration
    - Domains: `localhost.com`
    - Select Type of Load Balancer: `HTTP`
  - Default Origin Servers: `nginx-vm`
  - Vip Cconfigurations
    - Show Adbanced Fields: `enable`
    - Where to Advertise VIP: `Advertise Custome`
    - Configure
      - Select Where to Advertise: `Virtual Site`
      - Site Network: `Inside and Outside Network`
      - Virtual Site Reference: `vsite-adc`

## Confirm

If set, the domain name will be sent out from the DCS to DNS info.Set it to the CNAME record of any DNS server.
When you access the domain, you will see the Nginx WebUI.
