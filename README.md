<div align="center">
    <img src="./images/coderco.jpg" alt="CoderCo" width="300"/>
</div>


# URL Shortener — CoderCo DevOps Project 🚀

You’re shipping a production-ish service on AWS:

- **ECS Fargate** service behind **ALB** (+ **WAF**)
- **Blue/Green** deployments using **CodeDeploy** (canary or blue-green)
- **DynamoDB** for storage (PAY_PER_REQUEST, PITR on)
- **ECR** for container images
- **VPC** with **private subnets** + **VPC Endpoints** (no NAT gateways)
- **GitHub Actions** with **OIDC** (no static AWS keys)

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
   - `POST /shorten {"url":"https://example.com"}` ➜ returns `short`
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
