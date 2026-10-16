# Iron Core Gym — 3-tier deployment on AWS

A Vue 3 + Laravel gym management app, containerised with Docker and deployed on AWS
(eu-north-1) behind CloudFront, an Application Load Balancer and private subnets,
with RDS MySQL. Built and torn down as a hands-on cloud project (October 2026).

▶ Demo video: <!-- TODO: paste the unlisted YouTube or LinkedIn link -->

![Architecture](architecture/architecture.png)

## What it does
Visitors reach https://ahnafanan.site through Route 53 and CloudFront. Website files are
served from the CloudFront cache; `/api/*` and `/sanctum/*` bypass the cache and go to an
ALB, then to Docker containers on an EC2 instance with no public IP, then to RDS MySQL.
The full request path, step by step: [architecture/request-flow.md](architecture/request-flow.md)

## AWS services and how they're configured
| Layer | Service | Key configuration |
|---|---|---|
| DNS | Route 53 | Alias A/AAAA for apex and www → CloudFront; `origin` → ALB |
| Edge | CloudFront + WAF | 3 behaviors; `/api/*` CachingDisabled + AllViewer; HTTPS redirect |
| TLS | ACM | Certificates in us-east-1 (CloudFront) and eu-north-1 (ALB) |
| Load balancer | ALB | HTTPS:443 only from the CloudFront prefix list; health check `GET /` |
| Compute | EC2 t3.micro | Private subnet, no public IP, custom AMI, systemd boot script |
| Containers | Docker + ECR | `web` (nginx + Vue) proxies `/api` to `api` (Laravel) on a Docker network |
| Database | RDS MySQL | Private subnets, not public, port 3306 only from the EC2 security group |
| Outbound | NAT Gateway | Private instance reaches ECR, SSM and Parameter Store |
| Secrets | SSM Parameter Store | DB password as SecureString, read at boot via IAM role |
| Access | Session Manager | No SSH, no bastion, no open port 22 |

## Security layers
HTTPS end to end · WAF · ALB reachable only from CloudFront · private subnets ·
chained security groups (ALB → EC2 → RDS) · Session Manager instead of SSH ·
SecureString secrets · Sanctum trusted domains and secure cookies

## Problems I solved
- Admin login failed in production: missing APP_KEY and SANCTUM_STATEFUL_DOMAINS
- Session Manager "not connected": instance needed the IAM role and a NAT route
- Deploys showed stale pages: `index.html` set to no-cache, hashed assets cached 1 year

## Evidence
Screenshots and terminal output from the live deployment: [evidence/](evidence/)

## Code
- Frontend: https://github.com/ahnaf00/Iron-Core-Gym-Vue-JS/tree/aws-docker-deployment
- Backend: https://github.com/ahnaf00/Iron-Core-Gym-Laravel-Backend/tree/aws-docker-deployment

## Status
The AWS environment was deleted after the project to avoid costs. Teardown steps: [teardown.md](teardown.md)

## What I'd do next
Auto Scaling group across both AZs · Terraform for the whole stack · CI/CD with GitHub
Actions pushing to ECR · php-fpm instead of `artisan serve` · RDS Multi-AZ
