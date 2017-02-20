---
layout: post
title:  "How etherpad-lite, a real time collaborative editor, works?"
date:   2016-10-04
categories: blog
tags: [web,socket,node,server,document,editor,etherpad]
author: shobhit_garg
share: true
comments: true
excerpt: ""
---

Although this post is specific to etherpad-lite but i believe it would give you good insight on how a real time collaborative editor works.

Etherpad-lite is an open source real time collaborative editor, that means on it at a same time a number of users can edit the same document in real time. Developing a real time collaborative editor is a challenging task as there can be many number of users editing the document and you have to maintain the consistency for each user that too in real time. Here i am describing how etherpad manages the client-server communication and achieve the collaboration in real time. 

*In etherpad vocabulary word `pad` is used as an alternative to `document`.


![Diagram]({{ site.url }}/assets/etherpadlite.png)
<u>Architecture Diagram</u>

## Communication in Etherpad-lite:

Etherpad-lite uses socket connections for sending data to and fro between client and server. For creating ad maintaining these socket connections it uses [socket.io][officialsocketio] which is a popular open source library.You can read more on socket.io on my post [Detailed Explanation of SOCKET.IO][socketio-url] .
Etherpad-lite uses the concept of rooms of socket.io so please read the socket.io post carefully.It uses socket because it's a two way connection so both client and server can send message to each other.Also this is a persist connection so sending data to and fro just happens in real time.

Between etherpad server and database, communication is through standard tcp calls.



## Data Storage:
It uses a database for storing the data permanently. It stores pad content and it's revisions in database. Etherpad-lite provides the functionality to use databases like mysql, sqllite, redis, mongodb, casscandra etc. Apart from storing pad content in database it keeps the  pad content in memory as long as there is an active client on pad. So basically for opening a pad it loads the data from database and stores it in memory, then for every operation it uses the data stored in memory.It doesn't store the revisions content in memory.

For each client it also stores the sessionInfo in memory.In sessionInfo it basically stores the pad id, revision number and author info of that client.


## Changeset: 
Changeset is a delta between a old state a new state of a pad. So say pad is on state x and user makes some changes now pad is on state y. So a changeset is basically something which if you apply on x , it's state will be y. Important point to note here is changeset is always relative to a state. You can't apply changeset generated for state x to some other state.In etherpad every state is denoted by a number called revision. So if pad is on state n, and after applying changeset it goes from revision n to revision n+1 .



## Message Types

Each message , which client sends to server or server sends to client, contains a message type. Using message type client or server determines that message contains what information and it is for what action.Below are the some common message types and their use:

__CLIENT_READY:__ Message of this type is sent from client to server when it asks for the pad information just after establishing the socket connection.

__CLIENT_VARS:__ Message of this type is sent from server to client when it receives CLIENT_READY from client. It contains pad information.


__USER_CHANGES:__ Message of this type is sent from client to server when client sends a changeset to server.


__ACCEPT_COMMIT:__ Message of this type is sent from server to client on accepting the client changeset.

__NEW_CHANGES:__ Message of this type is sent from server to client to let it know about other user changesets if multiple users are working on same pad.


## How pad gets loaded into client browser?

Through http calls etherpad-lite loads the supporting scripts and html. It never loads the actual content of pad through it. When the scripts get loaded, it creates a socket connection with server. And after creating connecting it sends a message of type `CLIENT_READY` with information like padID and other security related token and passwords. Server on receiving this message verifies the user and store it's info in sessionInfo in local memory and then send the pad details to client with message type `CLIENT_VARS` . After that, corresponding communication socket joins the room of that padID. (Because server should know which all clients are connected to a pad so by adding socket to room this information remains on server.Using this room concept now you can find which all clients are connected to a particular pad and can send a message to all of them or a particular one). After reading this data which is sent with message type CLIENT_VARS browser loads the pad into memory and render it.

## How client sends it's changes to server and other clients?

This is pretty crucial part. Every pad stores on which revision it is currently on.
After a time interval of t second, client collects all the changes made on pad and creates a changeset. It sends this changeset to server with message type `USER_CHANGES`  and with the revision, pad is currently on. It sends this type of info to server:

