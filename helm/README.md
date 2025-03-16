# WMS Gateway Helm Chart

This Helm chart deploys [Apache APISIX](https://apisix.apache.org/) as an API gateway for the WMS (Warehouse Management System) services.

## Features

- API Gateway for WMS services
- Rate limiting (configurable, default: 100 requests per second)
- Expose APIs from Order and Inventory services, excluding healthcheck endpoints
- HTTP method restrictions for each endpoint

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+
- The WMS services deployed with the `wms-main` Helm chart

## Installation

### Two-Step Installation Process

This chart depends on APISIX CRDs being available in your cluster. We recommend using the following two-step installation process to ensure everything works correctly:

#### Step 1: Install APISIX Chart (with CRDs)

First, install the official APISIX Helm chart which automatically installs all required CRDs:

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

Wait until the APISIX deployment is complete and healthy. You can verify that the CRDs are installed with:

```bash
kubectl get crds | grep apisix
```

You should see something like:
```
apisixroutes.apisix.apache.org
apisixupstreams.apisix.apache.org
apisixpluginconfigs.apisix.apache.org
apisixconsumers.apisix.apache.org
```

#### Step 2: Install WMS Gateway Chart

Once the APISIX chart and CRDs are successfully installed, you can deploy the WMS Gateway chart:

```bash
# For local development
helm install wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-local.yaml

# For production
helm install wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-prod.yaml
```

## Configuration

The configuration is split into multiple files for better organization:

- **values.yaml**: Common configuration shared between all environments
- **values-prod.yaml**: Production-specific overrides
- **values-local.yaml**: Local development overrides

### Rate Limiting

Rate limiting is configured globally with environment-specific defaults:
- Production: 100 requests per second
- Local development: 1000 requests per second

You can customize this by modifying the `apisix.config.apisix.rateLimit.requestsPerSecond` value in the appropriate values file.

### HTTP Method Restrictions

Each route can be configured to accept only specific HTTP methods. The templates automatically collect all unique methods defined for the paths within a route.

### Adding New Routes

To add new routes for services, modify the `wmsRoutes` section in the values.yaml file:

```yaml
wmsRoutes:
  newService:
    enabled: true
    paths:
      - path: "/api/new-endpoint"
        methods: ["GET", "POST", "PUT", "DELETE"]
    excludePaths:
      - "/health"
```

Then, add the environment-specific settings in values-prod.yaml and values-local.yaml:

```yaml
# In values-prod.yaml
wmsRoutes:
  newService:
    host: "new-api.wms.local"
    serviceName: "wms-main-new-service"

# In values-local.yaml
wmsRoutes:
  newService:
    host: "localhost"
    serviceName: "wms-main-new-service"
```

## Integration with WMS Main Chart

This chart is designed to work alongside the `wms-main` chart. Make sure both charts are deployed in the same namespace.

## Accessing the Gateway

After installation, the API Gateway will be accessible at:

- For local development: http://localhost:30080
- For production: Depends on your ingress configuration

## Troubleshooting

### Common Issues

1. **"no matches for kind "ApisixRoute" in version "apisix.apache.org/v2"**:  
   This error indicates that the APISIX CRDs are not installed. Follow Step 1 of the installation process above to install the APISIX chart with CRDs.

2. **Connection Issues**:  
   If you cannot connect to the API Gateway, check that the service is running and properly exposed:
   ```bash
   kubectl get svc -l app.kubernetes.io/name=apisix
   ```

3. **Route Definition Errors**:
   If you encounter route definition errors, verify that your route definitions follow the correct APISIX CRD format. 