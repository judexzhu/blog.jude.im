---
title: "Traffic Policies in DestinationRule"
date: 2023-09-29
draft: false
categories: ['istio']
tags: ['istio']
---

The `TrafficPolicy` in Istio's `DestinationRule` is used to configure traffic behavior for the service. Below is a more detailed explanation of the fields within `TrafficPolicy`:

### 1. **LoadBalancer**
   - Configures the load balancing policy. You can specify load balancer settings at the mesh-wide, service, or subset level.
   - **Simple**: Enum, can be `ROUND_ROBIN`, `LEAST_CONN`, `RANDOM`, or `PASSTHROUGH`.
   - **ConsistentHash**: Provides consistent hashing-based load balancing.
     - **HttpHeaderName**: The name of the HTTP header used to obtain the hash key.
     - **UseSourceIp**: Boolean, use the source IP address as the hash key.
     - **HttpCookie**: Describes a cookie that will be used as the hash key.
     - **MinimumRingSize**: The minimum number of virtual nodes to use for the consistent hashing load balancer.

### 2. **ConnectionPool**
   - Configures connection pool settings for an upstream host. Connection pool settings can be specified and applied at the TCP level as well as at the HTTP level.
   - **Http**: Settings for HTTP connection pool.
     - **Http1MaxPendingRequests**: Maximum number of pending HTTP requests to a destination.
     - **Http2MaxRequests**: Maximum number of requests to a backend.
     - **MaxRequestsPerConnection**: Maximum number of requests per connection to a backend.
     - **MaxRetries**: Maximum number of retries that can be outstanding to all hosts in a cluster.
   - **Tcp**: Settings for TCP connection pool.
     - **MaxConnections**: Maximum number of connections to a destination.
     - **ConnectTimeout**: TCP connection timeout.

### 3. **OutlierDetection**
   - Outlier detection is a way to detect hosts that are underperforming or misbehaving. By removing them from the healthy load balancing set, you can ensure fewer requests are sent to these hosts.
   - **ConsecutiveErrors**: Number of errors before a host is ejected from the connection pool.
   - **Interval**: Time interval between ejection sweep analysis.
   - **BaseEjectionTime**: Minimum ejection duration.
   - **MaxEjectionPercent**: Maximum percentage of hosts in the load balancing pool for the upstream service that can be ejected.

### 4. **PortLevelSettings**
   - Traffic policies specific to individual ports.
   - Overrides the destination-level settings when there is a specific port-level setting.

### 5. **TLS**
   - Configures the TLS settings for connections to the upstream service.
   - **Mode**: Enum, can be `DISABLE`, `SIMPLE`, `MUTUAL`, or `ISTIO_MUTUAL`.
   - **ClientCertificate**: Path to the client certificate when making a TLS connection.
   - **PrivateKey**: Path to the private key when making a TLS connection.
   - **CaCertificates**: Path to the CA certificates when making a TLS connection.

### 6. **ExportTo**
   - Controls the visibility of the `DestinationRule` in other namespaces.
   - Can be set to `*` (visible to all namespaces), `.` (visible to current namespace), or a list of namespaces.

### 7. **TrafficPolicy**
   - Defines the default traffic policy for the destination.
   - Can be overridden by port-level settings.

### Example Explaination:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-service
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    connectionPool:
      http:
        http1MaxPendingRequests: 100
        maxRequestsPerConnection: 10
    outlierDetection:
      consecutiveErrors: 5
      interval: 5m
      baseEjectionTime: 15m
      maxEjectionPercent: 50
    tls:
      mode: SIMPLE
```

This example configures a `DestinationRule` with a round-robin load balancer, connection pool settings, outlier detection settings, and simple TLS mode for connections to `my-service`.

Below is a detailed explanation of each field in the provided `trafficPolicy` configuration from above example:

#### 1. **LoadBalancer**
   - **Simple: ROUND_ROBIN**
     - `ROUND_ROBIN`: Each healthy upstream host is selected in sequence, distributing incoming requests evenly across the group of servers.

#### 2. **ConnectionPool**
   - **Http:**
     - **Http1MaxPendingRequests: 100**
       - This field sets the maximum number of pending HTTP requests to a destination. If exceeded, new requests will be denied.
     - **MaxRequestsPerConnection: 10**
       - This field sets the maximum number of requests per connection to a backend. When this number is reached, the connection is closed.

#### 3. **OutlierDetection**
   - **ConsecutiveErrors: 5**
     - This field sets the number of consecutive 5xx errors before considering an upstream host as an outlier and ejecting it from the healthy backend pool.
   - **Interval: 5m**
     - This field specifies the time interval between ejection sweep analysis, meaning Istio checks whether any host should be ejected every 5 minutes.
   - **BaseEjectionTime: 15m**
     - This field sets the minimum amount of time a host is ejected. It is calculated as `baseEjectionTime * (2 ^ num_ejections)`.
   - **MaxEjectionPercent: 50**
     - This field sets the maximum percentage of hosts that can be ejected from the load balancing pool. Here, at most 50% of hosts can be ejected.

#### 4. **TLS**
   - **Mode: SIMPLE**
     - This field specifies the TLS mode used when communicating with the upstream service.
     - `SIMPLE`: TLS is used, but with no client certificate for mutual TLS.

For more detailed information and examples, you can refer to the [official Istio documentation](https://istio.io/latest/docs/reference/config/networking/destination-rule/#TrafficPolicy).