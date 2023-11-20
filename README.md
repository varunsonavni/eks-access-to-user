# AWS EKS Cluster Setup for New Users.

This guide outlines the steps to create a role for another AWS account inside an AWS EKS cluster account, granting the necessary permissions to access AWS EKS API. Additionally, it covers the creation of Kubernetes roles, role bindings, and the configuration of the `aws-auth` ConfigMap in the `kube-system` namespace.

## AWS IAM Role Setup (For Role Based access to EKS)

1. **Create a Role in AWS EKS Cluster Account:**

   Create a role in the AWS EKS cluster account with the necessary permissions to access the AWS EKS API. Ensure that the trust relationship includes the AWS account ID for the intended user.

2. **Retrieve Role ARN:**

   Note the Amazon Resource Name (ARN) of the created role, e.g., `arn:aws:iam::55889050022:role/dev-user-role`.

## Kubernetes RBAC Setup

3. **Create Kubernetes Role:**

   ```yaml
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: dev-view-role
     namespace: converj
   rules:
     - apiGroups: [""]
       resources: ["pods", "pods/log"]
       verbs: ["get", "list", "watch"]
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: dev-view-role-binding
     namespace: converj
   subjects:
     - kind: User
       name: dev-view-role
       apiGroup: rbac.authorization.k8s.io
   roleRef:
     kind: Role
     name: dev-view-role
     apiGroup: rbac.authorization.k8s.io
   ---

4.1. **Editing aws-auth configmap in Kubernetes (under mapRoles):**

   ```yaml

   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: aws-auth
     namespace: kube-system
   data:
     mapRoles: |
       - groups:
         - system:bootstrappers
         - system:nodes
         rolearn: arn:aws:iam::00000000000:role/eks-managed-ng-converj-dev
         username: system:node:{{EC2PrivateDNSName}}
       - groups:
         - system:masters
         rolearn: arn:aws:iam::00000000000:role/OrganizationAccountAccessRole
         username: cluster-admin
       - groups:
         - dev-view-access-group
         rolearn: arn:aws:iam::00000000000:role/OrganizationReadOnlyAccessRole
         username: dev-view-role

```
## AWS IAM Role Setup (For User Based access to EKS from AWS IAM USER)

4.2. **Editing aws-auth configmap in Kubernetes (under mapUsers):**

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: aws-auth
     namespace: kube-system
   data:
     mapRoles: |
       - groups:
         - system:bootstrappers
         - system:nodes
         rolearn: arn:aws:iam::00000000000:role/eksctl-robin-personal-cluster-nod-NodeInstanceRole-TUKH4Z187ANC
         username: system:node:{{EC2PrivateDNSName}}
     mapUsers: |
       - userarn: arn:aws:iam::00000000000:user/eks-trainee
         username: cluster-admin
       - userarn: arn:aws:iam::00000000000:user/eks-developer
         username: dev-view-role
```
Note:  **Here IAM Roles or Users is used for authentication & cluster role & binding are used for Authorization **

