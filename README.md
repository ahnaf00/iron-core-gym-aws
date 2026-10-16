# Iron Core Gym — 3-Tier Deployment on AWS

![AWS](https://img.shields.io/badge/AWS-eu--north--1-232F3E?logo=amazonwebservices&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-containerised-2496ED?logo=docker&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue_3-frontend-4FC08D?logo=vuedotjs&logoColor=white)
![Laravel](https://img.shields.io/badge/Laravel-backend-FF2D20?logo=laravel&logoColor=white)
![MySQL](https://img.shields.io/badge/RDS_MySQL-database-4479A1?logo=mysql&logoColor=white)

A full-stack gym management application (Vue 3 + Laravel) containerised with Docker and
deployed on AWS as a secure, production-style 3-tier architecture: **CloudFront edge →
Application Load Balancer → private EC2 → private RDS MySQL**. Served live at
`https://ahnafanan.site` during the project (October 2026), then torn down to avoid costs.

▶ **Demo video:** [media/presentation.mp4](media/presentation.mp4)

---

## Table of contents
- [Project summary](#project-summary)
- [Architecture](#architecture)
- [AWS services and configuration](#aws-services-and-configuration)
- [Security layers](#security-layers)
- [Screenshots](#screenshots)
  - [Live application](#1-live-application)
  - [DNS, TLS and edge](#2-dns-tls-and-edge)
  - [Load balancer](#3-load-balancer)
  - [Networking](#4-networking)
  - [Compute and containers](#5-compute-and-containers)
  - [Database](#6-database)
  - [End-to-end verification](#7-end-to-end-verification)
- [Problems solved](#problems-solved)
- [Repository layout](#repository-layout)
- [Source code](#source-code)
- [Status and teardown](#status-and-teardown)
- [Next steps](#next-steps)

---

## Project summary

**What was built**

- **Containerised the app.** The Vue frontend is built and served by nginx in a `web`
  container; the Laravel API runs in an `api` container. Both images are pushed to
  **Amazon ECR**.
- **Designed a custom VPC** (`10.0.0.0/16`) across two Availability Zones with public and
  private subnets, an Internet Gateway, and a NAT Gateway for outbound-only traffic.
- **Ran the app on a private EC2 instance** (no public IP) from a custom AMI. A systemd
  boot script reads secrets from **SSM Parameter Store** and starts the containers.
- **Put a managed database behind it.** RDS MySQL lives in private subnets and accepts
  connections only from the EC2 security group.
- **Fronted everything with CloudFront + WAF.** Static pages are cached at the edge;
  `/api/*` and `/sanctum/*` bypass the cache and go to an HTTPS-only ALB.
- **Secured the domain** with Route 53 and ACM certificates (us-east-1 for CloudFront,
  eu-north-1 for the ALB).
- **Removed SSH entirely.** Administration is done through **Session Manager** — no
  bastion host and no open port 22.

---

## Architecture

![Iron Core Gym AWS architecture](architecture/architecture.png)

**Request flow in brief**

1. The browser resolves `ahnafanan.site` via **Route 53** to a CloudFront edge.
2. **CloudFront** terminates HTTPS, runs **WAF**, and serves cached website files.
3. API calls and cache misses go over HTTPS to the **ALB**, which only accepts traffic from
   CloudFront's IP prefix list.
4. The ALB forwards HTTP:80 to the **EC2 instance** in a private subnet.
5. The `web` container serves the SPA and proxies `/api` to the `api` container (Laravel).
6. Laravel queries **RDS MySQL** on port 3306 inside the private subnets.

Outbound traffic from the instance (ECR image pulls, Parameter Store, Session Manager)
leaves through the NAT Gateway — nothing can open a connection inward.
Full step-by-step walkthrough: [architecture/request-flow.md](architecture/request-flow.md)

---

## AWS services and configuration

| Layer | Service | Key configuration |
|---|---|---|
| DNS | Route 53 | Alias A/AAAA for apex and `www` → CloudFront; `origin` → ALB |
| Edge | CloudFront + WAF | 3 behaviors; `/api/*` CachingDisabled + AllViewer; HTTP → HTTPS redirect |
| TLS | ACM | Certificates in us-east-1 (CloudFront) and eu-north-1 (ALB) |
| Load balancer | ALB | HTTPS:443 only from the CloudFront prefix list; health check `GET /` |
| Compute | EC2 t3.micro | Private subnet, no public IP, custom AMI, systemd boot script, IMDSv2 |
| Containers | Docker + ECR | `web` (nginx + Vue) proxies `/api` to `api` (Laravel) on a Docker network |
| Database | RDS MySQL | db.t4g.micro, private subnets, not public, 3306 only from the EC2 SG |
| Outbound | NAT Gateway | Private instance reaches ECR, SSM and Parameter Store |
| Secrets | SSM Parameter Store | DB password as SecureString, read at boot via IAM role |
| Access | Session Manager | No SSH, no bastion, no open port 22 |

---

## Security layers

| Layer | Control |
|---|---|
| Transport | HTTPS end to end (viewer → CloudFront → ALB), HTTP redirected to HTTPS |
| Edge | AWS WAF core protections on the CloudFront distribution |
| Origin lock-down | ALB security group accepts 443 only from the CloudFront managed prefix list |
| Network | EC2 and RDS in private subnets with no public IPs |
| Least privilege | Chained security groups: ALB → EC2 (port 80) → RDS (port 3306) |
| Access | Session Manager with IAM authentication instead of SSH |
| Secrets | SecureString parameters in SSM, fetched at boot via the instance IAM role |
| Application | Sanctum trusted domains and secure cookies |

---

## Screenshots

All screenshots were captured from the live deployment. Account IDs, ARNs, endpoints and
credentials are blurred.

### 1. Live application

<table>
  <tr>
    <td width="50%"><img src="media/01-website-home.png" alt="Website home page"></td>
    <td width="50%"><img src="media/02-website-class-schedule.png" alt="Class schedule page"></td>
  </tr>
  <tr>
    <td><b>Public website.</b> The Vue frontend served over HTTPS at <code>ahnafanan.site</code> through CloudFront.</td>
    <td><b>Class schedule.</b> Data loaded from the Laravel API and RDS via the uncached <code>/api/*</code> path.</td>
  </tr>
  <tr>
    <td><img src="media/03-admin-login-redacted.png" alt="Admin login"></td>
    <td><img src="media/04-admin-dashboard.png" alt="Admin dashboard"></td>
  </tr>
  <tr>
    <td><b>Admin login.</b> Sanctum cookie-based authentication working in production (credentials blurred).</td>
    <td><b>Admin dashboard.</b> Live member, trainer, class and message counts read from the database.</td>
  </tr>
  <tr>
    <td><img src="media/05-admin-members.png" alt="Admin members"></td>
    <td><img src="media/06-admin-trainers.png" alt="Admin trainers"></td>
  </tr>
  <tr>
    <td><b>Members management.</b> Search, filter and CRUD operations against RDS MySQL.</td>
    <td><b>Trainers management.</b> Trainer records with specialty, experience and status.</td>
  </tr>
  <tr>
    <td><img src="media/07-admin-pricing.png" alt="Admin pricing"></td>
    <td></td>
  </tr>
  <tr>
    <td><b>Pricing plans.</b> Basic, Premium and Elite plans managed from the admin panel.</td>
    <td></td>
  </tr>
</table>

### 2. DNS, TLS and edge

<table>
  <tr>
    <td width="50%"><img src="media/08-route53-hosted-zone-records.png" alt="Route 53 records"></td>
    <td width="50%"><img src="media/09-acm-certificate-issued.png" alt="ACM certificate"></td>
  </tr>
  <tr>
    <td><b>Route 53 hosted zone.</b> Alias A/AAAA records for the apex and <code>www</code> pointing to CloudFront, and an <code>origin</code> record pointing to the ALB.</td>
    <td><b>ACM certificate.</b> Issued for <code>ahnafanan.site</code> and <code>*.ahnafanan.site</code>, validated via DNS.</td>
  </tr>
  <tr>
    <td><img src="media/10-cloudfront-distribution-list.png" alt="CloudFront distribution"></td>
    <td><img src="media/11-cloudfront-behaviors.png" alt="CloudFront behaviors"></td>
  </tr>
  <tr>
    <td><b>CloudFront distribution.</b> <code>iron-core-cf</code> enabled with the custom domains and <code>origin.ahnafanan.site</code> as origin.</td>
    <td><b>Cache behaviors.</b> <code>/api/*</code> and <code>/sanctum/*</code> use CachingDisabled + AllViewer; the default behavior uses CachingOptimized. All redirect HTTP to HTTPS.</td>
  </tr>
  <tr>
    <td><img src="media/12-cloudfront-waf-security.png" alt="CloudFront WAF"></td>
    <td></td>
  </tr>
  <tr>
    <td><b>AWS WAF.</b> Core protections enabled on the distribution, with request metrics for the last 24 hours.</td>
    <td></td>
  </tr>
</table>

### 3. Load balancer

<table>
  <tr>
    <td width="50%"><img src="media/13-alb-listeners-and-rules.png" alt="ALB listeners"></td>
    <td width="50%"><img src="media/14-alb-security-group.png" alt="ALB security group"></td>
  </tr>
  <tr>
    <td><b>ALB listeners.</b> <code>iron-core-alb</code> spans eu-north-1a and 1b. HTTPS:443 forwards to <code>iron-core-tg</code> using a TLS 1.3 policy; HTTP:80 redirects to HTTPS (and is unreachable because the SG only allows 443).</td>
    <td><b>ALB security group.</b> A single inbound rule: HTTPS 443 from the CloudFront managed prefix list, so the origin cannot be reached directly.</td>
  </tr>
  <tr>
    <td><img src="media/15-target-group-health.png" alt="Target group health"></td>
    <td></td>
  </tr>
  <tr>
    <td><b>Target group health.</b> The private EC2 instance registered on port 80 and reporting <i>Healthy</i>.</td>
    <td></td>
  </tr>
</table>

### 4. Networking

<table>
  <tr>
    <td width="50%"><img src="media/16-vpc-details.png" alt="VPC details"></td>
    <td width="50%"><img src="media/17-vpc-resource-map.png" alt="VPC resource map"></td>
  </tr>
  <tr>
    <td><b>VPC.</b> <code>project-vpc</code> with CIDR <code>10.0.0.0/16</code>, DNS resolution and hostnames enabled.</td>
    <td><b>Resource map.</b> Four subnets (public + private in each AZ), separate route tables, the Internet Gateway <code>project-igw</code> and NAT Gateway <code>lab-nat</code>.</td>
  </tr>
  <tr>
    <td><img src="media/18-nat-gateway.png" alt="NAT gateway"></td>
    <td></td>
  </tr>
  <tr>
    <td><b>NAT Gateway.</b> Public NAT in <code>project-subnet-public1-eu-north-1a</code>, giving the private instance outbound-only internet access.</td>
    <td></td>
  </tr>
</table>

### 5. Compute and containers

<table>
  <tr>
    <td width="50%"><img src="media/19-ec2-instance-private.png" alt="EC2 private instance"></td>
    <td width="50%"><img src="media/20-ec2-instance-details.png" alt="EC2 instance details"></td>
  </tr>
  <tr>
    <td><b>Private EC2 instance.</b> <code>iron-core-app-private</code> (t3.micro) with <b>no public IPv4 address</b> and no public DNS — only a private IP (<code>10.0.130.106</code>).</td>
    <td><b>Instance configuration.</b> Running in <code>project-subnet-private1-eu-north-1a</code> with IAM role <code>ahnaf-ec2-role</code> and IMDSv2 required.</td>
  </tr>
  <tr>
    <td><img src="media/21-session-manager-connect.png" alt="Session Manager connect"></td>
    <td><img src="media/22-session-manager-docker-ps.png" alt="docker ps over Session Manager"></td>
  </tr>
  <tr>
    <td><b>Session Manager.</b> SSM agent online and connected through IAM authentication — no SSH keys, bastion or open inbound ports.</td>
    <td><b>Running containers.</b> <code>docker ps</code> over Session Manager shows the <code>web</code> (port 80) and <code>api</code> (port 8000) containers up.</td>
  </tr>
  <tr>
    <td><img src="media/23-ecr-repositories.png" alt="ECR repositories"></td>
    <td></td>
  </tr>
  <tr>
    <td><b>Amazon ECR.</b> Private <code>web</code> and <code>api</code> repositories with AES-256 encryption.</td>
    <td></td>
  </tr>
</table>

### 6. Database

<table>
  <tr>
    <td width="50%"><img src="media/24-rds-summary.png" alt="RDS summary"></td>
    <td width="50%"><img src="media/25-rds-connectivity.png" alt="RDS connectivity"></td>
  </tr>
  <tr>
    <td><b>RDS summary.</b> <code>lab-db</code> running MySQL Community on db.t4g.micro in eu-north-1a, database <code>appdb</code> on port 3306.</td>
    <td><b>Connectivity.</b> Placed in <code>project-vpc</code> via subnet group <code>lab-db-subnets</code>, <b>Publicly accessible: No</b>.</td>
  </tr>
  <tr>
    <td><img src="media/26-rds-security-group-rules.png" alt="RDS security group rules"></td>
    <td></td>
  </tr>
  <tr>
    <td><b>Security group rules.</b> <code>ahnaf-rds-sg</code> allows inbound MySQL only from the EC2 security group.</td>
    <td></td>
  </tr>
</table>

### 7. End-to-end verification

<table>
  <tr>
    <td width="50%"><img src="media/27-curl-https-test.png" alt="curl HTTPS test"></td>
    <td width="50%"></td>
  </tr>
  <tr>
    <td><b>HTTPS check from outside AWS.</b> <code>curl -I https://ahnafanan.site</code> returns <code>200 OK</code> from nginx via CloudFront (<code>X-Cache</code>, <code>Via</code>, <code>X-Amz-Cf-Pop</code> headers) with <code>Cache-Control: no-cache</code> on <code>index.html</code>.</td>
    <td></td>
  </tr>
</table>

More terminal output and ECR listings: [evidence/](evidence/)

---

## Problems solved

| Problem | Root cause | Fix |
|---|---|---|
| Admin login failed in production | Missing `APP_KEY` and `SANCTUM_STATEFUL_DOMAINS` | Added both to the production environment |
| Session Manager showed "not connected" | Instance lacked the IAM role and a route out | Attached the SSM-enabled IAM role and routed the private subnet through the NAT Gateway |
| Deploys showed stale pages | `index.html` was cached at the edge | Served `index.html` as no-cache; hashed assets cached for 1 year |

---

## Repository layout

```
.
├── architecture/
│   ├── architecture.drawio               # editable diagram source
│   ├── architecture.png                  # rendered diagram
│   └── request-flow.md                   # step-by-step request path
├── evidence/
│   ├── README.md                         # index of the evidence
│   ├── config/
│   │   ├── ecr-repositories.json         # ECR repository listing
│   │   ├── ecr-images-web.json           # images in the web repository
│   │   └── ecr-images-api.json           # images in the api repository
│   └── terminal/
│       └── laptop-checks.txt             # public HTTPS/DNS/origin checks from outside AWS
├── media/
├── teardown.md                           # how the environment was deleted
└── README.md
```

---

## Source code

- **Frontend (Vue 3):** https://github.com/ahnaf00/Iron-Core-Gym-Vue-JS/tree/aws-docker-deployment
- **Backend (Laravel):** https://github.com/ahnaf00/Iron-Core-Gym-Laravel-Backend/tree/aws-docker-deployment

---

## Status and teardown

The AWS environment was deleted after the project to avoid ongoing costs.
Teardown steps and verification: [teardown.md](teardown.md)

---

## Next steps

- Auto Scaling group across both Availability Zones
- Infrastructure as code with Terraform
- CI/CD with GitHub Actions building and pushing images to ECR
- php-fpm instead of `artisan serve` for the API container
- RDS Multi-AZ for database high availability
