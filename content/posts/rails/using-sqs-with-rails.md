---
title: "Using SQS With Rails"
date: 2022-08-21T17:07:33+05:30
tags: ["Ruby", "DevOps", "Rails", "SQS", "AWS"]
draft: false
---

In this post we're going to learn how to use SQS with Rails.

### What's SQS ?

SQS is a message queueing service provided by AWS. It can be used by two microservices to communicate using messages one acting as producer and the other as consumer. Using SQS, you can send, store, and receive messages between microservices while allowing them to scale horizontally.

SQS stores the messages send to it thus making sure messages are not lost. This also ensures that any downtime in interacting microservices doesn't affect the messages pushed into SQS.

### Setting up AWS SQS in Rails

AWS provides an official [gem] (https://github.com/aws/aws-sdk-ruby) for ruby applications. With V3 the gem has separate gem for each AWS service. Since we're using SQS we're going to use `aws-sdk-sqs` gem.

To use the gem add the following line to your Gemfile:

```ruby
gem "aws-sdk-sqs", '1.51.1'
```

You can change the version to the latest version available [here] (https://rubygems.org/gems/aws-sdk-sqs/). Run `bundle install` to install the gem.

Now that we've installed the gem we need to authenticate our Rails app to our AWS account. To do so we'll need to set the env variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.

After that initialize your SQS client like this:

```ruby
creds = Aws::Credentials.new(ENV['AWS_ACCESS_KEY_ID'], ENV['AWS_SECRET_ACCESS_KEY'])
sqs = Aws::SQS::Client.new(region: 'ap-south-1', credentials: creds)
```

You'll need to set the region in which the SQS queue is being created. In the above code we're using `ap-south-1` as the AWS region.

### Creating SQS Queues

We'll create a new SQS queue using sqs client we initialized in the previous code. AWS sdk provides `create_queue` method to create a SQS queue:

```ruby
sqs.create_queue(
  queue_name: 'test',
  attributes: { VisibilityTimeout: '10', MaximumMessageSize: '131072' }
)
```

In the creation we've additionally set two more configs:

1. Visibility timeout - Reading a message from SQS queue doesn't delete it. However, for a certain time it's not visible to the other clients trying to read the same message. This is set by setting the VisibilityTimeout. By default it's 30 seconds.
2. Maximum message size - There is limit to the size of messages we can send to SQS. This is specified by MaximumMessageSize config. By default the MaximumMessageSize is 256KB.

You'll be able to see the new queue in the AWS SQS queue dashboard:

![create-queue](/using-sqs-with-rails/create-queue.png)

### Sending message to SQS Queue

Let's send a message to our newly created SQS queue. To do so we'll use the `send_message` method.

You'll need the SQS queue's URL to send the message. Click on the queue in AWS SQS dashboard and copy the URL from there:

![queue-uri](/using-sqs-with-rails/queue-uri.png)

Pass the queue url to `send_message` method to send a message to the queue:

```ruby
queue_url = 'https://sqs.ap-south-1.amazonaws.com/153084803934/test'

sqs.send_message({
  queue_url: queue_url,
  message_body: "Hello World",
})
```

Let's check if the message was received by polling the queue. You can go to the queue page and click on Poll Message to receive the message:

![poll-message](/using-sqs-with-rails/poll-message.png)

The message can be seen. Also notice that the ReceiveCount of the message will be 1. The ReceiveCount signifies the number of times this message has been read by some client.

### Receiving message from SQS Queue

We received a message previously via the UI. Let's do it programmatically using the `receive_message` method provided by the AWS sdk:

```ruby
queue_url = 'https://sqs.ap-south-1.amazonaws.com/153084803934/test'

sqs.receive_message({
  queue_url: queue_url,
  max_number_of_messages: '1',
})

=>
#<struct Aws::SQS::Types::ReceiveMessageResult
 messages=
  [#<struct Aws::SQS::Types::Message
    message_id="1e78bd85-e35e-4dfd-924e-b33377e2943b",
    receipt_handle=
     "AQEBRQybsIx0ll8B........KGxLjomGZuGg53LbV6mf",
    md5_of_body="b10a8db164e0754105b7a99be72e3fe5",
    body="Hello World",
    attributes={},
    md5_of_message_attributes=nil,
    message_attributes={}>]>
```

This will return a ruby object. We can get the message from it by calling the body method of the messages property:

```ruby
result = sqs.receive_message({
  queue_url: queue_url,
  max_number_of_messages: '1',
})

puts result.messages[0].body
```

### What's polling ?

To receive messages from an SQS queue we've to keep checking if there are any new messages sent to the queue, this process is called polling. The time till which the client checks for new messages in a queue is defined by `wait_time_seconds` attribute set in `receive_message`. Here's an example:

```ruby
result = sqs.receive_message({
  queue_url: queue_url,
  max_number_of_messages: '5',
  wait_time_seconds: '20',
})
```

Here we've set receive_message to poll for 20 seconds for any new messages. According to the length the `wait_time_seconds` you can achieve two kinds of polling:

1. Long polling - when the polling time is high it's called long polling. This allows more messages to be received since the polling time is more and is more cost effective.
2. Short polling - by default the `wait_time_seconds` is 0 for `receive_message`. This will check if any new messages are present in the queue and return them immediately. This is known as short polling. It's inefficient since you'll get more empty responses.

Best practice is to go for long polling. It can also enabled by default when creating a queue:

```ruby
sqs.create_queue(
  queue_name: 'test',
  attributes: {
    VisibilityTimeout: '10',
    MaximumMessageSize: '131072',
    ReceiveMessageWaitTimeSeconds: '20',
  }
)
```

Setting `ReceiveMessageWaitTimeSeconds` will make sure the `receive_message` waits for 20 seconds by default.

### Deleting messages

A message can be deleted using `delete_message` method. Note that receiving a message doesn't delete it from the SQS queue so if you've processed the message and don't want to reprocess it by some other client, then delete it from the SQS queue.

The `receipt_handle` can be found in result object we received when receiving the message, you can call `results.messages[0].receipt_handle` to get it:

```ruby
queue_url = 'https://sqs.ap-south-1.amazonaws.com/153084803934/test'

sqs.receive_message({
  queue_url: queue_url,
  max_number_of_messages: '1',
})

=>
#<struct Aws::SQS::Types::ReceiveMessageResult
 messages=
  [#<struct Aws::SQS::Types::Message
    message_id="1e78bd85-e35e-4dfd-924e-b33377e2943b",
    receipt_handle=
     "AQEBRQybsIx0ll8B........KGxLjomGZuGg53LbV6mf",
    md5_of_body="b10a8db164e0754105b7a99be72e3fe5",
    body="Hello World",
    attributes={},
    md5_of_message_attributes=nil,
    message_attributes={}>]>
```

Then use the `receipt_handle` and pass it to the `delete_message` method.

```ruby
sqs.delete_message({
  queue_url: queue_url,
  receipt_handle: result.messages[0].receipt_handle,
})
```

### Deleting queue

Wrapping it up we'll clean up the queue we created. We can do that by using `delete_queue`:

```ruby
queue_url = 'https://sqs.ap-south-1.amazonaws.com/153084803934/test'

sqs.delete_queue(queue_url: queue_url)
```
