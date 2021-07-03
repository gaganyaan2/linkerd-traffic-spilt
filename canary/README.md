# linkerd-traffic-spilt
linkerd-traffic-spilt canary example

![alt text](https://github.com/koolwithk/linkerd-traffic-spilt/blob/main/canary/Linkerd-Canary.png?raw=true)

### Deploy apps and trafficsplit

```
kubectl create ns linkerd-test
kubectl annotate ns linkerd-test linkerd.io/inject=enabled

#install flagger
kubectl apply -k github.com/fluxcd/flagger/kustomize/linkerd

#deploy
kubectl apply -f appv1.yml -n linkerd-test
kubectl apply -f flagger.yml -n linkerd-test

```

## How to see the canary status?
```
#update the image name to see the status
kubectl -n linkerd-test set image deployment/appv1 appv1=httpd

#status
kubectl -n linkerd-test get ev --watch

watch kubectl -n linkerd-test get canary

```

## Note:

1. Test pod should be injected with linkerd-proxy sidecar
2. Linkerd viz and flagger should be installed
3. To expose this service outside of cluster we need to inject the linkerd-proxy sidecar into ingress controller

## Reference :

1. https://linkerd.io/2.10/tasks/canary-release/
2. https://docs.flagger.app/tutorials/linkerd-progressive-delivery
