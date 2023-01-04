---
title: "Basics of Scaling"
date: 2022-12-27T15:48:09+05:30
draft: true
---

In this blog we'll learn the basics required to scale an application. Lets start with how does the architecture of a small application looks like. At the most basic level any application has following parts:

- Client - which requests the API and gets back a response.
- Server - which processes the request and sends back the response.
- Database - which stores any data that the server needs to process any subsequent requests.

![Basics of API architecture](/system-design/basics-of-scaling/basic-api-architecture.png)
![Horizontally scaled API servers](/system-design/basics-of-scaling/horizontally-scaled-api-servers.png)
![Load balancer included API servers](/system-design/basics-of-scaling/load-balancer-included.png)
![Vertically scaled API architecture](/system-design/basics-of-scaling/vertically-scaled-architecture.png)
![Replicated Database](/system-design/basics-of-scaling/replicated-database.png)
![Master slave replicated architecture](/system-design/basics-of-scaling/master-slave-replicated-architecture.png)
![Master slave replication](/system-design/basics-of-scaling/master-slave-replication.png)
![Network partitioned master slave 0](/system-design/basics-of-scaling/network-partitioned-master-slave-0.png)
![Network partitioned master slave 1](/system-design/basics-of-scaling/network-partitioned-master-slave-1.png)
![Network partitioned master slave 2](/system-design/basics-of-scaling/network-partitioned-master-slave-2.png)
