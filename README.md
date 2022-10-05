# ArgoCD-ECR-Updater [![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/argocd-ecr-updater)](https://artifacthub.io/packages/search?repo=argocd-ecr-updater)

ECR currently requires token authentication.

ArgoCD supports username and password authentication for helm repos but does not have native functionality to refresh tokens

This image regenerates ECR tokens and updates an ArgoCD manifest with the updated token

See the following for more info
* https://docs.aws.amazon.com/AmazonECR/latest/userguide/ECR_on_EKS.html
* https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html

## Configuration
Create and apply a repository configuration
```
apiVersion: v1
kind: Secret
metadata:
  name: YOUR_REPO_SECRET_NAME
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  url: YOUR_AWS_ACCOUNT_ID.dkr.ecr.YOUR_AWS_REGION.amazonaws.com
  name: ecr
  type: helm
  enableOCI: "true"
  username: AWS
  password: TO_BE_AUTOGENERATED_AND_AUTOREFRESHED
```

## General parameters

| ENV VAR                        | DEFAULT        | Description                                                                                                                         |
|--------------------------------|----------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `ARGOCD_REPO_SECRET_NAME`      | `""`           | The name of the argocd repository secret to refresh.  Required.                                                                     |
| `ARGOCD_ECR_UPDATER_SYNC_CRON` | `0 */12 * * *` | cron for how often credentials should be refreshed.  defaults to every 12 hours to match the default token expiry.                  |
| `ARGOCD_ECR_REGISTRY`          | `None`         | Optionally specify a registry ID (ACCOUNT_ID) to use, if not the default registry for the account.  Useful for cross account login. |

## Monitoring
`/metrics` endpoint is exposed on port 8080 for external monitoring

## AWS Authentication
Pod-iam is the preferred method, which can be set via the serviceAccount annotations.

Otherwise (not recommended) aws access keys can be set via volume mounts, environment variables, or external secret refs
