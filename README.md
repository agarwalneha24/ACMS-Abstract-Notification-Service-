# ACMS-Abstract Notification Service
Abstract Notification Dispatch service for android and ios devices that is capable of notifying a user over all the devices (capped to N) that has been registered with the application. The service is agnostic and highly scalable.


                                         ABSTRACT NOTIFICATION SERVICE


PROJECT DESCRIPTION 

The Project aims at developing an abstract notification service that is capable of notifying the user on as many devices on which the user has registered with the application.

                                         CLIENT FACADE DEVELOPMENT

Maintenance of the record of all the registered user devices and their Token IDs’ corresponding to all user registered applications and facilitate the notification dispatcher with the same.

COMPONENT DESCRIPTION

User Registration:

End device users are registered over a particular application. Each user is registered with a username and password. User is supplied with a client Id by the application to which the user is being registered. Username, user device Id, device token,OS type and client Id are registration parameters provided to Client Facade. 

Client Registration:

These are the applications with which multiple users have been registered with their unique device tokens (in APN) or registration IDs’ (in FCM). A client secret and client Id is associated with each client to differentiate between the clients being registered with the Client Facade. Unique Client ID and Secret Key are generated by the client facade against the credentials supplied by the client. 

Notification Dispatch Request:

When the notifications are to be sent to the users registered with a particular client. Client generates a request to the client facade for dispatching the message to its respective users.

Client Facade:

Each device user is registered with the client facade with username and corresponding Device Token and Device ID. Each client is registered with the client facade which maintains a record of registered Unique Client IDs’ and their associated Secret Key.
User device being registered is mapped to their respective client using the client Id held with the client facade. Client Facade is responsible for maintaining the record of all registered user devices over a particular application with their respective device token Ids’. It also maintains the record of all registered applications with their corresponding credentials.

Database
Client Facade maintains tables containing
 User_Schema:
> Username
> Device Token
> Device ID
> OS Type
> Client ID
Client_Schema:
> Client ID
> Secret Key
> Application name
> Application Description
A database has been created to facilitate the storage of this data.
If the app user deletes the app from the device or the app has not been used for a while, their respective record is marked disabled.
This project involves use of MongoDB for database management.

                                           NOTIFICATION DISPATCHER


Receiving notification message along with token Ids and usernames from client facade and hitting integrators with it.

COMPONENT DESCRIPTION

Client Facade:

It will be synced by the clients and users. It will maintain two database tables. One will have the client Id, client secret of the applications provided by clients and the other one will have entries of all the users with their usernames and all the device tokens corresponding to that user that is generated when the user downloads the application in any device.

Notification Dispatch Mechanism:

Notification Dispatcher makes a function which is called by Client facade, through which the notification messages will be received from client facade along with the information about app of which the message is, the users and their device OS to which it has to be sent. This notification message is then sent to APNS or Firebase integrators for sending the notifications by calling their respective functions.
Language used here is NodeJS.

Device Token Resolver:

Mapping the device tokens with usernames.A single user can install the same app in many devices. Thus many token Ids can correspond to a single username. But the client facade will only send the notification message and the username to which the notification has to be sent so all the device tokens that correspond to that username have to be sent to the Integrators.

Mapping the device tokens with OS:

Each device token will correspond to a particular device with either android or iOS operating system. So, if the device is based on android OS then Firebase integrator will be hit and if the device will be iOS then APNS integrator will be hit.

Response reader:

If the notifications couldn’t be successfully sent to the device then an error statement will be returned to the specific integrator. It will store the corresponding device token in a queue format. Notification dispatcher will read this and mark this token Id as disabled in both the databases maintained by integrators and client facade.
Queue Software used here is Rabbitmq.

                                       APNS INTEGRATION

Sending Device token and payload to APNs which will send the notification to respective devices and logging the APNs responses.

COMPONENT DESCRIPTION

Notification Request Constructor:

Some header fields are required for delivering the notification while some are optional.
Required-Header fields:
method-POST
Path-path to the device token
apns push-type(alert, badge,sound etc)
apns expiration-type(date at which notification is no longer valid)
authentication(encrypted-token if we are using token-based authentication)
apns-topic-Bundle id of the application
apns-collapse id-used to coalesce multiple notifications into a single notification for the user.


Optional-Header fields:

apns id-a unique notification id (default value set by APNs)
apns-priority-setting the priority of the notification (default value-10)
Payload is constructed by combining message(data) with the type of user interactions(alert,badge or sound) that are to be performed.
   
