# Kubernetes Custom Resource Definition (CRD) Guide

A comprehensive, production-ready example of building a Kubernetes Custom Resource Definition (CRD) and controller from scratch.

## Table of Contents

- [Introduction](#introduction)
- [What is a CRD?](#what-is-a-crd)
- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Detailed Components](#detailed-components)
  - [CRD Definition](#crd-definition)
  - [Controller Implementation](#controller-implementation)
  - [RBAC Configuration](#rbac-configuration)
  - [Deployment](#deployment)
- [Usage Examples](#usage-examples)
- [Testing](#testing)
- [Monitoring & Debugging](#monitoring--debugging)
- [Production Considerations](#production-considerations)
- [Troubleshooting](#troubleshooting)
- [References](#references)

---

## Introduction

This project demonstrates how to build a production-grade Kubernetes controller using Custom Resource Definitions (CRDs). We'll create a "Website" resource that automatically manages associated ConfigMaps, showcasing the core patterns used by real-world Kubernetes operators.

### What You'll Learn

- How to define and install Custom Resource Definitions
- The Kubernetes controller pattern (watch, compare, reconcile)
- Production-ready error handling and status management
- RBAC configuration and security best practices
- Deploying controllers as Kubernetes Deployments
- Finalizers and owner references
- Events and observability

---

## What is a CRD?

A **Custom Resource Definition (CRD)** extends Kubernetes by allowing you to define your own custom resource types that work just like built-in resources (Pods, Deployments, Services, etc.).

### Key Concepts

- **Custom Resource (CR)**: An instance of your custom resource type
- **Controller**: A program that watches CRs and takes action to maintain desired state
- **Operator**: A controller that encodes operational knowledge (deployment, scaling, backup, etc.)

### Why Use CRDs?

- **Abstraction**: Create higher-level APIs tailored to your use case
- **Declarative**: Manage complex applications declaratively with `kubectl`
- **Kubernetes-native**: Leverage existing tooling, RBAC, and ecosystem
- **Automation**: Encode operational knowledge in code

### Example: The Website Resource

Instead of managing Pods and Services directly, you could create a `Website` resource:

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: my-blog
spec:
  url: "https://blog.example.com"
  owner: "platform-team"
  status: "active"
```

A controller watches for these resources and automatically creates/manages the underlying infrastructure.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│                                                              │
│  ┌────────────────┐         ┌──────────────────┐           │
│  │  Website CRD   │         │  Website CR      │           │
│  │  (Definition)  │────────▶│  (Instance)      │           │
│  └────────────────┘         └──────────────────┘           │
│                                      │                       │
│                                      │ watches               │
│                                      ▼                       │
│                          ┌─────────────────────┐            │
│                          │ Website Controller  │            │
│                          │   (Deployment)      │            │
│                          └─────────────────────┘            │
│                                      │                       │
│                                      │ creates/manages       │
│                                      ▼                       │
│                          ┌─────────────────────┐            │
│                          │    ConfigMap        │            │
│                          │  (Managed Resource) │            │
│                          └─────────────────────┘            │
└─────────────────────────────────────────────────────────────┘
```

### Controller Pattern

1. **Watch**: Monitor Website resources for changes (ADDED, MODIFIED, DELETED)
2. **Compare**: Check desired state (Website spec) vs actual state (ConfigMap exists?)
3. **Reconcile**: Take action to make actual state match desired state

---

## Prerequisites

### Required Tools

- Kubernetes cluster (1.19+)
  - Local: minikube, kind, Docker Desktop
  - Cloud: GKE, EKS, AKS
- `kubectl` CLI
- Docker (for building controller image)
- Python 3.11+ (for development)

### Python Dependencies

```bash
pip install kubernetes==29.0.0
```

### Verify Cluster Access

```bash
kubectl cluster-info
kubectl get nodes
```

---

## Quick Start

### 1. Install the CRD

```bash
kubectl apply -f website-crd-with-status.yaml
```

Verify:
```bash
kubectl get crd websites.example.com
```

### 2. Setup RBAC

```bash
kubectl apply -f rbac.yaml
```

### 3. Build and Deploy Controller

```bash
# Build Docker image
docker build -t website-controller:latest .

# For minikube
eval $(minikube docker-env)
docker build -t website-controller:latest .

# For kind
kind load docker-image website-controller:latest

# Deploy
kubectl apply -f deployment.yaml
```

### 4. Create a Website

```bash
kubectl apply -f test-website.yaml
```

### 5. Verify

```bash
# Check Website
kubectl get websites

# Check ConfigMap
kubectl get configmap production-site-config

# Check controller logs
kubectl logs -f deployment/website-controller
```

---

## Detailed Components

### CRD Definition

File: `website-crd-with-status.yaml`

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                url:
                  type: string
                  description: "The URL of the website"
                owner:
                  type: string
                  description: "Who owns this website"
                status:
                  type: string
                  enum: ["active", "maintenance", "offline"]
                  description: "Current operational status"
              required:
                - url
                - owner
            status:
              type: object
              properties:
                state:
                  type: string
                  description: "Current state of reconciliation"
                configMapName:
                  type: string
                  description: "Name of the created ConfigMap"
                message:
                  type: string
                  description: "Human readable message"
                lastUpdated:
                  type: string
                  format: date-time
      # Enable status subresource
      subresources:
        status: {}
      # Add printer columns for kubectl get
      additionalPrinterColumns:
      - name: URL
        type: string
        jsonPath: .spec.url
      - name: Owner
        type: string
        jsonPath: .spec.owner
      - name: Status
        type: string
        jsonPath: .spec.status
      - name: State
        type: string
        jsonPath: .status.state
      - name: Age
        type: date
        jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: websites
    singular: website
    kind: Website
    shortNames:
    - ws
```

#### Key Fields Explained

| Field | Description |
|-------|-------------|
| `group` | API group (like a namespace for your API) |
| `version` | API version (v1, v1alpha1, v1beta1, etc.) |
| `scope` | `Namespaced` or `Cluster` |
| `names.plural` | Used in API URLs and `kubectl get websites` |
| `names.singular` | Used in `kubectl describe website` |
| `names.kind` | Used in YAML manifests |
| `names.shortNames` | Shortcuts for kubectl (e.g., `kubectl get ws`) |
| `schema` | OpenAPI v3 schema defining resource structure |
| `subresources.status` | Enables separate status updates |
| `additionalPrinterColumns` | Custom columns in `kubectl get` output |

---

### Controller Implementation

File: `website-controller.py`

```python
from kubernetes import client, config, watch
from kubernetes.client.rest import ApiException
import time
import logging
import sys
import os
from datetime import datetime

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[logging.StreamHandler(sys.stdout)]
)
logger = logging.getLogger(__name__)

class WebsiteController:
    def __init__(self):
        """Initialize the controller with proper config"""
        try:
            # Try in-cluster config first (production)
            config.load_incluster_config()
            logger.info("Loaded in-cluster config")
        except config.ConfigException:
            # Fall back to kubeconfig (development)
            try:
                config.load_kube_config()
                logger.info("Loaded kubeconfig from ~/.kube/config")
            except config.ConfigException:
                logger.error("Could not load kubernetes config")
                sys.exit(1)
        
        self.custom_api = client.CustomObjectsApi()
        self.core_api = client.CoreV1Api()
        
        self.group = "example.com"
        self.version = "v1"
        self.plural = "websites"
        
        # Finalizer to ensure cleanup
        self.finalizer = "websites.example.com/finalizer"
        
        logger.info("Controller initialized successfully")
    
    def add_finalizer(self, name, namespace):
        """Add finalizer to prevent deletion until cleanup is done"""
        try:
            website = self.custom_api.get_namespaced_custom_object(
                group=self.group,
                version=self.version,
                namespace=namespace,
                plural=self.plural,
                name=name
            )
            
            finalizers = website.get('metadata', {}).get('finalizers', [])
            if self.finalizer not in finalizers:
                finalizers.append(self.finalizer)
                
                # Patch to add finalizer
                self.custom_api.patch_namespaced_custom_object(
                    group=self.group,
                    version=self.version,
                    namespace=namespace,
                    plural=self.plural,
                    name=name,
                    body={"metadata": {"finalizers": finalizers}}
                )
                logger.info(f"Added finalizer to Website {name}")
        except ApiException as e:
            logger.error(f"Failed to add finalizer: {e}")
    
    def remove_finalizer(self, name, namespace):
        """Remove finalizer after cleanup"""
        try:
            website = self.custom_api.get_namespaced_custom_object(
                group=self.group,
                version=self.version,
                namespace=namespace,
                plural=self.plural,
                name=name
            )
            
            finalizers = website.get('metadata', {}).get('finalizers', [])
            if self.finalizer in finalizers:
                finalizers.remove(self.finalizer)
                
                self.custom_api.patch_namespaced_custom_object(
                    group=self.group,
                    version=self.version,
                    namespace=namespace,
                    plural=self.plural,
                    name=name,
                    body={"metadata": {"finalizers": finalizers}}
                )
                logger.info(f"Removed finalizer from Website {name}")
        except ApiException as e:
            logger.error(f"Failed to remove finalizer: {e}")
    
    def update_status(self, name, namespace, status_data):
        """Update the status subresource of the Website"""
        try:
            self.custom_api.patch_namespaced_custom_object_status(
                group=self.group,
                version=self.version,
                namespace=namespace,
                plural=self.plural,
                name=name,
                body={"status": status_data}
            )
            logger.info(f"Updated status for Website {name}: {status_data}")
        except ApiException as e:
            logger.error(f"Failed to update status: {e}")
    
    def create_event(self, name, namespace, reason, message, event_type="Normal"):
        """Create a Kubernetes Event for debugging"""
        try:
            now = datetime.utcnow().isoformat() + "Z"
            event = client.CoreV1Event(
                metadata=client.V1ObjectMeta(
                    name=f"{name}.{int(time.time())}",
                    namespace=namespace
                ),
                involved_object=client.V1ObjectReference(
                    api_version=f"{self.group}/{self.version}",
                    kind="Website",
                    name=name,
                    namespace=namespace
                ),
                reason=reason,
                message=message,
                type=event_type,
                first_timestamp=now,
                last_timestamp=now,
                count=1
            )
            self.core_api.create_namespaced_event(namespace=namespace, body=event)
        except ApiException as e:
            logger.warning(f"Failed to create event: {e}")
    
    def reconcile_website(self, website):
        """Main reconciliation logic"""
        name = website['metadata']['name']
        namespace = website['metadata'].get('namespace', 'default')
        spec = website['spec']
        
        # Check if being deleted
        deletion_timestamp = website['metadata'].get('deletionTimestamp')
        if deletion_timestamp:
            logger.info(f"Website {name} is being deleted, cleaning up...")
            self.delete_configmap_for_website(website)
            self.remove_finalizer(name, namespace)
            return
        
        # Add finalizer if not present
        finalizers = website.get('metadata', {}).get('finalizers', [])
        if self.finalizer not in finalizers:
            self.add_finalizer(name, namespace)
        
        # Create/Update ConfigMap
        try:
            configmap = client.V1ConfigMap(
                metadata=client.V1ObjectMeta(
                    name=f"{name}-config",
                    namespace=namespace,
                    labels={
                        "managed-by": "website-controller",
                        "website": name
                    },
                    owner_references=[
                        client.V1OwnerReference(
                            api_version=f"{self.group}/{self.version}",
                            kind="Website",
                            name=name,
                            uid=website['metadata']['uid'],
                            controller=True,
                            block_owner_deletion=True
                        )
                    ]
                ),
                data={
                    "url": spec.get('url', ''),
                    "owner": spec.get('owner', ''),
                    "status": spec.get('status', ''),
                    "website-name": name
                }
            )
            
            try:
                self.core_api.read_namespaced_config_map(
                    name=f"{name}-config",
                    namespace=namespace
                )
                # Update existing
                self.core_api.replace_namespaced_config_map(
                    name=f"{name}-config",
                    namespace=namespace,
                    body=configmap
                )
                logger.info(f"Updated ConfigMap for Website {name}")
                self.create_event(name, namespace, "ConfigMapUpdated", 
                                f"ConfigMap {name}-config updated successfully")
            except ApiException as e:
                if e.status == 404:
                    # Create new
                    self.core_api.create_namespaced_config_map(
                        namespace=namespace,
                        body=configmap
                    )
                    logger.info(f"Created ConfigMap for Website {name}")
                    self.create_event(name, namespace, "ConfigMapCreated", 
                                    f"ConfigMap {name}-config created successfully")
                else:
                    raise
            
            # Update status
            self.update_status(name, namespace, {
                "state": "Ready",
                "configMapName": f"{name}-config",
                "lastUpdated": datetime.utcnow().isoformat() + "Z"
            })
            
        except Exception as e:
            logger.error(f"Error reconciling Website {name}: {e}")
            self.update_status(name, namespace, {
                "state": "Error",
                "message": str(e),
                "lastUpdated": datetime.utcnow().isoformat() + "Z"
            })
            self.create_event(name, namespace, "ReconciliationFailed", 
                            f"Failed to reconcile: {str(e)}", "Warning")
    
    def delete_configmap_for_website(self, website):
        """Delete ConfigMap (called during cleanup)"""
        name = website['metadata']['name']
        namespace = website['metadata'].get('namespace', 'default')
        
        try:
            self.core_api.delete_namespaced_config_map(
                name=f"{name}-config",
                namespace=namespace
            )
            logger.info(f"Deleted ConfigMap {name}-config")
            self.create_event(name, namespace, "ConfigMapDeleted", 
                            f"ConfigMap {name}-config deleted")
        except ApiException as e:
            if e.status == 404:
                logger.info(f"ConfigMap {name}-config already deleted")
            else:
                logger.error(f"Error deleting ConfigMap: {e}")
    
    def run(self):
        """Main control loop with error handling"""
        logger.info("Starting Website Controller...")
        logger.info(f"Watching {self.group}/{self.version}/{self.plural}")
        
        # Track resource version for efficient watching
        resource_version = None
        
        while True:
            try:
                w = watch.Watch()
                
                # List all existing resources on startup
                if resource_version is None:
                    logger.info("Performing initial sync of all Website resources")
                    resources = self.custom_api.list_cluster_custom_object(
                        group=self.group,
                        version=self.version,
                        plural=self.plural
                    )
                    resource_version = resources['metadata']['resourceVersion']
                    
                    # Reconcile all existing resources
                    for item in resources.get('items', []):
                        self.reconcile_website(item)
                
                # Watch for changes
                for event in w.stream(
                    self.custom_api.list_cluster_custom_object,
                    group=self.group,
                    version=self.version,
                    plural=self.plural,
                    resource_version=resource_version,
                    timeout_seconds=300
                ):
                    event_type = event['type']
                    website = event['object']
                    name = website['metadata']['name']
                    
                    # Update resource version
                    resource_version = website['metadata']['resourceVersion']
                    
                    logger.info(f"Event: {event_type} for Website {name}")
                    
                    if event_type in ['ADDED', 'MODIFIED']:
                        self.reconcile_website(website)
                    elif event_type == 'DELETED':
                        logger.info(f"Website {name} deleted")
                    
            except ApiException as e:
                if e.status == 410:
                    # Resource version too old, reset and restart
                    logger.warning("Resource version expired, restarting watch")
                    resource_version = None
                else:
                    logger.error(f"API Exception: {e}")
                    time.sleep(5)
            except Exception as e:
                logger.error(f"Unexpected error: {e}", exc_info=True)
                time.sleep(5)

if __name__ == "__main__":
    controller = WebsiteController()
    controller.run()
```

#### Controller Features

- ✅ **Finalizers**: Ensures cleanup happens before deletion
- ✅ **Status Updates**: Reports current state back to the resource
- ✅ **Events**: Creates Kubernetes events for debugging
- ✅ **Owner References**: Automatic garbage collection
- ✅ **Error Handling**: Robust retry logic
- ✅ **Initial Sync**: Reconciles existing resources on startup
- ✅ **Resource Version Tracking**: Efficient watch mechanism

---

### RBAC Configuration

File: `rbac.yaml`

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: website-controller
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: website-controller-role
rules:
# Permissions for Website CRD
- apiGroups: ["example.com"]
  resources: ["websites"]
  verbs: ["get", "list", "watch", "update", "patch"]
- apiGroups: ["example.com"]
  resources: ["websites/status"]
  verbs: ["get", "update", "patch"]
- apiGroups: ["example.com"]
  resources: ["websites/finalizers"]
  verbs: ["update"]
# Permissions for ConfigMaps
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
# Permissions for Events
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: website-controller-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: website-controller-role
subjects:
- kind: ServiceAccount
  name: website-controller
  namespace: default
```

#### RBAC Components

- **ServiceAccount**: Identity for the controller pod
- **ClusterRole**: Defines permissions needed
- **ClusterRoleBinding**: Grants permissions to the ServiceAccount

---

### Deployment

#### Dockerfile

File: `Dockerfile`

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy controller code
COPY website-controller.py .

# Run as non-root user
RUN useradd -m -u 1000 controller && chown -R controller:controller /app
USER controller

# Run the controller
CMD ["python", "-u", "website-controller.py"]
```

#### Requirements

File: `requirements.txt`

```
kubernetes==29.0.0
```

#### Kubernetes Deployment

File: `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: website-controller
  namespace: default
  labels:
    app: website-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: website-controller
  template:
    metadata:
      labels:
        app: website-controller
    spec:
      serviceAccountName: website-controller
      containers:
      - name: controller
        image: website-controller:latest
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          exec:
            command:
            - python
            - -c
            - "import sys; sys.exit(0)"
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          exec:
            command:
            - python
            - -c
            - "import sys; sys.exit(0)"
          initialDelaySeconds: 5
          periodSeconds: 10
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
```

---

## Usage Examples

### Creating a Website

File: `test-website.yaml`

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: production-site
spec:
  url: "https://production.example.com"
  owner: "platform-team"
  status: "active"
```

Apply:
```bash
kubectl apply -f test-website.yaml
```

### Viewing Websites

```bash
# List all websites
kubectl get websites
kubectl get ws  # using short name

# Output:
# NAME              URL                              OWNER           STATUS        STATE   AGE
# production-site   https://production.example.com   platform-team   active        Ready   5m

# Detailed view
kubectl describe website production-site

# YAML format
kubectl get website production-site -o yaml
```

### Updating a Website

```bash
# Edit interactively
kubectl edit website production-site

# Patch specific field
kubectl patch website production-site --type='json' \
  -p='[{"op": "replace", "path": "/spec/status", "value": "maintenance"}]'

# Using apply with updated file
kubectl apply -f test-website.yaml
```

### Deleting a Website

```bash
kubectl delete website production-site

# The controller will:
# 1. Receive DELETE event
# 2. Clean up ConfigMap
# 3. Remove finalizer
# 4. Allow deletion to complete
```

### Creating Multiple Websites

File: `multiple-websites.yaml`

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: blog-site
spec:
  url: "https://blog.example.com"
  owner: "marketing-team"
  status: "active"
---
apiVersion: example.com/v1
kind: Website
metadata:
  name: docs-site
spec:
  url: "https://docs.example.com"
  owner: "technical-writers"
  status: "maintenance"
---
apiVersion: example.com/v1
kind: Website
metadata:
  name: api-site
spec:
  url: "https://api.example.com"
  owner: "backend-team"
  status: "active"
```

Apply:
```bash
kubectl apply -f multiple-websites.yaml
```

---

## Testing

### Complete Test Script

File: `test.sh`

```bash
#!/bin/bash

set -e

echo "=== Testing Website Controller ==="

# Create website
echo -e "\n1. Creating Website..."
cat <<EOF | kubectl apply -f -
apiVersion: example.com/v1
kind: Website
metadata:
  name: test-site
spec:
  url: "https://test.example.com"
  owner: "test-team"
  status: "active"
EOF

# Wait for reconciliation
echo -e "\n2. Waiting for reconciliation..."
sleep 3

# Check Website status
echo -e "\n3. Checking Website status..."
kubectl get website test-site -o yaml | grep -A 10 "status:"

# Check ConfigMap was created
echo -e "\n4. Checking ConfigMap..."
kubectl get configmap test-site-config -o yaml

# Check controller logs
echo -e "\n5. Recent controller logs:"
kubectl logs deployment/website-controller --tail=20

# Check events
echo -e "\n6. Events for Website:"
kubectl get events --field-selector involvedObject.name=test-site

# Update website
echo -e "\n7. Updating Website status to maintenance..."
kubectl patch website test-site --type='json' \
  -p='[{"op": "replace", "path": "/spec/status", "value": "maintenance"}]'

sleep 2

# Verify ConfigMap updated
echo -e "\n8. Verifying ConfigMap update..."
kubectl get configmap test-site-config -o jsonpath='{.data.status}'
echo ""

# Test invalid resource (should fail validation)
echo -e "\n9. Testing CRD validation (should fail)..."
cat <<EOF | kubectl apply -f - 2>&1 || echo "✓ Validation working correctly"
apiVersion: example.com/v1
kind: Website
metadata:
  name: invalid-site
spec:
  url: "https://invalid.example.com"
  # Missing required 'owner' field
EOF

# Delete website
echo -e "\n10. Deleting Website..."
kubectl delete website test-site

sleep 2

# Verify cleanup
echo -e "\n11. Verifying cleanup..."
kubectl get configmap test-site-config 2>&1 || echo "✓ ConfigMap cleaned up successfully"

echo -e "\n=== Test Complete ==="
```

Make executable and run:
```bash
chmod +x test.sh
./test.sh
```

### Manual Testing Steps

```bash
# 1. Create a website
kubectl apply -f test-website.yaml

# 2. Verify it was created
kubectl get websites

# 3. Check the status
kubectl get website production-site -o jsonpath='{.status}' | jq

# 4. Verify ConfigMap was created
kubectl get configmap production-site-config

# 5. Check ConfigMap contents
kubectl get configmap production-site-config -o yaml

# 6. View controller logs
kubectl logs -f deployment/website-controller

# 7. Update the website
kubectl patch website production-site --type='json' \
  -p='[{"op": "replace", "path": "/spec/owner", "value": "new-team"}]'

# 8. Verify ConfigMap was updated
kubectl get configmap production-site-config -o jsonpath='{.data.owner}'

# 9. Check events
kubectl get events --sort-by='.lastTimestamp' | grep production-site

# 10. Delete the website
kubectl delete website production-site

# 11. Verify ConfigMap was deleted
kubectl get configmap production-site-config  # should return NotFound
```

---

## Monitoring & Debugging

### Controller Logs

```bash
# Follow logs in real-time
kubectl logs -f deployment/website-controller

# Last 100 lines
kubectl logs deployment/website-controller --tail=100

# Logs with timestamps
kubectl logs deployment/website-controller --timestamps

# Logs from previous crash (if pod restarted)
kubectl logs deployment/website-controller --previous

# Logs from specific pod
kubectl logs <pod-name>
```

### Check Controller Health

```bash
# Pod status
kubectl get pods -l app=website-controller

# Detailed pod info
kubectl describe pod -l app=website-controller

# Events related to controller pod
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### Check Resources

```bash
# List all websites
kubectl get websites --all-namespaces

# Watch for changes
kubectl get websites --watch

# Describe specific website
kubectl describe website production-site

# Get status
kubectl get website production-site -o jsonpath='{.status}' | jq
```

### Events

```bash
# All events sorted by time
kubectl get events --sort-by='.lastTimestamp'

# Events for specific website
kubectl get events --field-selector involvedObject.name=production-site

# Events in last hour
kubectl get events --field-selector type!=Normal
```

### RBAC Debugging

```bash
# Check if controller can list websites
kubectl auth can-i list websites \
  --as=system:serviceaccount:default:website-controller

# Check all permissions
kubectl auth can-i --list \
  --as=system:serviceaccount:default:website-controller
```

### Common Debugging Commands

```bash
# Check CRD is installed
kubectl get crd websites.example.com

# Check CRD details
kubectl describe crd websites.example.com

# Check ServiceAccount exists
kubectl get sa website-controller

# Check ClusterRole
kubectl describe clusterrole website-controller-role

# Check ClusterRoleBinding
kubectl describe clusterrolebinding website-controller-binding

# Check controller deployment
kubectl get deployment website-controller

# Restart controller
kubectl rollout restart deployment/website-controller
```

---

## Production Considerations

### High Availability

For production, you'll want multiple controller replicas with leader election:

```python
# Add to controller
from kubernetes.leaderelection import LeaderElection, LeaderElectionConfig

def run_with_leader_election(self):
    """Run controller with leader election"""
    config = LeaderElectionConfig(
        lock_name="website-controller-lock",
        lock_namespace="default",
        identity=os.environ.get("HOSTNAME", "unknown"),
        lease_duration=15,
        renew_deadline=10,
        retry_period=2
    )
    
    election = LeaderElection(config, self.custom_api)
    election.run(
        on_started_leading=self.run,
        on_stopped_leading=self.on_stopped_leading
    )

def on_stopped_leading(self):
    """Called when this instance loses leadership"""
    logger.warning("Lost leadership, exiting...")
    sys.exit(0)
```

Update deployment:
```yaml
spec:
  replicas: 3  # Multiple replicas for HA
```

### Metrics & Monitoring

Add Prometheus metrics:

```python
from prometheus_client import Counter, Gauge, Histogram, start_http_server

# Define metrics
reconcile_total = Counter(
    'website_reconciliations_total',
    'Total number of reconciliations',
    ['result']  # success/error
)

reconcile_duration = Histogram(
    'website_reconciliation_duration_seconds',
    'Time spent reconciling'
)

websites_total = Gauge(
    'websites_total',
    'Total number of Website resources'
)

configmaps_managed = Gauge(
    'website_configmaps_managed_total',
    'Total number of managed ConfigMaps'
)

# Start metrics server
start_http_server(8080)

# Use in code
@reconcile_duration.time()
def reconcile_website(self, website):
    try:
        # ... reconciliation logic ...
        reconcile_total.labels(result='success').inc()
    except Exception as e:
        reconcile_total.labels(result='error').inc()
        raise
```

Add to deployment:
```yaml
spec:
  template:
    spec:
      containers:
      - name: controller
        ports:
        - name: metrics
          containerPort: 8080
```

Create ServiceMonitor for Prometheus:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: website-controller
spec:
  selector:
    matchLabels:
      app: website-controller
  endpoints:
  - port: metrics
    interval: 30s
```

### Health Checks

Add proper health endpoints:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import threading

class HealthHandler(BaseHTTPRequestHandler):
    def __init__(self, controller):
        self.controller = controller
        super().__init__()
    
    def do_GET(self):
        if self.path == '/healthz':
            # Liveness check
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'OK')
        elif self.path == '/readyz':
            # Readiness check
            if self.controller.is_ready():
                self.send_response(200)
                self.end_headers()
                self.wfile.write(b'Ready')
            else:
                self.send_response(503)
                self.end_headers()
                self.wfile.write(b'Not Ready')

# Start health server
def start_health_server(controller):
    server = HTTPServer(('', 8081), lambda *args: HealthHandler(controller, *args))
    threading.Thread(target=server.serve_forever, daemon=True).start()
```

Update deployment:
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8081
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /readyz
    port: 8081
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Resource Limits

Always set resource requests and limits:

```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

### Security Best Practices

1. **Run as non-root**:
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
```

2. **Read-only root filesystem**:
```yaml
securityContext:
  readOnlyRootFilesystem: true
```

3. **Network policies** to restrict traffic

4. **Pod Security Standards**: Use restricted policy

### Graceful Shutdown

```python
import signal

class WebsiteController:
    def __init__(self):
        # ... existing init ...
        self.shutdown = False
        signal.signal(signal.SIGTERM, self.handle_shutdown)
        signal.signal(signal.SIGINT, self.handle_shutdown)
    
    def handle_shutdown(self, signum, frame):
        logger.info("Received shutdown signal, cleaning up...")
        self.shutdown = True
    
    def run(self):
        while not self.shutdown:
            # ... watch loop ...
        logger.info("Controller shutdown complete")
```

### Structured Logging

Use structured logging for better observability:

```python
import json
import logging

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_obj = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
        }
        if record.exc_info:
            log_obj['exception'] = self.formatException(record.exc_info)
        return json.dumps(log_obj)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
```

### Versioning & Upgrades

1. **CRD Versioning**: Support multiple versions
```yaml
versions:
  - name: v1
    served: true
    storage: true
  - name: v1beta1
    served: true
    storage: false
```

2. **Conversion Webhooks**: Convert between versions

3. **Rolling Updates**: Use deployment strategies
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

---

## Troubleshooting

### Common Issues

#### 1. Controller Pod Not Starting

**Symptoms**: Pod in CrashLoopBackOff

**Debug**:
```bash
kubectl describe pod -l app=website-controller
kubectl logs deployment/website-controller
```

**Common Causes**:
- Missing RBAC permissions
- Invalid kubeconfig
- Python dependencies missing
- Image pull errors

**Fix**:
```bash
# Check RBAC
kubectl get sa website-controller
kubectl get clusterrole website-controller-role
kubectl get clusterrolebinding website-controller-binding

# Rebuild image
docker build -t website-controller:latest .

# For minikube
eval $(minikube docker-env)
docker build -t website-controller:latest .
```

#### 2. CRD Not Found

**Symptoms**: `Error from server (NotFound): the server could not find the requested resource`

**Debug**:
```bash
kubectl get crd | grep websites
```

**Fix**:
```bash
kubectl apply -f website-crd-with-status.yaml
kubectl get crd websites.example.com
```

#### 3. Controller Not Watching Resources

**Symptoms**: Resources created but no ConfigMaps appear

**Debug**:
```bash
kubectl logs deployment/website-controller
```

**Common Causes**:
- RBAC permissions missing
- Wrong group/version in code
- Network issues

**Fix**:
```bash
# Verify permissions
kubectl auth can-i list websites \
  --as=system:serviceaccount:default:website-controller

# Check controller logs for errors
kubectl logs deployment/website-controller --tail=50
```

#### 4. Finalizers Stuck

**Symptoms**: Resource stuck in "Terminating" state

**Debug**:
```bash
kubectl get website <name> -o yaml | grep finalizers -A 5
```

**Fix**:
```bash
# Remove finalizer manually (last resort)
kubectl patch website <name> -p '{"metadata":{"finalizers":[]}}' --type=merge
```

#### 5. Status Not Updating

**Symptoms**: `.status` field empty or not updating

**Debug**:
```bash
kubectl get website <name> -o yaml
```

**Common Causes**:
- Status subresource not enabled in CRD
- Missing RBAC for status updates

**Fix**:
```bash
# Ensure CRD has status subresource
kubectl get crd websites.example.com -o yaml | grep -A 2 subresources

# Check RBAC
kubectl auth can-i update websites/status \
  --as=system:serviceaccount:default:website-controller
```

#### 6. Events Not Created

**Symptoms**: No events visible for resources

**Debug**:
```bash
kubectl get events --sort-by='.lastTimestamp'
```

**Fix**:
```bash
# Check RBAC for events
kubectl auth can-i create events \
  --as=system:serviceaccount:default:website-controller

# Update RBAC if needed
kubectl apply -f rbac.yaml
```

### Validation Errors

```bash
# Test CRD validation
cat <<EOF | kubectl apply -f -
apiVersion: example.com/v1
kind: Website
metadata:
  name: test
spec:
  url: "https://test.com"
  # Missing required 'owner' field - should fail
EOF
```

Expected error:
```
Error from server (BadRequest): error when creating "STDIN": 
Website in version "v1" cannot be created: 
spec.owner: Required value
```

---

## References

### Official Documentation

- [Kubernetes Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
- [CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
- [Kubernetes Python Client](https://github.com/kubernetes-client/python)
- [Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)

### Related Projects

- [Operator SDK](https://sdk.operatorframework.io/) - Framework for building operators
- [Kubebuilder](https://book.kubebuilder.io/) - SDK for building Kubernetes APIs in Go
- [KUDO](https://kudo.dev/) - Kubernetes Universal Declarative Operator
- [Metacontroller](https://metacontroller.github.io/metacontroller/) - Lightweight controller framework

### Example Operators

- [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)
- [MySQL Operator](https://github.com/mysql/mysql-operator)
- [Cert Manager](https://github.com/cert-manager/cert-manager)
- [External Secrets Operator](https://github.com/external-secrets/external-secrets)

### Learning Resources

- [Programming Kubernetes](https://www.oreilly.com/library/view/programming-kubernetes/9781492047094/) - Book by Michael Hausenblas & Stefan Schimanski
- [Kubernetes Operators](https://www.oreilly.com/library/view/kubernetes-operators/9781492048039/) - Book by Jason Dobies & Joshua Wood
- [Sample Controller](https://github.com/kubernetes/sample-controller) - Official example in Go

---

## License

This project is provided as-is for educational purposes.

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## Support

For issues and questions:

- Open a GitHub issue
- Check existing issues and documentation
- Review Kubernetes documentation

---

**Happy Controlling! 🚀**
