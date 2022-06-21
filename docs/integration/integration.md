Contact Info: ducna36@ghtk.co
# Installation Guide(First Phase):
We will show osquery intergration and graylog configure to capture cmd dangerous on server.

Input: Logstash push log from Osquery_results file to kafka of graylog recive log

Processing: Graylog pipeline

Output: Alert

Pipeline:
file osquery_results log was collected by logstash, and push to input of graylog is kafka graylogip:9092 with topic: osquery.

At Graylog we create input, indices, stream, rule, pipeline, alert and notification


#1.1 Input:

Message inputs are responsible for accepting log messages in Graylog. Some default message types are available by default in Graylog. But you might need to install additional plugins to enable Graylog to receive particular messages.

We create input type: Raw/Plaintext Kafka on graylog to recive log

![](../../images/graylog/graylog1.png) 

![](../../images/graylog/graylog2.png) 

#1.2 Indices

We create indices to save message to elasticsearch
![](../../images/graylog/graylog3.png)

#1.3 Stream

Graylog streams are a mechanism that route messages into categories in real time while they are being processed. You can define rules in Graylog to route messages into certain streams

We create Stream to route message from indices to stream provide pipeline
![](../../images/graylog/graylog5.png)

![](../../images/graylog/graylog4.png)


#1.4 Rule and Pipeline 

Graylog’s new processing pipelines plugin allows greater flexibility in routing, blacklisting, modifying, andenriching messages as they flow through Graylog.

click system/pipeline then click pipeline /manage rules

- You can follow the detailed full documentation graylog rules **[HERE](https://docs.graylog.org/docs/rules/)**

For example: rule new crontab, log from queries from osquery/fleet:

```sh
select command, path from crontab;
```

![](../../images/graylog/graylog6.png)

After we created the rules, we have to add rules to pipelines to classify message throw rule. Click manage pipelines

![](../../images/graylog/graylog7.png)

#1.5 Alert & Notification

After message log match with rule. We have to create a alert, Event Definitions allow you to create Alerts from different Conditions and alert on them.

####Step 1: Event Details

After clicking on Create Event Definition we see the Event Definition create wizard. We are on the first page called Event Details. Here we enter title and description of our Event Definition. Also we define a priority. The priority is a tool for the user to add a classification to a event. It will be later displayed in the events overview as a thermometer:


![](../../images/graylog/graylog8.png)

####Step 2: Condition

Configure how Graylog should create Events of this kind. You can later use those Events as input on other Conditions, making it possible to build powerful Conditions based on other

As Condition Type we choose Filter & Aggregation. Below the selection will now the Filter section be available.

![](../../images/graylog/graylog9.png)

#### FILTER
First of all we choose the Stream in which our log files are routed. If no stream for the web application was created, we highly recommend to do that. That way the query result will be limited to the logs of the web application and no other logs can influence the filter process.

Now we need to filter the incoming messages by search query, so we can later count the messages which are matching the filter.

Now we set Search within the last to 1 minutes and Execute search every to 1 minutes. Now the Event engine will execute the query every 1 minutes for a time range of 1 minutes.

![](../../images/graylog/graylog10.png)

####AGGREGATION
Since we want to aggregate on our events to count that we have more then 10 messages in 10 seconds, we choose Aggregation of results reaches a threshold for Create Events for Definition if... and go on with aggregation.

To be able to count the failed login attempts per user, we need an extractor on our incoming messages which is extracting our user name and store it in a field called user. Now we can expect that every message with login failed has a field user. And we use that field in the selection Group by Field(s).

In the last step of that page we add one aggregation rule. If count() is >= 10.

To summarize what we have done here:

1. We add a stream to minimize the messages we have to filter on.

2. Insert a query to filter the logs down to our failed logins.

3. Grouped our logs so the aggregation will be only applied per user.

4. Add a rule which states that we only raise an alert if the count more than or equals 1

![](../../images/graylog/graylog11.png)

####Step 3: Fields

Here we can add a custom field to our event. These fields can be used for several things.

At this case, we create fields : alert_message to show message alert to notification

![](../../images/graylog/graylog12.png)

####Step 4: Notification

We want to receive an telegram message when the event got raised. Configuring a notification, will elevate the event to an alert
we have to create channel to notification, email, telegram http notification ....	

![](../../images/graylog/graylog15.png)

![](../../images/graylog/graylog13.png)

Since we use an aggregation event here, the message backlog might not be really helpful so I leave it off. The backlog will show all messages within the time range of Search within the last and use the Query we entered. If you have a good enough query this can still be helpful. The number input will limit the amount of messages in the backlog.
####Step 5: Summary

We go on to the summary to have a last look at our Event Definition.

![](../../images/graylog/graylog14.png)
