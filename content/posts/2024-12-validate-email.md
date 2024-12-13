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

- [Introduction](#introduction)
- [Features and Capabilities](#features-and-capabilities)
    - [Comprehensive Email Validation](#comprehensive-email-validation)
    - [Lightweight and Scalable](#lightweight-and-scalable)
    - [API-Driven Architecture](#api-driven-architecture)
    - [Open-Source and Extensible](#open-source-and-extensible)
- [Releasing validate-email as Open Source](#releasing-validate-email-as-open-source)
- [Deploying validate-email in Kubernetes](#deploying-validate-email-in-kubernetes)
    - [Prerequisites](#prerequisites)
    - [Deployment Steps](#deployment-steps)
    - [Configuration Options](#configuration-options)
    - [Uninstall / Removal](#uninstall--removal)
- [Testing the application](#testing-the-application)
- [Conclusions](#conclusions)

<!-- /TOC -->

## Introduction

In today's digital landscape, verifying the validity of an email address is essential for reducing spam, ensuring smooth communication, and maintaining the integrity of user data. The `validate-email` open-source project, available on [GitLab](https://gitlab.com/besmirzanaj/validate-email), is designed to address this need. 

This solution offers the feature to validate an email addresses and is packaged for easy deployment in Kubernetes using Helm charts.

I am releasing this simple program today and this article provides an overview of its capabilities and guides you through deploying it in a Kubernetes environment.


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


## Releasing `validate-email` as Open Source

The decision to release `validate-email` as an open-source project stems from a commitment to fostering collaboration and innovation in the community. By making the code publicly available, I aim to:

1. **Encourage Community Contributions**:
   - Developers can suggest features, report issues, or contribute directly to the codebase.
   - This ensures continuous improvement and adaptability to emerging needs.

2. **Promote Transparency and Trust**:
   - Users can review the code to understand how email validation processes are implemented.
   - Open-source projects provide assurance of ethical practices and robust security.

3. **Enable Widespread Adoption**:
   - By removing barriers to entry, organizations of all sizes can integrate `validate-email` into their systems.
   - This democratizes access to reliable email validation tools.

Feel free to explore the source code on [GitLab](https://gitlab.com/besmirzanaj/validate-email), provide feedback, and contribute if you like. 


## Deploying `validate-email` in Kubernetes

### Prerequisites

Before deploying, ensure you have the following:
1. A Kubernetes cluster (local or cloud-based).
2. Helm installed on your local machine. Refer to the [Helm installation guide](https://helm.sh/docs/intro/install/) if needed.
3. Access to the `validate-email` Helm chart repository: [https://besmirzanaj.github.io/charts/](https://besmirzanaj.github.io/charts/).

### Deployment Steps

#### Step 1: Add the Helm Chart Repository

```bash
helm repo add besmirzanaj https://besmirzanaj.github.io/charts/
helm repo update
```

#### Step 2: Install the Chart

Deploy the `validate-email` application using the Helm chart:

```bash
helm install validate-email besmirzanaj/validate-email --namespace validate-email --create-namespace
```

This command installs the application in the `validate-email` namespace. You can customize the release name (`validate-email`) and namespace as needed.

#### Step 3: Verify the Deployment

Check the status of the pods to ensure the application is running:

```bash
kubectl get pods -n validate-email
```

Look for pods with names in the `validate-email` namespace and ensure their status is `Running`.

#### Step 4: Access the Application

Once deployed, you can access the `validate-email` API. Retrieve the service's external IP or port using:

```bash
kubectl get svc -n email-validation
```

The service exposes the API by default on port `9001` (depending on the configuration).


### Configuration Options

The Helm chart offers several configuration options. To customize your deployment, create a `values.yaml` file with the desired settings. For example you can define how many replicas you need for the deployment, the Service type and port, and if you do not need autorgenerated credentials, feel free to override the values below:

```yaml
replicaCount: 3
credentials:
  validate_jwt_key: ~
  validate_api_user: ~
  validate_api_password: ~
service:
  type: ClusterIP
  port: 9001
```

Apply the configuration during installation:
```bash
helm install validate-email besmirzanaj/validate-email -f values.yaml --namespace email-validation
```

### Uninstall / Removal

You can use the normal helm uninstall command to remove any trace of this software:

```bash
helm uninstall validate-email --namespace validate-email
```

## Testing the application

First expose the app locally:

```bash
kubectl --namespace validate-email port-forward service/validate-email 8080:9001
```

Then on a new terminal get the JWT secret key and the username password the helm chart created for you:

```bash
VALIDATE_JWT_SECRET_KEY=$(kubectl -n validate-email get secret validate-email-validate-email -o jsonpath="{.data.validate_jwt_key}" | base64 -d)
VALIDATE_API_USER=$(kubectl -n validate-email get secret validate-email-validate-email -o jsonpath="{.data.validate_api_user}" | base64 -d)
VALIDATE_API_PASSWORD=$(kubectl -n validate-email get secret validate-email-validate-email -o jsonpath="{.data.validate_api_password}" | base64 -d)
```

Define the host where the app is running, and its endpoints. We exposed in this example with port-forward:

```bash
VALIDATE_HOSTNAME="http://127.0.0.1"
VALIDATE_LOGIN_URL="$VALIDATE_HOSTNAME:8080/login"
VALIDATE_API_URL="$VALIDATE_HOSTNAME:8080/validate_email"
```

Get the token to use the app:

```bash
VALIDATE_TOKEN=$(curl -X POST "$VALIDATE_LOGIN_URL" -H "Content-Type: application/json" -d '{"username": "'"${VALIDATE_API_USER}"'", "password": "'"${VALIDATE_API_PASSWORD}"'"}' -s | jq .access_token -r)
```

Finally run the validation:

```bash
curl -s "$VALIDATE_API_URL" -X POST -d '{"email":"besmirzanaj@gmail.com"}' -H 'Content-type: Application/json' -H "Authorization: Bearer $VALIDATE_TOKEN"

{
  "email": "besmirzanaj@gmail.com",
  "status": true
}
```

## Conclusions

The `validate-email` open-source project is a powerful tool for ensuring email validity and maintaining data quality. With its easy-to-use Helm chart, you can quickly deploy it in a Kubernetes environment, scaling effortlessly to meet your application’s needs.

Explore the source code and contribute to the project on [GitLab](https://gitlab.com/besmirzanaj/validate-email). You can also use `validate-email` to improve your email validation workflows in a microservices environment.

