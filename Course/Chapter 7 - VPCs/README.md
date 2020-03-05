# CHAPTER 7 | VPC

## VPC

### [What's a VPC](https://aws.amazon.com/vpc/)

Think of a VPC as a virtual data centre in the cloud.

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

### What are the components of Amazon VPC

* Core components:
  * Subnet: A segment of a VPC’s IP address range where you can place groups of isolated resources. (Types: public, private, VPN-only)
  * Route tables
  * DHCP option sets
  * Security groups
  * Network access control lists
* Optional components:
  * Internet Gateway: The Amazon VPC side of a connection to the public Internet.
  * EIP addresses
  * NAT Gateway: A highly available, managed Network Address Translation (NAT) service for your resources in a private subnet to access the Internet.
  * Peering Connection: A peering connection enables you to route traffic via private IP addresses between two peered VPCs.
  * VPC Endpoints: Enables private connectivity powered by PrivateLink to services hosted in AWS, from within your VPC without using an Internet Gateway, VPN, Network Address Translation (NAT) devices, or firewall proxies.
    * VPC Gateway Endpoint for S3 and DynamoDB
    * VPC Interface Endpoint for other AWS resources
  * Egress-only Internet Gateway: A stateful gateway to provide egress only access for **IPv6** traffic from the VPC to the Internet.
  * Virtual private gateway: The Amazon VPC side of a VPN connection.
  * Customer gateway
  * VPN connection

