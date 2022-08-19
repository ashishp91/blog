---
title: "Deploying Django on ECS and ECR"
date: 2022-08-18T12:04:53+05:30
tags: ["DevOps", "Django", "ECS", "ECR"]
draft: false
---

Today we're going to see how to deploy a Django app on ECS using ECR.

### What's ECR and ECS ?

ECR or Elastic Container Register is place to store and manage Docker images. If you're familiar with Docker you must have heard of Dockerhub, ECR is same as that just for AWS. The Docker images in ECR are used by ECS.

ECS or Elastic Container Service is a container management service for AWS. If you want to run, stop or manage containers on an EC2 cluster, you'll use ECS to do so.

In this blog we'll deploy a Django app using ECS and ECR. Let's get started.

### Building a Dockerized Django app

We'll start with creating a Dockerized app. You can create your django app or use [this](https://github.com/ashish91/hello-world-django-app.git).

Once you've your app ready use the copy the following Dockerfile to your app:

```
# importing base image
FROM python:3.10

# updating docker host or host machine
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# changing current working directory to /usr/src/app
WORKDIR /usr/src/app

# copying requirement.txt file to present working directory
COPY requirements.txt ./

# installing dependency in container
RUN pip install -r requirements.txt

# copying all the files to present working directory
COPY . .

# informing Docker that the container listens on the
# specified network ports at runtime i.e 8000.
EXPOSE 8000

# running server
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

Now go to your project directory and build a docker image using following command:

```
docker build -t hello-world-django-app:version-1 .

