# F5 Distributed Cloud Services Tutorial

This document provides basic knowledge of DCS and basic configurations for using DCS Mesh/AppStack when using the functions of F5 Distributed Cloud Services (DCS) (<https://www.f5.com/cloud>). is intended for learning.

* This document is unofficial for F5 Distributed Cloud Service.

## easy explanation

DCS provides a Kubernetes infrastructure (AppStack) and a gateway (Mesh) that provides Application Security.
Free account can be obtained from <https://console.ves.volterra.io/signup/usage_plan>.

## Mesh function

Mesh provides App to App communication control and encryption, advanced Ingress/Egress gateway functionality, load balancing, and more.

* L3-L4 security
* L7 Security
* API Gateway
*LB

See here for other functions. <https://www.f5.com/cloud/products/platform-overview>

## AppStack Features

As a managed Kubernetes, AppStack provides features such as node management, application delivery, and enterprise-level secrets.

* Managed Kuberners
* Clustering
* Fleet management
* KMS

See here for other functions. <https://www.f5.com/cloud/products/platform-overview>

## Tutorial

1. [Basic Knowledge of DCS](./docs/1_dcs-tutorial.md)
1. [DCS Node installation and configuration](./docs/2_dcs-install.md)
1. [Configure Virtual Kubernetes](./docs/3_virtual_kubernetes.md)
1. [Ingress Gateway Settings](./docs/4_ingress_gateway.md)
1. [How to use multiple virtual sites](./docs/5_multiple_vsite.md)
1. [App to App connection settings](./docs/6_app_app.md)
1. [Application Delivery Controller](./docs/7_app_delivery_controller.md)
1. [Network policy](./docs/8_network_policy.md)
1. [Service policy (Ingress Gateway)](./docs/9_service_policy.md)
1. [Manage k8s](./docs/12_appstack_site.md)
