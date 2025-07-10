# VPC Implementation — NAT Gateway Full Resilience Design

## Overview

This project implements a highly resilient Virtual Private Cloud (VPC) design using multiple NAT Gateways (NATGWs) to enable secure outbound internet access for private subnets, following AWS best practices.

### VPC Structure

- **CIDR Block**: `10.16.0.0/16`
- **Availability Zones (AZs)**: A, B, and C

#### Subnets per AZ

| AZ | DB            | APP           | WEB            | Reserved    |
|-----|---------------|---------------|---------------|-------------|
| A   | 10.16.0.0/20  | 10.16.32.0/20 | 10.16.48.0/20 | 10.16.16.0/20 |
| B   | 10.16.64.0/20 | 10.16.96.0/20 | 10.16.112.0/20| 10.16.80.0/20 |
| C   | 10.16.128.0/20| 10.16.160.0/20| 10.16.176.0/20| 10.16.144.0/20|

---

## Key Components

- **NAT Gateways**: Placed in WEB subnets of each AZ, enabling outbound internet access for private subnets (APP, DB, Reserved).
- **Elastic IPs**: Assigned to each NAT Gateway to support public communication.
- **Route Tables**: Custom route tables for each AZ, configured to route `0.0.0.0/0` traffic through that AZ’s NAT Gateway.
- **Internet Gateway (IGW)**: Connected to the VPC; only WEB subnets use it directly.

---

## Implementation Steps

### 1. Create VPC, Subnets, and Route Tables

- Created using AWS CloudFormation.
- Defined a VPC with `/16` CIDR block.
- Subnets created for each AZ and each tier (DB, APP, WEB, Reserved).
- All subnets initially associated with a main route table.

### 2. Deploy NAT Gateways

- Created a NAT Gateway in each AZ’s WEB subnet.
- Assigned an Elastic IP to each NAT Gateway to enable outbound traffic.

### 3. Configure Custom Route Tables

- Created three custom route tables, one for each AZ.
- Each route table configured with a default route (`0.0.0.0/0`) pointing to that AZ’s NAT Gateway.

### 4. Update Subnet Associations

- Associated private subnets (APP, DB, Reserved) in AZ A with route table A.
- Associated private subnets in AZ B with route table B.
- Associated private subnets in AZ C with route table C.
- Ensures private subnets use their local NAT Gateway for outbound traffic, reducing latency and cross-AZ costs.

### 5. Validate Connectivity

- Verified that instances in private subnets could reach the internet via their respective NAT Gateways (e.g., for software updates).
- Confirmed that no inbound access is allowed, maintaining subnet privacy.

---

## Design Benefits

- **High resilience**: Each AZ has an independent NAT Gateway. Failure in one AZ does not affect others.
- **Cost optimization**: Avoids cross-AZ data transfer fees by using AZ-local NAT Gateways.
- **Clear separation of tiers**: Easier to scale and secure each subnet tier independently.
- **Secure outbound connectivity**: Private subnets can securely access the internet without exposing them directly.

---

## Conclusion

This implementation follows AWS architectural best practices for building a scalable and resilient network infrastructure. The use of per-AZ NAT Gateways and dedicated route tables ensures both high availability and cost efficiency.

---

### References

- [AWS NAT Gateway Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [AWS VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Scenario2.html)

---

