---
title: "CAP Theorem"
date: 2022-10-11T19:53:48+05:30
draft: false
---

In distributed systems, if there is a partition then CAP Theorem states that one can either have consistent data for all servers compromising availability or have all servers available with inconsistent data.

### Classical CA system

In the old days when the distributed systems were just a theory, all we had was one server and one client. Things were simple. Our client wrote and read data from the single server. There was no possibility of a partition as there was only one server. These systems were CA system.

![CA System](/system-design/cap-theorem/ca-system.png)

### Partition in Distributed Systems

After a few months we had to scale our system to serve clients in Asia and Europe. We decided to deploy multiple servers in Europe and Asia. This is how we ended up with a distributed system.

![Distributed System](/system-design/cap-theorem/distributed-system.png)

Everything was fine until one day a Network Partition occurred.

![Partitioned System](/system-design/cap-theorem/partitioned-system.png)

Wait a minute what's a Partition ?

In a distributed system a *Partition(or a Network Partition)* is when a set of servers are not able to communicate with another set of servers. In the above diagram we can see that Europe servers are not reachable to Asia servers anymore and vice versa.

This will create issues as updates to Asia servers cannot be propagated to Europe servers now and similarly updates to Europer servers cannot be propagated to Asia servers.

So how do we solve it ? If we think logically one way is to just not take any updates. This means all the servers will not take any request and thus affect Availability.

Another way is to allow the updates but not propagate them to the partitioned set of servers. Since the updates are not shared between the partitioned set of servers, the data will not be consistent among them thus affecting consistency.

We'll discuss real scenario next where we understand how to choose Availability or Consistency in case of a partition.

### AP system - Boxing match

*Scenario 1* - We've build a live streaming platform for boxing matches. Your team is responsible to show the watch count of the match. To make it scalable you've servers across Americas, Asia and Europe that stores the total watch count. Here's how your system looks like:

![Boxing Match Example](/system-design/cap-theorem/boxing-match-example.png)

During a live match the Asia servers gets partitioned from the rest of the system:

![Boxing Match Partitioned](/system-design/cap-theorem/boxing-match-partitioned.png)

Updates on the Asia servers are now not synced to Americas and Europe servers. What do you do to resolve this ?

Looking at the business use case we can speculate that there is no need for watch count to be most up to date all the time. If the watch count is inconsistent for a period of time then it'll not do any harm.

*This leads us to choose Availability over Consistency.* We'll allow watchers to be updated for Asia servers and the rest of the servers. This will cause the total watchers to be inconsistent but that's OK.

![AP system](/system-design/cap-theorem/ap-system.png)

When the network partition recovers, the two set of servers can share the number watchers since the last timestamp they're connected, they can add the diff to their counts. This is known as *Eventual Consistency*.

![Recovered AP system](/system-design/cap-theorem/recovered-ap-system.png)

### CP system - Bidding system

*Scenario 1* - We've build a bidding system for auction of valuable items. Your team is responsible to handle biddings from various auctions around the globe. To make that possible you've servers across Americas, Asia and Europe that can log bids for an auction. Here's how your system looks like:

![Bidding System Example](/system-design/cap-theorem/bidding-system-example.png)

The initial bid for the item is 1 million. Bidders from each continent can bid a higher value than the current bid. Now suddently the Asia servers gets partitioned from the network.

![Bidding System Partitioned](/system-design/cap-theorem/bidding-system-partitioned.png)

How will you resolve this ? Will you choose Availability or Consistency ?

Let's say for the sake of argument we choose Availability. In that case the bidders in Asia can also bid. Somebody in Asia bids 3 Million, this gets updated in Asia servers but not to the rest of the network. Europe and Americas bidder still see 1 Million. Now a European bidder bids 2 Million. The system allows this update, but hold on is this correct ?

![Bidding System Inconsistent](/system-design/cap-theorem/bidding-system-inconsistent.png)

There are two biddings now for the same item, Asia bidders see 3 Million while European and Americas bidders see 2 Million. The bidding for an item must always be one. This gives us the hint that the system must prioritize consistency.

To ensure consistency we can stop bidding altogether across the globe. We could also stop bidding only in Asia and allow bidding in Europe and Americas however that depends on the business use case.

![Bidding System Unavailable](/system-design/cap-theorem/bidding-system-unavailable.png)

Once the network partition recovers we will allow bidding again.

This time the Asian bidder will bid 3 Million, then the European bidder will see the new bid and won't bid 2 Million. It works because we chose Consistency over Availability.

![CP System](/system-design/cap-theorem/cp-system.png)

and that's all folks.
