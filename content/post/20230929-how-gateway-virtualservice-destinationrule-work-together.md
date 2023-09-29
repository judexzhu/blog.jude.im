---
title: "How Gateway, VirtualService and DestinationRule Work Together"
date: 2023-09-29
draft: false
categories: ['istio']
tags: ['istio']
---

In Istio, the Gateway, VirtualService, and DestinationRule resources work together to manage the traffic entering the mesh and route it to the appropriate services. Here’s a brief overview of how they interact:

### 1. **Gateway:**
   - **Role:** Manages the ingress (incoming) traffic.
   - **Selector:** Determines which workloads (usually Envoy proxies) implement the Gateway.
   - **Servers:** Defines the ports, protocol, and hosts for which the Gateway is responsible.

### 2. **VirtualService:**
   - **Role:** Defines the routing rules for how the requests reaching the Gateway are forwarded within the service mesh.
   - **Hosts:** Specifies the destination hosts to which the traffic is being sent.
   - **Gateways:** Associates the VirtualService with one or more Gateways.
   - **HTTP Routes:** Specifies the match conditions and routing destinations for HTTP traffic.

### 3. **DestinationRule:**
   - **Role:** Defines policies that apply to traffic intended for a service after routing has occurred.
   - **Host:** Identifies the destination host to which the traffic is being sent.
   - **Subsets:** Defines different versions or variants of the service.
   - **Traffic Policy:** Specifies load balancing, connection pool, and outlier detection settings.

### How They Connect/Correspond with Each Other:

1. **Gateway to VirtualService:**
   - The `gateways` field in the VirtualService specifies which Gateway(s) the VirtualService is associated with.
   - The VirtualService uses the `hosts` field to specify the destination hosts, which should match the `hosts` defined in the Gateway.

2. **VirtualService to DestinationRule:**
   - The `host` field in the DestinationRule should match one of the `hosts` specified in the VirtualService.
   - The VirtualService routes traffic to the service specified in its `hosts` field, and the corresponding DestinationRule applies policies to the traffic intended for that service.

3. **VirtualService to Service:**
   - The routing rules in the VirtualService determine which service (or service subset) the traffic is sent to, based on match conditions like URI paths, headers, etc.

### Example:

Here is a simplified example to illustrate the connection:

```yaml
# Gateway
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "myapp.example.com"

# VirtualService
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-virtualservice
spec:
  hosts:
  - "myapp.example.com"
  gateways:
  - my-gateway
  http:
  - match:
    - uri:
        prefix: "/api"
    route:
    - destination:
        host: myservice
        subset: v1

# DestinationRule
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destinationrule
spec:
  host: myservice
  subsets:
  - name: v1
    labels:
      version: v1
```

- **Gateway:** Listens for traffic on `myapp.example.com`.
- **VirtualService:** Associated with `my-gateway` and routes traffic with URI prefix `/api` to the `v1` subset of the `myservice` service.
- **DestinationRule:** Defines the `v1` subset for the `myservice` host with the label `version: v1`.

### Conclusion:

The Gateway, VirtualService, and DestinationRule in Istio are interconnected resources that collectively manage and route the traffic within the service mesh, allowing for fine-grained control over traffic behavior, routing, and policies.