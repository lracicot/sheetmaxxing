# Sheetmaxxing

Sheet cut optimizer (static site).

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-ffdd00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://buymeacoffee.com/lracicot)

## Hosting

**URL:** https://sheetmaxxing.com

`sheetmaxxing.louisracicot.com` and `opticutter.louisracicot.com` redirect to the primary domain.

### Domain setup

1. Register `sheetmaxxing.com` and create a Route53 hosted zone for it.
2. Point the domain's nameservers at the hosted zone.
3. Set the `SITE_HOSTED_ZONE_ID` GitHub variable to that zone's ID.

## Deploy

```bash
git push origin main
```

### GitHub Actions configuration

| Name | Type | Value |
|------|------|--------|
| `AWS_DEPLOY_ROLE_ARN` | secret | IAM role ARN for OIDC deploy (from stack output below) |
| `SITE_HOSTED_ZONE_ID` | variable | Route53 hosted zone ID for `sheetmaxxing.com` |
| `HOSTED_ZONE_ID` | variable | Route53 hosted zone ID for `louisracicot.com` (legacy redirects) |

```bash
gh secret set AWS_DEPLOY_ROLE_ARN --repo lracicot/sheetmaxxing
gh variable set SITE_HOSTED_ZONE_ID --repo lracicot/sheetmaxxing
gh variable set HOSTED_ZONE_ID --repo lracicot/sheetmaxxing
```

### One-time IAM bootstrap

Deploy the GitHub OIDC role once in the target account (`us-east-1`):

```bash
aws cloudformation deploy \
  --template-file cloud/github-oidc.cfn.yaml \
  --stack-name opticutter-github-oidc \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

If the GitHub OIDC provider already exists in the account, pass its ARN:

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
aws cloudformation deploy \
  --template-file cloud/github-oidc.cfn.yaml \
  --stack-name opticutter-github-oidc \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1 \
  --parameter-overrides "OIDCProviderArn=arn:aws:iam::${ACCOUNT_ID}:oidc-provider/token.actions.githubusercontent.com"
```

Set `AWS_DEPLOY_ROLE_ARN` to `arn:aws:iam::<account-id>:role/opticutter_github_cicd`, then push to `main` or re-run the workflow.
