<div align="center">
    <img src="./images/coderco.jpg" alt="CoderCo" width="300"/>
</div>


# URL Shortener — CoderCo DevOps Project 🚀

In this project, you will build and deploy a production-ready URL shortener service on AWS — the type of project you could proudly put in your portfolio or even run in a real environment.

The service takes a long URL and returns a shorter, unique code. Users can then access the short link and be redirected to the original URL. For example:

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

Everything else is on you. Read the AWS docs pls, iterate and commit small. Good luck!
