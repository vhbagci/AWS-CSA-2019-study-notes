# CHAPTER 8 | HA Architecture

## [Elastic Load Balancers](https://aws.amazon.com/elasticloadbalancing/)

Elastic Load Balancing is a highly available service that automaticall distributes incoming application traffic across multiple targets, such as Amazon EC2 instances. It includes options that provide felixibility and control of incoming requests to Amazon EC2 instance.

### [Types of loadbalancers](https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products)

* Application Load Balancer: Best for load balancing HTTP & HTTPS traffic. They operate at layer 7.
* Network Load Balancer: Load balancing TCP traffic where extreme performance is needed. They operate at layer 4. The most expensive option among load balancers.
* Classic Load Balancer: Legacy ELB, mostly operate at layer 4, but can go up to 7.

### X-Forwarded-For

(XFF) The HTTP header field is a common method for identifying the originating IP address of a client connecting to a web server through an HTTP proxy or load balancer.

### Load Balancers - Lab

* You must set up health checks in order to let the load balancer to understand if the instances are up and running.
* Load balancers won't expose any IP, instead they expose DNS names.

```bash
dig +short MyClassicELB-57286656.eu-west-2.elb.amazonaws.com
35.176.153.196
35.176.224.44
```

_At the time of writing, the lab instructs you to create one EC2 instance and configure the load balancer to simply point against it.
My suggestion is to create 2 instances instead and change the `index.html` in something like `instance_1` and `instance_2` so you can see what box the load balancer decides to send your requests. Doing so, allows you to also confirm that if an instance is down, the load balancer automatically forwards traffic to the remaining one still online._

### Configuration

#### Idle Connection Timeout

For each request that a client makes through a load balancer, the load balancer maintains two connections. Once connections is with the client and the other connection is to the back-end instance.
For each connection, the load balancer manages an idle timeout (60 seconds by default) that is triggered when no data is sent over the connection for a specified time period.
After the idle time-out period has elapsed, if no data has been sent or received, the load balancer closes the connections.

#### Cross-Zone Load Balancing

Cross-zone load balancing reduces the need to maintain equivalent numbers of back-end instances in each AZ and improves your application's ability to handle the loss of one or more back-end instances.
However, it's still recommended that you maintain approximately equivalent numbers of instances in each AZ for higher fault tolerance.

#### Connection Draining

You should enable connection draining to ensure that the load balancer stops sending requests to instances that are deregistering or unhelthy, while keeping the existing connections open.
This enables the load balancer to complete in-flight requests made to these instances.

#### Proxy Protocol

While using TCP or SSL, your load balancer forwards requests to the back-end instances without modifying the request headers.
When Proxy Control is enabled, a human-readable header is added to the request header with connection information such as the source IP address, destination IP address, and port numbers.

#### Sticky Sessions

By default, a load balancer routes each request independently to the registered instance with the smallest load.
However, you can use the sticky session feature(also known as session affinity), which enables the load balancer to bind a user's session to a specific instance.
This ensures that all requests from the user during the session are sent to the same instance.

If your application has it's own session cookie, you can configure Elastic Load Balancing so that the session cookie follows the duration specificed by the application's session cookie.
Otherwise, you can configure ELB to create a session cookie by specifying your own stickiness duration. The cookie is named AWSELB.

* Session cookie: Also called a transient cookie, a cookie that is erased when the user closes the Web browser.
  The session cookie is stored in temporary memory and is not retained after the browser is closed. Session cookies do not collect information from the user's computer.
  They typically will store information in the form of a session identification that does not personally identify the user.

#### Health Checks

ELB supports health checks to test the status of the registered EC2 instances behind an ELB load balancer. A health check is a prin, a connection attempt, or a page thats checked periodically.
You can set the time interval between health checks and also the amount of time to wait to respond in case the health check page includes a computational aspect.
Finally, you can set threshold for the number of consecutive health check failures before an instance is marked as unhealthy.

