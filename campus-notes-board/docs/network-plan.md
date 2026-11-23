
# Network Plan — Campus Notes Board

- Region: Europe (Stockholm) — eu-north-1
- VPC: campus-notes-vpc
- VPC CIDR: 10.42.0.0/16

## Subnets

AWS reserves 5 IPv4 addresses in every subnet.

| Subnet | CIDR | Usable range | Usable IPs | AZ | Purpose | Why this size |
|---|---|---|---:|---|---|---|
| public-subnet | 10.42.1.0/24 | 10.42.1.4–10.42.1.254 | 251 | eu-north-1a | Public application server | Enough addresses for the application |
| private-subnet-a | 10.42.2.0/24 | 10.42.2.4–10.42.2.254 | 251 | eu-north-1a | Private database subnet | RDS subnet group |
| private-subnet-b | 10.42.3.0/24 | 10.42.3.4–10.42.3.254 | 251 | eu-north-1a | Private database subnet | RDS subnet group |
| private-subnet-c | 10.42.4.0/24 | 10.42.4.4–10.42.4.254 | 251 | eu-north-1b | Private database subnet in another AZ | Two-AZ RDS coverage |

## Route tables

| Route table | Routes | Associated subnets |
|---|---|---|
| main | 10.42.0.0/16 → local | None explicitly associated |
| public-rt | 10.42.0.0/16 → local; 0.0.0.0/0 → Internet Gateway | public-subnet |
| private-rt | 10.42.0.0/16 → local | private-subnet-a, private-subnet-b, private-subnet-c |

## Questions

1. **What is a VPC, and how is it different from a subnet?**

A VPC is an isolated virtual network in AWS. A subnet is a smaller network range inside that VPC.

2. **What does /16 mean?**

It means the first 16 bits are used for the network portion of the IPv4 address.

3. **What makes a subnet public?**

A subnet is public when its associated route table has a route to an Internet Gateway.

4. **Why do we separate public and private subnets?**

The public subnet hosts the application server, while private subnets keep the database away from direct internet access.

5. **Why does RDS need a DB subnet group that covers two Availability Zones?**

It allows RDS to use subnets in different Availability Zones and supports high availability configurations.

6. **Why must the database not be publicly accessible?**

Keeping the database private reduces exposure to internet attacks. Only the application server should connect through the database security group.
