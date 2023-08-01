```yaml

apiVersion: policy.linkerd.io/v1beta1
kind: Server
metadata:
  name: nginx1-policy
  namespace: poc  
spec:
  podSelector:
    matchLabels:
      app: nginx1
  port: 80

---
### if pod does not have correct service account it will block the request.
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

```bash
[root@lp-k8control-1 linkerd]# k get pod
NAME                      READY   STATUS    RESTARTS   AGE
nginx1-7ffc6db6f-b9sdr    2/2     Running   0          33d
nginx2-76d55c7684-w44x5   2/2     Running   0          33d
nginx3-68b4897cf6-8zfqd   1/1     Running   0          33d
[root@lp-k8control-1 linkerd]# k exec -it ^C
[root@lp-k8control-1 linkerd]# k get pod -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
nginx1-7ffc6db6f-b9sdr    2/2     Running   0          33d   10.10.42.224   lp-knode-1.home   <none>           <none>
nginx2-76d55c7684-w44x5   2/2     Running   0          33d   10.10.42.251   lp-knode-1.home   <none>           <none>
nginx3-68b4897cf6-8zfqd   1/1     Running   0          33d   10.10.42.195   lp-knode-1.home   <none>           <none>

root@nginx2-76d55c7684-w44x5:/# curl 10.10.42.224/authors.json
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.25.1</center>
</body>
</html>

root@nginx2-76d55c7684-w44x5:/# curl 10.10.42.224/authors.jso

root@nginx2-76d55c7684-w44x5:/# curl 10.10.42.224/authors
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.25.1</center>
</body>
</html>


root@nginx3-68b4897cf6-8zfqd:/# curl 10.10.42.224/authors.jso
root@nginx3-68b4897cf6-8zfqd:/#
root@nginx3-68b4897cf6-8zfqd:/#
root@nginx3-68b4897cf6-8zfqd:/# curl 10.10.42.224/authors
root@nginx3-68b4897cf6-8zfqd:/# curl 10.10.42.224/authors/
root@nginx3-68b4897cf6-8zfqd:/# curl 10.10.42.224/authors.json
root@nginx3-68b4897cf6-8zfqd:/#
root@nginx3-68b4897cf6-8zfqd:/#
root@nginx3-68b4897cf6-8zfqd:/# curl 10.10.42.224/authors.json

### I noticed the first curl request goes thruogh.

```

- https://linkerd.io/2.13/reference/authorization-policy/#server
- https://linkerd.io/2.13/reference/httproute/