* InService: The status of the instances that are healthy at the time of the health check.
* OutOfService: The status of the instances that are unhealthy at the time of the health check.

## CloudWatch

Monitors resources and applications. It is used to collect and track metrics, create alarms that send notifications, and make changes to resources being monitored based on rules you define.

* Offers basis or detailed monitoring for supported AWS producs. Basic monitoring sends data points to CloudWatch every 5 minutes for a limited number of preselected metrics at no charge.
  Detailed monitoring (Disabled by default) sends data points every minute and allow data aggregation for an additional charge.
* Dashboard used to visualize what's happening with your AWS environment.
* Alarms can be set to notify when a specific threshold is hit.
* Events can be used to perform actions when state changes happen in your AWS resources.
* Logs can be aggregated in a single place to better troubleshoot. Remember that you need to install an agent on the EC2 instance.
* By default, metrics on EC2 instances are: CPU related, disk related, network related and status check related.

Remember that:

* CloudWatch: is for monitoring resources.
* CloudTrail: is for auditing.
* Amazon CloudWatch metric data is kept for 2 weeks.
* You can use the Amazon CloudWatch Logs Agent installer on existing Amazon EC2 instances to install and configure the CloudWatch Logs Agent.
* Amazon CloudWatch metrics provide hypervisor visible metrics.

## Auto Scaling

It allows you to automatically scale your Amazon EC2 capacity out and in using criteria that you define. You can ensure that the number of running EC2 instances increases during demand spikes or peak demand periods to maintain application performance and decreases automatically during demand lulls or troughs to minimize costs.

### Auto Scaling Palns

* Maintain Current Instance Levels: You can configure your Auto Scaling group to maintain a minimum or specified number of running instances at all times. Unhealthy instances are terminated and replaced with new ones.
* Manual Scaling
* Scheduled Scaling: It means that scaling actions are performed automatically as a function of time and date.
* Dynamic Scaling: Using scaling policy, Auto scaling process can be controlled dynamically. For eaxample, you can create a policy that adds more EC2 instances to the web tier when the network bandwith,
  measured by CloudWatch reaches a certain threshold.

### Components

#### [Lanch configurations](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchConfiguration.html) and [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)

* A launch configuration is an instance configuration template that an Auto Scaling group uses to launch EC2 instances. It is composed of the confiuration name, AMI, EC2 instance type, security group, and instance key pair.
* An Auto Scaling group contains a collection of Amazon EC2 instances that are treated as a logical grouping for the purposes of automatic scaling and management.
* Autoscaling group will automatically spread evenly on the number of instances across the AZ you selected once you configured it. So 3 AZ with 3 as group size, means 1 box in each AZ.
* Only the launch configuration name, AMI, and instance type are needed to create an Auto Scaling launch configuration. Identifying a key pair, security group, and a block device mapping are optional elements for an Auto Scaling launch configuration.

Launch configuration example:

```txt
Name: myLC
AMI: ami-0535d66c
Instance type: m3.medium
Security groups: sg-f57cde9d
Instance key pair: myKeyPair
```

#### Auto Scaling Group

An Auto Scaling Group is a collection of Amazon EC2 instances managed by the Auto Scaling service.

* An auto scaling group must contain a name and a minimum and maximum number of instances that be in the group.
* You can optionally specifydesired capacity, which is the of instances that the group must have at all times. If you don't specify it, the default desired capacity is the minimum number of instances that you specify.
* Each auto scaling group can have only once launch configuration at a time.

Auto Scaling Group example:

```txt
Name: myASG
Launch configuration: myLC
Availability Zones: us-east-1a and us-east-1c
Minimum size: 1
Desired capacity: 3
Maximum capacity: 10
Load balancers: myELB
```

#### Scaling Policy (Optional)

A scaling policy is used by Auto Scaling with CloudWatch alarms to determine when your Auto Scaling group should scale out or scale in.
Each CloudWatch alarm watches a single metric and sends messages to Auto Scaling when the metric breaches a threshold that you specify in your policy.
