# RabbitMQ Practical Examples

RabbitMQ is a message broker that use message queue internally

## 1) hello world

This Example simulate simple send and recieve message
![Basic Send Recieve](https://github.com/hossain-mo/rabbit-mq-practical-examples/blob/master/docs/basic_send_recieve.png?raw=true)

## 2) Work Queue

This Example simulate distribute time-consuming tasks among multiple workers

### Message acknowledgment
By default, RabbitMQ will send each message to the next consumer, in sequence. On average every consumer will get the same number of messages. This way of distributing messages is called round-robin.
If a consumer dies (its channel is closed, connection is closed, or TCP connection is lost) without sending an ack, RabbitMQ will understand that a message wasn't processed fully and will re-queue it. If there are other consumers online at the same time, it will then quickly redeliver it to another consumer. That way you can be sure that no message is lost, even if the workers occasionally die.

There aren't any message timeouts; RabbitMQ will redeliver the message when the consumer dies. It's fine even if processing a message takes a very, very long time.

Message acknowledgments were previously turned off by ourselves. It's time to turn them on by setting the fourth parameter to basic_consume to false (true means no ack) and send a proper acknowledgment from the worker, once we're done with a task.

### Message durability

We have learned how to make sure that even if the consumer dies, the task isn't lost. But our tasks will still be lost if RabbitMQ server stops.

When RabbitMQ quits or crashes it will forget the queues and messages unless you tell it not to. Two things are required to make sure that messages aren't lost: we need to mark both the queue and messages as durable.

First, we need to make sure that the queue will survive a RabbitMQ node restart. In order to do so, we need to declare it as durable. To do so we pass the third parameter to queue_declare as true:

### Fair dispatch 

You might have noticed that the dispatching still doesn't work exactly as we want. For example in a situation with two workers, when all odd messages are heavy and even messages are light, one worker will be constantly busy and the other one will do hardly any work. Well, RabbitMQ doesn't know anything about that and will still dispatch messages evenly.

This happens because RabbitMQ just dispatches a message when the message enters the queue. It doesn't look at the number of unacknowledged messages for a consumer. It just blindly dispatches every n-th message to the n-th consumer.

In order to defeat that we can use the basic_qos method with the prefetch_count = 1 setting. This tells RabbitMQ not to give more than one message to a worker at a time. Or, in other words, don't dispatch a new message to a worker until it has processed and acknowledged the previous one. Instead, it will dispatch it to the next worker that is not still busy.


![Work Queue](https://github.com/hossain-mo/rabbit-mq-practical-examples/blob/master/docs/work_queue.png?raw=true)



## 3) public-subscripe

we created a work queue. The assumption behind a work queue is that each task is delivered to exactly one worker. In this part we'll do something completely different -- we'll deliver a message to multiple consumers. This pattern is known as "publish/subscribe".
### Exchanges is the key
In previous parts of the tutorial we sent and received messages to and from a queue. Now it's time to introduce the full messaging model in Rabbit.

Let's quickly go over what we covered in the previous tutorials:

#### A producer is a user application that sends messages.
#### A queue is a buffer that stores messages.
#### A consumer is a user application that receives messages.
The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Actually, quite often the producer doesn't even know if a message will be delivered to any queue at all.

Instead, the producer can only send messages to an exchange. An exchange is a very simple thing. On one side it receives messages from producers and the other side it pushes them to queues. The exchange must know exactly what to do with a message it receives. Should it be appended to a particular queue? Should it be appended to many queues? Or should it get discarded. The rules for that are defined by the exchange type.


There are a few exchange types available: (direct, topic, headers and fanout) . We'll focus on the last one -- the fanout. Let's create an exchange of this type, and call it logs:

we use here fanout
![publish/subscripe](https://github.com/hossain-mo/rabbit-mq-practical-examples/blob/master/docs/exchanges.png?raw=true)

## 4) Routing
we're going to make it possible to subscribe only to a subset of the messages. For example, we will be able to direct only critical error messages to the log

we use here direct exchange

![publish/subscripe](https://github.com/hossain-mo/rabbit-mq-practical-examples/blob/master/docs/direct-exchange.png?raw=true)



## 5) Topic

Receiving messages based on a pattern
we're going to make it possible to subscribe only to a subset of the messages. For example, we will be able to direct only critical error messages to the log

we use here direct exchange

![publish/subscripe](https://github.com/hossain-mo/rabbit-mq-practical-examples/blob/master/docs/topic-exchange.png?raw=true)


## 6) Request Reply Pattern (RPC)

Receiving messages based on a pattern


![publish/subscripe](https://github.com/hossain-mo/rabbit-mq-practical-examples/blob/master/docs/rpc.png?raw=true)





