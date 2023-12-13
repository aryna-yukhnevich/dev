# Grafana Deployment options
The Timestream datasource plugin required: 
- https://grafana.com/grafana/plugins/grafana-timestream-datasource/

Costs depend on:
- the number of active users
- the dashboard complexity

## AWS Managed Grafana
Deployment:
- Grafana workspace can be created using CloudFormation

Docs: 
- https://aws.amazon.com/grafana/pricing/ 
- https://docs.aws.amazon.com/grafana/latest/userguide/getting-started-with-AMG.html 
- https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-grafana-workspace.html

Costs:
- Pay-per-use pricing with no infrastructure to manage, based on an active user license per workspace
- $9.00 (Editor license) per active editor or administrator user per workspace
- $5.00 (Viewer license) per active user per workspace


## Bitnami package for Grafana on AWS Marketplace
Deployment:
- Launch Grafana directly from the AWS Marketplace with pre-configured settings
- https://aws.amazon.com/marketplace/pp/prodview-4vshvaxh6go7i?sr=0-5&ref_=beagle&applicationId=AWSMPContessa 

Costs:
- EC2 instance costs: we can start with t3.medium, t3.large instance types, suitable for small to medium-sized deployments with moderate traffic.

## Containerized deployment on ECS or EKS
Deployment:
- Deploy Grafana as a Docker container
- Official Grafana Docker image: https://hub.docker.com/r/grafana/grafana/ 

Costs:
- Similar to EC2 costs but with additional costs for ECS or EKS

## Grafana on Kubernetes
Deployment:
- Deploy Grafana official Docker image in Kubernetes cluster
- Official Grafana Docker image: https://hub.docker.com/r/grafana/grafana/ 
- Deploy using Kubernetes manifests https://grafana.com/docs/grafana/latest/setup-grafana/installation/kubernetes/
- Deploy using Helm Charts https://github.com/grafana/helm-charts 

Costs:
- Similar to EC2 costs but with additional costs for EKS

## Grafana on Fargate
Deployment:
- Deploy a Grafana container on AWS Fargate using the Grafana Docker Image on DockerHub 
- CDK project sample: https://github.com/aws-samples/aws-cdk-grafana/tree/main

Costs:
- Fargate costs with additional costs for ECS 

## Grafana on EC2
Deployment:
- Cloudformation template can be used to create EC2 instance and install Grafana using a UserData script

Costs:
- EC2 instance costs

## Grafana Cloud
Deployment:
- SaaS offering

Costs:
- Pay-per-use pricing with no infrastructure to manage

Docs:
- https://grafana.com/pricing/
- https://grafana.com/auth/sign-up?refCode=gr83y6yqNccjKnX


## Deploy the dashboard to Grafana using AWSCDK
- https://pypi.org/project/cdk-grafana-json-dashboard-handler/ 