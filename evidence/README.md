# Evidence

Captured from the live deployment before teardown. Account IDs, emails, endpoints and secrets are redacted.

- `terminal/laptop-checks.txt` — public checks from outside AWS: HTTP→HTTPS redirect, CloudFront headers, the API returning JSON through `/api/*`, the origin timing out for direct access, DNS resolving to CloudFront.
- `config/` — ECR repository and image listings.
- `screenshots/` — console screenshots, numbered in the order they tell the story:

| # | File | Shows |
|---|---|---|
| 01 | site-home.png | The live site over HTTPS |
| 02 | site-admin.png | Admin dashboard after login |
| 03 | route53-records.png | Hosted zone records |
| 04 | acm-certs.png | Certificates in us-east-1 and eu-north-1, both Issued |
| 05 | cloudfront-general.png | Alternate domains, certificate, WAF |
| 06 | cloudfront-behaviors.png | `/api/*`, `/sanctum/*`, Default |
| 07 | cloudfront-origin.png | Origin `origin.ahnafanan.site`, HTTPS only |
| 08 | alb-listeners.png | HTTPS:443, HTTP:80 redirect |
| 09 | target-health.png | Target healthy |
| 10 | sg-alb.png | 443 from the CloudFront prefix list only |
| 11 | sg-ec2.png | 80 from the ALB security group only |
| 12 | sg-rds.png | 3306 from the EC2 security group only |
| 13 | vpc-resource-map.png | VPC resource map |
| 14 | nat.png | NAT Gateway and the private route table |
| 15 | ec2-instance.png | Private subnet, no public IPv4, IAM role |
| 16 | ssm-session.png | Session Manager running `sudo docker ps` |
| 17 | rds.png | Publicly accessible: No |
| 18 | ecr.png | Repositories `web` and `api` |
| 19 | parameter-store.png | `/lab/db/*` names only |
| 20 | iam-role.png | Instance role permissions |
