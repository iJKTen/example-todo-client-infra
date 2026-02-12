# example-todo-client-infra

AWS CloudFormation infrastructure for [todo.jaik.me](https://todo.jaik.me) -- an S3-hosted static site served via CloudFront with a CI/CD pipeline.

## Architecture

```
GitHub (iJKTen/example-todo-client)
  -> CodePipeline (Source)
  -> CodeBuild Build (npm install & build)
  -> CodeBuild Deploy (s3 sync with per-file cache headers + CloudFront invalidation)
  -> CloudFront (CDN + HTTPS + OAC)
  -> Route53 (todo.jaik.me)
```

## Environments

The stack is parameterized for three environments:

| Environment | Branch | Domain |
|---|---|---|
| dev | `dev` | `dev.todo.jaik.me` |
| staging | `staging` | `staging.todo.jaik.me` |
| prod | `main` | `todo.jaik.me` |

## Resources

| Resource | Purpose |
|---|---|
| **S3WebsiteBucket** | Hosts the built static site files |
| **PipelineArtifactBucket** | Stores CI/CD pipeline artifacts (auto-expires after 7 days) |
| **CloudFrontDistribution** | CDN with OAC, custom domain, HTTPS, and SPA error handling |
| **CloudFrontOAC** | Origin Access Control for secure S3 access via SigV4 |
| **TodoSitePipeline** | CodePipeline that builds and deploys on push to the configured branch |
| **BuildProject** | CodeBuild project (Node.js 20, runs `npm run build`) |
| **DeployProject** | CodeBuild project that syncs assets to S3 with cache headers and invalidates CloudFront |
| **DomainRecord** | Route53 A record aliasing to CloudFront |

## Prerequisites

This stack imports values from other stacks:

- `GlobalResourcesStack-GitHubConnectionArn` -- CodeStar connection to GitHub
- `Projects-CertificateArn` -- ACM certificate for `todo.jaik.me`
- `Todo-HostedZoneId` -- Route53 hosted zone ID

## Files

- `infra.yaml` -- Main CloudFormation template
- `deployment-dev.yaml` -- Deployment config for the dev environment
- `deployment-staging.yaml` -- Deployment config for the staging environment
- `deployment-prod.yaml` -- Deployment config for the prod environment
