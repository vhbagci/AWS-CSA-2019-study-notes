# CHAPTER 5 | Route53

Amazon Route 53 is a highly available and highly scalable AWS-provided DNS service.
Amazon Route 53 connects user requests to infrastructure running on AWS (for example, Amazon EC2 instances and Elastic Load Balancing load balancers).
It can also be used to route users to infrastructure outside of AWS.
With Amazon Route 53, your DNS records are organized into hosted zones that you configure with the Amazon Route 53 API. A hosted zone simply stores records for your domain.
You can create two types of hosted zones: public hosted zones and private hosted zones.

## DNS

_You won't be questioned in the exam about the next sections (up to ALIAS record).
It's just for your knowledge and very good for better understanding the course._

### [What's DNS](https://www.cloudflare.com/learning/dns/what-is-dns/)

The Domain Name System (DNS) is the phonebook of the Internet. Humans access information online through domain names, like nytimes.com or espn.com. Web browsers interact through Internet Protocol (IP) addresses. DNS translates domain names to IP addresses so browsers can load Internet resources. DNS **mostly use UDP** Port 53, but as time progresses, DNS will rely on TCP Port 53 more heavily. DNS has always been designed to use both UDP and TCP port 53 from the start, with UDP being the default, and fall back to using TCP when it is unable to communicate on UDP, typically when the packet size is too large (when response data size exceeds 512 bytes) to push through in a single UDP packet.

* DNS starts with TLDs (for example, .com, .edu).
* The Internet Assigned Numbers Authority (IANA) controls the TLDs in a root zone database, which is essentially a database of all available TLDs.
* DNS names are registered with a domain registrar. A registrar is an authority that can assign domain names directly under one or more TLDs.
* These domains are registered with InterNIC, a service of ICANN, which enforces the uniqueness of domain names across the Internet.
* Each domain name becomes registered in a central database, known as the WhoIS database.

### DNS Resolution

* Your browser asks the resolving DNS server what the IP address is for amazon.com.
* The resolving server does not know the address, so it asks a root server the same question. There are 13 root servers around the world, and these are managed by ICANN.
* The root server replies that it does not know the answer to this, but it can give an address to a TLD server that knows about .com domain names.
* The resolving server then contacts the TLD server. The TLD server does not know the address of the domain name either, but it does know the address of the resolving name server.
* The resolving server then queries the resolving name server.
* The resolving name server contains the authoritative records and sends these to the resolving server, which then saves these records locally so it does not have to perform these steps again in the near future.
* The resolving name server returns this information to the user’s web browser, which also caches the information.

### DNS Transfer

You can transfer already owned DNS to Route 53. First step is to transfer the existing domain registration from another registrar to Amazon Route 53 to configure it as your DNS service.

### [IPv4 vs IPv6](https://www.guru99.com/difference-ipv4-vs-ipv6.html)

* IPv4 is a 32-Bit IP Address and it support over 4 billion addresses. For example; 192.168.1.1.
* IPv6 is a 128-Bit more complex newer alphanumeric IP Address and was created to fulfil the need for more Internet addresses. It supports 340 undecillion addresses. For example; 2400:cb00:2048:1::c629:d7a2.

### [Top Level Domains:](https://en.wikipedia.org/wiki/Top-level_domain)

A top-level domain is one of the domains at the highest level in the hierarchical Domain Name System of the Internet (```.com``` ```.net``` ```.org``` as an example).

In the case of ```.co.uk```, ```.co``` is the second level domain and ```.uk``` is the top level domain.

### [SOA Record](https://en.wikipedia.org/wiki/SOA_record)

A Start of Authority record (abbreviated as SOA record) is a type of resource record in the Domain Name System (DNS) containing administrative information about the zone, especially regarding zone transfers.

Example of a SOA record

```bash
$ dig SOA +multiline google.com

[...]
;; ANSWER SECTION:
google.com.        55 IN SOA ns1.google.com. dns-admin.google.com. (
                238640061  ; serial
                900        ; refresh (15 minutes)
                900        ; retry (15 minutes)
                1800       ; expire (30 minutes)
                60         ; minimum (1 minute)
                )
[...]
```

### [NS Record](https://www.cloudflare.com/learning/dns/dns-records/dns-ns-record/)

NS stands for 'name server' and this record indicates which DNS server is authoritative for that domain (which server contains the actual DNS records)

```bash
$ dig NS google.com

[...]

;; ANSWER SECTION:
google.com.        2073    IN    NS    ns4.google.com.
google.com.        2073    IN    NS    ns1.google.com.
google.com.        2073    IN    NS    ns2.google.com.
google.com.        2073    IN    NS    ns3.google.com.

[...]
```

