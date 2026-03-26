---
name: openchoreo-networking
description: Use for OpenChoreo networking - Cilium service mesh, network policies, ingress configuration, and traffic management
---

# OpenChoreo Networking Skill

## When to Use This Skill

Use for:
- Configuring Cilium service mesh
- Creating network policies
- Setting up ingress and egress rules
- Load balancing and traffic management
- Service discovery and DNS

## Cilium Service Mesh

### Installation

```bash
# Install Cilium CLI
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-amd64.tar.gz{,.sha256sum}
sudo tar xzvfC cilium-linux-amd64.tar.gz /usr/local/bin

# Install Cilium
cilium install --version 1.15.0

# Verify
cilium status
```

### Network Policies

```yaml
# network-policy.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: customer-service-policy
  namespace: production
spec:
  endpointSelector:
    matchLabels:
      app: customer-service

  ingress:
    # Allow from API gateway
    - fromEndpoints:
        - matchLabels:
            app: api-gateway
      toPorts:
        - ports:
            - port: "8080"
              protocol: TCP

  egress:
    # Allow to database
    - toEndpoints:
        - matchLabels:
            app: postgres
      toPorts:
        - ports:
            - port: "5432"
              protocol: TCP

    # Allow DNS
    - toEndpoints:
        - matchLabels:
            k8s:io.kubernetes.pod.namespace: kube-system
            k8s-app: kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: UDP

    # Allow external HTTPS
    - toFQDNs:
        - matchPattern: "*.example.com"
      toPorts:
        - ports:
            - port: "443"
              protocol: TCP
```

## Ingress Configuration

```yaml
# envoy-gateway.yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: openchoreo-gateway
  namespace: openchoreo
spec:
  gatewayClassName: cilium
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
    - name: https
      protocol: HTTPS
      port: 443
      tls:
        mode: Terminate
        certificateRefs:
          - name: openchoreo-tls
---
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: customer-service-route
  namespace: production
spec:
  parentRefs:
    - name: openchoreo-gateway
      namespace: openchoreo
  hostnames:
    - api.openchoreo.local
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api/customers
      backendRefs:
        - name: customer-service
          port: 80
```

## Traffic Management

### Load Balancing

```yaml
# load-balancing.yaml
apiVersion: cilium.io/v2
kind: CiliumLoadBalancerIPPool
metadata:
  name: openchoreo-pool
spec:
  cidrs:
    - cidr: 192.168.1.0/24
---
apiVersion: v1
kind: Service
metadata:
  name: customer-service
spec:
  type: LoadBalancer
  loadBalancerIP: 192.168.1.10
  selector:
    app: customer-service
  ports:
    - port: 80
      targetPort: 8080
```

## Related Skills

- `openchoreo-devops`
- `openchoreo-security`
- `openchoreo-sre`

---

**Last Updated**: March 2025
