# CHAPTER 3 | IAM Identity Access Management

## [What's IAM](https://aws.amazon.com/iam/)

Allows you to manage users and their level of access to the AWS console at the global scope without region restriction. The three principals that can authenticate and interact with AWS resources are the root user, IAM users, and roles.
When you log in to the AWS Management Console as an IAM user or root user, you use a user name/password combination.
A program that accesses the API with an IAM user or root user uses a two-part access key.
A temporary security token authenticates with an access key plus an additional session token unique to that temporary security token.

## IAM Consist of the following

* Users: People, end users.
* Groups: Collection of users.
* Policies: A policy is a JSON document that defines one or more permissions to interact with AWS resources. Each permission includes the effect, service, action, and resource. It may also include one or more conditions.

```json
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "service-prefix:action-name",
        "Resource": "*",
        "Condition": {
            "DateGreaterThan": {"aws:CurrentTime": "2017-07-01T00:00:00Z"},
            "DateLessThan": {"aws:CurrentTime": "2017-12-31T23:59:59Z"}
        }
    }
}
```

* Roles: Is a way to allow one part of AWS to another part. IAM roles are prepackaged sets of permissions that have no credentials.

## Features

* Centralised control of your AWS account.
* Shared Access to your AWS account.
* Gives you granular Permission.
* Does Identity Federation (Active directory, Facebook, etc)
* MultiFactor Authentication.
* Provides temporary access for users/devices etc.
  * Amazon EC2 roles provide a temporary token to applications running on the instance.
  * Federation maps policies to identities from other sources via temporary tokens.
* Allows you to set up your own password rotation policy.
* Supports PCI DSS compliance.
* Integrated with a lot of AWS services.
* AWS allows two access keys to be valid simultaneously to make the rotation process straightforward:
  Generate a new access key, configure your application to use the new access key, test, disable the original access key, test, delete the original access key, and test again.
* If two policies contradict each other; that is, if one policy allows an action on a resource and another policy denies that action, the action is denied.

## [From Console](https://aws.amazon.com/console/)

IAM users sign-in link:
You can customize the URL used by users to sign-in

## New Users and account set up

* IAM is universal, the region is on a global level.
* Root account: Is the user you used to register on AWS, no one should use it to log-in.
* Always enable MFA on your Root account.
* New Users have no permissions once created.
* Access key ID & Secret are not the same as password.
* You can create and customize your password rotation policies.

## [Billing Alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)

You can set a billing alarm to get notified when the amount of spendings for the current month reaches your threshold.
Before doing it you probably have to go on:
```https://console.aws.amazon.com/billing/home?#/preferences```
and enable: Receive Billing Alerts

After that, you can go and use cloudwatch services to set a new Billing Alarm.
