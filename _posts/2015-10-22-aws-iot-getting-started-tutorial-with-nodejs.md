---
layout: post
published: true
title: AWS IoT - Getting started tutorial with node.js
description: "Getting started tutorial with node.js on the new AWS IoT service"
tags: [cloud drive,cloud gaming,free cloud server,ec2 pricing,cloud security alliance,free cloud computing,free cloud hosting,amazon ec2 pricing,cloud desktop,aws security,free cloud services,amazon s3,aws cloud,amazon s3 pricing,amazon ec2,the internet of things,aws console,aws services,cloud services,aws pricing,amazon s3 storage,aws ec2,amazon s3 backup,amazon storage,aws s3,ibm cloud,identity and access management,ec2 amazon,aws marketplace,identity access management,cloud connect,aws redshift,iot devices,cloud pc,aws rds,infrastructure as a service,virtual private cloud,hp cloud,aws internet of things,cloud database,internet of things security,linux cloud,what is the internet of things,aws elastic beanstalk,aws cognito,best cloud service,aws ses,aws device farm,iot applications,free cloud service,windows cloud computing,cloud computing free,cloud computing news,cloud computing tutorial,aws cloud provider,cloud computing certification,cloud computing leaders,aws aws aws,cloud aws,connected devices internet of things,aws cloud services,cheap cloud hosting,internet of things deutsch,cloud computing aws,aws applications,aws app development,internet of things devices connected,aws what is cloud computing,windows cloud server,free cloud computing services,aws databases,aws database,aws cloud hosting,cloud computing windows,internet of things cloud platform,aws cloud platform,aws application,aws service,database aws,aws in cloud computing,aws cloud computing,storage aws,host website aws,internet of things sdk,cloud hosting services,cloud computing with aws,ec2 training,security in aws,aws cloud storage,secure aws,aws computing,aws apps,aws for mobile apps,aws hosting,aws cloud solutions,cloud storage aws,aws site hosting,aws host website,ibm cloud services]
image: true
---

During his 2015 edition if the re:Invent show, AWS has released a new tool dedicated to our electronic devices.

The tool is a pub/sub system that includes a full SDK, TLS connections, and more...

Let's get started with a simple example and run our first application with the node.js platform (used by the [Beaglebone](http://beagleboard.org/black) for instance).

We first need to create our first project. As we're speaking about the internet-of-*thing* topic, AWS decided to call the projects *things*. Let's respect that super creativity by picking a super original name : `Tutorial`

![Create a new project][2]

You now have your first *thing* in your dashboard :

![You now have a thing in your list][3]

To be able to publish or receive any messages, we have to create a device into our project. To do it, we will click on our *thing* and select `Connect a device`

![Add a device to your thing][4]

Here, the wizard will ask you on which platform you want to develop your application, in our case we are gonna use *node.js*. The wizard will then give you the ability to download your certificates.

**!! Download those three ones now, as you'll not have the ability later !!**

Yes, as AWS does for all the credentials, it's a one time access, which makes sense on a security perspective.

![Select your platform and download your certificates][5]

You will also need a root certificate (also called CA certificate) to be able to open the *TLS* connection with the server. [This one is available on Verisign website](https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem). Download this file and name it `root-CA.crt`

Let's clean a bit and rename/move our certificate to our project folder. The names that I'll use in this example are:

`certificate.pem.crt`

`private.pem.key`

`public.pem.key`

`root-CA.crt`

If you want to copy/paste the code, use the same names.

![Rename and put those certificates in a folder][6]

We now have everything ready on AWS side, and you should see three items in your dashboard, a *thing*, a *certificate* and an *access policy*:

![You should now have three items][7]

The last step of the setup will be to install the required SDK. AWS has released a *npm* package for us:

```bash
# npm i aws-iot-device-sdk

```

I did create a `test.js` file in the project folder and required the sdk at the top:

```javascript
var awsIot = require('aws-iot-device-sdk');
```

Now that all our setup is ready, let's write our first lines of code!

We first need to define our *device*. To do it, we have to create a device object.

```javascript
var device = awsIot.device({
   keyPath: './certs/private.pem.key',
  certPath: './certs/certificate.pem.crt',
    caPath: './certs/root-CA.crt',
  clientId: 'Tutorial',
    region: 'eu-west-1'
});
```
Do not forget to adapt your region.

Now that we have our device object, let's act on the connection event that is triggered when the connection with the AWS IoT server has succeed:

```javascript
device
  .on('connect', function() {
    console.log('connected');
  });
```

