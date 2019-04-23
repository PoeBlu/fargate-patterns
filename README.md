- [**Fargate Design Patterns**](#fargate-design-patterns)
  * [Compendium and Code Examples of AWS Fargate Patterns.](#compendium-and-code-examples-of-aws-fargate-patterns)
  * [Introduction](#introduction)
  * [What is AWS Fargate?](#what-is-aws-fargate-)
  * [Components of AWS Fargate](#components-of-aws-fargate)
    + [Task](#task)
    + [Service](#service)
    + [Cluster](#cluster)
  * [AWS Fargate - the Good, Bad & Ugly](#aws-fargate---the-good--bad---ugly)
    + [Good & Bad: Pay Per Use](#good---bad--pay-per-use)
    + [Good: Low Complexity](#good--low-complexity)
    + [Good: Better Security](#good--better-security)
    + [Good: Faster Development](#good--faster-development)
    + [Good:  Scaling](#good---scaling)
    + [Bad: Limited Availability](#bad--limited-availability)
  * [Focus of these Behavioral Patterns](#focus-of-these-behavioral-patterns)
  * [*Container-on-Demand* Pattern](#-container-on-demand--pattern)
    + [Problem](#problem)
    + [Solution](#solution)
      - [Pattern Components](#pattern-components)
    + [Limitations](#limitations)
  * [Scaling Container Pattern](#scaling-container-pattern)
    + [Problem](#problem-1)
    + [Solution](#solution-1)
      - [Pattern Components](#pattern-components-1)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>
# Fargate Design Patterns

## Compendium and Code Examples of AWS Fargate Patterns.

**AWS Fargate** is the Uber of container service allowing engineers to hail a container by specifying their compute and memory needs. By providing incredible on-demand flexibility and removing the burden of resource provisioning just as Lambda did years ago to servers, Fargate is disrupting the container management technology.

## Introduction
Making software behave predictably in different environments where it is deployed during the lifecycle of an application is one of the biggest challenges of development. Subtle differences in system software on which developers have no control - even in same operating systems - can cause unexpected behaviors and are hard to debug and fix. 

Containers were invented to solve this problem. Containers encapsulate entire runtime environments for an application or service including dependent libraries, configurations which are just enough to run the application into software packages that are portable across operating systems. By sandboxing the application into just enough space and opening just the right ports for communication with the outside world, containers also increase the security of an application by reducing blast radius and increasing the number of services that can be run on a unit of hardware.

First released in 2013, Docker introduced the concept of containers. Kubernetes followed in 2014 allowing multiple Docker nodes running on different heterogenous hosts to be orchestrated by automating provisioning, networking, load-balancing, security and scaling across these nodes through a single dashboard or command line. Both of these technologies required the upkeep of the underlying cluster of servers & operating system through upgrades, patching, rehydration, and security management. Amazon introduced ECS and EKS as platform services to streamline this management process for Docker and Kubernetes respectively.

## What is AWS Fargate?

Put simply, **AWS Fargate** is a managed, compute engine provided by AWS to run your containers without having to worry about managing a cluster of servers. You don't have to choose server types, upgrade or patch servers or optimize container packing on your clusters. 

This is analogous to hailing an *Uber* car service. With Uber, you just tell what size car you want based on how many people are riding, if you want a car seat or want the car to be wheel-chair accessible. You don’t specify a Lexus or a Toyota. With Fargate, all you do is package your application in containers, specify the CPU and memory requirements, define networking and IAM policies, and launch the application. Fargate takes care of scaling so that you don't have to worry about provisioning enough compute resources for your containers to scale out or scale them in when they are not utilized.  In essence, Fargate separates the task of running containers from the task of managing the underlying infrastructure. Developers can simply specify the resources that each container requires, and Fargate will handle the rest. ***As a result, applications deployed on Fargate can save you time, manpower, and money***.

If you are used to traditional container management, you will really appreciate Fargate allowing you to focus on the 'Dev' part of designing and building your applications and reduce the 'Ops' part of managing infrastructure from your 'DevOps' responsibilities. 

## Components of AWS Fargate

![enter image description here](https://github.com/skarlekar/fargate-patterns/blob/master/images/Fargate%20Components.png)

### Task 
A *Task* is the blue print for your application running on Fargate. You use *Task Definition* to configure your tasks on Fargate and each time you launch a task in Amazon ECS.

The Task Definiton specifies which container repository and container image you want to use for running your Task. It also specifies the CPU, memory, the roles to use for executing the task. 
Fargate then knows which Docker image to use for containers, how many containers to use in the task, and the resource allocation for each container. 

### Service 
Fargate allows you to run and maintain a specified number of instances of a *Task Definition* simultaneously in an Amazon ECS cluster. This is called a *Service*. If any of your tasks should fail or stop for any reason, the Amazon ECS service scheduler launches another instance of your task definition to replace it and maintain the desired count of tasks in the service depending on the scheduling strategy used. 

In addition to maintaining the desired count of tasks in your service, you can optionally run your service behind a load balancer. The load balancer distributes traffic across the tasks that are associated with the service.

### Cluster
An Amazon ECS *Cluster* is a logical grouping of tasks or services. Clusters are AWS region specific and can contain tasks using both the Fargate and EC2 launch types.

## AWS Fargate - the Good, Bad & Ugly
###  Good & Bad: Pay Per Use
Fargate is a good choice if you are leaving a lot of compute power and memory foot-print unused. Unlike ECS or EKS, you only pay for the compute and memory that you actually use. It also integrates well with other AWS services allowing you to schedule tasks and run them based on events while automatically fading them out when not in use.

While Fargate provides you an opportunity to cut costs by charging you only for the time your container is running, the average per-hour cost for running Fargate is more than the per-hour cost of running ECS or EKS in spite of [major price reduction in Jan 2019](https://aws.amazon.com/blogs/compute/aws-fargate-price-reduction-up-to-50/) proving once again that there is no free lunch. The cost differential is the price you pay for not having to deal with the complexity of managing infrastructure or investing in time and resources to deal with the cluster management that comes with the traditional solutions. 

As a result, the onus is on you to make the right choice based on the size of your workload, availability of skilled resources to manage and secure clusters, etc.

### Good: Low Complexity
With its Container-as-a-Service model, you don't have to worry about the underlying infrastructure you need for deploying your container, how you will optimize usage or secure them. Instead, your focus reduces to the four walls of your container - its size, power, and communication with the outside world aka memory, CPU, and networking.

### Good: Better Security
Since you don't have to worry about securing the entire cluster of servers, your security concern is reduced to security within the container, the roles required to run your application, the ports that must be opened for the application that is running inside the container to communicate with the outside world, etc.

### Good: Faster Development
As the problems of systems management is alleviated, developers spend less time on operational issues and can focus on solving business problems building services.

### Good:  Scaling
As Fargate is serverless, scaling is taken care by the provider seamlessly. As a result, you do not have to consider the number of concurrent requests you can handle. Having said that, if you integrate Fargate with downstream *server-based* solutions, you should expect a increase in load on those components when your services running on Fargate scales out significantly.

### Bad: Limited Availability
While AWS is rolling out Fargate to as many regions as they can, it is not as available as Lambdas, ECS or EKS. As of April 2019, Fargate is not available in GovCloud, Sao Paulo, Paris, Stockholm, Japan and China.

## Behavioral Design Patterns for AWS Fargate
Behavioral patterns provide solution for the better interaction between components and foster lose coupling while providing the flexibility to extend these components easily independent of each other.

In this section, we will explore three behavioral design patterns for AWS Fargate viz., the ***Container-on-Demand***, ***Scaling-Container*** and ***Sidecar-Assembly*** patterns that allows Fargate to be used just like Lambdas for heavy on-demand tasks where Lambda is not suitable, or allow you to run containers traditionally but without having to manage infrastructure. Additionally, we will explore how to attach sidecar containers to a parent container to provide supporting features for the application. 

We will use the ***Container-on-Demand*** pattern to build an on-demand video thumbnail service to generate thumbnail images from video files. With this pattern, you can spin the containers on demand and immediately decommission after the task is run.

We will use the ***Scaling-Container*** to build an auto-scaling service that finds the value of the coins thrown on a table from an image. With this pattern, you will have a small footprint always running and scale up or down as the processing demands.

Later we will explore the ***Sidecar-Assembly*** pattern to deploy components of an application into a separate containers to provide isolation and encapsulation. 

## *Container-on-Demand* Pattern
### Context & Problem
AWS Lambda lets you run functions as a service. This allows you to build applications as a conglomeration of serverless microservices which react to events, eschewing development of core functionalities, easy deployment, automatic scaling and fault tolerance. But Lambda has  many [resource limitations](https://docs.aws.amazon.com/lambda/latest/dg/limits.html) and in general, it is not efficient for running long-running jobs. 

For instance these are current limitations on Lambda (as of April 2019):
-   The default deployment package size is 50 MB.
-   Memory range is from 128 to 3008 MB.
-   Maximum execution timeout for a function is 15 minutes.      
-   Request and response (synchronous calls) body payload size can be up to to 6 MB.
-   Event request (asynchronous calls) body can be up to 128 KB .

These are severe limitations for processing several types of applications including machine learning models where the size of libraries go much above the maximum deployment package size of 250MB or may take longer than 15 minutes to run a batch.

As a result, it is not possible to run large workloads or long running processes on Lambda. Further, the resource limitation around size of the software package restricts the type of workloads your can run on Lambda. For instance, if you have a machine learning model that requires usage of large libraries such as Scikit, Numpy etc, it is impossible to fit the software package in a Lambda deployment.

### Solution
Deploy your software package in a container as a Fargate Task. Invoke the task using a Lambda. The Fargate Task is started from a dormant state. Once the process is complete and the output written to the output repository, the Task is automatically stopped. As a result of this, you pay only for the time the Task is running. Additionally, you can preconfigure the size of the task (vCPU, memory, environment variables to pass parameters to the job) or override it for every invocation.

![Container-on-Demand Pattern](https://github.com/skarlekar/fargate-patterns/blob/master/images/container-on-demand-pattern.png)

The entry point in the container can be as trivial as a shell script or could be complex as a web service. But the point to note here is the job submitted to the Fargate Task in this case should be asynchronous. As a result large software packages running large workloads can be run using this pattern.

#### Pattern Components
- **Input Repository** - The input for your *Processor* is stored here and should be reachable by the processor. This could be S3-based object store or a data base. Ideally, this repository should notify the task invoker when a new object is uploaded or updated.
- **Task Invoker** - A short-running function that is used to invoke your Processor. This could be a Lambda function or a synchronous service running as part of another larger process chain.
- **Processor** - A long-running task that is the core of the pattern. It is invoked by the Task Invoker. This could be a Fargate Task that reads its input from the Input Repository, processes it and writes back the output to the Output Repository. The Fargate task can be configured to use one or more containers (with a maximum of 10).
- **Output Repository** - Results of the Fargate Task are stored here. Again, this could be a S3 store or a database and could be optionally configure to emit events on inserts and updates.

### Limitations
While using this pattern Fargate puts Lambdas on steroids, Fargate has its [own resource limitations](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service_limits.html) due to it serverless nature. For instance, the number of tasks using the Fargate launch type, per region, per account cannot be more than 50 or the maximum container storage for tasks using the Fargate launch type cannot be over 10GB. 

If you think your workloads will breach these limitations, you should seriously consider AWS EMR or AWS Glue for your solution's tech stack.

## Scaling Container Pattern
### Context & Problem
In the [problem](https://github.com/skarlekar/fargate-patterns#problem) section of the [Container-on-Demand](https://github.com/skarlekar/fargate-patterns#container-on-demand-pattern) pattern we discussed how the limitations on long-processes rules out Lambda for such asynchronous workloads. Here the main problem is the restrictions on the time it takes to run the jobs which cannot exceed 15 minutes. While the Container-on-Demand pattern solves this issue, for synchronous web services that execute within these limits, the main limitations are the ***size of the deployment package***, networking, local temporary store or the language supported in Lambda. 

As of this writing in April 2019, AWS Lambda natively supports Java, Go, PowerShell, Node.js, C#, Python, and Ruby code. Most recently AWS Lambda provides a Runtime API which allows you to use any additional programming languages to author your functions. While the concept of allowing you to bring your own runtime is radical, it is not straight forward as can be seen from this author's experiment [here](https://github.com/skarlekar/lambda-custom-runtime).

***How do we run synchronous services where the size of the deployment package exceeds the Lambda limits?*** 

While Lambda Layers mitigate some of this issue by allowing artifacts to be shared between Lambdas, it introduces it [own set of issues](https://lumigo.io/blog/lambda-layers-when-to-use-it/), especially around testing Lambdas locally and layers still count towards the 250MB hard limit on the unzipped deployment package size.

***What if you want to run an always-on services that can scale on-demand?*** 

Note that, the Container-on-Demand pattern spins up task to execute the job and spins it down. For asynchronous workloads the time taken to spin-up is not an issue. But for synchronous web services, time is dear. 

### Solution
Following is a possible solution to use a Fargate Service fronted by an Application Load Balancer.

- Deploy your service in a Fargate Task
- Open ports for two-way communication in the Task and Container
- Create an ECS Service to wrap around the Fargate Task. 
- Attach an Application Load Balancer in front of the Fargate Service .
- Register an auto-scaling target with rules on when to scale out your service and when to scale it in.

![enter image description here](https://github.com/skarlekar/fargate-patterns/blob/master/images/scaling-container-pattern.png) 

#### Pattern Components

- **Fargate Task** - A Fargate task that has its ports open for two-way communication using one or more containers (within a maximum limit of ten containers).
- **ECS Service** - An ECS service that uses the Fargate Task from above identifying the desired count of tasks that must be run at any given time.
- **Application Load Balancer** - An Application Load Balancer with a listener to forward requests to the ECS Service.
- **API Gateway** - An *optional* API gateway configured to forward requests to the application load balancer.
- **Web Interface** - An *optional* browser-based interface for allowing users to post requests to the service. This could be a simple HTML form.

## Sidecar Assembly Pattern
### Problem
Services require orthogonal technical capabilities, such as monitoring, logging, configuration, and networking services. While the components encapsulating these orthogonal capabilities can be integrated into the main service, it will leave the main service exposed to the vagaries of these components. For instance,  they will not be well isolated, and an outage in one of these components can affect other components or the entire servive. Also, they usually need to be implemented using the same language as the parent service. As a result, the component and the main service have close interdependence on each other.

One option is to deploy these orthogonal components as separate services allowing each component to have its own life-cycle and be built using different languages. While this gives more flexibility, deploying these features as separate services can add latency to the application. 

### Solution
Co-deploy the orthogonal components along with the main service by placing them in their own containers. Containers in a task are co-deployed together in the same host thereby not affecting the latency of the service signficantly for the communication between them. As a result of this co-deployment, the sidecar and the main service can access the same resources. This allows the sidecar to monitor system resources used by both the sidecar and the primary service.

![enter image description here](https://github.com/skarlekar/fargate-patterns/blob/master/images/side-car-assembly.png)

This pattern can also enable applications to be composed of heterogeneous components and services that has expanded capabilities beyond what is provided by these individual services. In essence, to reinforce that the whole is greater than the sum of its parts. The sidecar also shares the same lifecycle as the parent application, being created and retired alongside the parent.

---


## Instructions for Running the Examples
### Prerequisites 
All the examples require a few prerequisites to be performed. These range from creating appropriate roles in IAM for the Lambda and Fargate to invoke AWS services. For instance, the Lambda to invoke the Fargate task, the Fargate task to read files from the S3 bucket and write back responses to it. Additionally, the S3 bucket must be prepped to notify the Lambda function.

> All these examples uses AWS CLI to invoke various AWS services. To
> avoid the idiosyncrasies of personal development environments,  an EC2
> instance running Ubuntu 16.04 LTS was used to run the AWS CLI and
> deploy the AWS services.
> 
> As you run the various shell scripts indicated in the instructions,
> pay close attention to scripts requiring you to *source* shell
> scripts. This is done to carry over the environment variables
> generated in one script to subsequent ones.

#### AWS IAM setup for executing the AWS CLI commands
To be able to run the aws cli commands in the following exercises, create a group in IAM with the following permissions as shown below and assign it to the IAM user you will be using to work on this exercise.

![FargateDeveloper Group](https://github.com/skarlekar/fargate-patterns/blob/master/images/FargateDeveloperGroup.png)

#### Development Environment setup
Start a brand new EC2 instance running Ubuntu 16.04 LTS as your development environment and in the terminal window clone the Git repository to your development environment

    $ git clone https://github.com/skarlekar/fargate-patterns.git

 Run the *prereqs-ubuntu.sh* script to install Docker, Python, pip, aws cli and other development enablers in your environment.

    $ cd fargate-patterns/prerequisities
    $ prereqs-ubuntu.sh

Log out of your shell and log back for the newly installed programs to take effect.

Verify that you are able to run awscli in your newly minted environment

    $ aws --version
    aws-cli/1.16.144 Python/2.7.10 Darwin/18.5.0 botocore/1.12.134

####  Create AWS roles for Fargate
Create AWS roles ecsTaskExecutionRole and taskRole in IAM for Fargate to access other AWS services on your behalf

    $ source ./create-roles.sh

Ensure the role was created by verifying the TASK_ROLE_ARN variable was set

    $ echo $TASK_ROLE_ARN

####  Create VPC, Subnets and Security Group
Create VPC, Subnets and Security groups for running Fargate

    $ source ./create-vpc-subnets.sh

Ensure the role was created by verifying the VPC, SUBNET1, SUBNET2 and SECURITYGROUP variables was set.

    $ echo echo $VPC, $SUBNET1, $SUBNET2, $SECURITYGROUP

#### Create an Application Load Balancer
Create an application load balancer for the Bean-Counter service we will create later

    $ source ./create-alb.sh

Ensure the ALB, Target Group and Listener were created successfully.

    $ echo $ALB_ARN $TG_ARN $LISTENER_ARN

Do not close the terminal or the shell. You will need the environment variables for running the commands below in the examples.

## Examples
The following code examples demonstrates these behavioral patterns.

### Tom Thumb - A Video Thumb-nail Generator Task
Tom Thumb is a video thumb-nail generator task. It is implemented following the ***Container-on-Demand*** pattern.

In a typical usage, an user uploads a video file to a S3 bucket. A trigger is set on the S3 bucket to notify a Lambda function in the event of a file upload to the *video* folder in the bucket. The Lambda is deployed with a Python code to extract the name of the video file from the Lambda notification event and [invoke a Fargate task](https://github.com/skarlekar/tom-thumb/blob/85f5dc8527ed9c8b917119ee4f94cd61621e1b42/lambda/lambda-function.py#L29-L63). The Fargate task consists of one container that uses ffmpeg application to decode the video and freeze an image at a given position in the video. The frozen image is written to a pre-configured folder in a S3 bucket.

### Setup Instructions

In the same shell that you used to run the prerequisites, run the following commands.

#### Create a repository in ECR 
Create a repository in ECR for storing the Tom-Thumb container image

    $ source ./create-tom-thumb-repository.sh

If the repository already exists, you will get an error message. This is expected. Make sure that the variable ECR_REPO_URI is set

    $ echo $ECR_REPO_URI

#### Build the Docker Image
Build Docker image and push to ECR repository

    $ ./push-to-ecr.sh

Ensure the latest image was pushed to the ECR Repository.

![ECR Repository](https://github.com/skarlekar/fargate-patterns/blob/master/images/ecr-repository-tom-thumb.png) 

#### Create the Log Group
Create the tom-thumb log group

    $ ./create-tom-thumb-log-group.sh

This will create a log group called */ecs/tom-thumb-service*

#### Create the ECS Cluster
Create the tom-thumb cluster in ECS

    $ ./create-tom-thumb-cluster.sh

This will create an ECS cluster called tom-thumb-cluster.
![Tom-Thumb Cluster Creation](https://github.com/skarlekar/fargate-patterns/blob/master/images/cluster-creation-tom-thumb.png)

#### Generate the Task Definition
Generate tom-thumb task definition from the template by passing an URL for a sample video and the duration in the video where you want the frame captured for the thumbnail.

    $ ./generate-tom-thumb-task-definition.sh https://s3.amazonaws.com/your-bucket-name/raw/samplevideo.mp4 10

This will create a temp directory and write the *register-tom-thumb-task-definition.json* file.  Inspect this file and notice that the task contains one container and it uses the my-ecs-tasks-role you created earlier to run the Fargate task.

> Referring to  [the ECS
> documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)
> you can see that the execution role is the IAM role that executes ECS
> actions such as pulling the image and storing the application logs in
> cloudwatch. On the otherhand, the TaskRole is the IAM role used by the
> task itself. For example, if your container wants to call other AWS
> services like S3, Lambda, etc,  it uses the permissions from the
> TaskRole to perform those actions. You need the TaskRole to avoid
> storing the access keys in a config file on the container instance.

#### Register the Task Defintion
Register the tom-thumb task definition in ECS and verify it has been created in the Task Definition section of ECS.

    $ ./register-tom-thumb-task.sh

![Register tom-thumb task-definition](https://github.com/skarlekar/fargate-patterns/blob/master/images/register-task-definition-tom-thumb.png)

#### Generate the parameters for running the task
Generate the parameters for running the task as follows. This will generate a file run-tom-thumb-task.json in the temp directory. 

    $ ./generate-run-tom-thumb-task.sh https://s3.amazonaws.com/your-bucket/raw/samplevideo.mp4 12 your-output-bucket

Notice that there is a section for overrides. You can make changes to this file if you want to change the parameters after the fact that the task has been registered.

#### Manually run the task
Verify the task runs and generates the thumbnail as desired.
$ ./run-tom-thumb-task.sh
Go to the tom-thumb-cluster and verify that the task is running and the thumbnail was generated.

![Manual verification of Task registration](https://github.com/skarlekar/fargate-patterns/blob/master/images/manually-run-task.png)

#### Create a Lambda Trigger
Create a Lambda to automatically trigger the Fargate Task when a video file lands in the desired bucket.

    $ cd lambda

#### Set the S3 bucket ARN
Identify a bucket that will notify the lambda when a video file is uploaded. Note down its ARN and set the S3_BUCKET_ARN variable.
$ EXPORT S3_BUCKET_ARN=arn:aws:s3:::your-bucket-name

#### Create Lambda 
Create the policies and roles required for the lambda to invoke the Fargate task. 

    $ source ./create-lambda-role.sh
    $ echo $LAMBDA_ROLE_ARN

This will create a new role called *my-run-task-lambda-role*. Verify that the role is created through the IAM section of the AWS console.

#### Create a Log Group for Lambda
Create the log group required for the lambda to post logs to CloudWatch

    $ ./create-task-runner-log-group.sh

#### Package the Lambda
Package the python code that has the function that will be triggered through the Lambda when a video file is uploaded. The following script will create a zip file with the Python code.

    $ ./package-lambda.sh

#### Deploy the Lambda
Deploy the zip file with the Lambda function on AWS. If the function already exists, it will be updated. This script also adds the permission for the Lambda to be invoked when a file is uploaded to the S3 bucket mentioned in the S3_BUCKET_ARN variable.

    $ ./create-lambda.sh

Verify the Lambda ***task-runner*** was created through the console and the following enviroment variables are set right for the following:

- SUBNET1
- SUBNET2
- SECURITYGROUP

Additionally, verify that the Lambda permissi
Note: An update to the function does not update the environment variables. 

### Testing Tom-Thumb
- Create a folder called 'video', 'thumbnail' and 'raw' in the S3 bucket that you chose for this project. The Bucket ARN for this should match the S3_BUCKET_ARN variable you set earlier. 

- In the Console go to the Advanced Settings in the Properties tab of the S3 bucket and create a notification event to trigger the ***task-runner*** lambda that was created earlier when a file is dropped into a particular folder in your S3 bucket.

![S3 Notification Setting 1](https://github.com/skarlekar/fargate-patterns/blob/master/images/s3-notification-setting-1.png)

![S3 Notification Setting 2](https://github.com/skarlekar/fargate-patterns/blob/master/images/s3-notification-setting-2.png)
- Upload a video file in the 'video' folder of the bucket and verify a thumbnail is created in the 'thumbnail' folder. It will take around a minute for the process to complete depending upon the size of the video file.

---
### Bean-Counter - A Coin Counter Service
Bean Counter is a coin counter service. It will analyze an image of coins and return the total value of the coins in the image. It works only on US Mint issued coined and does not recognize any denomination above a quarter dollar coin. It also assumes that the picture contains a quarter. The quarter is used to calibrate the size of the coins. It is implemented following the ***Scaling-Container*** pattern.

In a typical usage, an user navigates to the URL of the ALB on the browser and enters the URL for the service along with the location of the image file containing the picture of the coins. The Bean-Counter service then invokes the Fargate Task and returns the response to the browser.

### Setup Instructions

In the same shell that you used to run the prerequisites, run the following commands.

#### Create a repository in ECR 
Create a repository in ECR for storing the Tom-Thumb container image

    $ source ./create-bean-counter-repository.sh

If the repository already exists, you will get an error message. This is expected. Make sure that the variable ECR_REPO_URI is set

    $ echo $ECR_REPO_URI

#### Build the Docker Image
Build Docker image and push to ECR repository

    $ ./push-to-ecr.sh

Ensure the latest image was pushed to the ECR Repository.

#### Create the Log Group
Create the bean-counter log group

    $ ./create-bean-counter-log-group.sh

This will create a log group called */ecs/bean-counter-service*

#### Create the ECS Cluster
Create the bean-counter cluster in ECS

    $ ./create-bean-counter-cluster.sh

This will create an ECS cluster called tom-thumb-cluster.

#### Generate the Task Definition
Generate bean-counter task definition from the template.

    $ ./generate-bean-counter-task-definition.sh 

This will create a temp directory and write the *register-bean-counter-task-definition.json* file.  Inspect this file and notice that the task contains one container and it uses the my-ecs-tasks-role you created earlier to run the Fargate task.

#### Register the Task Defintion
Register the bean-counter task definition in ECS and verify it has been created in the Task Definition section of ECS.

    $ ./register-bean-counter-task.sh

#### Generate the Service Definition
Generate bean-counter service definition from the template.

    $ ./generate-bean-counter-service-definition.sh 

This will create a temp directory and write the *create-bean-counter-service-definition.json* file.  Inspect this file and notice that it contains the target group for the service under the load balancers section. This ties the load balancer to the service. Also notice the desiredCount variable set to 2. 

#### Create the Bean-Counter Service
Create the bean-counter service from the service defintion file generated in the previous step.
$ ./create-bean-counter-service.sh

Verify that the service has been created and two tasks are being provisioned for the service.

![Bean-counter Service Creation Check](https://github.com/skarlekar/fargate-patterns/blob/master/images/create-bean-counter-service-1.png)

![Bean-counter Service Task Provision Check](https://github.com/skarlekar/fargate-patterns/blob/master/images/create-bean-counter-service-2.png)

#### Test the Service
Retrieve the DNS name of the application load balancer. Cut & paste the DNS in the browser.

    $ export DNS=$(aws elbv2 describe-load-balancers | jq '.LoadBalancers[] | if .LoadBalancerName == "My-Fargate-ALB" then .DNSName else null end' | grep -v null | sed "s/\"//g")
    $ echo $DNS
    My-Fargate-ALB-xxxxxxx.us-east-1.elb.amazonaws.com

![Bean counter landing page](https://github.com/skarlekar/fargate-patterns/blob/master/images/bean-counter-browser-1.png)
![Bean counter test output](https://github.com/skarlekar/fargate-patterns/blob/master/images/bean-counter-browser-2.png)

#### Set the Scaling Policy for the Service
Set a target scaling policy for the service such that desired count of the service is set to 2 and can increase to 4 on demand. The auto-scaling-policy.json specifies that the when the combined load on the service breaches 75% the service should scale-out. A cool-out period of 60 seconds is also specified so that the service doesn't thrash around.

    $ ./set-scaling-policy.sh

#### Test the Scaling Policy
Use Apache Bench to hit the server $100,000 times with 100 concurrent threads with a timeout of 120 seconds to see the service scale out. You will have to wait for the cooling period to see the scaling out. Scaling in will take 15 minutes after scale out. Verify this on the ECS console.

    $ ./test-scaling.sh

Following is the output of running Apache Bench:    
![Output of Apache Bench](https://github.com/skarlekar/fargate-patterns/blob/master/images/auto-scaling-output.png)

In the following picture you can see that Fargate has scaled-out as a result of the load.
![Fargate caught in action](https://github.com/skarlekar/fargate-patterns/blob/master/images/scaling-demo.png)

## Conclusion
Each application is unique and solving different needs based on the business requirements. If the task of infrastructure management is too onerous and/or if you only want to pay for your computing time, then Fargate may be the right choice for you. 

On the other hand, if you need greater control of the network resources or have large container workloads that warrant maintaing a cluster of servers to run ECS or EKS, then stick with the latter.

### Scenarios where Fargate is most Beneficial
Fargate can be used with any type of containerized application. However, this doesn’t mean that you will get the same benefit in every scenario. Fargate would be most beneficial for projects that need to reduce the time from ideation to realization such as proof-of-concepts and well-designed, decoupled, micro service based architectures deployed in production environments.

**Applications can consist of a mix of Fargate & Lambda to exploit the Serverless model.**

Use Lambdas for small & tight services with low memory (<3GB) and small request-response cycles (<15 mins).

Use containers deployed on Fargate for:
- Existing legacy services that cannot be trivially refactored, 
- Applications written in languages not supported by Lambda,
- Need to use large libraries that cannot fit into a Lambda profile (Quantlib, Scikit etc),
- Where you need more control over networking, compute horsepower or memory
- Use cases that requires a long in-process runtimes.
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA2MDYzMTA3MCwxNDg0MjM0NTEzLDEyNT
A3MDI0NDIsLTE1MzQyNTg2MzUsLTEzMTgzMDcyNDcsOTkwNDk2
MjYsMTE5MDI4Nzk3OSwxNzc2MjQxMjQwLC0xNzE4NTEwNDM3LD
g2MjQxNjc2MSw5OTY5ODI1ODYsMjM2NDYyOTQwLC01NzcyNDM3
ODksLTg1MzA1NTE2OCwtMTc4MzQzMTI5MCwtMTQxMDUxMzEwMy
wtMjExNDQwNjI1OCwxMDczNDIzODQ2LC00NTc2NDU0MDUsLTky
MDAzNzE4OF19
-->