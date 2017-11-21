---
revealOptions:
    transition: "none"
---

### Networking in Kubernetes

In this section we'll look at some of the more fundamental concepts of
networking within Kubernetes.

---

### Disclaimer

We won't be looking at any specific implementation, as each
product is implemented slightly differently.

---

### Recap

Four levels of abstraction:
- Cluster
- Node
- Pod
- Container

---

### Cluster

- is made up of multiple machines
- all networked together using a network solution

---

### Node

- has external IP address
- has internal IP address
- is allocated a subnet to assign to any Pods it runs (ie. 10.0.0.0/24
  for GKE)
- knows the relevant Node for each internal subnet
<!-- mention NAT -->

---

### Pod

- has internal IP address
- assigned by host Node

---

### Container

- 'Inherits' the IP address of its Pod
- can refer to containers in same Pod via 'localhost'

---

### The fundamental requirements

- no NAT between containers
- no NAT between containers & nodes
- the IP that a container sees itself as is the same IP that others see it as

---

### No NAT between Containers

- all containers can communicate with all other containers without NAT
- Inter-node communication is transparent to Pods.

---

### no NAT between Containers & Nodes

- A Node must be able to directly communicate any Pod in the cluster.
- A Pod must be able to directly communicate with any Node in the cluster.
- Again, must all be transparent.

---

### Implementations

- Flannel (Overlay Network)
- Google Compute Engine (Overlay-free)
- Project Calico (Overlay-free)
- More exist (https://kubernetes.io/docs/concepts/cluster-administration/networking/)

---

### Flannel

- Default plugin for Kubernetes
- Simple
- Consistent Performance

---

### Google Compute Engine

- Great if you control your network.
- Can get native speed with no overhead.

---

### Calico

- Don't need to control the network (AWS, Azure)
- Finer-Grained access controls than default (Flannel)


---

### Exercise

These exercises are designed to help with getting an understanding of
pods.

---

### Multiple Containers in a Pod

Containers in a Pod share the same IP Address and can talk to one
another via localhost.

---

### Multiple Containers in a Pod

Let's first create our Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: "multi-container"
spec:
  containers:
    -
      args:
        - "9999999"
      command:
        - "sleep"
      image: "appropriate/curl"
      name: "shell"
    -
      image: "nginx"
      name: "nginx"
```

---

### Multiple Containers in a Pod

We can now jump into it:

```
$ kubectl exec -it multi-container -c shell /bin/sh
$ curl localhost
<welcome to Nginx>
...
```

---

### Access Pod from Node

First we need to grab the Ip address of the Pod:

```
$ kubectl describe pod multi-container | grep IP:
```

We can then (from the Node), directly communicate with the Pod:

```
$ curl $POD_ID
<welcome to Nginx>
...
```

---

### Access Node from Pod

Setup temporary server to listen on Node and Grab the Node Ip address:

```
$ nc -l 8000 < multi-container.yml & &> /dev/null
$ ip addr
```

Use kubectl to exec back into our _multi-container_, then try curling
the IP address, Port combination.

```
$ curl $NODE_IP:8000
apiVersion: v1
...
```

---

### Cleanup

All Done!

We can tidy everything up with

```
$ kubectl delete pods multi-container
```

---

[FIN](./01_outline.md)
