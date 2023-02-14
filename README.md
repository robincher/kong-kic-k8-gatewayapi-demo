# Kong KIC Gateway Demonstration

[![][kong-logo]][kong-url]


Sample Recipe to start using [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/) feature in Kong Kubernetes Ingress Controller


### Create Kubernetes Cluster

For testing, let's just create a GKE.

```
gcloud container clusters create robincher-scratch-cluster --num-nodes=1 --machine-type=n1-standard-1 --region=asia-southeast1 --project={{project-id}}

# Initi Kubectl
gcloud container clusters get-credentials robincher-scratch-cluster \
    --region=asia-southeast1
```
 
### Install Kong KIC

```
kubectl apply -f https://raw.githubusercontent.com/Kong/kubernetes-ingress-controller/v2.8.0/deploy/single/all-in-one-dbless.yaml

```

### Install Gateway API Associated Resources

Since the K8 Gateway API is still in tech preview and not available in kubernetes distribution by deafult, you need to install the Gateway API associated resources and admission controller [manually](https://gateway-api.sigs.k8s.io/guides/#installing-gateway-api).

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v0.5.1/experimental-install.yaml
```

### Update Feature Flag for Gateway API in Kong KIC

Next we need to restart Kong to recognize those Gateway Resources. Upon restarting, we can enable the feature flag for Gateway API support in Kong KIC

```
# Restart KIC
kubectl rollout restart -n NAMESPACE deployment DEPLOYMENT_NAME

# Set Env Var to enable feature flag
kubectl set env -n default deployment/ingress-kong CONTROLLER_FEATURE_GATES="GatewayAlpha=true" -c ingress-controller
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

Next we deploy sample echo service and a HTTPRoute

```
# Deploy Sample Service
kubectl apply -f 02_Deployment.yaml

# Deploy HTTPROute
kbuectl apply -f 03_HTTPRoute.yaml
```

# Testing

```
curl -i http://kong.example/echo --resolve kong.example:80:$PROXY_IP

http $PROXY_IP/echo
```

[kong-url]: https://konghq.com/
[kong-logo]: https://konghq.com/wp-content/uploads/2018/05/kong-logo-github-readme.png