# Custom Resource Definitions (CRDs) in Kubernetes

## Overview

A **Custom Resource Definition (CRD)** allows you to extend Kubernetes by defining your own resource types.
Once defined, these custom resources behave just like native Kubernetes objects such as `Pods`, `Deployments`, or `Services`.

With CRDs, you can:

* Define new APIs inside Kubernetes
* Store structured domain-specific data
* Manage custom resources using `kubectl`
* Build higher-level abstractions and operators

---

## What Is a CRD?

Kubernetes comes with many built-in resource types:

* Pods
* Deployments
* Services
* ConfigMaps

A **CRD** lets you define **your own resource types** that integrate natively with Kubernetes.

Once a CRD is installed:

* Kubernetes recognizes the new resource
* You can create, read, update, and delete it
* Validation is enforced using an OpenAPI schema
* Objects are stored in `etcd` like any other Kubernetes resource

---

## Why Use CRDs?

CRDs are useful when Kubernetes’ built-in resources are too low-level for your use case.

### Example Use Case

Instead of managing databases using raw Pods and Services, you could define a custom resource:

```bash
kubectl get databases
```

Each `Database` resource could contain:

* Database engine version
* Backup schedule
* Storage size
* Replication settings

This provides a **clean, declarative, and domain-specific API**.

---

## Example: Website CRD

This example defines a custom resource named **Website**, used to track websites with metadata such as URL, owner, and status.

---

## Step 1: Define the CRD

Create a CRD definition file.

### `website-crd.yaml`

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # Format: <plural>.<group>
  name: websites.example.com
spec:
  # API group for the custom resource
  group: example.com

  # Supported versions
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
                  enum:
                    - active
                    - maintenance
                    - offline
                  description: "Current status of the website"
              required:
                - url
                - owner

  # Resource scope
  scope: Namespaced

  # Naming configuration
  names:
    plural: websites
    singular: website
    kind: Website
    shortNames:
      - ws
```

---

## CRD Field Breakdown

### `apiVersion`

Specifies the Kubernetes API used to define CRDs.

```yaml
apiextensions.k8s.io/v1
```

---

### `metadata.name`

Must follow the format:

```text
<plural>.<group>
```

Example:

```yaml
websites.example.com
```

---

### `spec.group`

Acts as a namespace for your custom API.

Example:

```yaml
example.com
```

---

### `spec.versions`

Defines one or more API versions.

Key fields:

* `served`: Whether this version is accessible via the API
* `storage`: The version used to store data in `etcd`
* `schema`: OpenAPI v3 schema for validation

---

### `spec.scope`

Determines where resources exist:

* `Namespaced` → Resources belong to a namespace
* `Cluster` → Resources are cluster-wide

---

### `spec.names`

Controls how users interact with the resource:

| Field        | Purpose                        |
| ------------ | ------------------------------ |
| `plural`     | Used in URLs and `kubectl get` |
| `singular`   | Alias for CLI usage            |
| `kind`       | Used in YAML manifests         |
| `shortNames` | CLI shortcuts                  |

---

## Step 2: Install the CRD

Apply the CRD to your cluster:

```bash
kubectl apply -f website-crd.yaml
```

Verify installation:

```bash
kubectl get crd websites.example.com
```

At this point, Kubernetes recognizes the `Website` resource type.

---

## Step 3: Create Custom Resources

Now create actual `Website` objects.

### `my-website.yaml`

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: company-homepage
  namespace: default
spec:
  url: "https://example.com"
  owner: "engineering-team"
  status: "active"
```

Apply it:

```bash
kubectl apply -f my-website.yaml
```

---

## Step 4: Interact with Custom Resources

Use standard Kubernetes commands.

```bash
# List all websites
kubectl get websites

# Using short name
kubectl get ws

# Describe a website
kubectl describe website company-homepage

# Output as YAML
kubectl get website company-homepage -o yaml

# Delete a website
kubectl delete website company-homepage
```

---

## Step 5: Additional Examples

```yaml
apiVersion: example.com/v1
kind: Website
metadata:
  name: blog-site
spec:
  url: "https://blog.example.com"
  owner: "marketing-team"
  status: "maintenance"
---
apiVersion: example.com/v1
kind: Website
metadata:
  name: docs-site
spec:
  url: "https://docs.example.com"
  owner: "technical-writers"
  status: "active"
```

---

## Validation Example

The schema enforces required fields.
The following will **fail validation** because `owner` is missing:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: example.com/v1
kind: Website
metadata:
  name: bad-website
spec:
  url: "https://bad.example.com"
EOF
```

---

## What CRDs Do (and Don’t Do)

### What CRDs Do

* Define new resource types
* Validate input using schemas
* Store structured data in Kubernetes
* Integrate with RBAC, admission, and tooling

### What CRDs Don’t Do

* Automatically create Pods, Services, or infrastructure
* Perform reconciliation or automation by themselves

---

## Controllers and Operators

To make CRDs **actionable**, you need a **controller** (also called an operator).

A controller can:

* Watch custom resources
* React to create/update/delete events
* Create or modify Kubernetes resources
* Update status fields

Example actions for a `Website` controller:

* Create a Deployment and Service
* Configure ingress or DNS
* Update website status based on health checks

---

## Cleanup

```bash
kubectl delete website company-homepage
kubectl delete crd websites.example.com
```

---

## Summary

* CRDs extend Kubernetes with custom APIs
* They enable domain-specific abstractions
* They integrate seamlessly with `kubectl`
* Controllers bring CRDs to life

CRDs are a foundational building block for **Kubernetes Operators** and advanced platform engineering.

---


