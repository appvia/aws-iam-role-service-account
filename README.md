# aws-iam-role-service-account

![Version: 0.1.0](https://img.shields.io/badge/Version-0.1.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 4.11.0](https://img.shields.io/badge/AppVersion-4.11.0-informational?style=flat-square)

AWS IAM role for Service Account (IRSA) Helm chart for the [Terraform-operator](https://github.com/isaaguilar/terraform-operator).

## Prerequisites
```bash
helm repo add isaaguilar https://isaaguilar.github.io/helm-charts
helm repo update
helm upgrade --install terraform-operator isaaguilar/terraform-operator \
  --version v0.2.1 --namespace tf-system --create-namespace
```

## Get Repo Info
```bash
helm repo add appvia-community https://appvia-community.storage.googleapis.com
helm repo update
helm search repo appvia-community
```

## Install Chart
```bash
helm install [RELEASE_NAME] appvia-community/aws-iam-role-service-account \
  --namespace [NAMESPACE] \
  --create-namespace \
  --set aws.region=[AWS_REGION] \
  --set aws.credentials=[AWS_CREDENTIALS] \
  --set iam.role_name=[AWS_IAM_ROLE_NAME] \
  --set iam.cluster_service_accounts=[K8S_SERVICE_ACCOUNT]
```

## Upgrade Chart
```bash
helm upgrade --install [RELEASE_NAME] appvia-community/aws-iam-role-service-account \
  --namespace [NAMESPACE] \
  --create-namespace \
  --set aws.region=[AWS_REGION] \
  --set aws.credentials=[AWS_CREDENTIALS] \
  --set iam.role_name=[AWS_IAM_ROLE_NAME] \
  --set iam.cluster_service_accounts=[K8S_SERVICE_ACCOUNT]
```

## Uninstall Chart
```bash
helm uninstall [RELEASE_NAME]
```

## Example Usages

### Install consuming a Kubernetes Secret with AWS Credentials
```bash
helm upgrade --install aws-iam-role-service-account appvia-community/aws-iam-role-service-account \
  --namespace my-ns \
  --set aws.region=eu-west-2 \
  --set aws.credentials[0].secretNameRef.key=data \
  --set aws.credentials[0].secretNameRef.name=tf-aws-secrets \
  --set aws.credentials[0].secretNameRef.namespace=my-ns \
  --set iam.role_name=my-app \
  --set iam.cluster_service_accounts.my-cluster[0]="my-ns:my-app-sa"
```

### Install generating a temporary ServiceAccount and setting an IRSA Annotation for authentication to AWS
```bash
helm upgrade --install aws-iam-role-service-account appvia-community/aws-iam-role-service-account \
  --namespace my-ns \
  --set aws.region=eu-west-2 \
  --set aws.credentials[0].serviceAccountAnnotations."eks\.amazonaws\.com/role-arn"="arn:aws:iam::123456789012:role/existing-irsa-role-name" \
  --set iam.role_name=my-app \
  --set iam.cluster_service_accounts.my-cluster[0]="my-ns:my-app-sa"
```

> **Note:** The AWS IAM Policy Document would require a wildcard to support the randomly generated ServiceAccount names, in the form `system:serviceaccount:<my-namespace>:tf-<chart-name>-*`.

### Install leveraging an existing IRSA to provision AWS Resources
```bash
helm upgrade --install aws-iam-role-service-account appvia-community/aws-iam-role-service-account \
  --namespace my-ns \
  --set aws.region=eu-west-2 \
  --set kubernetes.serviceAccount=irsa-name \
  --set iam.role_name=my-app \
  --set iam.cluster_service_accounts.my-cluster[0]="my-ns:my-app-sa"
```

### Example Secret contents
```
apiVersion: v1
kind: Secret
metadata:
  name: tf-aws-secrets
type: Opaque
data:
  AWS_ACCESS_KEY_ID: ZXhhbXBsZS1rZXktaWQ=
  AWS_SECRET_ACCESS_KEY: ZXhhbXBsZS1zZWNyZXQta2V5
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| aws.credentials | object | `{}` | The AWS credentials to be used for provisioning the IAM role. See [supported credential types](http://tf.isaaguilar.com/docs/references/configuration/#credentials-v1alpha1-tf) |
| aws.region | string | `""` | The AWS region where the IAM role should be created |
| kubernetes.serviceAccount | string | `""` | Name of the k8s ServiceAccount to use for creating Terraform Pods. Can be used for AWS authentication if an IRSA annotation is already present on the resource. |
| iam.cluster_service_accounts | object | `{}` | EKS cluster and k8s ServiceAccount pairs. Each EKS cluster can have multiple k8s ServiceAccounts. See [Terraform example usage](https://github.com/terraform-aws-modules/terraform-aws-iam/tree/master/modules/iam-eks-role#iam-eks-role) [MUTABLE] |
| iam.role_name | string | `""` | Name of IAM role [IMMUTABLE]|
| terraform.module | string | `"https://github.com/terraform-aws-modules/terraform-aws-iam.git//modules/iam-eks-role"` | The HashiCorp official Terraform module |
| terraform.moduleVersion | string | `"v4.11.0"` | The version of the Terraform module used to create an IAM role |
| terraform.version | string | `"1.1.4"` | The version of Terraform used |

:warning: A change to an `IMMUTABLE` parameter after initial creation will force a re-creation of the AWS IAM role. However a `MUTABLE` parameter is a configurable option by design and can be changed.

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.7.0](https://github.com/norwoodj/helm-docs/releases/v1.7.0)
