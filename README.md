# SQSD 

A simple alternative to the Amazon SQS Daemon ("sqsd") used on AWS Beanstalk worker tier instances.

[AWS Beanstalk](http://aws.amazon.com/elasticbeanstalk/) provides a simple to use *Worker Environment Tier* 
([more info](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features-managing-env-tiers.html)) that greatly streamlines the deployment of passive worker microservices for background or async processing. 

![aws-eb-worker](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-messageflow-worker.png)
*(diagram by AWS - available [here](http://aws.amazon.com/elasticbeanstalk/))*

As the included diagram portrays, in a common workflow, the worker instance will consume messages sent to a specified [Amazon SQS](http://aws.amazon.com/documentation/sqs/) from another service (e.g.: a web server or another worker). These messages will be received by the worker via POST requests. This eliminates the necessity of configuring a worker as an always-on service, as well as having to add code for reading and consuming messages from an AWS SQS queue. In other words, the worker is implemented as a standard RESTful API/Service that will react to a message sent to it at an specific endpoint via a POST request. This is an awesome approach by Amazon to microservices and [reactive design](www.reactivemanifesto.org/).

The conversion of the SQS message to a POST request is executed by what AWS calls the *"SQS Daemon"* or *"Sqsd"*. This is a simple daemon they pre-install in the worker tier instances that is constantly monitoring an specific AWS SQS queue (provided by configuration) for new messages. When new messages arrive, it constructs a `POST` request and sends it to a specific endpoint (also provided via configuration). If the endpoint consumes it without errors and returns a `2**` HTTP Code in its response, the *"Sqsd"* deletes the message from the queue to signal that its consumption was successful.

However, even though this approach is extremely powerful, Amazon does not provide the code of this daemon as open source. Therefore, we have reproduced its behavior by creating our own version of the *"Sqsd"* free for everyone to use. Moreover, we have provided lots of customization and configuration properties so that it can be molded to your specific use cases.

But even more important! **We have "dockerized" it** so that you can use it as a Docker container along your worker (even link it to it). This makes this microserviced worker approach even more powerful as it can be easily pre-configured and pre-packaged to be deployed automatically along your services using your favorite Docker orchestration frameworks or even the recently announced [Amazon EC2 Container Service](http://aws.amazon.com/ecs/).

Following are detailed instructions of configuration and usage with and without Docker. Any changes, suggestions or Forks are welcome!

## Technologies / Environments Used 
- Groovy 2.3.7 
- Java JDK 7+ 
- AWS Java SDK 1.9.6

## Usage 

### Configuration 
There are 2 ways to configure the `sqsd`'s properties: Environment Variables or a configuration file. You must set one of the two options.

#### Using Configuration File 
Custom properties are loaded from `config/sqsd-config.groovy`.

#### Using Environment Variables 
Environment Variables and defaults are loaded from `config/sqsd-default-config.groovy`.

- `AWS_ACCESS_KEY_ID` - Your AWS Access Key (**required**)
- `AWS_SECRET_ACCESS_KEY` - Your AWS secret access secret (**required**)
- `SQS_QUEUE_REGION_NAME` - The region name of the AWS SQS queue (default: `us-east-1`)
- `SQSD_QUEUE_URL` - Your queue URL. You can instead use the queue name but this takes precedence over queue name. (**optional/required**)
- `SQSD_QUEUE_NAME` - Your queue name (**optional/required**)
- `SQSD_MAX_MESSAGES_PER_REQUEST` - Max number of messages to retrieve per request (max: `10`, default: `10`)
- `SQSD_WAIT_TIME_SECONDS` - Long polling wait time when querying the queue (max: `20`, default: `20`)
- `SQSD_HTTP_HOST` - Host address to your service (default: `http://127.0.0.1`)
- `SQSD_HTTP_PATH` - Your service endpoint/path where to POST the messages (default: `/`)
- `SQSD_HTTP_REQUEST_CONTENT_TYPE` - Message MIME Type (default: `application/json`)

### Running / Executing  

#### Using Groovy CLI 
This script has been tested with `Groovy 2.3.7`:

    groovy sqsd.groovy

#### Using Docker (with localhost service)  

    cd /your/sqsd/local/path
	docker build -t someImageName .
	docker run someImageName

#### Using Docker (with remote service) 
TBD

#### Using Docker (with linked container service) 
TBD

