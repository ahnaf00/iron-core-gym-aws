# Request flow

How one request to https://ahnafanan.site travels through the stack.

1. **DNS.** The browser asks Route 53 for `ahnafanan.site`. An alias A/AAAA record returns CloudFront edge IPs.
2. **Edge TLS.** The browser opens HTTPS to the nearest CloudFront edge, using the ACM certificate from us-east-1. Plain HTTP gets a 301 to HTTPS.
3. **WAF.** AWS WAF inspects the request at the edge.
4. **Cache or forward.** Website files are served from the edge cache when present (`index.html` is no-cache, hashed assets are cached for a year). `/api/*` and `/sanctum/*` are never cached and forward all viewer headers, cookies and the Host header.
5. **Origin.** On a miss or an API call, CloudFront connects over HTTPS to `origin.ahnafanan.site`, the ALB, which uses an ACM certificate from eu-north-1.
6. **ALB security group.** The ALB accepts port 443 only from the AWS-managed CloudFront origin-facing prefix list. Direct requests to the origin time out.
7. **Private instance.** The ALB forwards HTTP:80 to the EC2 instance in a private subnet. The instance has no public IP, and its security group accepts port 80 only from the ALB.
8. **Containers.** The `web` container (nginx + the built Vue app) serves the SPA and proxies `/api`, `/sanctum` and `/storage` to the `api` container (Laravel, port 8000) over a shared Docker network.
9. **Database.** Laravel queries RDS MySQL in the private subnets on port 3306, which the RDS security group allows only from the EC2 security group. The response returns the same way.

**Outbound path.** The instance reaches ECR (image pulls), Parameter Store (secrets) and Systems Manager (Session Manager) through the NAT Gateway in a public subnet. Nothing can open a connection inward.
