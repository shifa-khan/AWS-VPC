# AWS-VPC
This project demonstrates how to create a VPC that you can use for servers in a production environment.

# VPC Production Environment

This example demonstrates how to create a Virtual Private Cloud (VPC) that can be used for servers in a production environment. To improve resiliency, the servers are deployed in two Availability Zones using an Auto Scaling group and an Application Load Balancer (ALB). The servers are deployed in private subnets and receive requests through the load balancer for additional security. The servers can connect to the internet using a NAT gateway deployed in both Availability Zones to improve resiliency.

## Architecture Overview

![Architecture Diagram](architecture-diagram.png)  <!-- Add your architecture diagram here -->

## Components

- **VPC**: A virtual network dedicated to your AWS account.
- **Subnets**: Two private subnets for the servers and two public subnets for the load balancer and NAT gateways.
- **Auto Scaling Group**: Ensures that the correct number of Amazon EC2 instances are running to handle the load.
- **Application Load Balancer**: Distributes incoming application traffic across multiple targets.
- **NAT Gateway**: Allows instances in the private subnets to connect to the internet or other AWS services, but prevents the internet from initiating a connection with those instances.

## Steps to Create the VPC

1. **Create a VPC**:
   - Define a CIDR block, for example, `10.0.0.0/16`.

2. **Create Subnets**:
   - Create two public subnets, one in each Availability Zone (AZ).
   - Create two private subnets, one in each AZ.

3. **Create an Internet Gateway**:
   - Attach it to the VPC.

4. **Create NAT Gateways**:
   - Create a NAT Gateway in each public subnet.

5. **Configure Route Tables**:
   - Create a route table for public subnets with a route to the internet via the Internet Gateway.
   - Create a route table for private subnets with a route to the internet via the NAT Gateways.

6. **Launch EC2 Instances**:
   - Configure an Auto Scaling group to manage the instances in private subnets.

7. **Create an Application Load Balancer**:
   - Distribute traffic across the EC2 instances in private subnets.

## Security Groups

- **Public Subnet Security Group**: Allows HTTP and HTTPS traffic from the internet to the load balancer.
- **Private Subnet Security Group**: Allows traffic from the load balancer to the EC2 instances.

## Connecting to the Internet

Instances in private subnets connect to the internet through the NAT gateways, ensuring they are not directly accessible from the internet, thus enhancing security.

## Diagram

![Architecture Diagram](architecture-diagram.png)  <!-- Add your architecture diagram here -->

## Conclusion

By following these steps, you can set up a robust and secure VPC environment suitable for production workloads, with improved resiliency and security.