Connection Establisher:

For establishing connection between provider and APNs-configuration is required in the online developer account which can be either token-based authentication or certificate-based authentication.Here in this project token-based authentication is used.

APNs Notification Dispatcher:

Requests must include payload,Request header field comprising Provider server Authentication token,device token and other header fields which APNs forward it to respective devices.Language used is Javascript with Node js runtime environment.

APNS Response Handler:

APNs provide a response to each POST request the server transmits. Each response contains a header with fields indicating the status of the response. If the request was successful, the body of the response is empty. If an error occurred, the body contains additional information about the error.Responses are stored in the DBMS along with the status if successfully received and Invalid device tokens are produced in RabbitMQ asynchronous queue.

ESTABLISHING TOKEN BASED CONNECTION 

Token based Authentication:

Token-based authentication offers a stateless way to communicate with APNs.Stateless communication is faster than certificate-based communication because it does not require APNs to look up the certificate, or other information, related to the provider server.

There are other advantages of using token-based authentication:
We can use the same token from multiple provider servers.
We can use one token to distribute notifications for all of your company’s apps.

Authentication Procedure:

Encrypt the resulting data using the authentication token signing key obtained from the developer account.Provider server must include the resulting encrypted data with all  notification requests


                                                  ANDROID FIREBASE INTEGRATION

Sending Device token and payload to Firebase which will send the notification to respective devices and logging the responses.

COMPONENT DESCRIPTION

Notification Payload Constructor:

Notification content which is sent to FCM comprises the payload extracted from message and the token id. Notification messages have a predefined set of user-visible keys and an optional data payload of custom key-value pairs. The Firebase Admin SDK and the FCM v1 HTTP protocol both allow your message requests to set all fields available in the message object. This includes a common set of fields and user specific fields.

FCM Notification Dispatcher:

To programmatically send notification messages using the Admin SDK or the FCM protocols, set the notification key with the necessary predefined set of key-value options for the user-visible part of the notification message.  FCM can send a notification message including an optional data payload. In such cases, FCM handles displaying the notification payload, and the client app handles the data payload. 

FCM Response Handler:

Log Entry is retrieved from the FCM response. It must contain token_id, encrypted_payload, timestamp and status. Once we have created logs-based metrics to monitor our functions, we can create charts and alerts based on these metrics. For example, we could create a chart to visualize latency over time, or create an alert to let you know if a certain error occurs too often.

TOOLS USED

NPM(Node Package Manager):

It contains online repositories for node.js packages/modules and provides aids package installation, version management, and dependency management.

NodeJS:

Node.js is a Javascript-based environment that is easily understood by most of the browsers. Here, the Javascript is Server-Side instead of serving client-side. Node.js is definitely fast and it allows to explore a dynamic range of data at real-time. It allows code sharing. The most important reason for the rising popularity of Node.js is that the programmer can code the server-side as well as client-side using it. 

Express Module:

Express is a minimal and flexible Node.js web application framework that provides a robust set of features to develop web and mobile applications. It facilitates the rapid development of Node based Web applications. Following are some of the core features of Express framework −
Allows to set up middlewares to respond to HTTP Requests.
Defines a routing table which is used to perform different actions based on HTTP Method and URL.
Allows to dynamically render HTML Pages based on passing arguments to templates.


MongoDB:

Using NoSQL(MongoDB) over MySQL

Since with mostly Non-relational Database in the logs,it is easy to handle large data.It is magically faster than MySQl Database.
In MongoDB, Structure of a single object is clear.There is no fixed schema which ensures flexibility of the data-type.
There are no complex joins.
MongoDB offers better performance for mass read and write operations than MySQL.
MongoDB can be a disadvantage when there are a lot of write queries in the relational database but with our logs working on a Non-Relational database,it is not going to lower the performance of logging.
With the Time to live(TTL) feature in MongoDB,it is easier to implement the log life feature in MongoDB than in MySQL.

RabbitMQ:

RabbitMQ is an asynchronous Messaging queue to store the undelivered notification.The basic architecture of a message queue is simple - there are client applications called producers that create messages and deliver them to the message queue. Another application, called the consumer, connects to the queue and gets the messages to be processed.
RabbitMQ has a high community-support.It has very good documentation.There is no powerful alternative for RabbitMQ to create an asynchronous messaging queue.

     


