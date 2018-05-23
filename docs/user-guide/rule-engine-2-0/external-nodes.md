---
layout: docwithnav
title: External Nodes
description: Rule Engine 2.0 External Nodes

---

External Nodes used are used to interact with external systems.

* TOC
{:toc}


# AWS SNS Node

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sns.png)

Node publish messages to AWS SNS (Amazon Simple Notification Service).

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sns-config.png)

- **Topic ARN pattern** - can be set direct topic name for message publishing 
or pattern can be used, that will be resolved to the real ARN Topic name using Message metadata. 
- **AWS Access Key ID** and **AWS Secret Access Key** are the credentials of an AWS IAM User with programmatic access. More information on AWS access keys can be found [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html). 
- **AWS Region** must correspond to the one in which the SNS Topic(s) are created. Current list of AWS Regions can be found [here](http://docs.aws.amazon.com/general/latest/gr/rande.html).

In the following example, topic name depends on Device Type and there is a Message that contains **deviceType** field in Metadata:
{% highlight javascript %}
{
    deviceType: controller
}
{% endhighlight %}

For publishing message in **controller**'s topic, we will set this pattern in **Topic ARN pattern**:

{% highlight bash %}
arn:aws:sns:us-east-1:123456789012:${deviceType}
{% endhighlight %}

In runtime, pattern will be resolved to <code>arn:aws:sns:us-east-1:123456789012:controller</code>

**Published payload** - Node will publish full Message payload to the SNS.
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the SNS.

**Outbound message** from this node will contain response **messageId** and **requestId**
 in Message metadata. Original Message payload, type and originator will not be changed.

<br/>

# AWS SQS Node

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sqs.png)

Node publish messages to the AWS SQS (Amazon Simple Queue Service).

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-aws-sqs-config.png)

- **Queue Type** - SQS queue type. Can be *Standard* or *FIFO*.
- **Queue URL Pattern** - Pattern for building Queue URL. For example <code>${deviceType}</code>.
Can be set direct Queue URL for message publishing or pattern can be used, that will be resolved to the real Queue URL using Message metadata.
- **Delay** - delay in seconds, used to delay a specific message.
- **Message attributes** - optional list of message attributes to publish.
- **AWS Access Key ID** and **AWS Secret Access Key** are the credentials of an AWS IAM User with programmatic access. More information on AWS access keys can be found [here](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html). 
- **AWS Region** must correspond to the one in which the SQS Queue(s) are created. Current list of AWS Regions can be found [here](http://docs.aws.amazon.com/general/latest/gr/rande.html).

In the following example, Queue URL depends on Device Type and there is a Message that contains **deviceType** field in Metadata:

{% highlight javascript %}
{
    deviceType: controller
}
{% endhighlight %}

For publishing message in **controller**'s Queue, we will set this pattern in **Queue URL pattern**:

{% highlight bash %}
https://sqs.us-east-1.amazonaws.com/123456789012/${deviceType}
{% endhighlight %}

In runtime, pattern will be resolved to <code>https://sqs.us-east-1.amazonaws.com/123456789012/controller</code>

**Published body** - Node will publish full Message payload to the SQS. 
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the SQS.

**Published attributes** - optional list of attributes can be added for publishing message in SQS. It is a collection of <NAME> - <VALUE> pairs.
Both, NAME and VALUE, could be a static values or patterns that will be resolved using Message metadata.

If **FIFO** queue is selected, then Message ID will be used as **deduplication ID** and Message originator as **group ID**.

**Outbound message** from this node will contain response **messageId**, **requestId**, **messageBodyMd5**, **messageAttributesMd5** 
and **sequenceNumber** in Message metadata. Original Message payload, type and originator will not be changed. 

<br/>

# Kafka Node

![image](/images/user-guide/rule-engine-2-0/nodes/external-kafka.png)

Kafka Node sends messages to Kafka brokers. Expects messages with any message type. Will send record via Kafka producer to Kafka server.

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-kafka-config.png)

- **Topic pattern** - can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code>
- **bootstrap servers** - list of kafka brokers separated with comma.
- **Automatically retry times** - number of attempts to resend message if connection fails.
- **Produces batch size** - batch size in bytes for grouping messages with the same partition.
- **Time to buffer locally** - max local buffering window duration in ms.
- **Client buffer max size** - max buffer size in bytes for sending messages.
- **Number of acknowledgments** - number of acknowledgments node requires to received before considering a request complete.
- **Key serializer** - by default org.apache.kafka.common.serialization.StringSerializer
- **Value serializer** - by default org.apache.kafka.common.serialization.StringSerializer
- **Other properties** - any other additional properties could be provided for kafka broker connection.

**Published body** - Node will send full Message payload to the Kafka topic. 
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the Kafka.

**Outbound message** from this node will contain response **offset**, **partition** and **topic** properties in the Message metadata. 
Original Message payload, type and originator will not be changed.

<br/>

# MQTT Node

![image](/images/user-guide/rule-engine-2-0/nodes/external-mqtt.png)

Publish incoming message payload to the topic of the configured MQTT broker with QoS **AT_LEAST_ONCE**. 

