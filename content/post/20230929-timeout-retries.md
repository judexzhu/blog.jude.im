---
title: "Resiliency: Timeout and Retries"
date: 2023-09-29
draft: false
categories: ['istio']
tags: ['istio']
---

In Istio, resiliency features like timeouts and retry policies are configured using the `VirtualService` resources.

### 1. **Timeouts:**
   - **Purpose:** Timeouts in Istio define how long the Envoy proxy should wait for a response from the destination service.
   - **Configuration:** Specified in the `VirtualService` resource.
   - **Behavior:** If the destination service does not respond within the specified timeout duration, the Envoy proxy will terminate the request and return an HTTP 504 Gateway Timeout error to the client.

#### Example of Timeout Configuration:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: example-service
spec:
  hosts:
    - example-service
  http:
    - route:
        - destination:
            host: example-service
            subset: v1
      timeout: 5s  # 5 seconds timeout for requests
```

In this example, any request to the `v1` subset of `example-service` that takes longer than 5 seconds to receive a response will be terminated by the Envoy proxy, and the client will receive an HTTP 504 Gateway Timeout error.

### 2. **Retries:**
   - **Purpose:** Retries in Istio define how the Envoy proxy should handle failed requests to the destination service.
   - **Configuration:** Specified in the `VirtualService` resource.
   - **Behavior:** If a request fails due to conditions specified in the retry policy, the Envoy proxy will retry the request based on the number of attempts and conditions defined in the policy.

#### Example of Retry Configuration:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: example-service
spec:
  hosts:
    - example-service
  http:
    - route:
        - destination:
            host: example-service
            subset: v1
      retries:
        attempts: 3  # Number of retry attempts
        perTryTimeout: 2s  # Timeout per retry attempt
        retryOn: 5xx,connect-failure  # Conditions to retry on
```

In this example:
   - If a request to the `v1` subset of `example-service` fails due to a 5xx error or a connection failure, Envoy will retry the request.
   - Each retry attempt has a `perTryTimeout` of 2 seconds.
   - Envoy will make up to 3 retry attempts.

### Detailed Behavior:
   - **Timeouts:** The timeout duration includes all retry attempts. So, if the timeout is 5s and the per-try timeout is 2s, you may not get 3 retry attempts as the overall timeout will be reached after the second attempt.
   - **Retries:** The delay between retries is subject to a backoff strategy, typically involving exponential backoff with jitter, to avoid overwhelming the upstream service.
   - **Combination:** When using both timeouts and retries, careful consideration is needed to avoid unintended interactions between the overall timeout and the per-try timeout.

### Conclusion:
Timeouts and retries are critical resiliency features in Istio, allowing you to control the behavior of service communication in the face of slow or failing services. Properly configuring these settings in `VirtualService` resources can help in maintaining service availability and providing a better user experience.