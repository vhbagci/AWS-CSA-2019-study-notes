# CHAPTER 10 | Serverless

## [Lambda](https://aws.amazon.com/lambda/)

AWS Lambda lets you run code without provisioning or managing servers. You pay only for the compute time you consume - there is no charge when your code is not running.
Lambda basically is based on triggers.

You can use lambda in two ways:

* Event-driven based: Lambda runs your code based on events, such, new file on S3 or a new alarm on cloudwatch
* Compute-service based: Lambda runs your code based on HTTP requests using an API Gateway or API calls made using AWS SDKs.

AWS lambda, supports: C#, Node.js, Python, Go and Java.

Lambda is charged as follow:

* First 1M requests per month are free. $0.20 PER 1M requests thereafter.
* Duration: You are charged for the amount of memory you allocate on your functions. First 400,000 GB-seconds per month, up to 3.2M seconds of computing time, are free.
* Your functions can't go over 15 minutes in run-time.

_Make sure you have a look at what [CORS](https://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html) is.