Configuration:

![image](/images/user-guide/rule-engine-2-0/nodes/external-mqtt-config.png)

- **Topic pattern** - can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code>.
- **Host** - MQTT broker host.
- **Port** - MQTT broker port.
- **Connection timeout** - timeout in seconds for connecting to MQTT broker.
- **Client ID** - optional client identifier used for connecting to MQTT broker. If not specified, default generated clientId will be used.
- **SSL Enable/Disable** - enable/disable secure communication.  
- **Credentials** - MQTT connection credentials. Can be either *Anonymous*, *Basic* or *PEM*.

Different Authentication credentials are supported for external MQTT broker:

- Anonymous - no authentication
- Basic - username\password pair is used for authenticating
- PEM - PEM certificates are used for Authentication

If **PEM** credentials type is selected, the following configuration should be provided:

- CA certificate file
- Certificate file
- Private key file
- Private key password

<br/>

**Published body** - Node will send full Message payload to the MQTT topic.
If required, Rule Chain can be configured to use chain of Transformation Nodes for sending correct Payload to the MQTT broker.

In case of successful message publishing, original Message will be passed to the next nodes via **Success** chain, 
otherwise **Failure** chain is used.

<br/>

# RabbitMQ Node
Publish incoming message payload to the RabbitMQ.

Configuration parameters:

- **Exchange name pattern** - the exchange to publish the message to. Can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code> .
- **Routing key pattern** - the routing key. Can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code> .
- **Message properties** - routing headers. Supported headers **{BASIC \| TEXT_PLAIN \| MINIMAL_BASIC \| MINIMAL_PERSISTENT_BASIC \| PERSISTENT_BASIC \| PERSISTENT_TEXT_PLAIN}**
- **Host**
- **Port**
- **Virtual host** - the virtual host to use when connecting to the broker
- **Username** - AMQP user name to use when connecting to the broker
- **Password** - AMQP password to use when connecting to the broker
- **Automatic recovery** - enables or disables automatic connection recovery
- **Connection timeout** - timeout in seconds for connecting to MQTT broker
- **Handshake timeout** - timeout in seconds for connecting to MQTT broker
- **Client properties** - additional properties that are sent to the server during connection startup 

**Published body** - Node will send full Message payload to the RabbitMQ. If required, Administrator can configure Chain of Transformation Nodes for sending correct Payload.

In case of successful message publishing, original Message will be passed to the next nodes via **Successful** chain, 
otherwise **Failure** chain is used.

<br/>

# Rest API Call Node
Invoke REST API calls to the external REST server.

Configuration parameters:

- **Endpoint URL pattern** - Can be a static string, or pattern that is resolved using Message Metadata properties. For example <code>${deviceType}</code>
- **Request method** - **{GET \| POST \| PUT \| DELETE}**
- **Headers** - request headers, constructed from Message Metadata

**Endpoint URL**

URL can be a static string or a pattern. Only Message metadata is used for resolving patterns. 
So property names that are used in the patterns must exist in the Message Metadata, otherwise raw pattern will be added into header.

For example, if Message payload contains property **deviceType** with value **container**, then this pattern: 

<code>http://localhost/api/${deviceType}/update</code> 

will be resolved to 

<code>http://localhost/api/container/update</code>   

**Headers**

Collection of header name/value can be configured. Those headers will be added into Rest request. Pattern should be used for configured both header name and header value.
For example <code>${deviceType}</code>. Only Message metadata is used for resolving patterns. 
So property names that are used in the pattern must exist in the Message Metadata, otherwise raw pattern will be added into header. 

**Request body** - Node will send full Message payload to the configured REST endpoint. If required, Administrator can configure Chain of Transformation Nodes for sending correct Payload.

*Outbound message* from this node will contain responce **status**, **statusCode**, **statusReason** and responce **headers** in the Message metadata.
Outbound Message payload will be the same as responce body. Original Message type and originator will not be changed.

In case of successful request, outbound message will be passed to the next nodes via **Successful** chain, 
otherwise **Failure** chain is used.

**!!! TODO-RE - add link to tutorial**
{: style="color:red" }

<br/>

# Send Email Node
Node sends incoming message using configured Mail Server. This Node works only with messages that where created using 
**to Email** transformation Node, please connect this Node with **to Email** Node using **Successful** chain.

Configuration parameters:

- **Use system SMTP settings**
- **SMTP protocol** - **{SMTP \| SMTPS}**
- **SMTP host**
- **SMTP port**
- **Timeout ms**
- **Enable TLS** - Enable \ Disable TLS
- **Username** -
- **Password**

This Node can work with default System SMTP Settings.
Please find more details about [how to configure default System SMTP Settings.](/docs/user-guide/ui/mail-settings/)

If another SMTP server required for this node - disable **Use system SMTP settings** checkbox and configure SMTP server manually.

In case of successful mail sending, original Message will be passed to the next nodes via **Successful** chain, 
otherwise **Failure** chain is used.

**!!! TODO-RE - add link to tutorial**
{: style="color:red" }
