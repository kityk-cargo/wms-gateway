# Getting Started with WMS Gateway

This guide provides step-by-step instructions for setting up and using the WMS Gateway API.

## Prerequisites

Before you begin, make sure you have the following installed:

- Kubernetes 1.16+ cluster
- Helm 3.0+
- kubectl CLI tool configured to communicate with your cluster
- WMS Main services deployed (if you want to route traffic to them)

## Installation

The installation process consists of two steps: first installing the APISIX base chart with CRDs, then installing the WMS Gateway chart.

### Step 1: Install APISIX with CRDs

First, add the APISIX Helm repository and install the official APISIX chart:

```bash
# Add the APISIX Helm repository
helm repo add apisix https://charts.apiseven.com
helm repo update

# Install APISIX with CRDs
helm upgrade --install apisix apisix/apisix \
  --set ingress-controller.enabled=true \
  --set etcd.enabled=true \
  --set ingress-controller.config.apisix.adminKey=YOUR_KEY
```

Verify that the CRDs have been installed:

```bash
kubectl get crds | grep apisix
```

You should see output listing the APISIX CRDs:
```
apisixroutes.apisix.apache.org
apisixupstreams.apisix.apache.org
apisixpluginconfigs.apisix.apache.org
apisixconsumers.apisix.apache.org
```

Verify that APISIX is running:

```bash
kubectl get pods -l app.kubernetes.io/instance=apisix
```

### Step 2: Install WMS Gateway

Once APISIX and its CRDs are successfully installed, you can proceed with installing the WMS Gateway:

For local development:
```bash
helm install wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-local.yaml
```

For production:
```bash
helm install wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-prod.yaml
```

Verify the installation:

```bash
kubectl get pods -l app.kubernetes.io/instance=wms-gateway
```

## Accessing the Gateway

### Local Development

In local development mode, the gateway is exposed using a NodePort service:

- API Gateway: http://localhost:30080

You can access the following sample endpoints:

- Order Service: http://localhost:30080/orders
- Inventory Service: http://localhost:30080/api/v1/products

### Production

In production mode, the gateway is deployed as a ClusterIP service. You may need to set up an Ingress or use port forwarding to access it.

## Customizing Configurations

### Modifying Rate Limits

You can modify the rate limiting by editing the appropriate values file and upgrading the Helm release:

```bash
# Edit values-local.yaml or values-prod.yaml
# Then upgrade the release
helm upgrade wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-local.yaml
```

### Adding New Routes

To add new routes, follow these steps:

1. Edit the `wmsRoutes` section in `values.yaml` to define the new service and its routes
2. Add environment-specific settings in `values-local.yaml` and `values-prod.yaml`
3. Upgrade the Helm release to apply the changes

## Troubleshooting

### Common Issues

1. **Routes not being created**:
   - Check that the APISIX CRDs are installed: `kubectl get crds | grep apisix`
   - Verify that the APISIX routes are created: `kubectl get apisixroutes`

2. **Connection issues**:
   - For local development, verify the NodePort service is running: `kubectl get svc -l app.kubernetes.io/name=apisix`
   - Check if the pods are healthy: `kubectl get pods -l app.kubernetes.io/instance=wms-gateway`

3. **Rate limiting issues**:
   - Verify the rate limit settings in the values file
   - Check the APISIX logs for any rate limiting events: `kubectl logs -l app.kubernetes.io/name=apisix`

For more detailed troubleshooting, refer to the [APISIX Documentation](https://apisix.apache.org/docs/apisix/getting-started/). 