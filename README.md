<div align="center">
    <img src="./images/coderco.jpg" alt="CoderCo" width="300"/>
</div>

# URL Shortener - Infamous CoderCo ECS Project's Son

The CoderCo ECS Project has given birth to this new challenge.

For a long time, our community cut their teeth on the infamous CoderCo ECS project - a well-known rite of passage that sharpened skills in containerisation, Terraform and AWS deployment. It became a badge of honour for anyone completing our DevOps Roadmap.

Now, we’re raising the bar. This new and improved ECS project keeps the solid foundations of its predecessor but adds more production-ready features, security hardening and cost-efficient design patterns you’ll see in the real world.

You’re not just deploying an app here - you’re engineering an end-to-end, modern AWS workload with blue/green deployments, VPC-only connectivity, WAF protection and a CI/CD pipeline powered by GitHub OIDC.

It’s the next journey of our CoderCo tradition.

## Project Overview

In this project, you will build and deploy a production-ready URL shortener service on AWS - the type of project you could proudly put in your portfolio or even run in a real environment.

The service takes a long URL and returns a shorter, unique code. Users can then access the short link and be redirected to the original URL. For example:

```bash
POST /shorten  { "url": "https://example.com/my/very/long/path" }
→ { "short": "abc123ef", "url": "https://example.com/my/very/long/path" }

GET /abc123ef
→ HTTP 302 redirect to https://example.com/my/very/long/path
```

You’ll containerise the provided Python app, deploy it to AWS ECS Fargate behind an Application Load Balancer (ALB), and store the short-to-long URL mappings in Amazon DynamoDB.

Your infrastructure will be Terraform-managed and you will be shipping a production-ish service on AWS:

- **ECS Fargate** service behind **ALB** (+ **WAF**) - you could try this in Lambda but think about it carefully (if it makes sense or not)

- Runs inside private subnets (no public IPs)
- Accesses AWS services via VPC Endpoints (no NAT gateways)

- Has blue/green or canary deployments via AWS CodeDeploy for zero-downtime releases

- Is protected by AWS WAF rules on the ALB

- **DynamoDB** for storage (PAY_PER_REQUEST, PITR on)

- Uses GitHub Actions OIDC for CI/CD — no long-lived AWS keys or static creds please!

The Python app is provided. You must do **all infrastructure + CI/CD**.

## Rules

- No long-lived AWS keys in GitHub. Use **OIDC**.
- **Tasks must run in private subnets** with **VPC endpoints**. No NAT.
- Must implement **blue/green** with **automatic rollback on failed health checks**.
- **WAF** with the appropriate rules attached to ALB.
- Use **Terraform** (split modules + envs). State in **S3** with **DDB lock**.
- Cost-conscious. No unnecessary resources pls.


## ⚠️ IMPORTANT: Costs & Teardown

This project deploys real AWS infrastructure that will incur costs if left running.

As soon as you complete your deployment and take your screenshots/demos, tear down your resources:

```bash
cd infra/envs/dev && terraform destroy -auto-approve
```

Keep DynamoDB, ALB and WAF especially in mind - these run 24/7.

If you want to re-deploy later, your state is in the remote backend and can be re-applied.

💡 Tip: You can also test most of this locally using LocalStack for ECS, ECR, DynamoDB, S3 and CodeDeploy before going live on AWS. This can dramatically reduce costs during development. [LocalStack docs](https://docs.localstack.cloud/aws/getting-started/)

## Deliverables

1. Working service URL (ALB DNS or Route53) with:
   - `GET /healthz` ➜ `{"status":"ok"}`
   - `POST /shorten {"url":"https://coderco.io/shorten"}` ➜ returns `short`
   - `GET /{short}` ➜ HTTP 302 to original URL
2. GitHub Actions:
   - **CI**: build, unit tests, image scan (tool of your choice), push to ECR on `main`.
   - **CD**: terraform `plan` (PR) and `apply` (main) using OIDC; trigger CodeDeploy canary.
3. Evidence:
   - Screenshot of OIDC role trust policy
   - CodeDeploy deployment screen showing canary + rollback test
   - WAF associated to ALB
   - VPC Endpoints list (S3/DDB/ECR/logs/etc.)
4. Short README section: decisions + trade-offs (bullets).

## Minimal Guidance (do not ask for more 😉)

- Infra goes in `infra/` using provided folder layout.
- You must create `infra/global/backend` for Terraform state (S3+DDB) and run it once.
- Use two **target groups** (blue/green). Health check path: `/healthz`.
- App container port: **8080**. No public IPs.
- App needs env var: `TABLE_NAME`.

## Acceptance Criteria (we will check)

- No NAT gateways on the bill; tasks still pull from ECR and write logs.
- CodeDeploy canary shifts traffic and auto-rolls back if health checks fail.
- App IAM role limited to `dynamodb:GetItem/PutItem` on your table only.
- Execution role able to pull from ECR and write CloudWatch logs.
- GitHub workflow uses `id-token: write` and assumes your deploy role.

## Bonus (optional)

- HTTPS (ACM + 443 listener)
- Infracost/tfsec/Trivy in CI
- Route53 DNS + friendly hostname
- CloudWatch dashboard (p50/p95 latency, 5xx, healthy host count)

## Extra special & outside project scope (if you want to have a variation to your project)

1. App logic changes (you provide the base, they extend it)

  - Add an analytics endpoint /stats/{short} to count and return redirect hits (DDB update).

  - Add an expiry TTL for shortened links (DDB TTL attribute).

  - Add a /bulk-shorten endpoint to shorten multiple URLs in one request.

  - Store metadata (created_at, creator_ip) alongside the link.

2. AWS integrations (you can pick 2-3 extras or all, whatever is required.)

  - Push click events to SQS or Kinesis for later processing.

  - Store request logs in S3 via Firehose.

  - Use Parameter Store or Secrets Manager for TABLE_NAME instead of env var.

  - Add CloudFront in front of the ALB (bonus for caching).

3. Security patterns

  - Require an API key via API Gateway in front of the ALB.

  - Add IP rate limiting via WAF rules.

4. Monitoring patterns

  - Add CloudWatch dashboard (p50/p95 latency, 5xx, healthy host count)

Everything else is on you. Read the AWS docs pls, iterate and commit small. Good luck!
