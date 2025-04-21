# Platform Crossplane Extensions

This repository contains Crossplane extensions for deploying standardized infrastructure components across different cloud providers, with a focus on AWS resources.

## Overview

The platform-crossplane-extensions repository provides:

- **Composite Resource Definitions (XRDs)** - Define standardized infrastructure interfaces
- **Compositions** - Implement XRDs using specific cloud provider resources
- **Kustomize overlays** - Enable multi-tenant/organization customization

Currently supported resources:
- AWS EC2 instances
- AWS DynamoDB tables

## Repository Structure

```
platform-crossplane-extensions/
├── xrds/                           # Composite Resource Definitions
│   └── aws/
│       └── ec2-dynamodb-xrd.yaml   # EC2 + DynamoDB XRD
├── compositions/                   # Implementation details
│   └── aws/
│       └── ec2-dynamodb-comp.yaml  # EC2 + DynamoDB composition
└── orgs/                          # Organization-specific overlays
    └── devplatform.io/            # Example organization
        ├── kustomization.yaml     # Kustomize configuration
        └── patches/               # Customization patches
            ├── xrd-variables.yaml        # XRD customizations
            └── composition-variables.yaml # Composition customizations
```


## Customization

This repository uses Kustomize to enable organization-specific customizations while maintaining a single source of truth for infrastructure components.

### How It Works

1. Base XRDs and Compositions use a placeholder organization (`myorg.io`)
2. Organization-specific overlays patch resources with their own domain and versions
3. JSON patch format enables precise modifications of resource names and nested fields

### Creating a New Organization Overlay

1. Create a directory under `orgs/` with your organization name (e.g., `orgs/yourcompany.io/`)
2. Copy the kustomization.yaml and patches from an existing organization
3. Update the patches with your organization name and desired API versions

Example kustomization.yaml:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../../xrds/aws
- ../../compositions/aws

patches:
- path: patches/xrd-variables.yaml
  target:
    group: apiextensions.crossplane.io
    kind: CompositeResourceDefinition
    name: awsec2dynamodbinstances.myorg.io
    version: v1
- path: patches/composition-variables.yaml
  target:
    group: apiextensions.crossplane.io
    kind: Composition
    name: awsec2dynamodbinstances.myorg.io
    version: v1

# Deployment
## Prerequisites
- Kubernetes cluster
- Crossplane installed
- ArgoCD installed (optional, for GitOps deployment)

## Manual Deployment
```bash
# Build and apply for a specific organization
cd orgs/devplatform.io
kustomize build | kubectl apply -f -
```

## ArgoCD Deployment
Create an ArgoCD application that points to your organization's directory:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: devplatform-crossplane-extensions
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/yourorg/platform-crossplane-extensions.git
    path: orgs/devplatform.io
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: crossplane-system
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

# Available Resources
EC2 + DynamoDB Instance
Creates an EC2 instance and a DynamoDB table together as a composite resource.

Usage:

```yaml
apiVersion: devplatform.io/v1alpha1
kind: EC2DynamoDBInstance
metadata:
  name: my-app-instance
spec:
  instanceType: t2.micro
```