---
title: "Websockets and Phoenix Deep Dive"
date: 2025-03-01T12:03:35Z
draft: true
---

## There's Websockets, and there's Phoenix

In a recent deep dive of websockets, I kept on finding myself thinking
"Well how does this work in Phoenix/LiveView"? With Elixir being so
embedded in the distributed world it does feel like we get a lot of stuff "for free".
You get to create a distributed app in simple ways by simply using
the heavy lifting provided by Phoenix yet I find myself having gaps in
knowledge on how things are being done.

To start our quest I want to show you what question really peaked my interest.

NOTE: I'm using Hugo for the server and not entirely sure how to tweak images.
I'm hoping to fix it one day

## The Cost of State

The typical REST APIs can perfectly rely on HTTP has a limitation.
Only the client can initiate requests. Looking at an image from ably
hopefully helps show this. The client is the one who must start the request in
order to get back information.

![standard http flow](/images/websockets-and-phoenix-deep-dive/http-request-response-cycle.webp)

In the case where you do want the server to send back information to the client
websockets can be a great fit, but let's consider the downside. Websockets
means that the server has to be "smarter". It needs to maintain state, each
client that connects to us has its own view of the world through its
own websocket connection. **Note that even with websockets we're not real-time
yet.** The client can make some updates to the UI and the server will
return data back to it within the same websocket connection, but
the server will not broadcast this out to all clients by default!

The last thing I really want to highlight is that in a setting where we
only have one server, life is good. However, if you are trying to
horizontally scale your application, then you might have a gateway in
between you and your servers. That's where my rabbit hole started:

- The Great Hussein Nasser on Scaling Websockets: <https://youtu.be/XgFzHXOk8IQ?feature=shared&t=7993>
- Alby Realtime mentioning the same scaling problem: <https://youtu.be/vXJsJ52vwAA?feature=shared&t=464>

We'll dig into this more soon but this image is all it takes to add some
complexity to the system.

![gateway](/images/websockets-and-phoenix-deep-dive/gateway-conundrum.jpg)

## Additional Complexity: User Specific State

Let's make the example more concrete. Let's say I am using slack. I start
a conversation with a friend and we both send messages to each other.
I have an image I want to send to them but I have that image on my phone.
From the user perspective, we'd expect.

1. All the messages I've sent from the computer are visible on my phone.
2. The image I upload will be quickly visible on my laptop.

If slack uses websockets, then this means that even though I am a single
user, there are two different websocket connections to the server. If
the state updates I sent through my laptop went to one server, then that
state lives there. Now if I connect to my phone, it's vital that I am
creating a new websocket connection on the same server as my laptop
since otherwise I wont see those changes.

### HTTP is Stateless, Websockets is Stateful

The way that we use HTTP is that when our phone connects to our server,
it can safely connect to any server it wants since it's not a stateful.
This just means that in this request/response flow, it's expected that
our request will load the most recent state through something
like a database.

With websockets on the other hand, it expects the server to properly
manage the **live** state. This means that we need to properly
manage users and which server they access when scaling.

## Broadcasting Updates to All

Websockets core strength is that communication is bidirectional. We can
use this to do the something like the following.

- Me and my friends are chatting in the slack channel
- I send a message to the channel
- Without our client requesting for new updates, the server will be
  able to notify my friends that I have sent a new message and show that

This leads us to the next problem, how do we notify all the interested
parties that there was an update?

The answer is PubSub but what I want to focus on is how Phoenix handles this
in our distrubuted world.

We'll go through 3 resources to really understand how Phoenix handles this.
These articles cover more detail than what I mention but I like the articles
based on what clicked for me.

1. How pg2 works and how Phoenix leverages it - <https://www.poeticoding.com/distributed-phoenix-chat-with-pubsub-pg2-adapter/>
2. Understanding additional Phoenix Internals at
   play like Transport and Channels - <https://zohaib.me/guts-of-phoenix-channels/>
3. Creator of Phoenix's TLDR: <https://stackoverflow.com/a/34259366/8262460>

### One Group Processes to rule them all, pg2 (Who needs to receive live updates)

To recap our problem, we have some update from a single client, how do other
clients receive that update? We're going to work through the slack example
with the following system that borrows from Alvise Susmel's
[Distributed Phoenix Chat with PubSub PG2 Adapter](https://www.poeticoding.com/distributed-phoenix-chat-with-pubsub-pg2-adapter/)

- We have two servers that are running Phoenix
- We have 4 clients (2 on each server)

![pg2 phoenix](/images/websockets-and-phoenix-deep-dive/pg2-phoenix.webp)

### How does our distributed application handle multiple Phoenix nodes? (pg2)

"Each Phoenix node starts its own local `PubSub.PG2Server`
and registers it in a `pg2` group with name `{:phx, YourApplication.PubSub}`"

This means that all Phoenix nodes are aware of each other since there's a global
group process storing that information. Once a node comes online, that distrubted/
global process knows of it.

### How does our distributed application handle multiple clients per node? (ETS)

Now let's imagine we have the two clients join a single node.
How do we "register" the fact
that on an update, we should notify these people? Introducing `Phoenix.PubSub.Local`.
Phoenix uses a GenServer and an ETS to handle this.

### Fin: How both local and remove clients receive updates

1. Locally Connected Clients: Every time a client joins a node,
   their pid is recorded by `Phoenix.PubSub.Local` and manged by ETS
2. Remotely Connected Clients: Every time a new phoenix server node joins,
   it is recorded by `Phoenix.PubSub.PG2Server` and manged by `pg2`

## Resources

- HTTP vs Websocket Basics: <https://ably.com/topic/websockets-vs-http#which-to-choose-web-sockets-or-http>
- Highlihts the case the issues with scaling websockets: <https://tsh.io/blog/how-to-scale-websocket/>
- Guts of Phoenix Channels: <https://zohaib.me/guts-of-phoenix-channels/>
- Distributed Phoenix Chat with PubSub PG2 Adapter: <https://www.poeticoding.com/distributed-phoenix-chat-with-pubsub-pg2-adapter/>
