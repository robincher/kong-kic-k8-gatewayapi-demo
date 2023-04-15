# Kong KIC Gateway Demonstration

[![][kong-logo]][kong-url]


Sample Recipe to start using [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) feature in Kong Kubernetes Ingress Controller


### Create Kubernetes Cluster

For testing, let's just create a GKE.

```
gcloud container clusters create robincher-scratch-cluster --num-nodes=1 --machine-type=n1-standard-1 --region=asia-southeast1 --project={{project-id}}

# Initi Kubectl
gcloud container clusters get-credentials robincher-scratch-cluster \
    --region=asia-southeast1 --project={{project-id}}
```
 
### Install Kong KIC

```
kubectl apply -f https://raw.githubusercontent.com/Kong/kubernetes-ingress-controller/v2.9.2/deploy/single/all-in-one-dbless.yaml

```

### Install Gateway API Associated Resources

Since the K8 Gateway API is still in tech preview and not available in kubernetes distribution by deafult, you need to install the Gateway API associated resources and admission controller [manually](https://gateway-api.sigs.k8s.io/guides/#installing-gateway-api).

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v0.6.2/experimental-install.yaml
```

### Update Feature Flag for Gateway API in Kong KIC

Next we need to restart Kong to recognize those Gateway Resources. Upon restarting, we can enable the feature flag for Gateway API support in Kong KIC

```
# Restart KIC
# kubectl rollout restart -n NAMESPACE deployment DEPLOYMENT_NAME
kubectl rollout restart -n kong deployment/ingress-kong

# Set Env Var to enable feature flag
kubectl set env -n kong deployment/ingress-kong CONTROLLER_FEATURE_GATES="GatewayAlpha=true" -c ingress-controller
```

### GatewayClass and Gateway

Next we need to bind the Gateway Class and Gateway to Kong Kubernetes Ingress Controller. **Kubernetes Ingress Controller recognizes the kong IngressClass and konghq.com/kic-gateway-controller GatewayClass by default**. Setting the CONTROLLER_INGRESS_CLASS or CONTROLLER_GATEWAY_API_CONTROLLER_NAME environment variable to another value overrides these defaults.

```
# Set up Gateway
kubectl apply -f 01_Gateway.yaml

# Check Gateway Status
kubectl get gateway kong

# Response, where Kong Gateway already bind to the KIC
NAME   CLASS   ADDRESS        READY   AGE
kong   kong    203.0.113.42   True    4m46s

```

### Deploy Test Service and HTTPRoute

Next we deploy sample echo and httpbin service and a HTTPRoute with traffic splitting

```
# Deploy Sample Services
kubectl apply -f 02a_Deployment.yaml
kubectl apply -f 02b_Deployment.yaml

# Deploy HTTPROute
kubectl apply -f 03_HTTPRoute.yaml
```

# Testing

```

export PROXY_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" service -n kong kong-proxy)

http $PROXY_IP/echo Host:kong.example person:robin
```

[kong-url]: https://konghq.com/
[kong-logo]: https://konghq.com/wp-content/uploads/2018/05/kong-logo-github-readme.png