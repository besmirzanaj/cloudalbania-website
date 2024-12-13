---
title: 'Announcing validate-email: An Open-Source Email Validation Solution'
date: "2024-12-13"
description: "Announcing validate-email: An Open-Source Email Validation Solution"
draft: false
tags: 
  - email
  - helm
  - k8s
  - kuernetes
  - microservice
  - open source

---

<!-- TOC -->

- [Features and Capabilities](#features-and-capabilities)
    - [Comprehensive Email Validation](#comprehensive-email-validation)
    - [Lightweight and Scalable](#lightweight-and-scalable)
    - [API-Driven Architecture](#api-driven-architecture)
    - [Open-Source and Extensible](#open-source-and-extensible)
- [Releasing Validate-Email as Open Source](#releasing-validate-email-as-open-source)
- [Deploying Validate Email in Kubernetes](#deploying-validate-email-in-kubernetes)
    - [Prerequisites](#prerequisites)
    - [Deployment Steps](#deployment-steps)
- [Configuration Options](#configuration-options)
- [Conclusion](#conclusion)

<!-- /TOC -->

I am releaseing this simple program today and this article provides an overview of `validate-email`'s capabilities and guides you through deploying it in a Kubernetes environment.

---

## Features and Capabilities

### Comprehensive Email Validation
- **Syntax Checking**: Ensures that email addresses conform to the standard email format.
- **Domain Validation**: Verifies the existence of the email domain (e.g., `@example.com`).
- **SMTP Validation**: Checks the deliverability of the email by connecting to the mail server.

### Lightweight and Scalable
- Written in Python for high performance and simplicity.
- Optimized for deployment in containerized environments.

### API-Driven Architecture
- Offers RESTful API endpoints for seamless integration with your applications.
- Supports JSON-based requests and responses for modern web services.

### Open-Source and Extensible
- Freely available under an open-source license.
- Designed with extensibility in mind, allowing developers to adapt it to specific needs.

---

## Releasing Validate-Email as Open Source

The decision to release `validate-email` as an open-source project stems from a commitment to fostering collaboration and innovation in the community. By making the code publicly available, I aim to:

1. **Encourage Community Contributions**:
   - Developers can suggest features, report issues, or contribute directly to the codebase.
   - This ensures continuous improvement and adaptability to emerging needs.

2. **Promote Transparency and Trust**:
   - Users can review the code to understand how email validation processes are implemented.
   - Open-source projects provide assurance of ethical practices and robust security.

3. **Enable Widespread Adoption**:
   - By removing barriers to entry, organizations of all sizes can integrate Validate Email into their systems.
   - This democratizes access to reliable email validation tools.

Feel free to explore the source code on [GitLab](https://gitlab.com/besmirzanaj/validate-email), provide feedback, and contribute if you like. 

---

## Deploying Validate Email in Kubernetes

### Prerequisites

Before deploying, ensure you have the following:
1. A Kubernetes cluster (local or cloud-based).
2. Helm installed on your local machine. Refer to the [Helm installation guide](https://helm.sh/docs/intro/install/) if needed.
3. Access to the Validate Email Helm chart repository: [https://besmirzanaj.github.io/charts/](https://besmirzanaj.github.io/charts/).

### Deployment Steps

#### Step 1: Add the Helm Chart Repository

```bash
helm repo add besmirzanaj https://besmirzanaj.github.io/charts/
helm repo update
```

#### Step 2: Install the Chart

Deploy the Validate Email application using the Helm chart:

```bash
helm install validate-email besmirzanaj/validate-email --namespace email-validation --create-namespace
```

This command installs the application in the `email-validation` namespace. You can customize the release name (`validate-email`) and namespace as needed.

#### Step 3: Verify the Deployment
Check the status of the pods to ensure the application is running:
```bash
kubectl get pods -n email-validation
```
Look for pods with names starting with `validate-email` and ensure their status is `Running`.

#### Step 4: Access the Application
Once deployed, you can access the Validate Email API. Retrieve the service's external IP or port using:
```bash
kubectl get svc -n email-validation
```
The service typically exposes the API on port 80 or 443 (depending on the configuration).

---

## Configuration Options

The Helm chart offers several configuration options. To customize your deployment, create a `values.yaml` file with the desired settings. For example:

```yaml
replicaCount: 3
environment:
  SMTP_CHECK: true
  LOG_LEVEL: debug
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

Apply the configuration during installation:
```bash
helm install validate-email besmirzanaj/validate-email -f values.yaml --namespace email-validation
```

---

## Conclusion

The Validate Email open-source project is a powerful tool for ensuring email validity and maintaining data quality. With its easy-to-use Helm chart, you can quickly deploy it in a Kubernetes environment, scaling effortlessly to meet your application’s needs.

Explore the source code and contribute to the project on [GitLab](https://gitlab.com/besmirzanaj/validate-email). Start leveraging Validate Email today to improve your email validation workflows.

