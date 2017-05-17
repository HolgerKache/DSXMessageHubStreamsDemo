
<img src="https://i0.wp.com/developer.ibm.com/messaging/wp-content/uploads/sites/18/2015/08/MessageHubBanner1.png?resize=562%2C229&ssl=1" alt="Drawing" style="width: 470px;"/>

# Receive message events from IBM Message Hub

In this python notebook, you will learn how to simulate streams by pushing sample messages to Message Hub which is currently based on Kafka 0.10.0.1. You will also consume these messages to perform analytics.

Message Hub is a cloud-based platform as a service technology predestined for streaming batch and real-time data to run analytics applications. Thereby you are able to gain valuable insights into your data. We will use the [Kafka Rest API](https://console.ng.bluemix.net/docs/services/MessageHub/messagehub025.html#messagehub025 "Documentation") which is designed to provide a quick start for beginners. Once you are finished you are ready to use the main [Kafka API](https://console.ng.bluemix.net/docs/services/MessageHub/messagehub050.html#messagehub050 "Documentation") which is used for high throughout use cases. Especially in the era of the Internet of Things there will be many application scenarios. So let's get started.

## 1. Basics

Message Hub acts like a real-time messaging system or data pipeline used for all incoming and outgoing communication. The most important terms are

- Topic
- Producer
- Consumer
- Cluster 

A producer sends a message to a specific topic within a Kafka cluster and a consumer receives this message by subscribing to this topic. 

A message can be addressed to a specific consumer group. The message is sent only to one consumer instance. The benefit is that the process of sending a message from end to end can be done in a shorter period of time. After the message is sent, the consumer instance shares the received message with the whole group. 

<CENTER><img src="http://kafka.apache.org/090/images/producer_consumer.png
" alt="Drawing" style="width: 275px;"/></CENTER>

(Img.1:  Basic architecture, [Kafka Documentation](http://kafka.apache.org/090/documentation.html "Documentation"))<br></br>

## 2. Setup and preparation

In this section you will create your own Message Hub instance and perform some prerequisites. 


```python
!pip install --upgrade requests
```

    Requirement already up-to-date: requests in /gpfs/global_fs01/sym_shared/YPProdSpark/user/s0d3-ac5d8d40b1dea6-f436cf316f63/.local/lib/python3.5/site-packages



```python
import requests 
```

Let’s get started. To test Message Hub no credit card is needed. If you have registered your account with a credit card <b>be aware</b> that costs can occur. 

1. Please navigate to [Bluemix](http://bluemix.net/ "Bluemix"). 
2. Login and click on catalog. 
3. Search for “Message Hub” in the Application Services section.
4. Create an instance 
5. In the "Manage" section create a topic name. The name will be needed later. 
6. Copy the Service Credentials without its {braces} and paste them into the next code snippet. 
7. Do not hesitate to use all code snippets in this blog as desired


```python
credentials ={
 "instance_id": "xxx",
  "mqlight_lookup_url": "https://mqlight-lookup-prod01.messagehub.services.us-south.bluemix.net/Lookup?serviceId=xxx",
  "api_key": "xxx",
  "kafka_admin_url": "https://kafka-admin-prod01.messagehub.services.us-south.bluemix.net:443",
  "kafka_rest_url": "https://kafka-rest-prod01.messagehub.services.us-south.bluemix.net:443",
  "kafka_brokers_sasl": [
    "kafka03-prod01.messagehub.services.us-south.bluemix.net:9093",
    "kafka04-prod01.messagehub.services.us-south.bluemix.net:9093",
    "kafka05-prod01.messagehub.services.us-south.bluemix.net:9093",
    "kafka01-prod01.messagehub.services.us-south.bluemix.net:9093",
    "kafka02-prod01.messagehub.services.us-south.bluemix.net:9093"
  ],
  "user": "xxx",
  "password": "xxx"}
```

## 3. Create a Kafka consumer group and consumer instance 

The core of this sample notebook are the get and post requests. There are four parts of our HTTP requests we are sending.

 - Method (Get and Post)
 - Url (defines the source of the destination)
 - Header (Needed for authentication)
 - Body (contains the data we want to submit)

A consumer group can contain one or more consumer instances. 


```python
kafkaTopic = 'json'
```


```python
consumerInstance = 'instance1'
consumerGroup = 'group1'

authToken = credentials['api_key']
kafkaRestUrl = credentials['kafka_rest_url']
```

The header has to be defined one time only and will be used for all upcoming requests.


```python
import json

headers = {
    'X-Auth-Token': authToken,
    'Content-Type': 'application/vnd.kafka.v1+json'
}
```

The consumer instance will be sent within the body. 


```python
body1 = json.dumps({
    'name': consumerInstance,
    'format': 'binary',
    'auto.offset.reset': 'smallest'
    })
```

The following method sends a post request including both the authentication and consumer group details. 


```python
def setConsumerInstanceAndGroup(consumerGroup, body1, headers):
    
    response = requests.post(kafkaRestUrl + "/consumers/" + consumerGroup, data=body1, headers=headers)

    print(response.status_code, response.reason, response.text)
    result = response.json()
    print(result)
    consumerUrl = result['base_uri']
    print(consumerUrl)
 
    return response
```

This method has to be executed one time only. <b>Existing consumer group/instance combinations will cause a 409 conflict</b>. Change the group or instance name in that case and repeat.


```python
setConsumerInstanceAndGroup(consumerGroup, body1, headers)
```

    200 OK {"instance_id":"instance1","base_uri":"https://kafka-rest-prod01.messagehub.services.us-south.bluemix.net/consumers/group1/instances/instance1"}
    {'base_uri': 'https://kafka-rest-prod01.messagehub.services.us-south.bluemix.net/consumers/group1/instances/instance1', 'instance_id': 'instance1'}
    https://kafka-rest-prod01.messagehub.services.us-south.bluemix.net/consumers/group1/instances/instance1





    <Response [200]>



## 4. Push a message event to Kafka 

Here you can set the name of the topic. The name must be the same you have created manually in your Message Hub instance before. The first letter of the topic name should be in upper case.


```python
url = kafkaRestUrl + "/topics/" + kafkaTopic
```

Here you can set some messages. Edit the message and execute the method more often to produce more events.

_Note: The values submitted in the records array have to be binary encoded for our consumer instance. JSON or Avro encoding is not currently supported with Message Hub REST API.
To encode the value as binary, they have to be kept in byte arrays (not strings) and be decoded from the receiver into String values. _


```python
import binascii

body2 = json.dumps({
    'records': [
            {'value': binascii.hexlify(b"Mercury ").decode('utf-8')},
            {'value': binascii.hexlify(b"Venus   ").decode('utf-8')},
            {'value': binascii.hexlify(b"Earth   ").decode('utf-8')},
            {'value': binascii.hexlify(b"Mars    ").decode('utf-8')},
            {'value': binascii.hexlify(b"Jupiter ").decode('utf-8')},
            {'value': binascii.hexlify(b"Saturn  ").decode('utf-8')},
            {'value': binascii.hexlify(b"Uranus  ").decode('utf-8')},
            {'value': binascii.hexlify(b"Neptune ").decode('utf-8')}
        ]
    }, ensure_ascii=False).encode('utf8')

print(body2)
```

    b'{"records": [{"value": "4d65726375727920"}, {"value": "56656e7573202020"}, {"value": "4561727468202020"}, {"value": "4d61727320202020"}, {"value": "4a75706974657220"}, {"value": "53617475726e2020"}, {"value": "5572616e75732020"}, {"value": "4e657074756e6520"}]}'



```python
def pushMessageToKafka(kafkaTopic, url, body2, headers):
    
    response = requests.post(url, data=body2, headers=headers)
    print(body2)
     
    print(response.status_code, response.reason, response.text)
    
    return response
```

To send this message, execute this method one or more times. Feel free to go back and <b>change the text of the message and execute this method again</b> to simulate a data stream. 


```python
pushMessageToKafka(kafkaTopic, url, body2, headers)
```

    b'{"records": [{"value": "4d65726375727920"}, {"value": "56656e7573202020"}, {"value": "4561727468202020"}, {"value": "4d61727320202020"}, {"value": "4a75706974657220"}, {"value": "53617475726e2020"}, {"value": "5572616e75732020"}, {"value": "4e657074756e6520"}]}'
    200 OK {"offsets":[{"partition":0,"offset":137,"error_code":null,"error":null},{"partition":0,"offset":138,"error_code":null,"error":null},{"partition":0,"offset":139,"error_code":null,"error":null},{"partition":0,"offset":140,"error_code":null,"error":null},{"partition":0,"offset":141,"error_code":null,"error":null},{"partition":0,"offset":142,"error_code":null,"error":null},{"partition":0,"offset":143,"error_code":null,"error":null},{"partition":0,"offset":144,"error_code":null,"error":null}],"key_schema_id":null,"value_schema_id":null}





    <Response [200]>



Each time, the message is assigned to a partition uniquely identified by an ID called <b>offset</b>. 

<CENTER><img src="https://kafka.apache.org/0102/images/log_consumer.png" alt="Drawing" style="width: 400px;"/></CENTER>
Source: Sample partition https://kafka.apache.org/documentation/#introduction

## 5. Receive a message event from Kafka

The loop below will run for some amount of time but we have to eventually stop it to process the results. What we have here is not an actual streaming receiver as needed for Spark Streaming. We only demonstrate Spark core functionality here by getting messages from Message Hub loaded into a Dataframe. 

Now it’s time to receive our messages by defining the following Method. 


```python
from pprint import pprint
import time
import base64

def getMessageFromKafka(maxArrayLength, maxIterations, consumerUrl, headers):
    results = []
    length = 0
    iteration = 0
    while (length < maxArrayLength):
        if (iteration > maxIterations): break
        
        response = requests.get(kafkaRestUrl + "/consumers/"+consumerGroup+"/instances/"+consumerInstance+"/topics/"+kafkaTopic, headers=headers)        
        print (response, response.reason, response.text)

        data = response.text

        x = json.loads(data)
        length = length + len(x)
        iteration = iteration + 1
        
        print ('===============================')
        print ('Number of incoming messages: ', len(x))
        print ('===============================')
        
        for obj in x:
            value = binascii.unhexlify(obj['value']).decode('utf-8')
        
            print(value)
            results.append(value)
            
    return results
```

Finally execute the following method to receive all sent message events. Other parameters are set to limit the length of the message array and the number of iterations. 


```python
maxArrayLength = 2000
maxIterations = 1

headers2 = {
    'X-Auth-Token': authToken,
    'Accept': 'application/vnd.binary.v1+json'
}

results = getMessageFromKafka (maxArrayLength, maxIterations, url, headers2)
```

    <Response [200]> OK [{"key":null,"value":"4d65726375727920","partition":0,"offset":137},{"key":null,"value":"56656e7573202020","partition":0,"offset":138},{"key":null,"value":"4561727468202020","partition":0,"offset":139},{"key":null,"value":"4d61727320202020","partition":0,"offset":140},{"key":null,"value":"4a75706974657220","partition":0,"offset":141},{"key":null,"value":"53617475726e2020","partition":0,"offset":142},{"key":null,"value":"5572616e75732020","partition":0,"offset":143},{"key":null,"value":"4e657074756e6520","partition":0,"offset":144}]
    ===============================
    Number of incoming messages:  8
    ===============================
    Mercury 
    Venus   
    Earth   
    Mars    
    Jupiter 
    Saturn  
    Uranus  
    Neptune 
    <Response [200]> OK []
    ===============================
    Number of incoming messages:  0
    ===============================


The reading of the data starts from partition 0 beginning with the lowest ID (offset).

## 6. Delete the Kafka consumer instance

Next we will drop the consumer instance so we avoid the 409 conflicts the next time we try to create an instance with the same name.


```python
def deleteConsumerInstance(consumerGroup, consumerInstance, headers):
    
    response = requests.delete(kafkaRestUrl + "/consumers/" + consumerGroup + "/instances/" + consumerInstance,
                               headers=headers)

    print(response.status_code, response.reason, response.text)
    return response
```

The expected response from this call is a 204 (No Content) message.


```python
deleteConsumerInstance(consumerGroup, consumerInstance, headers)
```

    204 No Content 





    <Response [204]>



## 7. Perform analytics to gain insights 

Once we received the data we are free to choose what we want to do with it. In this case we will create a simple tag cloud in Brunel. The message of the event is extracted and stored in the following list.


```python
results
```




    ['Mercury ',
     'Venus   ',
     'Earth   ',
     'Mars    ',
     'Jupiter ',
     'Saturn  ',
     'Uranus  ',
     'Neptune ']



Next we transform the result into a Pandas dataframe. All we want is to construct a generic dataframe so we can inspect the data we got from the topic.

If index 0 is empty please execute the method again until the first index contains a word. 


```python
import pandas as pd

results_pd = pd.DataFrame(results, columns=["Message"])

pd.set_option('display.max_columns', 500)
results_pd.head(20)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Message</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mercury</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Venus</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Earth</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mars</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Jupiter</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Saturn</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Uranus</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Neptune</td>
    </tr>
  </tbody>
</table>
</div>



After creating a pandas dataframe we can do everything supported in Spark, including Machine Learning, Visualization, Graphing, etc.


```python
results_pd.to_csv('results_pd',encoding='utf-8')
```


```python
import brunel

%brunel cloud data('results_pd') label(Message) :: width=700, height=400
```


<!--
  ~ Copyright (c) 2015 IBM Corporation and others.
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ You may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->


<link rel="stylesheet" type="text/css" href="/data/jupyter2/7106bd0a-f68e-461a-a0d3-ac5d8d40b1de/nbextensions/brunel_ext/brunel.2.3.css">
<link rel="stylesheet" type="text/css" href="/data/jupyter2/7106bd0a-f68e-461a-a0d3-ac5d8d40b1de/nbextensions/brunel_ext/sumoselect.css">

<style>
    
</style>

<div id="controlsid7722f66c-370b-11e7-bcfb-002590fb6e1c" class="brunel"/>
<svg id="visid7722f3ba-370b-11e7-bcfb-002590fb6e1c" width="700" height="400"></svg>





    <IPython.core.display.Javascript object>




```python

```


```python

```
