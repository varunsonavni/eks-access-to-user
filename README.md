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
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: dev-view-role
     namespace: converj
   rules:
     - apiGroups: [""]
       resources: ["pods", "pods/log"]
       verbs: ["get", "list", "watch"]

3. **Editing aws-auth configmap in Kubernetes:**

   ```yaml
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




## AWS IAM Role Setup (For User Based access to EKS from AWS IAM USER)