At this step, if we run our application, we will see that it is correctly connecting to the AWS IoT servers.

```bash
# node test.js
connected
```


Ok, this is a good start, but let's do something a bit more sexy.

AWS IoT offers us a messaging queue service that allows us to publish message over topics and receive message from topic we have subscribed to.

We can also define actions on AWS side for messages that have been published on specific topics.

Let's subscribe to our first topic:

```javascript
device
  .on('connect', function() {
    console.log('connected');
    device.subscribe('topic_1', function(error, result) {
      console.log(result);
    });
  });
```

If we run the application, we receive back the confirmation that we have correctly subscribed to the `topic_1` (name that we have choose arbitrarily)

```bash
# node test.js
connected
[ { topic: 'topic_1', qos: 0 } ]
```

We will also need a piece of logic that will display a message when one will be published on the topic:

```javascript
device
  .on('message', function(topic, payload) {
    console.log('message', topic, payload.toString());
  });
```

You can run your application, but you will not see anything unexpected here. Indeed a message has to be published to make our application reacts.

Unfortunately, AWS doesn't offer yet the ability to publish a message from the interface like they do for SQS, we will then need to publish a message ourself if we want to see something happening. We gonna take that as a opportunity to discover the third concept of those AWS IoT features, the *rules*.

Those give us the ability to define an action when the server receives a message.

Let's create our first one, to do it, you have to click on the `Create a ressource` button:

![Let's add a new ressource][8]

Then you will have to select the *rule* type.

![Pick up the rule item in the menu][9]

We gonna write the query to match to any messages published to our new queue `topic_2`:

![Let's pick up all messages from topic 2][10]

And we will pick the republishing action. That way we'll be able to republish our messages to the queue we have subscribed to : `topic_1`

![republish messages to an other queue][11]

![Let's use the queue we've subscribed to][12]

To be able to publish to our queue, we need permissions, which means creating a new policy in the IAM. Hopefully, AWS helps us here by automating the process. Indeed, if you click on `Create a new role` you are redirected to a pre-filled policy generator:

![We have to create a policy to have access][13]

Just validate it as it is and it will be automatically added to our *rule*:

![The policy is automatically added][14]

Now, just click on `Add Action` and you'll be able to finish the *rule* creation process:

![Your action is added][15]

Now, let's publish a message to that topic and read what we get in return:

```javascript
device
  .on('connect', function() {
    console.log('connected');
    device.subscribe('topic_1');
    device.publish('topic_1', JSON.stringify({ test_data: 2}));
  });
```

Then if we run our application, we receive back our message in return:

```bash
# node test.js
connected
[ { topic: 'topic_1', qos: 0 } ]
message topic_1 {"test_data":1}
```

Here is the end of our tutorial, it's a very basic introduction to the usage of AWS IoT, but it gives you a good view of the pub/sub system that this service offers. With that process in mind, you could easily tweak it to:

- Store sensors values into a dynamoDb
- Light up a led from a web server
- ...

You can also find more advanced examples on the [AWS JS SDK repository](https://github.com/aws/aws-iot-device-sdk-js).

[2]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/2-create-a-new-project.png "Create a new project"
[3]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/3-you-now-have-a-thing-in-yout-list.png "You now have a thing in your list"
[4]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/4-add-a-device-to-your-thing.png "Add a device to your thing"
[5]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/5-select-your-plateform-and-download-your-certificates.png "Select your platform and download your certificates"
[6]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/6-rename-and-put-those-certificates-in-a-folder.png "Rename and put those certificates in a folder"
[7]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/7-you-should-now-have-three-items.png "You should now have three items"
[8]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/8-let-s-add-a-new-ressource.png "Let's add a new ressource"
[9]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/9-pick-up-the-rule-item-in-the-menu.png "Pick up the rule item in the menu"
[10]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/10-let-s-pick-up-all-message-from-topic-2.png "Let's pick up all messages from topic 2"
[11]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/11-republish-message-to-an-other-queue.png "republish messages to an other queue"
[12]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/12-let-s-use-the-queue-we-ve-subscribed-to.png "Let's use the queue we've subscribed to"
[13]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/13-we-have-to-create-a-policy-to-have-access.png "We have to create a policy to have access"
[14]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/14-the-policy-is-automatically-added.png "The policy is automatically added"
[15]: https://blog.louisborsu.be/images/posts/2015-10-22-aws-iot-getting-sarted-tutorial-with-nodejs/15-your-action-is-added.png "Your action is added"
