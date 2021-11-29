#  Simple Queue Service and Kinesis


## Simple Queue Service (SQS)

- SQS is a managed service that allows these different application components to pass messages back and forth. 

- SQS is highly available and elastic, capable of handling hundreds of thousands of messages per second.

### Queues

- SQS lets you create queues to hold messages for processing — a message can be up to 256 KB in size

- SQS uses the term *producer* to describe the component that places messages into the queue and *consumer* to describe the component that reads messages from the queue
- The process works as follows:
  1. A producer places one (or more messages) into a queue using the `SendMessage` action — messages in a queue are said to be *in-flight*
  
  2. A consumer checks or polls the queue for new messages and processes them using the `ReceiveMessage` action
  
     After the consumer processes a message, it deletes it from the queue using the `DeleteMessage` action

- SQS supports batching which enables a producers to perform actions on a batch of up to 10 messages at a time

#### Visibility Timeout

- When a consumers takes a message from a queue, the message remains in the queue until the message is deleted. 
- Once a consumer receives a message, SQS doesn't allow other consumers to read the same message for a period of time known as the  *visibility timeout* — this prevents multiple consumers processing the same message
- The default visibility timeout is 30 seconds, but can range from 0 seconds to 12 hours

#### Retention Period

- Messages can only be retain for a period of time.
- The default retention period is 4 days, but you can configure it to be anywhere from 1 minute to 14 days.

#### Delay Queues and Message Times

- Delay queues let you postpone the delivery of new messages to a queue for a number of seconds — the default is 0 seconds, and can be as long as 15 minutes
- Message timers let you specify an initial invisibility period for a message added to a queue — the default is 0 seconds, and can be as long as 15 minutes
- Settings a message timer will override the delay queue settings


### Queue Types

- SQS offers two different types of queues: standard and first-in, first-out (FIFO)

#### Standard Queues

- Standard queues (default type) offer almost unlimited throughput, and can have up to 120,000 in-flight messages
- Messages may be delivered out-of-order, and occasionally may be delivered more than once

#### First-In, First-Out (FIFO) Queues

- First-in, first-out (FIFO) queues support sending about 3,000 messages per second into a queue. Messages are only delivered once, in the order they were received. FIFO queues can handle about 20,000 in-flight messages.
- FIFO Queues can be divided up into message groups to maintain message ordering within a subset of messages in the queue  — this is particularly useful when you have multiple producers sending data into a queue, such as two temperature sensors sending data every 5 seconds
- In practice, using message groups is akin to having two separate queues

### Polling

- When receiving messages from a queue, you can use short polling (which is the default) or long polling
- In short polling, SQS checks only a subset of its servers for waiting messages. This gives you an immediate response, but it is possible to get an empty response when there are items in the queue, alas you will need to poll the queue multiple times to be sure.
- In long polling, SQS ensures that you receive any messages waiting in the queue. Each poll will check all the servers servicing the queue, and may take unto 20 seconds. Long polling is cheaper than short polling, as you pay per poll.

### Dead-Letter Queues

- When a message is held within a queue that is unable to be processed by multiple consumers, this is known as a *Dead Letter*.
- SQS can automatically move all *Dead Letter* messages from your queue and place them into a dead-letter queue after it’s been received so many times.
- To setup a dead letter queue, you'll need to ensure that it is of the same type as the source queue, and you must setup the `maxReceiveCount` value to be the maximum number of times a message can be received before it’s moved to the dead-letter queue.
- A dead-letter queue has it's own retention period, however, when a message is moved from a source queue to a dead-letter queue, the original creation date is kept, meaning that the retention period is also left intact.



## Kinesis

- Amazon Kinesis is a collection of services that let you collect, process, store, and deliver streaming data. 
- Kinesis can perform real-time ingestion of gigabytes per second from thousands of sources, making it perfect for things like audio and video feeds, application logs, and telemetry data.

### Kinesis Video Streams

- Kinesis Video Streams is designed to ingest and index almost unlimited amounts of streaming video data, such as from webcams, security cameras, and smartphones.
- Kinesis Video Streams uses a producer-consumer model. The data source that feeds data into a Kinesis stream is called the *producer*, and a *consumer* pulls data from a Kinesis video stream for playback or processing
- A producer may also send non-video data such as subtitles, audio or GPS coordinates as part of the stream
- A consumer maybe a video player, or a custom application running on EC2

### Kinesis Data Streams

- Kinesis Data Streams is a data pipeline that can aggregate, buffer, and reliably store data from producers until a consumer is ready to process it. 
- Consumers might include big data analytics applications such as MapReduce. 
- Kinesis Data Streams can rapidly ingest and store different types of binary data, including:
  - Application logs
  - Stock trades
  - Social media feeds
  - Financial transactions Location tracking information

- Producers feed data into a Kinesis Data Stream, and that data is stored in a data record. A consumer retrieves a data record using a partition key and sequence number. Kinesis Data Streams are indexed by the partition key and sequence number which allows for data to be retrieved in a particular order (rather than time based such as Video Streams)
- Multiple consumers can read from a stream concurrently, a process called *fan-out* — for example one process maybe scanning logs for suspicious activity, while another compresses the data for archival purposes
- The maximum throughput of a stream depends on the number of shards you configure — each shard supporting up to five read transactions per second, while write support unto 1,000 records per second.

### Kinesis Data Firehose

- Kinesis Data Firehose can ingest streaming data and transform it before sending it to a destination. Data transformation may include cleaning data or converting it to a different format
-  For example, you may need to change JSON-formatted data to Apache Parquet format before sending it to Hadoop. Kinesis Data Firehose uses Lambda functions to per- form the data transformation, giving you the flexibility to create custom transformations.
- Kinesis Data Firehose can send a copy of your untransformed data to an S3 bucket for safe keeping.

### Kinesis Data Firehose vs. Kinesis Data Streams

- Kinesis Data Streams uses a producer-consumer model, whereas Kinesis Video Streams and Kinesis Data Streams use a one-to-many model, allowing multiple consumers to subscribe to a single data stream.
- Kinesis Data Firehose allows for the ingesting of data and sends the data to one more more destinations. Kinesis Data Firehose is tightly integrated with managed AWS services such as Redshift, S3 or Splunk.

