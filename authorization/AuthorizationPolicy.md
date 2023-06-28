```yaml

apiVersion: policy.linkerd.io/v1beta1
kind: Server
metadata:
  namespace: poc
  name: nginx1-policy
spec:
  podSelector:
    matchLabels:
      app: nginx1
  port: 80

---
apiVersion: policy.linkerd.io/v1alpha1
kind: AuthorizationPolicy
metadata:
  name: nginx1-policy
  namespace: poc
spec:
  targetRef:
    group: policy.linkerd.io
    kind: Server
    name: nginx1-policy
  requiredAuthenticationRefs:
    - name: default
      kind: ServiceAccount

---
apiVersion: policy.linkerd.io/v1beta2
kind: HTTPRoute
metadata:
  name: nginx1-httproute
  namespace: poc1
spec:
  parentRefs:
    - name: nginx1-policy
      kind: Server
      group: policy.linkerd.io
  rules:
    - matches:
      - path:
          value: "/authors.json"
        method: GET
      - path:
          value: "/authors/"
          type: "PathPrefix"
        method: GET

```

- https://linkerd.io/2.13/reference/authorization-policy/#server
- https://linkerd.io/2.13/reference/httproute/