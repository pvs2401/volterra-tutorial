# Network policy

Network Policy provides Ingress/Egress security for L3-L4.
The Ingress Rule sets the traffic coming into the Endpoint, and the Egress Rule sets the traffic coming out of the Endpoint.
Because this policy works statefully, you do not need to configure the Ingress Rule to return traffic that the Egress Rule allows.

![network_policy1](./pics/network_policy1.svg)

The same is true for communication in the same Namespace, with the Ingress Rule setting the traffic coming into the Endpoint and the Egress Rule setting the traffic coming out of the Endpoint.

![network_policy2](./pics/network_policy2.svg)

## Network policy structure

The configuration creates an Ingress/Egress condition in `Netrowk Policy` and applies Network Policy Rules to Namespaces in 'Active Network Policies'.

![network_policy3](./pics/network_policy3.svg)

## Network Policy

### Communication control to the Internet

Create namespace: 'security' and set vk8s to Virutal site.
Name: `pref-tokyo`
Site type: `CE`
Site Selecter Expression: `pref:tokyo`

- If you are a Free user, delete the existing Namespace first and then create it.

Create a known key in Shared Configuration.

- If you are a Free user, delete the existing Known label `pref:osaka` first and then create it.

Label key: `app`

label value:

- `allow-client`
- `deny-client`
- `server-app`

![shared_label](./pics/shared_label.png)

Create 2 Pods, app:allow-client and app:deny-client with different labels.

deny-client

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deny-client
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deny-client
  template:
    metadata:
      labels:
        app: deny-client
    spec:
      containers:
        - name: deny-client
          image: dnakajima/netutils:1.3
```

allow-client

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: allow-client
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: allow-client
  template:
    metadata:
      labels:
        app: allow-client
    spec:
      containers:
        - name: allow-client
          image: dnakajima/netutils:1.3
```

The Pod you created, deny-client, denies access to Google-DNS.
The creation procedure is as follows.
1. Create a rule that allows everything
2. Create a rule to deny Google-DNS
3. Applying Rules


![network_policy_same_node](./pics/network_policy_same_node.svg)

Create a Network Policy in `Manage` -> `vk8s Network Policies' -> 'Network Policies'.

![network_policy_ui1](./pics/network_policy_ui1.png)


1. Because there is an implicit Deny, create a policy to allow everything.

You create a rule from 'Add network policy'.

- name: `allow-any`
  - Policy For Endpoints
    - Endpint(s): `Any Endpoints`
  - Ingress Rules
    - Name: `allow-any-ingress`
    - Action: `Allow`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`
  - Egress Rules:
    - Name: `allow-any-egress`
    - Action: `Allow`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`

![network_policy_block1](./pics/network_policy_block1.png)

![network_policy_block2](./pics/network_policy_block2.png)

2. Create a rule to deny communication from `deny-client' to Google-DNS.

- name: `deny-client`
  - Policy For Endpoints
    - Endpint(s): `Label Selector`
      - Selector Expression: `app:in (deny-client)`
  - Ingress Rules
    - Name: `allow-any-ingress`
    - Action: `Allow`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`
  - Egress Rules: (Rule-1)
    - Name: `deny-destination`
    - Action: `Deny`
    - Logging Action: 'Log' # Show Displays when Advanced Fields is enabled.If you enable Log, Site Security displays the log.
    - Select Other Endpoint: `IPv4 Prefix List`
      - IPv4 Prefix List: `8.8.4.4/32`, `8.8.8.8/32`
    - Select Type of Traffic to Match: `Match All Traffic`
  - Egress Rules: (Rule-2)
    - Name: `allow-others`
    - Action: `Allow`
    - Logging Action: `Do Not Log`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`

3. Apply the rules you created.

Select the policy you created from `Manage` -> `vk8s Network Policies' -> 'Active Network Policies' and apply it.Since the policies are evaluated in order from 1, set the individual policy settings to come to the wakaban.

  -  Active Network Policies: [1: deny-client, 2: allow-client]

![network_policy_block3](./pics/network_policy_block3.png)

You can check the filter from the Pod.You can connect to the target Pod from the Virtual K8s Pods via Exec to Container.

![network_policy_block4](./pics/network_policy_block4.png)

After selection, you can connect to the container by bash by selecting deny-client or allow-client from Container to exec to and putting bash in Command to execute.

- You can also download kubeconfig and connect with kubectl.

I can see that deny-client cannot ping 8.8.8.8 due to google-dns policy, but allow-client can ping.

![network_policy_block5](./pics/network_policy_block5.png)

You can see the log that hit the filter from System -> Site Security.
The logs show the Pod name of the source, the IP address and protocol of the destination, and the policies that were hit.

![network_policy_block6](./pics/network_policy_block6.png)

### Communication control within the same Kubernetes Clouster

Create an additional server-app.Also add `server-app` as Key to App label in Shared namespace.

In this case, only app:allow-client can communicate to app:server-app, and app:deny-client can communicate to app:server-app.
Free does not allow you to create more than 3 Deplyoments, so you will need more than Individual tenant contracts after this.

![network_policy_same_node1](./pics/network_policy_same_node1.svg)

app: Create web Pods and services.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server-app
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: server-app
  template:
    metadata:
      labels:
        app: server-app
    spec:
      containers:
        - name: server-app
          image: dnakajima/inbound-app:1.0
          ports:
            - containerPort: 8080
              protocol: TCP
```

```
apiVersion: v1
kind: Service
metadata:
  name: server-app
  labels:
    app: server-app
  annotations:
    ves.io/virtual-sites: security/pref-tokyo
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector:
    app: server-app
  type: ClusterIP
```

1. Because there is an implicit Deny, create a policy to allow everything.

You create a rule from 'Add network policy'.

- name: `allow-any`
  - Policy For Endpoints
    - Endpint(s): `Any Endpoints`
  - Ingress Rules
    - Name: `allow-any-ingress`
    - Action: `Allow`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`
  - Egress Rules:
    - Name: `allow-any-egress`
    - Action: `Allow`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`

2. deny-Creates a rule for the client.

- name: `deny-client`
  - Policy For Endpoints
    - Endpint(s): `Label Selector`
      - Selector Expression: `app:in (deny-client)`
  - Ingress Rules
    - Name: `allow-any-ingress`
    - Action: `Allow`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`
  - Egress Rules: (Rule-1)
    - Name: `deny-destination`
    - Action: `Deny`
    - Logging Action: 'Log' # Show Displays when Advanced Fields is enabled.If you enable Log, Site Security displays the log.
    - Select Other Endpoint: `Label Selector`
      -  Selector Expression: `app:in (server-app)`
    - Select Type of Traffic to Match: `Match All Traffic`
  - Egress Rules: (Rule-2)
    - Name: `allow-others`
    - Action: `Allow`
    - Logging Action: `Do Not Log`
    - Select Other Endpoint: `Any Endpoint`
    - Select Type of Traffic to Match: `Match All Traffic`


Create a Network Policy for the Deny client and create Ingress Rules and Egress Rules.

3. Apply the rules you created.

Select the policy you created from Active Network Policies and apply it.Since the policies are evaluated in order from 1, set the individual policy settings to come to the wakaban.

  -  Active Network Policies: [1: deny-client, 2: allow-client]

I can see that the deny-client cannot curl to server-app because it has a policy of deny-client, but allow-client can curl.

![network_policy_same_node5](./pics/network_policy_same_node3.png)
![network_policy_same_node6](./pics/network_policy_same_node4.png)
