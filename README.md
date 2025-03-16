# wms-gateway
WMS API gateway

## API Gateway Configuration

This project uses Apache APISIX as the API gateway to expose and manage APIs from various WMS microservices.

## Installation

This project is deployed as a Helm chart. For detailed installation instructions, see the [helm chart README](./helm/README.md).

The installation process uses a two-step approach:
1. First install the official APISIX Helm chart which provides CRDs
2. Then install the WMS Gateway Helm chart

```bash
# Step 1: Install APISIX with CRDs
helm upgrade --install apisix apisix/apisix \
  --set ingress-controller.enabled=true \
  --set etcd.enabled=true \
  --set ingress-controller.config.apisix.adminKey=YOUR_KEY

# Step 2: Install WMS Gateway
helm install wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-local.yaml
```

### Available Routes

#### Order Management Service

All order management endpoints are accessible via `/api/orders` and routed to the wms-order-management service:

- **Create Order**: POST `/api/orders`
- **Get All Orders**: GET `/api/orders`
- **Get Order by ID**: GET `/api/orders/{id}`
- **Update Order**: PUT `/api/orders/{id}`
- **Delete Order**: DELETE `/api/orders/{id}`
- **Allocate Inventory**: POST `/api/orders/{id}/allocate`
- **Update Order Status**: PUT `/api/orders/{id}/status`

#### Inventory Management Service

Inventory endpoints are grouped into three categories:

**Locations**
- **List Locations**: GET `/api/inventory/locations`
- **Get Location by ID**: GET `/api/inventory/locations/{id}`
- **Create Location**: POST `/api/inventory/locations`
- **Update Location**: PUT `/api/inventory/locations/{id}`

**Products**
- **List Products**: GET `/api/inventory/products`
- **Get Product by ID**: GET `/api/inventory/products/{id}`
- **Create Product**: POST `/api/inventory/products`

**Stock**
- **List Stock**: GET `/api/inventory/stock`
- **Add Stock (Inbound)**: POST `/api/inventory/stock/inbound`
- **Remove Stock (Outbound)**: POST `/api/inventory/stock/outbound`

### Rate Limiting

The Order Management API has a rate limit of 100 requests per second.

### Configuration

The APISIX configuration is located in the Helm chart values files. To modify the configuration:

1. Edit the appropriate values file:
   - `helm/values.yaml`: Common settings
   - `helm/values-local.yaml`: Local development settings
   - `helm/values-prod.yaml`: Production settings

2. Upgrade the helm release:
   ```bash
   helm upgrade wms-gateway ./wms-gateway/helm -f ./wms-gateway/helm/values-local.yaml
   ```

For more information about Apache APISIX configuration options, visit [APISIX Documentation](https://apisix.apache.org/docs/apisix/getting-started/).
