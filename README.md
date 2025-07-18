# VPC Peering Setup for Cross-Service Finance Data Access

## Overview

This README describes the process of setting up VPC peering between three services, each residing in its own Amazon VPC (Virtual Private Cloud):

- **Service A**: Requires access to Finance Service data.
- **Service B**: Requires access to Finance Service data.
- **Finance Service**: Provides sensitive financial data and resides in a separate VPC.

The objective is to enable secure, private communication so that Service A and Service B can access data from the Finance Service, while all three services remain isolated in their individual VPCs.

---

## Architecture Diagram
<img width="973" height="371" alt="aws vpc peering" src="https://github.com/user-attachments/assets/7ff75ab5-a182-4df5-8566-ba68e6a2ec9c" />

- VPC A ↔ VPC Finance (peering connection) and accept the request
- VPC B ↔ VPC Finance (peering connection) and accept the request
- No direct peering between VPC A and VPC B
<img width="1892" height="960" alt="created with 3 vpcs for 3 different applications" src="https://github.com/user-attachments/assets/c79a994c-265c-482d-9d0d-43df3aa9a893" />

---

## Prerequisites

- AWS CLI configured, or access to AWS Console
- Permissions to create VPC peering connections, modify route tables, and update security groups
- VPC IDs, CIDR blocks, and service details for all three VPCs

---

## Steps

### 1. Create VPC Peering Connections

Assuming the following VPC IDs:
- VPC A: `vpc-AAAA1111`
- VPC B: `vpc-BBBB2222`
- VPC Finance: `vpc-FIN3333`
  
#### a. Peering: Service A ↔ Finance Service

```bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-AAAA1111 \
  --peer-vpc-id vpc-FIN3333 \
  --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=VPCPeering-A-Finance}]'
```
or in UI

<img width="1852" height="828" alt="creating peering group for marketing and finance" src="https://github.com/user-attachments/assets/cf7fc20a-ebfa-4e5e-880d-68b9dbafcce5" />
<img width="1621" height="891" alt="accept requests on both sides" src="https://github.com/user-attachments/assets/30e8c0df-c6cd-445b-b6d2-e341dc9ade0e" />

#### b. Peering: Service B ↔ Finance Service

```bash
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-BBBB2222 \
  --peer-vpc-id vpc-FIN3333 \
  --tag-specifications 'ResourceType=vpc-peering-connection,Tags=[{Key=Name,Value=VPCPeering-B-Finance}]'
```

- Accept the peering connection from the target VPC (Finance).

---

### 2. Update Route Tables

For each VPC, update the respective route tables to allow traffic to the peered VPC’s CIDR.

- In **VPC A**: Add route to Finance VPC CIDR via peering connection
<img width="1785" height="791" alt="route table of marketing ggroup to finance via peer" src="https://github.com/user-attachments/assets/4270ee7b-3cb1-4e1b-8d8d-6bb20d730232" />

- In **VPC B**: Add route to Finance VPC CIDR via peering connection
- In **VPC Finance**: Add routes to VPC A and VPC B CIDRs via respective peering connections
A and finace:
<img width="1812" height="847" alt="route table of finanace to marketing via peer" src="https://github.com/user-attachments/assets/5d8edb58-e110-4d13-a27f-e4cbf51ce00f" />
---

### 3. Update Security Groups

- Allow inbound traffic on required ports from VPC A and VPC B’s CIDRs in the Finance Service security group.
- Optionally, restrict outbound rules in VPC A and VPC B to only allow access to Finance Service endpoints.
  
<img width="1882" height="665" alt="inbound rules to allow access for finance group" src="https://github.com/user-attachments/assets/4ce4a19e-f70e-4dbf-820a-3b023c53598c" />

---

### 4. Test Connectivity

- From Service A and Service B, attempt to connect to the Finance Service endpoints (using private IPs).
- Use tools like `telnet`, `curl`, or application-specific clients.
<img width="1060" height="585" alt="data passing from developer group to finance group" src="https://github.com/user-attachments/assets/ba311300-9493-489d-b930-5df2280d9d26" />
<img width="1561" height="453" alt="monitoring resources utilization when it is connected" src="https://github.com/user-attachments/assets/a05b3f02-c9ad-4139-adaa-5646059f4629" />

---

## Best Practices & Notes

- **No Transitive Peering**: VPC peering is not transitive. Service A cannot access Service B directly.
- **Least Privilege**: Limit security group rules to only necessary ports and CIDRs.
- **Tag Resources**: Tag your peering connections, routes, and security groups for easy identification.
- **Audit**: Periodically review routes and permissions for compliance and security.

---

## Troubleshooting

- Ensure route tables are updated and associated with correct subnets.
- Check for overlapping CIDR blocks (peering does not work if VPC CIDRs overlap).
- Validate security group and NACL rules.
- Confirm peering connections are in the "active" state.

---

## References

- [AWS VPC Peering Documentation](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [AWS CLI Reference - ec2 create-vpc-peering-connection](https://docs.aws.amazon.com/cli/latest/reference/ec2/create-vpc-peering-connection.html)
