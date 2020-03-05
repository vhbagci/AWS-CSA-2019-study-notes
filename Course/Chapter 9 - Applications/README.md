# CHAPTER 9 | Applications

## [SQS](https://aws.amazon.com/sqs/)

Amazon Simple Queue Service (SQS) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

* It's basically a message queue service, like Kafka for example. You add items in the queue, and then someone will ask for these objects from the queue.
* SQS uses three different identifiers; queue URL, message ID, and receipt handle.
* The default retention period is 4 days and it can go up to 14 days.
* It's a pull based system.
* Type of queues:
  * Default: All queues are standard by default. They don't have an order. Ordering is best-effort, so they don't guarantee ordering on default queues. At least once delivery is performed.
  * FIFO: (first in first out) These queues guarantee ordering. Only once delivery is performed.
* You can have duplicates if you don't manage your visibility timeouts well. SQS supports up to 12 hours of maximum visibility timeout. Default visibility timeout is 30 seconds.
* SQS message consist of attributes as metadata (upto 10 attributes) and message body.
* You can have up to 120,000 message in flight.
* Long polling can be done by sending a WaitTimeSeconds argument to ReceiveMessage up to 20 seconds.
* SQS policy can be used to grant access to other AWS accounts.
* After a message has been successfully sent to a queue, it cannot be recalled.

_Read the [SQS faqs](https://aws.amazon.com/sqs/faqs/) before taking the exam._

## [SWF](https://aws.amazon.com/swf/)

Amazon SWF (Simple Workflow Service) is an Amazon Web Services tool that helps developers coordinate, track and audit multi-step, multi-machine application jobs.

* A workflow is a collection of activities that carry out a specific goal.
* It ensures that a task is assigned only once and is never duplicated.
* Retention can go up to 1 year.
* SWF actors are activity workers, workflow starters, and deciders.
* SWF Activity Workers: Workers are programs that interact with Amazon SWF to get tasks, process received task, and return the result.
* SWF Decider: The decider is a program that controls the coordination of tasks.
* SWF Domains: Domains isolate a set of types, executions, and task lists from others within the same account.
* Workflows in different domains cannot interact.

## [SNS](https://aws.amazon.com/sns/)

Amazon SNS enables message filtering and fan out to a large number of subscribers, including serverless functions, queues, and distributed systems. Additionally, Amazon SNS fans out notifications to end users via mobile push messages, SMS, and email.

* In the exams it will be around Auto Scaling questions.
* SNS stores copies of the messages on multiple AZ.
* Push-based
* It can notify through:
  * AWS Lambda functions
  * HTTP/S webhooks
  * Mobile push
  * SMS
  * Email / Email-JSON
  * Amazon SQS
* When you delete an SNS topic, it my take about 30-60 seconds based on number of subscribers to fully delete the topic. It's allowed to recreate the same topic once it's fully deleted.
* After a message has been successfully published to a topic, it cannot be recalled.
  
## [Elastic Transcoder](https://aws.amazon.com/elastictranscoder/)

Amazon Elastic Transcoder is a media transcoding service.

## [API Gateway](https://aws.amazon.com/api-gateway/)

Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale.

* You can enable request caching.
* Scales automatically.
* You can throttle requests.
* By default https protocol is used for endpoints. Custom certificates can be attached.
* Connect to CloudWatch to log all requests.

## [Kinesis](https://aws.amazon.com/kinesis/)

Amazon Kinesis makes it easy to collect, process, and analyze real-time, streaming data so you can get timely insights and react quickly to new information.

* Kinesis streams: It consists of shards, where each stream is saved and stored for 24h, up to 7 days. You can have multiple shards on each stream. A consumer then read from the shards and process the data.
* Kinesis Firehose: It doesn't have shards, so data traverses the Firehose. Data has to be processed immediately with lambda or stored in S3 for example.
  * It can stream data to Amazon S3, Amazon Redshift, Amazon Elasticsearch Service, and Splunk.
* Kinesis Analytics: It allows you to analyze the data that exists in Kinesis Firehose of streams.
