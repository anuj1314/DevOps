# Building a Simple Kubernetes Controller in Python

A beginner-friendly guide to understanding Kubernetes controllers by building a working example.

## Overview

This tutorial demonstrates how to build a simple Kubernetes controller in Python that manages custom resources. We'll create a controller that watches for `Website` resources and automatically creates and manages ConfigMaps based on them.

## What Will Our Controller Do?

Our controller will:

- **Watch** for Website resources
- **Create** a ConfigMap when a Website is created
- **Update** the ConfigMap when a Website is updated
- **Delete** the ConfigMap when a Website is deleted

Simple, but demonstrates the core concepts of the Kubernetes controller pattern!

## Prerequisites

### 1. Install the Kubernetes Python Client

```bash
pip install kubernetes
```

### 2. Ensure CRD is Installed

Make sure you have the Website CRD installed in your cluster:

```bash
kubectl apply -f website-crd.yaml
```

## The Controller Code

Create a file named `website-controller.py`:

```python
# website-controller.py
from kubernetes import client, config, watch
import time
import logging

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

class WebsiteController:
    def __init__(self):
        """Initialize the controller"""
        # Load kubernetes config (from ~/.kube/config if running locally)
        try:
            config.load_kube_config()
            logger.info("Loaded kube config from ~/.kube/config")
        except:
            # If running inside a pod, use in-cluster config
            config.load_incluster_config()
            logger.info("Loaded in-cluster config")
        
        # Create API clients
        self.custom_api = client.CustomObjectsApi()
        self.core_api = client.CoreV1Api()
        
        # Our CRD details
        self.group = "example.com"
        self.version = "v1"
        self.plural = "websites"
    
    def create_configmap_for_website(self, website):
        """Create a ConfigMap for a Website resource"""
        name = website['metadata']['name']
        namespace = website['metadata'].get('namespace', 'default')
        spec = website['spec']
        
        # Create ConfigMap with website data
        configmap = client.V1ConfigMap(
            metadata=client.V1ObjectMeta(
                name=f"{name}-config",
                namespace=namespace,
                labels={
                    "managed-by": "website-controller",
                    "website": name
                }
            ),
            data={
                "url": spec.get('url', ''),
                "owner": spec.get('owner', ''),
                "status": spec.get('status', ''),
                "website-name": name
            }
        )
        
        try:
            # Check if ConfigMap already exists
            existing = self.core_api.read_namespaced_config_map(
                name=f"{name}-config",
                namespace=namespace
            )
            # Update existing ConfigMap
            self.core_api.replace_namespaced_config_map(
                name=f"{name}-config",
                namespace=namespace,
                body=configmap
            )
            logger.info(f"Updated ConfigMap {name}-config in namespace {namespace}")
        except client.exceptions.ApiException as e:
            if e.status == 404:
                # ConfigMap doesn't exist, create it
                self.core_api.create_namespaced_config_map(
                    namespace=namespace,
                    body=configmap
                )
                logger.info(f"Created ConfigMap {name}-config in namespace {namespace}")
            else:
                logger.error(f"Error managing ConfigMap: {e}")
    
    def delete_configmap_for_website(self, website):
        """Delete the ConfigMap for a Website resource"""
        name = website['metadata']['name']
        namespace = website['metadata'].get('namespace', 'default')
        
        try:
            self.core_api.delete_namespaced_config_map(
                name=f"{name}-config",
                namespace=namespace
            )
            logger.info(f"Deleted ConfigMap {name}-config from namespace {namespace}")
        except client.exceptions.ApiException as e:
            if e.status == 404:
                logger.info(f"ConfigMap {name}-config already deleted")
            else:
                logger.error(f"Error deleting ConfigMap: {e}")
    
    def handle_event(self, event):
        """Handle a watch event for Website resources"""
        event_type = event['type']
        website = event['object']
        name = website['metadata']['name']
        
        logger.info(f"Event: {event_type} for Website {name}")
        
        if event_type in ['ADDED', 'MODIFIED']:
            # Website was created or updated
            self.create_configmap_for_website(website)
        elif event_type == 'DELETED':
            # Website was deleted
            self.delete_configmap_for_website(website)
    
    def run(self):
        """Main loop - watch for Website resources and handle events"""
        logger.info("Starting Website Controller...")
        logger.info(f"Watching for {self.group}/{self.version} {self.plural}")
        
        while True:
            try:
                # Create a watch stream for Website resources
                w = watch.Watch()
                for event in w.stream(
                    self.custom_api.list_cluster_custom_object,
                    group=self.group,
                    version=self.version,
                    plural=self.plural
                ):
                    self.handle_event(event)
                    
            except client.exceptions.ApiException as e:
                logger.error(f"API Exception: {e}")
                logger.info("Retrying in 5 seconds...")
                time.sleep(5)
            except Exception as e:
                logger.error(f"Unexpected error: {e}")
                logger.info("Retrying in 5 seconds...")
                time.sleep(5)

if __name__ == "__main__":
    controller = WebsiteController()
    controller.run()
```

