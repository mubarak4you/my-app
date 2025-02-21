---
id: ingress
title: Enable Ingress Traffic with Istio APIs
---

#### Enable Ingress Traffic with Istio APIs

#### Purpose of the Ingress Gateway and Managed Service Mesh Control Plane

In Istio, the **Ingress Gateway** is a specialized Envoy proxy that manages inbound traffic to the service mesh. It allows you to define entry points into the mesh, control traffic flow, and apply routing rules before the traffic reaches your services. This setup enhances security, observability, and traffic management capabilities.

The **Managed Service Mesh Control Plane** oversees the configuration and management of the data plane components, including the ingress gateways and sidecar proxies. By decoupling control and data planes, Istio ensures efficient management of traffic policies and telemetry collection without impacting application performance.

#### Deploying a Web Application with Ingress Configuration

We will deploy the `httpbin` application, a simple HTTP request and response service, to demonstrate ingress traffic management.

#### Step 1: Deploy the `httpbin` Pod
This pod manifest specifies the pod deployment.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/istio-ingress-gateway/pod.yaml
```

#### Step 2: Deploy the `httpbin` Service
This service manifest specifies the `httpbin` service for the pod.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/istio-ingress-gateway/service.yaml
```

#### Step 3: Deploy the `httpbin` Virtual Service
This service manifest specifies a `VirtualService` to route traffic from the gateway to the `httpbin` service.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/istio-ingress-gateway/virtual-service.yaml
```

#### Step 4: Requirements for Kubernetes TLS Secrets
Before configuring the TLS-enabled ingress gateway, ensure you have a valid TLS certificate and private key. You will need to create a Kubernetes secret to store these credentials.

If you plan to serve HTTPS traffic through the ingress gateway, you'll need to configure TLS. Generate a TLS certificate and private key for your domain, then create a Kubernetes secret to store these credentials:

#### TLS Secret
This secret manifest deploys a secret which includes a certificate crt an key along with the necessary secret annotation to have the secret reflected reflected in the asm-ingressgateway namespace.

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/istio-ingress-gateway/tls-secret.yaml
```

Reference the secret in your `Gateway` configuration:

```yaml
tls:
  mode: SIMPLE
  credentialName: httpbin-tls-secret
```

#### Step 5: Configuring TLS for Ingress Gateway
Define an Istio `Gateway` resource to handle HTTPS traffic using a TLS secret:

```bash
kubectl apply -f https://gitlab.verizon.com/google-containers/gke-sample-applications/-/raw/main/istio-ingress-gateway/gateway.yaml
```

#### Step 6: Configuring DNS for the Ingress Gateway
To associate a domain name with your ingress gateway's external IP, create a [DNS] `A` record pointing your desired domain to the ingress gateway's IP address, which is the external IP of the clusters ASM ingress gateway service. 

[DNS]: https://dns.verizon.com/req.php
