# AWS Spot Instance Management Automation

This repo is helping professionals to easily manage their spot instance over on-demand to save cost on mission non-criticale applications.

1.	Project Objective
•	For non-critical workloads, use Spot Instances to save money over On-Demand Instances.
2.	General Information
I.	Spot Instance:
•	A Spot Instance is an instance that uses spare EC2 capacity that is available for less than the On-Demand price. Because Spot Instances enable you to request unused EC2 instances at steep discounts, you can lower your Amazon EC2 costs significantly. The hourly price for a Spot Instance is called a Spot price. The Spot price of each instance type in each Availability Zone is set by Amazon EC2 and is adjusted gradually based on the long-term supply of and demand for Spot Instances. Your Spot Instance runs whenever capacity is available and the maximum price per hour for your request exceeds the Spot price.
•	Spot Instances are a cost-effective choice if you can be flexible about when your applications run and if your applications can be interrupted. For example, Spot Instances are well-suited for data analysis, batch jobs, background processing, and optional tasks. For more information, see Amazon EC2 Spot Instances.
II.	On Demand Instance:
•	With On-Demand Instances, you pay for compute capacity by the second with no long-term commitments. You have full control over its lifecycle—you decide when to launch, stop, hibernate, start, reboot, or terminate it.
•	There is no long-term commitment required when you purchase On-Demand Instances. You pay only for the seconds that your On-Demand Instances are in the running state. The price per second for a running On-Demand Instance is fixed and is listed on the Amazon EC2 Pricing, On-Demand Pricing page.
III.	AWS Lambda
•	AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources for you. These events may include changes in state or an update, such as a user placing an item in a shopping cart on an ecommerce website. You can use AWS Lambda to extend other AWS services with custom logic or create your own backend services that operate at AWS scale, performance, and security. AWS Lambda automatically runs code in response to multiple events, such as HTTP requests via Amazon API Gateway, modifications to objects in Amazon Simple Storage Service (Amazon S3) buckets, table updates in Amazon DynamoDB, and state transitions in AWS Step Functions.
•	Lambda runs your code on high availability compute infrastructure and performs all the administration of your compute resources. This includes server and operating system maintenance, capacity provisioning and automatic scaling, code and security patch deployment, and code monitoring and logging. All you need to do is supply the code.


IV.	Amazon EventBridge 
•	Amazon EventBridge is a serverless event bus that makes it easy to connect applications together using data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services. EventBridge delivers a stream of real-time data from event sources, such Zendesk, Datadog, or PagerDuty, and routes that data to targets like AWS Lambda. You can set up routing rules to determine where to send your data to build application architectures that react in real time to all of your data sources. EventBridge allows you to build event-driven architectures, which are loosely coupled and distributed. This improves developer agility as well as application resiliency. EventBridge makes it easy to build event-driven applications because it takes care of event ingestion and delivery, security, authorization, and error-handling for you. EventBridge leverages the CloudWatch Events API, so CloudWatch Events users can access their existing default bus, rules, and events in the new EventBridge console, as well as in the CloudWatch Events console. 
•	Amazon EventBridge is a serverless event bus that makes it easy to connect applications together using data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services. EventBridge delivers a stream of real-time data from event sources, such Zendesk, Datadog, or PagerDuty, and routes that data to targets like AWS Lambda
V.	Amazon SNS
•	Amazon Simple Notification Service (Amazon SNS) is a managed service that provides message delivery from publishers to subscribers (also known as producers and consumers). Publishers communicate asynchronously with subscribers by sending messages to a topic, which is a logical access point and communication channel. Clients can subscribe to the SNS topic and receive published messages using a supported endpoint type, such as Amazon Kinesis Data Firehose, Amazon SQS, AWS Lambda, HTTP, email, mobile push notifications, and mobile text messages (SMS).
VI.	Amazon S3
•	Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. Customers of all sizes and industries can use Amazon S3 to store and protect any amount of data for a range of use cases, such as data lakes, websites, mobile applications, backup and restore, archive, enterprise applications, IoT devices, and big data analytics. Amazon S3 provides management features so that you can optimize, organize, and configure access to your data to meet your specific business, organizational, and compliance requirements.

3.	Workflow

I.	Interuption Notification:
•	Whenever AWS trying to reserve the spot instance, they will notify by using CloudWatch Events.
II.	Event Bridge.
•	When the interruption event occurs, we will initiate the Event Rule to trigger the SNS topic with the payload of existing spot instance id
III.	Amazon SNS 
•	The SNS topic is configured with lambda invoke protocol, this can trigger the AWS Lambda function with the instance id as payload
IV.	Lambda Function:
•	Lambda function contains AWS SDK (Boto3) scripts written in python 3.8.
Process:
1.	SNS Topic invoke the lambda with existing instance id.
2.	Using Instance ID that is received from SNS, Lambda will initiate the termination request for the existing instance and detach the attached volume.
3.	Once the existing instance state is changed into “Terminated”. Lambda will check for the existing instance configuration details from the config CSV file that is present in the S3 bucket.
PATH: 
4.	Once the configuration details declared, lambda will get the list of suited spot instance type list provided by INFRA team.
5.	Lambda will initiate the iteration to request spot instance from AWS.
6.	Based on the spot instance availability pipeline will redirect the request into below mentioned process:
CASE – I: Spot Instance Creation

I.	If the instance type that is mentioned in the list available, Lambda will follow the case 1 process
II.	Lambda will Initiate the spot instance creation request
III.	Once the instance state is changed to “Created”. Temporarily lambda will stop the instance.
IV.	Once the instance state is changed to “Stopped”. Lambda will detach the root volume from the instance
V.	Once the volume is successfully detached it will attach the application volumes to the instance
VI.	Once the volume is successfully attached, lambda will trigger the start instance request
VII.	Once the instance is successfully started, lambda will update the log file present in the S3 bucket.
PATH: 
CASE – II: On-Demand Instance Creation

I.	If the list of suitable instance types is not available. Lambda will initiate the on-demand instance request.
II.	Lambda will Initiate the spot instance creation request
III.	Once the instance state is changed to “Created”. Temporarily lambda will stop the instance.
IV.	Once the instance state is changed to “Stopped”. Lambda will detach the root volume from the instance
V.	Once the volume is successfully detached it will attach the application volumes to the instance
VI.	Once the volume is successfully attached, lambda will trigger the start instance request
VII.	Once the instance is successfully started, lambda will update the log file present in the S3 bucket.
VIII.	And lambda will initiate the EventBridge rule to trigger another lambda named “SpotAvailTest” for once every 4 hours
CASE – III: SpotAvailTest Lambda

I.	Once the “SpotAvailTest” lambda is initiated with the existing instance id payload. 
II.	Lambda will check for the existing instance configuration details from the config CSV file that is present in the S3 bucket.
III.	Once the configuration details declared, lambda will get the list of suited spot instance type list provided by INFRA team.
IV.	Lambda will initiate the iteration to request spot instance from AWS.
V.	Once spot instance available it will create and repeat the same process mentioned earlier
VI.	If the mentioned list of spot instance is not available lambda ends. 


 