## Understanding the Code

### 1. Initialization (`__init__`)

```python
config.load_kube_config()  # Connect to your Kubernetes cluster
self.custom_api = client.CustomObjectsApi()  # For CRDs
self.core_api = client.CoreV1Api()  # For ConfigMaps
```

The controller initializes by connecting to Kubernetes and creating API clients for managing custom resources and ConfigMaps.

### 2. The Watch Loop (`run` method)

```python
for event in w.stream(self.custom_api.list_cluster_custom_object, ...):
```

This continuously watches for changes to Website resources. Every time something happens (create, update, delete), we get an event.

### 3. Event Handling (`handle_event` method)

- **ADDED**: A new Website was created → create ConfigMap
- **MODIFIED**: A Website was updated → update ConfigMap
- **DELETED**: A Website was deleted → delete ConfigMap

### 4. ConfigMap Management

The controller creates ConfigMaps that store the website information. This is simple but demonstrates the pattern used by production controllers.

## Running the Controller

### Step 1: Ensure the CRD is Installed

```bash
kubectl apply -f website-crd.yaml
```

### Step 2: Run the Controller

```bash
python website-controller.py
```

You should see output like:

```
2025-01-27 10:00:00 - INFO - Starting Website Controller...
2025-01-27 10:00:00 - INFO - Watching for example.com/v1 websites
```

### Step 3: Create a Website Resource

In another terminal, create a Website:

```bash
kubectl apply -f my-website.yaml
```

Example `my-website.yaml`:

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: company-homepage
spec:
  url: "https://example.com"
  owner: "engineering-team"
  status: "active"
```

### Step 4: Watch the Controller Logs

You'll see the controller react to the creation:

```
2025-01-27 10:00:05 - INFO - Event: ADDED for Website company-homepage
2025-01-27 10:00:05 - INFO - Created ConfigMap company-homepage-config in namespace default
```

### Step 5: Verify the ConfigMap was Created

```bash
kubectl get configmap company-homepage-config -o yaml
```

Output:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: company-homepage-config
  labels:
    managed-by: website-controller
    website: company-homepage
data:
  url: "https://example.com"
  owner: "engineering-team"
  status: "active"
  website-name: "company-homepage"
```

### Step 6: Update the Website

```bash
kubectl edit website company-homepage
# Change status from "active" to "maintenance"
```

Watch the controller logs - it automatically updates the ConfigMap!

### Step 7: Delete the Website

```bash
kubectl delete website company-homepage
```

The controller automatically deletes the ConfigMap too!

## The Kubernetes Controller Pattern

This controller demonstrates the core pattern used throughout Kubernetes:

1. **Watch**: Monitor resources for changes
2. **Compare**: Check desired state (Website spec) vs actual state (ConfigMap exists?)
3. **Act**: Take action to make actual state match desired state

This is exactly what production operators like the Prometheus Operator, MySQL Operator, and others do - they just manage real infrastructure instead of ConfigMaps!

## Making It Production-Ready

To make this controller production-ready, you'd want to add:

### 1. Status Updates

Update the Website resource's status field to reflect the current state:

```python
# Update status to show ConfigMap was created
self.custom_api.patch_namespaced_custom_object_status(
    group=self.group,
    version=self.version,
    namespace=namespace,
    plural=self.plural,
    name=name,
    body={"status": {"configMapCreated": True}}
)
```

### 2. Error Handling

Add retry logic and exponential backoff for transient failures.

### 3. Finalizers

Ensure cleanup happens before deletion:

```python
# Add finalizer to Website resource
finalizer = "website-controller.example.com/finalizer"
```

### 4. Kubernetes Events

Create events for debugging:

```python
self.core_api.create_namespaced_event(
    namespace=namespace,
    body=client.V1Event(...)
)
```

### 5. Health Checks

Add liveness and readiness probes when running as a Pod.

### 6. Deploy as a Pod

Run the controller inside the cluster using a Deployment.
