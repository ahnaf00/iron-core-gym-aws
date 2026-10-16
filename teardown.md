# Teardown

The AWS environment was deleted after the project to avoid costs.
Region: eu-north-1 (Stockholm), plus us-east-1 for CloudFront's certificate.

Torn down on: <!-- TODO: date -->

## Day 1
1. Route 53: deleted the A/AAAA records for the apex, `www` and `origin`.
2. CloudFront: disabled `iron-core-cf` and cancelled its Free flat-rate plan (a distribution on a plan can only be deleted once the billing cycle ends).
3. Load balancer: deleted `iron-core-alb`, then target group `iron-core-tg`.
4. EC2: terminated the app instance and `lab-golden`.
5. AMI: deregistered the custom AMI, deleted its snapshot and the key pair.
6. RDS: deleted `lab-db` without a final snapshot, then the DB subnet group.
7. NAT Gateway `lab-nat` deleted, then its Elastic IP released.
8. ACM (eu-north-1): deleted the ALB certificate.
9. ECR: deleted repositories `web` and `api` with their images.
10. Parameter Store `/lab/db/*` parameters and IAM role `ahnaf-ec2-role` deleted.
11. VPC `project-vpc` deleted with its subnets, route tables, internet gateway and security groups.
12. CloudWatch log groups deleted; unused CLI access keys deactivated.

## After the plan's billing cycle ended
13. CloudFront distribution deleted.
14. The WAF web ACL created by the Free plan deleted.
15. ACM certificate in us-east-1 and its validation CNAME deleted.
16. Hosted zone: <!-- TODO: kept for the case-study site, or deleted -->

## Verification
- Billing and Cost Management → Bills, checked on the day and two days later.
- EC2 dashboard in eu-north-1 and us-east-1: all counts 0, no Elastic IPs or volumes.
- Resource Explorer: nothing named `iron`, `lab` or `project` left.