{% highlight javascript %}

baseRev:k
changeset:"Z:7d>0|c=70*0*1*2*3*4*5=1$"
type:"USER_CHANGES"
//k is an integer
{% endhighlight %}

Now the question arrive is how server handle these changesets sent by the user.

<u>Case 1: When client pad and server is on same revision</u>

This message reflects that this changeset is for pad revision k.  When this changeset reaches to server, server finds the HEAD revision of pad from pad content stored in memory. If the revision of pad is k, it applies the changeset to pad and change the pad revision to k+1 . It also stores the changeset sent by user in a separate row of database marking it revision k+1 so that this can be sent to other clients and pad changes hierarchy can be maintained. After that server sends message to client with type `ACCEPT_COMMIT` and revision no. of this changeset. This message is like:

{% highlight javascript %}

type: "ACCEPT_COMMIT", 
newRev: k+1
//check server is marking the client changeset as k+1 revision

{% endhighlight %}

(client will be able to send another changeset to server only after receiving this message of type ACCEPT_COMMIT)

Now as i mentioned previously that socket rooms maintains the info of clients connected to the pad and sessionInfo store the info on which revision they are on. Now to task is to send the changeset to other clients of this pad. For each client of pad server checks the revision no. which client is currently on and send the changesets accordingly. For example if other client is on revision k-1 then server sends changesets k and k+1 to client. If other client is on state k then server only sends the changeset k+1 to the client. All these messages are sent with type `NEW_CHANGES` and with information of changeset and revision. Like:

{% highlight javascript %}

changeset:"Z:7d>0|c=70*0*1*2*3*4*5=1$"
newRev:k+1
type:"NEW_CHANGES"

{% endhighlight %}

(On receiving this changes client update it's pad content according to the changeset and update it's revision number.)

<u>Case 2: When client pad and server is on different revision</u>

Server find the HEAD revision of pad from database and it is not k. It is something bigger than k say k+2. It means some other clients have updated the pad already or it is possible that this client has lost some messages of type NEW_CHANGES sent by server. As i mentioned earlier in Changeset section that every changeset is relative to a revision and the changeset client has sent is relative to revision k so we can't apply it to pad of revision k+2. So for each changeset after k, server modifies the revision sent by client and send the changeset which it is processing. I make you confused,Right? Let's take an example. Suppose pad is on revision k+2. There are two revisions after k first one is k+1 and second one is k+2. 

For changeset k+1:

Currently the changeset sent by user is relative to changeset k, call this changeset UserChangeset<k>. So first server modifies the revision sent by client and make it relative to revision k+1, call it UserChangeset<k+1>. (Initially it was relative to k as client revision was k). Now server sends the actual k+1 revision to client using message type NEW_CHANGES as client was on k revision.Now client is on revision k+1. 

For changeset k+2:

Server makes the UserChangeset<k+1> realtive to changeset k+2, lets call it UserChangeset<k+2>. Now server sends the actual k+2 revision to client using message type NEW_CHANGES as client was on k+1 revision. Now client is on revision k+2.As pad was on revision k+2 and this UserChangeset<k+2> is relative to changeset k+2 , this can be applied to pad. So server applies this changeset to pad and stores the pad as revision k+3 . Finally server stores the user changeset as revision k+3 so it sends message with type ACCEPT_COMMIT to client with revision k+3. For other clients connected to that pads it does the same operation it does in Case 1.




Until the client gets the ACCEPT_COMMIT message for the sent changeset, it doesn't send another changeset. If it doesn't get ACCEPT_COMMIT message for a time t it gets disconnected.


## Why etherpad stores data in main memory?

Etherpad runs on nodejs which is event based. If you shift everything to some global cache etherpad will start facing issue in synchronizing data between multiple clients. Because say if server receives two changeset at almost the same time it may read the same revision of pad and later apply same revision to both the changesets, in that case we can lose one changeset. There are lots of other issue in maintaining consistency between clients of same pads. Also accessing in memory data is fast which helps etherpad in making the collaboration fast.


[socketio-url]:     {{ site.url }}/blog/socket-io/
[officialsocketio]: http://socket.io/
