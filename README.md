# Crossplane
This repository contains the Helm values to deploy [Crossplane](https://github.com/crossplane/crossplane) to the Kubernetes clusters, with an API Gateway as an example.

>Crossplane is an open source Kubernetes extension that transforms your Kubernetes cluster into a universal control plane

## Documentation

📖 [CrossPlane](https://docs.crossplane.io/latest/)

📖 [SRE Deployment Guide](./docs/sre-deployment-guide.md)

## Prerequisites
Before diving into the deployment process, make sure you satisfy the following criteria:

- **Cluster Configuration**: Ensure you have access to and have configured the clusters you're deploying to. 
- **Clone the Repository**: Clone this repository to your local machine.
  
## Setup the Helm Chart to crossplane:
Command to perform the Helm Chart deployment:

````bash

helm install crossplane \
crossplane-stable/crossplane \
--namespace crossplane-system \
--create-namespace -f values.yaml

````

### Chart Details
To better understand the components being deployed, here are the details of the Helm chart:

| Field             | Details                                                                                                    |
| ----------------- | ---------------------------------------------------------------------------------------------------------- |
| **Home Page**     | [CrossPlane](https://docs.crossplane.io/)                                          |
| **GitHub**        | [Crossplane](https://github.com/crossplane/crossplane)                      |
| **ArtifactHub**   | [crossplane](https://artifacthub.io/packages/helm/crossplane/crossplane) |
| **Chart Name**    | crossplane                                                                                         |
| **Chart Repo**    | https://charts.crossplane.io/stable                                                                      |
| **Chart Version** | `1.16.0-rc.0.223.gda85751d`                                                                                                   |
| **App Version**   | `1.16.0-rc.0.223.gda85751d`                                                                                                  |

---

## values.yaml


<details>
<summary><em><strong>Ver arquivo:</strong></em></summary>

**values.yaml**
```sh
provider:
  packages: 
    - xpkg.upbound.io/upbound/provider-aws-apigateway:v1.3.1
extraObjects:
  - apiVersion: aws.upbound.io/v1beta1
    kind: ProviderConfig
    metadata:
      name: aws-provider-apw
      namespace: crossplane-system 
    spec:
      credentials:
        source: WebIdentity
        webIdentity:
          roleARN: "arn:aws:iam::XXXXXXXXXXXXX:role/crossplane-apw-k8s"

```
</details>

## Create Necessary Permission

This configuration sets up a Crossplane provider to interact with AWS API Gateway. To make this work, you must first create an IAM role in AWS with the necessary permissions and a trust relationship that allows the Kubernetes service account to assume the role using Web Identity. Once the IAM role is configured, apply this ProviderConfig to your Kubernetes cluster. This will enable Crossplane inside the cluster to access AWS resources.

## General Infos

This Helm chart performs the creation of the AWS API Gateway (RestAPI) resource via Crossplane, enabling the creation of a fully customized body as shown in the Official AWS Documentation. We are demonstrating only a few functionalities in the examples below, with emphasis on the x-amazon-apigateway-integration and the creation of multiple paths and their configurations.

## Setup the Helm Chart to RESTAPI:
Command to perform the Helm Chart deployment:

````bash

helm install restapi-crossplane . --namespace crossplane-system  -f values.yaml

````

## Custom values
 ````values.yaml````

Example:
```sh
restapi:
  name: api-name-here
  description: "This is my API for demonstration purposes"
  region: us-east-1
  endpointConfiguration:
    types: [PRIVATE]
    vpcEndpointIds: [vpce-YOUR-VPC-ENDPOINT-ID]
  body: >-
    ${jsonencode({
        openapi = "3.0.1"
        info = {
          title   = "example"
          version = "1.0"
        }
        paths = {
          "/api" = {
            get = {
              x-amazon-apigateway-integration = {
                httpMethod           = "GET"
                payloadFormatVersion = "1.0"
                type                 = "HTTP_PROXY"
                uri                  = "http://apps.k8s.com/api"
                connectionType       = "VPC_LINK"
                connectionId         = "ID-DO-VPC-LINK-AQUI, ex 'hvbfr55i'"
              }
            }
          }
          "/user" = {
            get = {
              x-amazon-apigateway-integration = {
                httpMethod           = "GET"
                payloadFormatVersion = "1.0"
                type                 = "HTTP_PROXY"
                uri                  = "http://apps.k8s.com/user"
                connectionType       = "VPC_LINK"
                connectionId         = "ID-DO-VPC-LINK-AQUI, ex 'ghyt55i'"
                requestParameters = {
                  "integration.request.header.example-header" = "'app-example-header'"
                }
              }
            }
          }
        }
      })}
```


### Notes:
 
- For requestParameters and other possibilities of x-amazon-apigateway-integration, please refer to the Official AWS Documentation.
Happy Deploying!

