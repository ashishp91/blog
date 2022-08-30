---
title: "Deploy ALB with ECS using Django"
date: 2022-08-26T15:27:54+05:30
draft: false
---

In this blog we're going to deploy an ALB(Application Load Balancer) with ECS.
Before creating an ALB follow [this]({{< ref "deploying-django-on-ecs-and-ecr" >}} "this") blog to create an ECS cluster with a sample Django app deployed to it.

To use an ALB over our ECS cluster we'll follow these steps:

1. Create a Target Group.
2. Create an ALB. Use the target group in the ALB.
3. Create a service in our ECS cluster. Then attach the ALB in our ECS service.

Finally we'll have our traffic for our ECS cluster routed through our ALB.

### Create a Target Group

The first thing we need to do is create a target group. A target group is a collection of EC2 instances or ip addresses or lambda functions which an ALB can route traffic to. So let's create a target group for the EC2 instances we created in our ECS cluster.

To create a Target Group go to EC2 -> Load Balancing -> Create target group:

![create-target-groups](/deploy-alb-with-ecs-using-django/create-target-groups.png)

Next configure the target group:

1. Select Instances for the target type.
2. Set a name.
3. In health check set the path as /hello as our Django app exposes that endpoint.
4. Additionally in Advanced health check settings set Success codes as 200-301. We're doing this because our /hello endpoint by default redirects.

Then register targets as the EC2 instances created by the ECS cluster. We've created 2 EC2 instances in our ECS cluster so we're going to register those.

1. Select the targets.
2. Click on Include as pending below.
3. Click on Create group.

![register-targets](/deploy-alb-with-ecs-using-django/register-targets.png)

### Create Application Load Balancer

Now that we've a target group we'll create an ALB. An ALB is a load balancer managed by AWS which distributes incoming traffic among the targets registered in the target group.

To create an ALB go to EC2 -> Load Balancers -> Create Load Balancer

![create-alb](/deploy-alb-with-ecs-using-django/create-alb.png)

In load balancer type select ALB. In create ALB page:

1. Set a name for the ALB.
2. Select 2 availability zones in Mappings. The AZs must be same the one EC2 instances are running in. For eg. I've 2 instances in my ECS cluster in ap-south-1a and ap-south-1b so I have selected ap-south-1a and ap-south-1b in my ALB.
3. Under Listeners and routing select target group we created earlier.
4. Click on Create load balancer.

![config-alb](/deploy-alb-with-ecs-using-django/config-alb.png)

### Create Service in ECS Cluster

Currently we don't have any containers running on our EC2 instances. We'll create a service in our ECS cluster to create and deploy containers on our EC2 instances.

Go to our ECS Cluster -> Services -> Click on Create

![create-service](/deploy-alb-with-ecs-using-django/create-service.png)

Configure the ECS Service:

1. Select EC2 in launch type.
2. Select hello-world-django-task in the Task Definition.
3. Set the Service Name as alb-ecs-service.
4. Set the Number of tasks to 1-2.

Finally your config should look like:

![config-service](/deploy-alb-with-ecs-using-django/config-service.png)

Next we'll configure our service to use the ALB we created earlier. To do so:

1. Select Application Load Balancer in Load balancing.
2. Select the load balancer name that was created earlier.
3. In Container to load balance section specify the container name we used while creating the Task Definition. Click on Add to load balancer.
4. For Production listener port use 80:HTTP.
5. Select the target group we created earlier.

Finally your config should look like:

![config-network](/deploy-alb-with-ecs-using-django/config-network.png)

Click on Next. Leave Auto Scaling as it is and click on Next. In Review page review the settings:

![review-settings](/deploy-alb-with-ecs-using-django/review-settings.png)

Click on Create Service. It'll take some time for the Tasks to be created and deployed. Once done they'll be in Running state:

![service-tasks](/deploy-alb-with-ecs-using-django/service-tasks.png)

After they're in running state go to the Target Group we created earlier. The target should be in healthy state:

![target-group-healthy-state](/deploy-alb-with-ecs-using-django/target-group-healthy-state.png)

To verify everything is working copy your ALB's DNS name and visit /hello.

Since we've not setup SSL you'll need to go to http url and not https, the url must be http://alb-ecs-83104975.ap-south-1.elb.amazonaws.com/hello.
