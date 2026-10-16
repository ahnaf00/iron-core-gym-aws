# Evidence

Captured from the live deployment before teardown. Account IDs, emails, endpoints and secrets are redacted.

- `terminal/laptop-checks.txt` — public checks from outside AWS: HTTP→HTTPS redirect, CloudFront headers, the API returning JSON through `/api/*`, the origin timing out for direct access, DNS resolving to CloudFront.
- `config/` — ECR repository and image listings.
- [`../media/`](../media/) — screenshots from the live site and the AWS console (IDs, ARNs, endpoints and the account blurred), plus the presentation video:

- [01-website-home.png](../media/01-website-home.png)
- [02-website-class-schedule.png](../media/02-website-class-schedule.png)
- [03-admin-login-redacted.png](../media/03-admin-login-redacted.png)
- [04-admin-dashboard.png](../media/04-admin-dashboard.png)
- [05-admin-members.png](../media/05-admin-members.png)
- [06-admin-trainers.png](../media/06-admin-trainers.png)
- [07-admin-pricing.png](../media/07-admin-pricing.png)
- [08-route53-hosted-zone-records.png](../media/08-route53-hosted-zone-records.png)
- [09-acm-certificate-issued.png](../media/09-acm-certificate-issued.png)
- [10-cloudfront-distribution-list.png](../media/10-cloudfront-distribution-list.png)
- [11-cloudfront-behaviors.png](../media/11-cloudfront-behaviors.png)
- [12-cloudfront-waf-security.png](../media/12-cloudfront-waf-security.png)
- [13-alb-listeners-and-rules.png](../media/13-alb-listeners-and-rules.png)
- [14-alb-security-group.png](../media/14-alb-security-group.png)
- [15-target-group-health.png](../media/15-target-group-health.png)
- [16-vpc-details.png](../media/16-vpc-details.png)
- [17-vpc-resource-map.png](../media/17-vpc-resource-map.png)
- [18-nat-gateway.png](../media/18-nat-gateway.png)
- [19-ec2-instance-private.png](../media/19-ec2-instance-private.png)
- [20-ec2-instance-details.png](../media/20-ec2-instance-details.png)
- [21-session-manager-connect.png](../media/21-session-manager-connect.png)
- [22-session-manager-docker-ps.png](../media/22-session-manager-docker-ps.png)
- [23-ecr-repositories.png](../media/23-ecr-repositories.png)
- [24-rds-summary.png](../media/24-rds-summary.png)
- [25-rds-connectivity.png](../media/25-rds-connectivity.png)
- [26-rds-security-group-rules.png](../media/26-rds-security-group-rules.png)
- [27-curl-https-test.png](../media/27-curl-https-test.png)