# if you're using Mac M1/M2 chip
docker buildx build --platform=linux/amd64 -t hello-world-django-app:version-1 .
```

Verify that your docker image is build:

```
docker images | grep hello-world-django-app
```


### Creating ECR repository

We've now dockerized our Django app. We need to push the image to ECR in order for ECS to use it while creating containers. For that we'll need a ECR repository.

Go to ECR from your AWS account. From the sidebar go to Repositories, once you do that you'll see something like this:

![ecr-repositories](/deploying-django-on-ecs-and-ecr/ecr-repositories.png)

In the Create Repository page fill in:

1. Select Private - Private repositories are managed by IAM and repository policy permissions.
2. Set a repository name.

Leave the other config as it is.

![create-repository](/deploying-django-on-ecs-and-ecr/create-repository.png)

### Registering the Dockerized Django app to ECR

Now we've a ECR repository where we can push our Django Docker image.

For pushing the Docker image we'll use the aws command line. Export your AWS_ACCESS_KEY and AWS_SECRET_KEY so aws command line can use that for authentication:

```
export AWS_ACCESS_KEY_ID=******
export AWS_SECRET_ACCESS_KEY=******
```

You can generate your access keys by going to IAM -> Users -> Your User -> Security Credentials. Then generate your Access Keys.

Once you've exported the credentials use following command to authenticate docker to your aws account.

```
aws ecr get-login-password --region {REGION} | docker login --username AWS --password-stdin {AWS_ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com
```

Replace {REGION} and {AWS_ACCOUNT_ID} with your region and 12 digit account ID. For me it's ap-south-1 and 153084803934 so for me the command will be:

```
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 153084803934.dkr.ecr.ap-south-1.amazonaws.com

Login Succeeded

Logging in with your password grants your terminal complete access to your account.
For better security, log in with a limited-privilege personal access token. Learn more at https://docs.docker.com/go/access-tokens/
```

Now that you've authenticated docker to use aws. You'll need to push the docker image to ECR repository. For this tag the docker image with the ECR repository. It'll be in the URI field on the ECR repository page:

![ecr-repository-uri](/deploying-django-on-ecs-and-ecr/ecr-repository-uri.png)

Copy the uri. To tag the docker image get the docker image id:

```
> docker images

REPOSITORY               TAG         IMAGE ID       CREATED        SIZE
hello-world-django-app   version-1   44a813757a0d   18 hours ago   1.01GB
```

Then tag the image with ECR repo URI:

```
docker tag 44a813757a0d 153084803934.dkr.ecr.ap-south-1.amazonaws.com/hello-world-django-app
```

Finally push the docker image to ECR

```
docker push 153084803934.dkr.ecr.ap-south-1.amazonaws.com/hello-world-django-app
```

When the upload is complete you can check the image in ECR by clicking the hello-world-django-app repository.

### Creating a Cluster in ECS

Next we'll need to create an ECS cluster to create EC2 instances where we can run our dockerized django app.
For that go to ECS, click on Clusters from the sidebar, then click on create Cluster:

![create-cluster](/deploying-django-on-ecs-and-ecr/create-cluster.png)

In the cluster template select EC2 Linux + Networking, this will create an Auto Scaling Group with linux AMI:

![cluster-template](/deploying-django-on-ecs-and-ecr/cluster-template.png)

The next page is for configuring the cluster, you'll a number of configuration but we'll set the bare minimum:

1. Set the cluster name.
2. Choose t3.micro/t2.micro if you want to use the Free tier. Otherwise choose the instance type you want.
3. In Networking, choose the default VPC, the default subnet and default security group.

Finally your cluster configuration should look like:

![create-repository](/deploying-django-on-ecs-and-ecr/configure-cluster.png)

### Running Tasks in ECS

To run containers on the EC2 instances we'll need to create task definition. Task definition allows describing config for a container, in layman terms it's how you tell ECS what image to run, how much memory the container uses, what ports to expose from the container etc.

Go to Task Defintions sidebar and click on Create new Task Defintion.

![create-task-definition](/deploying-django-on-ecs-and-ecr/create-task-definition.png)

Select EC2 in the launch type:

![task-definition-launch-type](/deploying-django-on-ecs-and-ecr/task-definition-launch-type.png)

Lastly configure the task definition. Set the following properties:

1. Set the name of the Task definition.
2. Set Task memory as 128 and Task CPU as 1 vCPU.

In Container Definitions select Add container. In the container side bar set:

1. Container name.
2. Set ECR container image uri in Image. Get the uri by going to ECR repository -> ECR Image -> Copy Image URI.
3. Memory Limits as 128.
4. In Port mapping set Host port to 8000 and Container Port to 8000. Leave protocol as TCP. We've set the port as 8000 in the Dockerfile for our dockerized django app.

![container-definition](/deploying-django-on-ecs-and-ecr/container-definition.png)

Once done create the task definition. Now we need to run the task which will run the container image in the EC2 instances of the ECS cluster. To do so:

1. Go to the ECS cluster we created.
2. Scroll down and select Tasks then click on Run new task.
![cluster-run-task](/deploying-django-on-ecs-and-ecr/cluster-run-task.png)
3. In Run Task select EC2 and the Task Definition as the one we created before. Click on Run Task.
![run-task](/deploying-django-on-ecs-and-ecr/run-task.png)
4. After a while the new Task will go to Running state from Pending state which means your container is now running on the EC2 instance.

### Expose app to public internet

If you try to go to the EC2 url you'll see a time out error. This is because we've not opened the EC2 instance to accept request from the internet. To fix the security group to allow inbound traffic on port 8000.

1. Go to the running EC2 instance.
![running-ec2-instance](/deploying-django-on-ecs-and-ecr/running-ec2-instance.png)
2. Open Security tab. Click on the default security group.
![ec2-instance-security-tab](/deploying-django-on-ecs-and-ecr/ec2-instance-security-tab.png)
3. Edit the Inbound rules to add TCP from any ip(0.0.0.0) on port 8000. Finally your inbound rules should look like:
![ec2-inbound-rules](/deploying-django-on-ecs-and-ecr/ec2-inbound-rules.png)

After some time go to ec2-65-1-91-75.ap-south-1.compute.amazonaws.com:8000/hello where ec2-65-1-91-75.ap-south-1.compute.amazonaws.com:8000/hello is your EC2 public DNS. You should now see Hello World!

Congratulations you've deployed a Django app on AWS ECS and ECR.