![VPC_Diagram](https://docs.aws.amazon.com/vpc/latest/userguide/images/default-vpc-diagram.png)

* You can have multiple VPCs in a region (default up to 5).
* You can have 1 internet gateway per VPC.
* 1 Subnet = 1 AZ
* You cannot have a subnet stretched over multiple availability zones.
* You can have multiple subnets in one availability zone.
* Security groups are Stateful, instead, Network ACLs are stateless.
* IPSec security protocol is supported by VPC.

### Default VPC

* Amazon provides a default VPC to immediately deploy instances.
* All Subnets in default VPC have a route out to the internet.

### Creating a new VPC

When we create a VPC, we pick a name tag and enter a CIDR block for it. By default it creates the following components:

* Default Main Route table
* Network Access Control List (NACL) - not associated with any subnets
* Default Security Group
* It won't create any subnets, nor it will create a default Internet gateway.
* US-East-1A in an AWS account can be a completely different availability zone to US-East-1A in another AWS account. The AZ's are randomized.

### Subnet

* Subnets can be public, private, or VPN-only.
* Public subnet is one in which the associated route table directs the subnet's traffic to the Amazon VPC's Internet Gateway(IGW).
* A private subnet is one in which the associated route table does not direct the subnet's traffic to the Amazon VPC's IGW.
* A VPN-only subnet is one in which the associated route table directs the subnet's traffic to the Amazon VPC's Virtual PRivate Gateway(VPG) and does not have a route to the IGW.
* Regardless of the subnet type, the internal IP address range of the subnet is always private.
* The minimum size subnet that you can have uses CIDR block /28, 16 IP address. 5 of these addresses are reserved, so 11 address can be used.
* The maximum size subnet is /16.

### Security Group

* Security groups can't span VPCs.
* By default security groups within the same VPC do not allow access to each other.
  * In order to allow access, rule should be created under security group by setting source by specifying CIDR block or security group name that wants to access it.
* While creating a security group, you pick a name, a description and a VPC for it. Then Inbound and Outbound rules can be defined during the creation process.
* While creating EC2 instances, we pick VPC (network), subnet, and security group. A public instance will get public IP assigned to it through subnet configuration.
* You may change the rules for the default security group, but you may not delete the default security group.

### VPC Peering

* You can peer one VPC to another VPC using private IP subnets.
* You can peer VPC's with others AWS accounts as well as with other VPC's in the same account.

### How to VPC Peering

* Overlapping CIDR Blocks is not supported: You can't connect two VPC's that have the same CIDR.
* Transitive Peering is not supported:

    You have a VPC peering connection between VPC A and VPC B (pcx-aaaabbbb), and between VPC A and VPC C (pcx-aaaacccc). There is no VPC peering connection between VPC B and VPC C. You cannot route packets directly from VPC B to VPC C through VPC A.

    ![transitive-peering](https://docs.aws.amazon.com/vpc/latest/peering/images/transitive-peering-diagram.png)

### Build Your Own Custom VPC

* [VPC and Subnet Sizing](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#vpc-subnet-basics) The first four IP addresses and the last IP address in each subnet CIDR block are not available for you to use, and cannot be assigned to an instance.
* While creating a subnet, you pick a name tag, VPC, AZ, IPv4 / IPv6 CIDR block.
  * Auto assign public IP address and default subnet options are both turned off.
  * Reserved IPs are deducted from the available IPs count. The first four IP addresses and the last IP address in each subnet CIDR block are not available to use, and cannot be assigned to an instance.
    * First address Network address
    * Second address Reserved by AWS for the VPC router
    * Third address Reserved by AWS. The IP address of the DNS server is always the base of the VPC network range plus two.
    * Fourth address is reserved by AWS for future use.
    * Last address is network broadcast address.
  * All subnets that haven't been explicitly associated with any routes tables will use the default main route table. These subnets can communicate with each other.
* To access to Internet from private subnets while keeping them private, NAT instances or NAT gateways are used.
  * NAT instances are created as EC2 instances specifying VPC and public subnet with auto assigned public IP address. Security groups should have access to Internet for the preferred protocols.
    * For Nat Instances you have to disable the [Source/Destination Checks](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html#EIP_Disable_SrcDestCheck). Through UI, follow through options Actions - Networking - Change Source/Dest. Check to disable it.
    * Each EC2 instance performs source/destination checks by default. This means that the instance must be the source or destination of any traffic it sends or receives.
    * However, a NAT instance must be able to send and receive traffic when the source or destination is not itself.
    * NAT instance must be in a public subnet.
    * In order to use the NAT instance, a route out of the private subnet to the NAT instance needs to be defined in the main (private) route table with destination value as 0.0.0.0/0, target as the Instance -> NAT instance.
    * The amount of traffic that NAT instance can support depends on the instance size.
    * They are always behind a security group.

  * NAT Gateway can be created by selecting the public subnet and after getting assigned an Elastic IP.
    * In order to use the NAT gateway, a route out of the private subnet to the NAT Gateway needs to be defined in the main (private) route table with destination value as 0.0.0.0/0, target as the NAT Gateway.
    * Provides redundancy inside the Availability Zone.
    * You can have one NAT gateway inside an AZ.
    * Preferred by the enterprise. Use [Nat Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html#nat-gateway-basics) instead of Nat Instances.
    ![nat-gateway](https://docs.aws.amazon.com/vpc/latest/userguide/images/nat-gateway-diagram.png)
    * Starts at 5Gbps and scales currently to 45Gbps.
    * No need to patch
    * Not associated with security groups
    * Automatically assigned a public IP address due to Elastic IP usage.
    * No need to disable Source/Destination Checks.
    * To create an Availability Zone-independent architecture, create a NAT gateway in each AZ and configure routing to ensure that resources use the NAT gateway in the same AZ.

### Internet Gateway (IGW)

An IGW is a horizontally scaled, redundant, and highly available Amazon VPC component that allows communication between instances in your Amazon VPC and the internet.

You can create an Internet Gateway for a VPC by picking a name tag.

* By default Internet gateway is created as a detached one. Attach it to the VPC.
* You can only have one Internet gateway attached to a VPC.

### Elastic IP (EIP)

* An EIP address is a static, public IP address in the pool for the region that you can allocate to your account and release.
* EIPs allow you to maintain a set of IP address that remain fixed while the underlying infrastructure may change over time.

### Route Table

* By default, main route table allows internal communication between subnets.
* All subnets that haven't been explicitly associated with any routes tables will use the default main route table.
* Following conventions, we should keep the default main route table private and define a separate public route table for Internet access.
* Public route table should have a route definition as below:
  * Destination: 0.0.0.0/0 (For IPv4) or ::/0 (For IPv6) Target: Internet Gateway
* Once the public route table is created, associate public subnets with it.
* A subnet can only be associate with one route table at a time.

### [Network Access Control Lists vs Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html)

![vpc&NACL](https://docs.aws.amazon.com/vpc/latest/userguide/images/security-diagram.png)

* NACL can be created picking up a name tag then selecting a VPC.
* VPC automatically comes with a default NACL that enables all traffic for inbound and outbound.
* By default new custom NACLs deny all traffic for inbound and outbound.
* NACL cannot be deployed in multiple VPCs.
* A subnet can be associated with one NACL at the time.
* A NACL might be associated with multiple subnets at the time.
* Each subnet must be associated with a network ACL, if you don't, default NACL will be used.
* NACL contains a numbered list of rules that are evaluated in order starting with the lowest numbered rule. Rule 100 comes before Rule 200. Deny rule should come before allow rule in order.
* Rules are applied in numerical order (starting from the lowest), so when you should create the first rule having number 100 and add others on incremental of 100.
* NACL are **stateless** (opposite of Security Groups); responses to allowed inbound traffic are subject to the rules for outbound traffic (and vice versa).
* NACLs have separate inbound and outbound rules, and each rule can either allow or deny traffic.
* Remember to open [ephemaral ports](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports) on your outbound rules only.
* If you have to block specific IP addresses, use network ACL not Security Groups.

### Custom VPC's and ELB's

* Design tip: Remember that ELB needs at least two public AZ's, so when designing your network, remember to create at least two public subnets in two AZ.

### [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)

VPC Flow Logs is a feature that enables you to capture information about the IP traffic going to and from network interfaces in your VPC. Flow log data can be published to Amazon CloudWatch Logs and Amazon S3. After you've created a flow log, you can retrieve and view its data in the chosen destination.
Flow logs can be created at 3 levels;

* VPC
* Subnet
* Network Interface Level

You cannot enable flow logs for VPCs that are peered with your VPC unless the peer VPC is in your account.
You cannot tag a flow log.
After you've created a flow log, you can't associate a different IAM role with the flow log.
Not all IP traffic is monitored:

* Traffic generated by instances when they contact the Amazon DNS server. If you use your own DNS server, then all traffic to the DNS server is logged.
* Traffic generated by a Windows instance for Amazon Windows license activation.
* Traffic to and from 169.254.169.254 for instance metadata.
* DHCP traffic.
* Traffic to the reserved IP address for the default VPC router.

### Bastion Host (Jump Box)

* A Bastion is used to securely administer EC2 instance (Using SSH or RDP). Typically you put it in a public subnet that has internet access.
* You cannot use a NAT Gateway as a Bastion host.

### Direct Connect

AWS Direct Connect is a cloud service solution that makes it easy to establish a dedicated network connection from your premises to AWS.

* It directly connects your data center to AWS.
* Useful for high throughput workloads or if you need a stable and reliable secure connection.

### [VPC Enpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)

A VPC endpoint enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection.

There are two types of VPC endpoints:

* Interface Endpoints
* Gateway Endpoints (Supports S3 and DynamoDB)

### Virtual Private Gateway (VPG), Customer Gateway (CGW), Virtual Private Network (VPN)

* A VPG is the VPN concentrator on the AWS side of the VPN connection between the two networks.
* A CGW is a pysical device or a software application on the customer's side of the VPN connection.
* After these two elements of an Amazon VPC has been created, the final step is to create a VPN tunnel. The VPN tunnel is established after traffic is generated from the customer's side of the VPN connection.
