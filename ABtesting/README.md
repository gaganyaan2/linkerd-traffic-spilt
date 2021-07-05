# linkerd A/B testing
linkerd A/B testing example

### Deploy apps and trafficsplit

```
kubectl create ns linkerd-test
kubectl annotate ns linkerd-test linkerd.io/inject=enabled

#install flagger
kubectl apply -k github.com/fluxcd/flagger/kustomize/linkerd

#install nginx ingress
kubectl create ns ingress-nginx
helm upgrade -i nginx-ingress stable/nginx-ingress -n ingress-nginx --set controller.service.type=NodePort

#get ingress nodeport
kubectl --namespace ingress-nginx get services -o jsonpath="{.spec.ports[0].nodePort}" nginx-ingress-controller

#apply ingress for app.example.com
kubectly apply -f ingress.yml

#deploy
kubectl apply -f appv1.yml -n linkerd-test
kubectl apply -f ab-flagger.yml -n linkerd-test

```

## How to see the canary status?
```
#update the image name to see the status
kubectl -n linkerd-test set image deployment/appv1 appv1=httpd

#random load
while(true); do curl -HHost:app.example.com "http://192.168.0.184:30988/" ; sleep 0.5; done

#status
kubectl describe canary appv1 -n linkerd-test

#To canary new deployment add header
curl -H 'X-Canary: always'  -HHost:app.example.com "http://192.168.0.184:30988/"

```

## Note:

1. Test pod should be injected with linkerd-proxy sidecar
2. Linkerd viz, flagger and nginx ingress should be installed
3. To expose this service outside of cluster we need to inject the linkerd-proxy sidecar into ingress controller

## Reference :

1. https://linkerd.io/2.10/tasks/canary-release/
2. https://docs.flagger.app/tutorials/linkerd-progressive-delivery#linkerd-ingress
