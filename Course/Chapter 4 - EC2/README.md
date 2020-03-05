# CHAPTER 4 | EC2 The Backbone of AWS

## EC2

### [What's EC2](https://aws.amazon.com/ec2/)

Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is designed to make web-scale cloud computing easier for developers.

### [EC2 Options](https://aws.amazon.com/ec2/pricing/)

* [On demand](https://aws.amazon.com/ec2/pricing/on-demand/): You pay for computing capacity by per hour or per second depending on which instances you run.
* [Reserved Instance (RI)](https://aws.amazon.com/ec2/pricing/reserved-instances/): Provide a significant discount (up to 75%) compared to On-Demand pricing and provide a capacity reservation when used in a specific Availability Zone. You have to enter a contract.
* [Spot](https://aws.amazon.com/ec2/spot/): Amazon EC2 Spot instances allow you to request spare Amazon EC2 computing capacity for up to 90% off the On-Demand price.

  * If you terminate an instance, you will pay for the complete hour.
  * If Amazon terminates the instance, you won't pay for the complete hour.
  * The following are the possible reasons that Amazon EC2 might [interrupt your Spot Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html):
    * Price – The Spot price is greater than your maximum price.
    * Capacity – If there are not enough unused EC2 instances to meet the demand for Spot Instances, Amazon EC2 interrupts Spot Instances. The order in which the instances are interrupted is determined by Amazon EC2.
    * Constraints – If your request includes a constraint such as a launch group or an Availability Zone group, these Spot Instances are terminated as a group when the constraint can no longer be met.
  * The default Amazon EC2 instance limit for all regions is 20.

* [Dedicated Hosts](https://aws.amazon.com/ec2/dedicated-hosts/): Is a physical server with EC2 instance capacity fully dedicated to your use.

### [What's EBS](https://aws.amazon.com/ebs/)

Provides persistent block storage volumes for use with Amazon EC2 instances in the AWS Cloud. EBS is automatically replicated in a specific AZ

### [Amazon EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)

#### SSD

SSD are good for random access.

* General Purpose SSD (GP2): General purpose SSD volume that balances price and performance.
  * 3 [IOPS](https://en.wikipedia.org/wiki/IOPS) per GB, up to 10k IOPS. Maximum throughput is 160MB.
  * Volume size: 1 GiB - 16 TiB. Billed based on the amount of data space provisioned, regardless of how much data actually used.
  * For instance, a 1TB volume would provide a baseline performance of 3,000 IOPS. A 5 TB volume will not provide a 15,000 IOPS baseline, as it would hit the cap at 10,000 IOPS.
  * It supports the ability to burst to up to 3,000 IOPS for extended periods of time for volumes under 1 TB.
    * For instance, if you have a 500 GB volume you can expect a baseline of 1,500 IOPS. Wheneveryou're not using these IOPS, they are accumulated as I/O credits. Duringg heavy traffic, credits will be used to provide 3,000 IOPS until they're depleted.
  
* Provisioned IOPS SSD (io1): Designed for IO intensive, use it if you need more than 10k IOPS.
  * 30 IOPS per GB, up to 20k IOPS. Maximum throughput is 320MB.
  * Volume size: 4 GiB - 16 TiB.

#### HDD (Magnetic)

Are great for sequential access (processing log files, bigdata work flows as an example).

* Throughput Optimized HDD (st1): Magnetic disk, this can't be a boot volume (root volume).
  * Average 500 IOPS. Maximum throughput is 500MB. Volume size: 1 GiB - 16 TiB.
* Cold HDD (sc1): Lower cost storage, like file servers, can't be a boot volume.
  * Average 250 IOPS. Maximum throughput is 250MB. Volume size: 1 GiB - 16 TiB.
* Magnetic (standard): Lowest cost per gigabyte of all EBS. It's bootable and it's from the previous storage generation.
  * Average 100 IOPS. Maximum throughput is 40-90MB. Volume size: 1 GiB - 1 TiB.

### [RAID Arrays using EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html)

As you would do in a bare-metal server, you can also create [RAID](https://en.wikipedia.org/wiki/RAID) arrays within AWS EC2 boxes using EBS volumes.

* Remember that the RAID available in AWS is only **software**.
* You can have two types of RAID:
  * [RAID 0](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_0) – splits ("stripes") data evenly across two or more disks. When I/O performance is more important than fault tolerance; for example, as in a heavily used database (where data replication is already set up separately). You can use RAID 0 configurations in scenarios where you are using heavy databases with perhaps mirroring and replication.
  * [RAID 1](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_1) – consists of an exact copy (or mirror) of a set of data on two or more disks. When fault tolerance is more important than I/O performance; for example, as in a critical application. With RAID 1 you get more data durability in addition to the replication features of the AWS cloud.
  * RAID 5 and RAID 6 are not recommended for Amazon EBS because the parity write operations of these RAID modes consume some of the IOPS available to your volumes. Depending on the configuration of your RAID array, these RAID modes provide 20-30% fewer usable IOPS than a RAID 0 configuration. Increased cost is a factor with these RAID modes as well; when using identical volume sizes and speeds, a 2-volume RAID 0 array can outperform a 4-volume RAID 6 array that costs twice as much.

Remember also that you can create [snapshots of your RAID arrays](https://aws.amazon.com/premiumsupport/knowledge-center/snapshot-ebs-raid-array/)

### Launch an EC2 Instance - Lab

* Termination protection is turned off by default.
* The EBS root volume by default is deleted at termination.
* Default AMI's (provided by Amazon) cannot be encrypted.
* Additional volumes can be encrypted.
* Once the instance is created:
  * You can change the instance type only within the same instance type family, or you can change the Availability Zone.
  * You cannot change the operating system nor the instance type family.
* Running instance with enhanced networking provides the following advantages:
  * More pacets per second (PPS)
  * Lower latency
  * Less jitter
* There is a very small hourly charge for allocated elastic IP addresses that are not associated with an instance.

## Security groups - Lab

### [What's a security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

A security group acts as a virtual firewall for your instance to control inbound and outbound traffic.

* All Inbound traffic is Blocked by Default.
* All Outbound traffic is Blocked by Default.
* All security groups changes are applied immediately.
* Security groups are stateful. For example, if you allow the request to come in, automatically responses can go out even if you don't have anything on the outbound section of your security group.
* You can specify only allow rules, not deny rules.
* When there are multiple security groups associated with an instance, all the rules are aggregated.

## EBS Volumes & Encrypt Root Device Volume - Lab

* Instances and Volumes MUST be in the same AZ.
* Snapshots exists in S3.
* Snapshots are a point in time copies of Volumes.
* Snapshots are incremental (only differences are saved).
* Snapshot of root volumes require the instance to be stopped.
* _You can't delete a snapshot of an EBS volume that is used as the root device of a registered AMI._
* You can change EBS volumes size and type on the fly.
* To move an EC2 volume from one AZ/Region to another, take a snap or an image of it, then you can copy them to the new AZ/Region.
* Snapshots of encrypted volumes are encrypted automatically.
* Volumes restored from an encrypted snapshot will be encrypted as well.
* You can share snapshots only if they are not encrypted, these snapshots can be made public.

### [AMI Types](https://aws.amazon.com/amazon-linux-ami/instance-type-matrix/)

* [EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html): Amazon EBS provides durable, block-level storage volumes that you can attach to a running instance
  * EBS takes less time to provision.
  * EBS volumes can be kept once the instance is terminated.
* [Instance Store / Ephemeral storage](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html): This storage is located on disks that are physically attached to the host computer.
  * Instance stores are low-durability, high-IOPS storage that is included for free with the hourly cost of an instance.
  * Instance Store Volumes can't be stopped, if the host fails, you lose data.
  * You can reboot the instance without losing data.
  * You can not detach Instance Store Volumes.
  * Instance store volumes cannot be kept once the instance is terminated.

### IAM Roles with EC2 - Lab

* Roles avoid you to store credentials inside EC2 instances in order to communicate with other AWS services.
* Roles can be assigned after an EC2 instance has been provisioned.
* Roles are universal, you can use them in any region.

### [EC2 Instance Metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

Within your local instance run:

 ```http://169.254.169.254/latest/meta-data```

 This will give you the top level metadata items, something like this.

 ```text
 [ec2-user ~]$ curl http://169.254.169.254/latest/meta-data/
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hostname
iam/
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/
```

If you want to access a specific item, simply add it at the end of your request:

```bash
[ec2-user ~]$ curl http://169.254.169.254/latest/meta-data/ami-id
ami-12345678
```

There is also one URL for user data

```bash
[ec2-user ~]$ curl http://169.254.169.254/latest/user-data/
```

### [Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)

You can launch or start instances in a placement group, which determines how instances are placed on the underlying hardware. When you create a placement group, you specify one of the following strategies for the group:

* Cluster – clusters instances into a low-latency group in a single Availability Zone.
* Spread – spreads instances across underlying hardware and can spread in multiple Availability Zones.
* Partition – spreads instances across logical partitions, ensuring that instances in one partition do not share underlying hardware with instances in other partitions.

Some notes about placement groups:

* The name you specify for a placement group must be unique within your AWS account.
* Only specific types of instances can be launched in a placement group.
* You can't merge placement groups.
* You can't move an existing instance into a placement group.
* If the exam refers to placement groups, without mentioning which type, it's most probably talking about the Cluster ones since those are the old ones.

## [EFS](https://aws.amazon.com/efs/)

Amazon EFS provides scalable file storage for use with Amazon EC2. You can create an EFS file system and configure your instances to mount the file system

* Supports NFSv4.
* You only pay for the storage you use.
* Scale up to petabytes.
* Can support thousands of concurrent connections.
* Data is stored across multiple AZ.
* Read after write consistency.
* You need to make sure that the EC2 instance that needs to connect with the EFS volume, is associated with the same security group you have on the EFS volume.
* You can assign permissions at the file level and at the folder level.
