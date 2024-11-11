# Fake Shop

This repository is forked from the [sample project](https://github.com/KubeDev/fake-shop) shared in the Cloud & DevOps Immersion program. Fake Shop is an ecommerce application developed in Python, and the challenge is to perform its deployment in a Kubernetes cluster. 

## Table of Contents

- [AWS Infrastructure Setup Guide for App Deployment](#aws-infrastructure-setup-guide-for-app-deployment)
  - [Prerequisites](#prerequisites)
  - [Steps to Create AWS Infrastructure](#steps-to-create-aws-infrastructure)
    - [1. Create IAM Roles](#1-create-iam-roles)
      - [Role 1: EKS Cluster Role](#role-1-eks-cluster-role)
      - [Role 2: EC2 Worker Node Role](#role-2-ec2-worker-node-role)
    - [2. Configure the Network Infrastructure](#2-configure-the-network-infrastructure)
    - [3. Create the Kubernetes Cluster](#3-create-the-kubernetes-cluster)
    - [4. Add Worker Node Groups to the Kubernetes Cluster](#4-add-worker-node-groups-to-the-kubernetes-cluster)
    - [5. Perform CLI Actions](#5-perform-cli-actions)
   

# AWS Infrastructure Setup Guide for App Deployment

This guide will help you create the AWS infrastructure needed to deploy the application on a Kubernetes Cluster. If you have any questions during the process, please feel free to reach out.

## Prerequisites

- An active AWS account.

## Steps to Create AWS Infrastructure

### 1. Create IAM Roles

#### Role 1: EKS Cluster Role
- **Description**: An IAM role is an identity that you create with specific permissions. Roles can be assumed by trusted entities and provide temporary credentials.
- **AWS Service**: EKS (eks.amazonaws.com)
- **Use Case**: Allows the EKS cluster's Kubernetes control plane to manage AWS resources on your behalf.
- **Permission Policy**: `AmazonEKSClusterPolicy`
- **Role Name**: `role-eks-cluster`

#### Role 2: EC2 Worker Node Role
- **Description**: This role allows EC2 instances to call AWS services on your behalf.
- **AWS Service**: EC2 (ec2.amazonaws.com)
- **Use Case**: Enables EC2 instances to interact with services like CloudWatch and Systems Manager.
- **Permission Policies**: `EC2ContainerRegistryReadOnly`, `EKS_CNI_Policy`, `EKSWorkerNodePolicy`
- **Role Name**: `role-ec2-worker`

### 2. Configure the Network Infrastructure

Use AWS CloudFormation to create and manage resources with templates.

- **Template**: Every stack is based on a template (in JSON or YAML format) that contains configuration information about the AWS resources included in the stack.
  
- **Amazon S3 URL Template**: 
  [VPC Private Subnets Template](https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml)
  
- **Stack Name**: `stack-eks-cluster`

### 3. Create the Kubernetes Cluster

- **Service**: Elastic Kubernetes Service (EKS) - The most trusted way to start, run, and scale Kubernetes.
  
- **Cluster Name**: `eks-cluster-k8s`
  
- **Cluster IAM Role**: `role-eks-cluster` (previously created)

- **Networking**: Use the VPC and subnets created by the CloudFormation stack.
  
- **Cluster Endpoint Access**: Public and private (The cluster endpoint will be accessible from outside your VPC, but worker node traffic will remain within your VPC).

### 4. Add Worker Node Groups to the Kubernetes Cluster

- **Path**: Access Cluster > Compute > Node groups > Add node group
  
- **Definition**: A Node Group is a collection of EC2 instances that provide compute capacity to your Amazon EKS cluster. You can add multiple nodes to your cluster.
  
- **Node Group Name**: `default-ec2-node-group`
  
- **Node IAM Role**: `role-ec2-worker` (previously created)

- **Node Group Scaling Configuration**: Specify the desired number of nodes to launch initially, as well as the maximum and minimum number of nodes the group can scale to.
  
- **Subnets**: Choose only the private subnets in your VPC (this is a best practice).

### 5. Perform CLI Actions

1. Set your AWS Access Key by running:
```bash
   aws configure
```
2. Run the commands below:
```bash
aws eks update-kubeconfig --name eks-cluster-k8s
kubectl apply -f k8s/deployment.yaml
kubectl get service
```
