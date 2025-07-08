
# Secure Kubernetes Cluster with Istio and OPA Gatekeeper

This project sets up a secure Kubernetes cluster using Istio as a service mesh and Open Policy Agent (OPA) Gatekeeper for policy enforcement. It is built and tested on a local Kind cluster with Kubernetes v1.26.

## What This Project Does

- Secures pod-to-pod traffic with Istio mutual TLS (mTLS)
- Applies fine-grained AuthorizationPolicy for services
- Enforces security policies using OPA Gatekeeper
- Blocks pods that attempt to run as root

## Tech Stack

- Kind – Run Kubernetes locally in Docker
- Istio 1.26 – Service Mesh for traffic control and security
- OPA Gatekeeper – Policy controller for Kubernetes
- kubectl, istioctl, and YAML configuration files

## Project Structure

```
.
├── mtls-policy.yaml                # Istio PeerAuthentication policy
├── authz-policy.yaml              # Istio AuthorizationPolicy
├── disallowed-root-template.yaml  # OPA ConstraintTemplate
├── disallow-root-constraint.yaml  # OPA Constraint instance
└── README.md                      # This file
```

## How to Run

### 1. Create the Kind cluster

```bash
kind create cluster --name istio-opa-cluster --image kindest/node:v1.26.0
```

### 2. Install Istio (Demo profile)

```bash
istioctl install --set profile=demo -y
```

### 3. Enable sidecar injection

```bash
kubectl label namespace default istio-injection=enabled
```

### 4. Apply Istio policies

```bash
kubectl apply -f mtls-policy.yaml --validate=false
kubectl apply -f authz-policy.yaml --validate=false
```

### 5. Install OPA Gatekeeper

```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml --validate=false
```

Wait for pods to be ready:

```bash
kubectl get pods -n gatekeeper-system -w
```

### 6. Apply OPA templates and constraints

```bash
kubectl apply -f disallowed-root-template.yaml --validate=false
kubectl apply -f disallow-root-constraint.yaml --validate=false
```

## Example Policy Test

Try to run a root pod (should be blocked):

```bash
kubectl run root-pod --image=nginx --overrides='{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": { "name": "root-pod" },
  "spec": {
    "containers": [{
      "name": "nginx",
      "image": "nginx",
      "securityContext": {
        "runAsNonRoot": false
      }
    }]
  }
}' --dry-run=client -o yaml | kubectl apply -f -
```

Expected output:

```
Error from server ([denied by k8sdisallowedroot]): running as root is not allowed
```

## Tear Down

```bash
kind delete cluster --name istio-opa-cluster
```

## Notes

- Tested with Kubernetes v1.26, Istio 1.26.2, and Gatekeeper latest (via GitHub)
- Use --validate=false to avoid OpenAPI schema issues when applying CRDs

## Author

Made by Mangalam 
Email: shrivastavam017@gmail.com
