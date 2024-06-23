# VPC Production Environment

This example demonstrates how to create a Virtual Private Cloud (VPC) that can be used for servers in a production environment. To improve resiliency, the servers are deployed in two Availability Zones using an Auto Scaling group and an Application Load Balancer (ALB). The servers are deployed in private subnets and receive requests through the load balancer for additional security. The servers can connect to the internet using a NAT gateway deployed in both Availability Zones to improve resiliency.

## Architecture Overview

![Architecture Diagram](./architecture.png)


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
   - Create a NAT Gateway in each public subnet, one per AZ.

5. **Configure Route Tables**:
   - Create a route table for public subnets with a route to the internet via the Internet Gateway.
   - Create a route table for private subnets with a route to the internet via the NAT Gateways.
  
   **Note**: For this project I used the vpc and more option which eventually created all 5 components for me.

   ### Steps to Create EC2 Instances Using Auto Scaling Groups

1. **Create a Launch Template**:
   - **Purpose**: The launch template contains settings common to all the EC2 instances launched using it.
   - **Steps**:
     - Name the template.
     - Provide the AMI for the instance.
     - Specify the instance type.
     - Provide the key pair for SSH access.
     - Attach the security group.
     - Select the VPC in which to launch the instances.
     - Add inbound security group rules (keep port 22 open for SSH and port 8000 open for application access).
     - Add volume settings.

2. **Create an Auto Scaling Group**:
   - **Navigate to Auto Scaling Groups**:
     - Name the Auto Scaling group.
     - Select the launch template created in the previous step.
     - Select the VPC and specify the subnets. As per the architecture diagram, the subnets should be private. Select the appropriate Availability Zones (AZs) and subnets.
   - **Load Balancer**:
     - For now, you can skip selecting a load balancer and attach it later.
   - **Desired Capacity**:
     - Set the desired size of the Auto Scaling group by adjusting the desired capacity.

### Accessing Instances in Private Subnets Using a Bastion Server (Jump Server)

Instances in private subnets cannot connect to the internet directly. To enable secure access, I have used the concept of a **Bastion Server or Jump Server**. Here's how to set it up:

1. **Create a Bastion Server**:
   - Create an instance with the same configurations as the instances in the private subnet.
   - Ensure the instance is in a public subnet with an associated Elastic IP for external access.

2. **Transfer the Private Key**:
   - Download the key pair to your local computer.
   - Use the Secure Copy Protocol (SCP) to copy the key from your local computer to the bastion server. Below is the command to do this:
     ```sh
     scp -i path/to/your/local/private/key.pem path/to/private-key-to-copy.pem ec2-user@your-instance-public-dns:/path/to/destination/
     ```

3. **Login to the Bastion Server**:
   - Use SSH to log in to the instance:
     ```sh
     ssh -i path/to/your/local/private/key.pem ec2-user@your-instance-public-dns
     ```

4. **Verify the File Transfer**:
   - Check that the file has been transferred correctly:
     ```sh
     ls -l /path/to/destination/
     ```

5. **Update File Permissions**:
   - Change the permissions of the key file on the bastion server:
     ```sh
     chmod 600 /path/to/private-key.pem
     ```

6. **Connect to the Private Instance from the Bastion Server**:
   - From the bastion server, use SSH to connect to the private instance:
     ```sh
     ssh -i /path/to/private-key.pem ec2-user@private-ip
     ```

**Tip**: Ensure your security groups are configured to allow SSH access between the bastion server and the private instances.

### Creating a basic Application 

1. **Create a Basic HTML File**:
   - Create a basic HTML file (e.g., `index.html`) that you want to serve.
2. **Using Python HTTP server**
   - Use Python's built-in HTTP server to serve the HTML file on port 8000.
   - ''' python3 -m http.server 8000 '''

  ### Attaching an Application Load Balancer

1. **Create a Target Group**:
   - Create a target group in the same VPC as the instances.
   - Use the same security group permissions as the instances.
   - Ensure the desired ports are open on the security groups (e.g., port 80 for HTTP, port 443 for HTTPS, port 8000).

2. **Create a Load Balancer**:
   - Create an Application Load Balancer (ALB) and select the previously created target group.
   - Ensure the load balancer is in the same VPC as your instances.
   - Configure the load balancer to use the appropriate subnets (public subnets) and security groups.

3. **Access Your Application**:
   - Use the DNS name provided by the load balancer to access your application.
   - Example:
     ```plaintext
     http://your-load-balancer-dns-name
     ```

**Tip**: Ensure the security groups associated with your load balancer allow inbound traffic on the necessary ports.

## Conclusion

By following these steps, you can set up a robust and secure VPC environment suitable for production workloads, with improved resiliency and security.