### [A Record](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/)

The ‘A’ stands for ‘address’ and this is the most fundamental type of DNS record, it indicates the IP address of a given domain

```bash
$ dig A google.com

[...]

;; ANSWER SECTION:
google.com.        62    IN    A    216.58.201.14

[...]
```

### [TTL, Time to Live](https://en.wikipedia.org/wiki/Time_to_live#DNS_records)

When a caching (recursive) nameserver queries the authoritative nameserver for a resource record, it will cache that record for the time (in seconds) specified by the TTL.

### [CNAME Record](https://support.dnsimple.com/articles/cname-record/)

CNAME records can be used to alias one name to another. CNAME stands for Canonical Name.

A common example is when you have both example.com and www.example.com pointing to the same application and hosted by the same server.

```bash
$ dig www.dnsimple.com

[...]

;; ANSWER SECTION:
www.dnsimple.com.    3595    IN    CNAME    dnsimple.com.
dnsimple.com.        55    IN    A    104.245.210.170

[...]
```

CNAME can't be used on the root domain. (This is a contractual limitation imposed by the RFC 1912 and RFC 2181, not a technical one.) It must be either an A record or an Alias.

### [ALIAS record](https://support.dnsimple.com/articles/alias-record/)

An ALIAS record is a virtual record type we created to provide CNAME-like behaviour on apex domains.
An apex domain is a custom domain that does not contain a subdomain, such as example.com. Apex domains are also known as base, bare naked, root apex, or zone apex domains.
An apex domain is configured with an A, ALIAS, or ANAME record through your DNS provider.
For example, if your domain is example.com and you want it to point to a hostname like myapp.herokuapp.com, you can’t use a CNAME record, but you can use an ALIAS record. The ALIAS record will automatically resolve your domain to one or more A records at resolution time, and resolvers see your domain simply as if it had A records.

In AWS you have to use ALIAS records to point your root domain to other DNS records such as your ELB.

## DNS IN AWS

Amazon Route 53 can route queries to a variety of AWS resources such as an Amazon CloudFront distribution, an Elastic Load Balancing load balancer, an Amazon EC2 instance, a website hosted in an Amazon S3 bucket, and an Amazon Relational Database (Amazon RDS).

### [Routing policies available in AWS](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)

* Simple routing policy: Use for a single resource that performs a given function for your domain. You can have 1 record with multiple addresses, then Route 53 returns all addresses to the user in a random order. It doesn't support health checks.
* Weighted routing policy: Use to route traffic to multiple resources in proportions that you specify. You can send 40% of the traffic on one IP and 60% to another IP. You can set health checks on individual record sets.
* Latency routing policy: Use when you have resources in multiple AWS Regions and you want to route traffic to the region that provides the lowest network latency for the end user.
* Failover routing policy: Use when you want to configure active-passive failover. You need to create a health check before for the primary set.
* Geolocation routing policy: Use when you want to route traffic based on the location of your users.
* Multivalue answer routing policy: Use when you want Route 53 to respond to DNS queries with up to eight healthy records selected at random. Basically, it's simple routing policy with health checks. Each record can have one address.
* Geoproximity routing policy: Use when you want to route traffic based on the location of your resources and, optionally, shift traffic from resources in one location to resources in another. You must use Route 53 traffic flow.

#### Health Checks

* If a record set fails a health check, it'll be removed  from Route 53 until it passes the health check.
* You can set SNS notifications to alert you if a health check is failed.

## Exam Tips

* ELBs do not have pre-defined IPv4 addresses; you resolve to them using a DNS name.
* Understand the difference between an Alias record and a CNAME.
* Given the choice, always choose an Alias record over a CNAME.
* **Common DNS Types:**
  * SOA: Start of authority record. The start of a zone is defined by the SOA; therefore, all zones must have an SOA record by default.
  * NS: Name server record
  * A: IPv4 address record
  * AAAA: IPv6 address record
  * CNAME: Canonical name record maps a name to another name. It should be used only when there are no other records on that name.
  * MX: A mail exchange record specifies the mail server responsible for accepting email messages on behalf of a domain name.
  * PTR: Pointer records are used for the Reverse DNS (Domain Name System) lookup. Using the IP address you can get the associated domain/hostname. An A record should exist for every PTR record. The usage of a reverse DNS setup for a mail server is a good solution.
  * SPF: Sender policy framework commonly used to stop email spoofing and spam by verifying authorized senders of mail from your domain.
  * SRV: Service locator
  * TXT: Text record can be used to store arbitrary and unformatted text with a host. For example; human-readable information about a server, network, and other accounting data.
* You can buy domain names directly from AWS.
* It can take up to 3 days to register the domain depending on the circumstances.
