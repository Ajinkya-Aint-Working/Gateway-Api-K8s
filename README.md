
# Multi-Namespace Gateway API Setup (GKE Autopilot)

This repository contains a complete example of using **Kubernetes Gateway API** on **GKE Autopilot** to route _multiple domains_ to _multiple services_ across _different namespaces_, while sharing a **single global static IP**.

The structure uses:

- **One shared Gateway**
- **Multiple HTTPRoutes across namespaces**
- **Multiple Services in isolated namespaces**
- **ReferenceGrant** for secure cross-namespace routing

## Folder Structure

```
Gateway-Api-K8s/
├── gateway.yaml
├── routes/
│   ├── project-a.yaml
│   └── project-b.yaml
└── ReferenceGrants/
    ├── Ref-svc-A.yaml
    └── Ref-svc-B.yaml
```

## How the Architecture Works

### ✔ One Gateway → Many Domains
A single external Gateway receives all traffic using one static IP.

### ✔ Each Project Has Its Own HTTPRoute
Each HTTPRoute defines:

- The hostname (domain)
- The service backend
- The routing rules
- The namespace isolation

### ✔ Services Live in Their Own Namespaces
Example:

- `project-a` contains `service-a`
- `project-b` contains `service-b`

### ✔ ReferenceGrant Allows Secure Cross-Namespace Routing
Gateway lives in a different namespace (`gateway-system`),
so Services must **explicitly allow Routes** to reference them.

Without a ReferenceGrant, Kubernetes **blocks the routing**.

## Component Breakdown

### 1️⃣ Gateway (`gateway.yaml`)
- Holds the shared external IP
- Terminates TLS
- Accepts HTTPRoutes from any namespace

### 2️⃣ HTTPRoutes (`routes/*.yaml`)
Each HTTPRoute:

- Lives in its own project namespace
- Defines routing for its domain
- Attaches to the shared Gateway
- Forwards traffic to a service

### 3️⃣ ReferenceGrants (`ReferenceGrants/*.yaml`)
Required because:

- Gateway and Routes are in different namespaces
- Routes reference Services
- Kubernetes blocks this by default for safety

## Request Flow

```
User → DomainA.com → Shared Gateway (gateway-system)
       ↓ hostname match
HTTPRoute in project-a
       ↓ backend reference
Service-a in namespace project-a
```

For project-b:

```
User → DomainB.com → Shared Gateway
       ↓
HTTPRoute in project-b
       ↓
Service-b in namespace project-b
```

## Deployment Steps

### 1. Create namespaces
```
kubectl create ns gateway-system
kubectl create ns project-a
kubectl create ns project-b
```

### 2. Apply the Gateway
```
kubectl apply -f gateway.yaml
```

### 3. Apply HTTPRoutes
```
kubectl apply -f routes/project-a.yaml
kubectl apply -f routes/project-b.yaml
```

### 4. Apply ReferenceGrants
```
kubectl apply -f ReferenceGrants/Ref-svc-A.yaml
kubectl apply -f ReferenceGrants/Ref-svc-B.yaml
```

### 5. Deploy services (`service-a`, `service-b`) into their namespaces

### 6. Point DNS
In Hostinger or any DNS provider:

```
domainA.com → A → <STATIC_IP>
domainB.com → A → <STATIC_IP>
```

## Validation

### Check Gateway status:
```
kubectl get gateway -n gateway-system
kubectl describe gateway shared-gateway -n gateway-system
```

### Check Route status:
```
kubectl get httproute -A
kubectl describe httproute -n project-a
kubectl describe httproute -n project-b
```

`Accepted: True` means routing is working.

## Final Result

- One static IP  
- Multiple isolated projects  
- Independent routing per domain  
- Secure separation using ReferenceGrants  
- Fully production-ready Gateway API design  

