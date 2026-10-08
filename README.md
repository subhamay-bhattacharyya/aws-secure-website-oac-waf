# Secure Web Hosting on AWS: S3, CloudFront & WAF

<!-- Row 1: Status - Most Important -->
[![Release](https://img.shields.io/github/v/release/subhamay-bhattacharyya/aws-secure-website-oac-waf?label=Release)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/releases)&nbsp;[![Release Workflow](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/actions/workflows/release.yaml)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya/aws-secure-website-oac-waf)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya/aws-secure-website-oac-waf)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/badge/Languages-Python%20%7C%20YAML-blue)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya/aws-secure-website-oac-waf)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazonaws&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya/aws-secure-website-oac-waf)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya/aws-secure-website-oac-waf)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya/aws-secure-website-oac-waf)](https://github.com/subhamay-bhattacharyya/aws-secure-website-oac-waf/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/5c33f0ba73bbfc669ffff84c6999f63e/raw/aws-secure-website-oac-waf.json)](https://gist.github.com/subhamay-bhattacharyya/5c33f0ba73bbfc669ffff84c6999f63e)

This repository deploys a static website ("Foody Woody") to a secure, versioned S3 bucket on AWS using CloudFormation. The root stack in `cloudformation/template.yaml` provisions the bucket through a nested stack. CloudFront with Origin Access Control (OAC), the bucket policy, and AWS WAF are still in progress (see [Roadmap](#roadmap)).

## Repository Layout

```text
cloudformation/
├── template.yaml          # Root stack: invokes the nested S3 bucket stack
├── parameters.json        # Parameter values (devl)
└── stack-config.json      # Stack name, template, and parameter file used by CI
website/
├── index.html             # Single-page site: Home, About, Services, Menu, Contact
├── web.css                # Styles (light and dark themes)
└── web.js                 # Mobile menu, active section link, scroll-to-top, theme toggle
.env/environments.yaml     # Maps ci and devl to the AWS-SCS-C03-SANDBOX environment in us-east-1
.github/workflows/         # CI, environment setup, branch creation, Claude integration
```

## Root Stack (`cloudformation/template.yaml`)

The root stack creates a single `AWS::CloudFormation::Stack` resource, `S3BucketNestedStack`. It loads the nested template from:

```text
https://<NestedStacksS3BucketName>.s3.us-east-1.amazonaws.com/cfn-nested-aws-s3-bucket/template.yaml
```

The nested template is not part of this repository. Its stated purpose, per the root template metadata, is to create a versioned, encrypted S3 bucket with public access blocked and optional logging and website configuration. The root stack always passes `WebsiteConfiguration: "true"`.

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `NestedStacksS3BucketName` | `subhamay-cfn-templates-bucket-270453428528-us-east-1` | S3 bucket that holds the nested template |
| `ProjectName` | `proj-ztc` | Bucket name prefix. Lowercase letters, numbers, and hyphens; max 20 characters |
| `BucketBaseName` | `foody-woody` | Base name for the bucket. Lowercase letters, numbers, hyphens, and dots; max 20 characters |
| `Environment` | `devl` | Deployment environment label |
| `KmsKey` | `SB-KMS` | KMS key for encryption. Accepts a key name, an `alias/` name, or a key ARN. Leave empty for no KMS encryption |
| `EnableBucketKey` | `false` | `true` or `false`. Enables S3 Bucket Keys to reduce KMS cost; applies only when `KmsKey` is set |
| `CiSuffix` | `""` | Optional suffix appended to the bucket name, used for CI deployments |

### Bucket Naming

The nested stack builds the bucket name from the parameters:

```text
{ProjectName}-{BucketBaseName}-{AccountId}-{Environment}-{Region}[-{CiSuffix}]
```

Example: `proj-ztc-foody-woody-123456789012-devl-us-east-1`. The exact pattern is set by the nested template, so confirm it against the deployed bucket name.

### Outputs

Outputs are exported with the stack name as a prefix, so other stacks can import them.

| Output | Export name | Value |
|--------|-------------|-------|
| `NestedStackId` | `<stack-name>-NestedStackId` | ID of the nested stack |
| `S3BucketName` | `<stack-name>-BucketName` | Name of the S3 bucket |
| `S3BucketArn` | `<stack-name>-BucketArn` | ARN of the S3 bucket |
| `NestedStackOutputs` | — | Bucket name and ARN as a text block |

## Website (`website/`)

The site is a static, single-page "Foody Woody" restaurant landing page with these features:

- Sections for Home, About, Services, Menu of the week, and Contact
- Responsive navigation with a mobile menu toggle
- Active navigation link that tracks the section in view
- Scroll-to-top button
- Light and dark theme toggle, saved in `localStorage`

The template does not yet upload the `website/` files to the bucket or serve them through CloudFront.

## Deploying

The root stack has no IAM resources, so no `--capabilities` flag is needed. Deploy with the default parameters, or override any of them:

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name aws-secure-website-oac-waf-stack \
  --region us-east-1 \
  --parameter-overrides \
    ProjectName=proj-ztc \
    BucketBaseName=foody-woody \
    Environment=devl \
    KmsKey=SB-KMS \
    EnableBucketKey=false \
    CiSuffix=""
```

Read the bucket name from the stack outputs after the deploy finishes:

```bash
aws cloudformation describe-stacks \
  --stack-name aws-secure-website-oac-waf-stack \
  --query 'Stacks[0].Outputs[?OutputKey==`S3BucketName`].OutputValue' \
  --output text
```

## CI/CD Workflows

- **`ci.yaml`** runs only when started manually (`workflow_dispatch`). It reads `cloudformation/stack-config.json`, then calls a reusable workflow in the `ci` environment to deploy the stack. On `main`, it also generates the changelog and creates a GitHub release. The push and pull request triggers are commented out.
- **`setup-environments.yaml`** runs manually to set up the AWS environments.
- **`create-branch.yaml`** creates a `CFN-` prefixed feature branch when an issue is assigned.
- **`claude.yaml`** and **`claude-code-review.yaml`** respond to `@claude` mentions and review pull requests.

## Roadmap

Completed:

- [x] AWS S3

Pending:

- [ ] AWS CloudFront (with Origin Access Control)
- [ ] AWS Bucket Policy
- [ ] AWS WAF
- [ ] AWS CloudWatch
- [ ] AWS CloudTrail

## Best Practices Implemented

- Versioning enabled on the bucket (per the nested template's stated purpose)
- Public access blocked on the bucket (per the nested template's stated purpose)
- KMS encryption with an optional S3 Bucket Key
- Bucket names scoped by project, account, environment, and region
- Outputs exported for cross-stack references

## License

MIT
