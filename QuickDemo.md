# QUICK DEMO

## 1. NGINX Plus DEMO

### Deploy Controller
~~~
kubectl apply -f nginx-ingress-controller/nginx-plus-ingress_full-deployment.yml
~~~
### Deploy applications
~~~
kubectl create -f colors.yml
~~~
### Get resources deployed 
~~~
kubectl get all --namespace nginx-ingress
kubectl get all --namespace default
~~~

### Get Ports for quick access
~~~
kubectl get service/nginx-ingress -n nginx-ingress -o json|jq '[.spec.ports]'


export INGRESS_IP=$(curl -s ifconfig.co )


export INGRESS_HTTP_PORT=$(kubectl get service/nginx-ingress -n nginx-ingress -o json|jq '[[.spec.ports][][]|select(.name=="http").nodePort][0]')

~~~

### Create Ingress Controller
~~~
kubectl create -f nginx-ingress-controller/colors-nginx-plus-ingress-controller.yml

kubectl get ingress 
kubectl get ingress colors-nginx-ingress -o yaml
~~~

## Demos (Basic Routing, Active Health Checks, Simple Rewriting and Session Persistence)

---
## Basic Routing to Backends
~~~
http ${INGRESS_IP}:${INGRESS_HTTP_PORT}/text Host:red.example.com

http ${INGRESS_IP}:${INGRESS_HTTP_PORT}/text Host:blue.example.com
~~~
---

### Active Health Checks
~~~
kubectl get pods --selector app==blue -o name

kubectl exec -ti $(kubectl get pods --selector app==blue -o name|awk -F "/" 'END { print $2 }')   -- touch /tmp/down

http ${INGRESS_IP}:${INGRESS_HTTP_PORT}/health Host:blue.example.com

http ${INGRESS_IP}:${INGRESS_HTTP_PORT}/text Host:blue.example.com

kubectl exec -ti $(kubectl get pods --selector app==blue -o name|awk -F "/" 'END { print $2 }')   -- rm /tmp/down

http ${INGRESS_IP}:${INGRESS_HTTP_PORT}/health Host:blue.example.com
~~~
---
### Ingress Controller rewriting 
~~~
http ${INGRESS_IP}:${INGRESS_HTTP_PORT}/red/text Host:blue.example.com
~~~
---
### Ingress Controller Sticky Sessions Persistence
~~~
http --session red ${INGRESS_IP}:${INGRESS_HTTP_PORT}/text Host:red.example.com

http --session red ${INGRESS_IP}:${INGRESS_HTTP_PORT}/text Host:red.example.com
~~~
----

## 2. KONG DEMO

### We deploy complete Kong Environment with Postgresql.
~~~
kubectl create -f kong-ingress-controller/kong-ingress_full-deployment.yml
~~~

### We have created 'kong' namespace so we will wait until all Kong components are ready.
~~~
kubectl get all --namespace kong
~~~

### For quick access we prepare some envirnment variables (for Kong admin and Kong proxy access)
~~~
kubectl get service/kong-ingress-controller -n kong -o json|jq '[.spec.ports][0].nodePort]'

export KONG_ADMIN_PORT=$(kubectl get service/kong-ingress-controller -n kong -o json|jq '[.spec.ports][][].nodePort')

export KONG_ADMIN_IP=$(curl -s ifconfig.co)

kubectl get service/kong-proxy -n kong -o json|jq '[.spec.ports]'

export KONG_PROXY_IP=$(curl -s ifconfig.co)

export KONG_HTTP_PORT=$(kubectl get service/kong-proxy -n kong -o json|jq '[[.spec.ports][][]|select(.name=="kong-proxy").nodePort][0]')
~~~
### Now we deploy black-ingress ingress resource for accesing black application
~~~
kubectl apply -f kong-ingress-controller/black-kong-ingress-controller.yml
~~~

### We just set up host routing
~~~
kubectl get ingress black-kong-colors-ingress -o yaml
~~~

### And now we can access our applications setting host headers
~~~
http ${KONG_PROXY_IP}:${KONG_HTTP_PORT}/text Host:black.example.com
~~~

### We now add a KongPluggin resource for reate limitting to route
~~~
kubectl create -f kong-ingress-controller/kong-ratelimiting-plugin.yml 
~~~


### And now we apply rate limit to our deployed ingress resource
~~~
kubectl patch ingress black-kong-colors-ingress \
  -p '{"metadata":{"annotations":{"rate-limiting.plugin.konghq.com":"add-ratelimiting-to-route\n"}}}'
~~~

### And now we can access again our application
~~~
http ${KONG_PROXY_IP}:${KONG_HTTP_PORT}/text Host:black.example.com
~~~

---
> **NOTES:**
> ### We can review targets because black-app has 2 replicas
> ~~~
> http ${KONG_ADMIN_IP}:${KONG_ADMIN_PORT}/upstreams/default.black-svc.80/targets
> ~~~
> 
> ### Review now the plugins on Kong 
> ~~~
> http ${KONG_ADMIN_IP}:${KONG_ADMIN_PORT}/plugins
> ~~~
> 
> ### We can review the routes deployed
> ~~~
> http ${KONG_ADMIN_IP}:${KONG_ADMIN_PORT}/routes
> ~~~
---