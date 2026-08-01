# AWS Cloud Networking & DevOps Projects

A two-part hands-on AWS portfolio covering core networking fundamentals and production-style application delivery — built from scratch in the AWS Console, with every design decision, security control, and troubleshooting step documented.

| | Assignment 1 | Assignment 2 |
|---|---|---|
| **Focus** | VPC & Networking | Application Load Balancer |
| **Core service** | VPC, IGW, NAT Gateway, Route Tables | EC2, ALB, Target Groups, Route 53 |
| **Bonus completed** | ✅ Bastion Host, ✅ CloudWatch Monitoring | ✅ Custom domain via Route 53 |


---

## Table of Contents

- [Project 1 — VPC & Networking](#project-1--vpc--networking)
- [Project 2 — Application Load Balancer](#project-2--application-load-balancer)
- [Skills Demonstrated](#skills-demonstrated)
- [Repository Contents](#repository-contents)

---

## Project 1 — VPC & Networking

**Objective:** Design and build a fully segmented custom VPC from the ground up — public and private subnets, correct internet routing for each tier, and EC2 instances deployed across both — establishing the networking foundation that Project 2 builds on.

### What was built

- A custom VPC (`10.0.0.0/16`) with one **public** and one **private** subnet, each in its own `/24` block.
- An **Internet Gateway** attached to the VPC for the public subnet, and a **NAT Gateway** (backed by an Elastic IP) in the public subnet so the private subnet gets outbound-only internet access.
- Two separate **route tables** — the public one defaulting to the IGW, the private one defaulting to the NAT Gateway — so each tier's traffic is routed correctly and in isolation.
- A **public EC2 instance** with a public IP, reachable via SSH/HTTP only from the administrator's IP.
- A **private EC2 instance** with no public IP at all, reachable only via the bastion host's security group — referenced by security-group ID rather than a static IP, so the rule stays valid even if the bastion's address changes.

### Bonus objectives (both completed)

- **Bastion Host**: a dedicated jump box in the public subnet, providing the only path into the private instance. Access uses **SSH agent forwarding** rather than copying the private key onto the bastion, so the key never leaves the local machine.
- **CloudWatch Monitoring**: detailed monitoring enabled across all four running instances (public EC2, private EC2, bastion, and an additional server).

### Notable troubleshooting

- **CIDR overlap**: an early attempt to reuse `10.0.0.0/24` for the private subnet was rejected, since the /16 VPC's first two octets are fixed but the third octet must be unique per subnet — corrected to a distinct, non-overlapping `10.0.7.0/24` block.
- **HTTP "connection refused" vs. timeout**: a refused connection (rather than a timeout) was used to correctly diagnose the issue as application-layer, not networking — confirmed with `netcat` that no process was listening on port 80, ruling out the route tables, security groups, and NACLs before ever touching the network configuration.

---

## Project 2 — Application Load Balancer

**Objective:** Deploy two EC2 web servers across separate Availability Zones behind an Application Load Balancer, so all traffic reaches a single highly-available entry point and neither instance is ever directly reachable from the internet.

### What was built

- Two EC2 instances (Amazon Linux, NGINX) in **separate Availability Zones**, each returning a distinct response so load balancing could be visually verified.
- An **internet-facing Application Load Balancer** spanning both public subnets, with an HTTP:80 listener forwarding to a health-checked target group.
- **Security group isolation**: the ALB accepts HTTP from anywhere, while each EC2 instance accepts HTTP *only* from the ALB's security group — direct public access was removed once health checks passed.
- Verified round-robin distribution across both instances, and confirmed both targets reporting **healthy**.

### Bonus objective (completed)

- **Custom domain via Route 53**: a public hosted zone with an alias A record pointing the domain's root at the ALB — including working around a registrar limitation (Cloudflare-registered domains can't repoint their nameservers without a full transfer) by registering the domain directly through Route 53 instead.

### Notable troubleshooting

- A **missing route to the internet gateway** on a newly created subnet silently broke the EC2 user-data script on first boot, since the instance had no path to the package repositories — traced via the VPC resource map and cloud-init logs.
- **Unhealthy targets** after locking down the security groups were root-caused to a **missing outbound rule on the ALB's own security group** — confirmed step-by-step using `curl` (server itself was healthy), NGINX access logs (ALB traffic was never arriving), and the ALB's resource map, before finding the fix.
- A final DNS mystery — the custom domain timing out in-browser despite `dig` resolving correctly — turned out to be the browser defaulting to **HTTPS** against an ALB with only an HTTP listener configured.

---

## Skills Demonstrated

- **Core networking**: VPC design, CIDR planning, subnetting, route tables, Internet/NAT gateways
- **High availability**: multi-AZ deployment, Application Load Balancer, target groups, health checks
- **Security**: least-privilege security groups, security-group-to-security-group referencing, bastion-host access patterns, SSH agent forwarding
- **DNS**: Route 53 hosted zones, alias records, registrar/nameserver constraints
- **Observability**: CloudWatch monitoring, NGINX access log analysis
- **Systematic troubleshooting**: isolating network-layer vs. application-layer failures, tracing traffic path-by-path using logs, resource maps, and command-line tools (`curl`, `dig`, `netcat`)

---

## Repository Contents

```
.
├── README.md
├── Asg_1         # Assignment 1 full report — VPC & Networking
└── Asg_2     # Assignment 2 full report — Application Load Balancer
```

Each report includes the full architecture diagram, annotated console screenshots for every step, and a written narrative covering both the implementation and the troubleshooting process.
