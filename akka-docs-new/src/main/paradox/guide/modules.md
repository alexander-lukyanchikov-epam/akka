# Akka Libraries and Modules

Before we delve further into writing our first actors, we should stop for a moment and look at the set of libraries 
that come out-of-the-box. This will help you identify which modules and libraries provide the functionality you 
want to use in your system. 

### Actors (`akka-actor` library, the core) 

The use of actors across Akka libraries provides a consistent, integrated model that relieves you from individually 
solving the challenges that arise in concurrent or distributed system design. From a birds-eye view, 
actors are a programming paradigm that takes encapsulation (one of the pillars of OOP) to its extreme. 
Unlike objects, actors encapsulate not only their 
state but their execution. Communication with actors is not via method calls but by passing messages. While this 
difference seems to be minor, this is actually what allows us to break clean from the limitations of OOP when it 
comes to concurrency and remote communication. Don’t worry if this description feels too high level to fully grasp 
yet, in the next chapter we will explain actors in detail. For now, the important point is that this is a model that 
handles concurrency and distribution at the fundamental level instead of ad hoc patched attempts to bring these 
features to OOP.

Problems that actors solve include:

* How to build and design high-performance, concurrent applications?
* How to handle errors in a multi-threaded environment?
* How to protect my project from the pitfalls of concurrency?

### Remoting

Remoting enables actors that are remote from each other, living on different computers, to seamlessly exchange messages. 
While distributed as a JAR artifact, Remoting resembles a module more than it does a library. You enable it mostly 
with configuration, it has only a few APIs. Thanks to the actor model, remote and local message sends look exactly the 
same. The patterns that you use locally on multi-core systems translate directly to remote systems. 
You will rarely need to use Remoting directly, but it provides the foundation on which the Cluster subsystem is built.
 
Some of the problems Remoting solves are

* How to address actor systems living on remote hosts?
* How to address individual actors on remote actor systems?
* How to turn messages to bytes on the wire?
* How to manage low-level, network connections (and reconnections) between hosts, detect crashed actor systems and hosts, 
  all transparently?
* How to multiplex communications from unrelated set of actors on the same network connection, all transparently?

### Cluster

If you have a set of actor systems that cooperate to solve some business problem, then you likely want to manage these set of 
systems in a disciplined way. While Remoting solves the problem of addressing and communicating with components of 
remote systems, Clustering gives you the ability to organize these into a "meta-system" tied together by a membership
protocol. **In most cases, you want to use the Cluster module instead of using Remoting directly.** 
Cluster provides an additional set of services on top of Remoting that most real world applications need. 

The problems the Cluster module solves (among others) are

* How to maintain a set of actor systems (a cluster) that can communicate with each other and consider each other as part of the cluster?
* How to introduce a new system safely to the set of already existing members?
* How to detect reliably systems that are temporarily unreachable?
* How to remove failed hosts/systems (or scale down the system) so that all remaining members agree on the remaining subset of the cluster?
* How to distribute computations among the current set of members?
* How do I designate members of the cluster to a certain role, in other words to provide certain services and not others?

### Cluster Sharding

Persistence solves the problem of restoring an actor’s state from persistent storage after system restart or crash. 
It does not solve itself the problem of distributing a set of such actors among members of an Akka cluster. 
Sharding is a pattern that mostly used together with Persistence to balance a large set of persistent entities 
(backed by actors) to members of a cluster and also migrate them to other systems if one of the members crash.
 
The problem space that Sharding targets:

* How do I model and scale out a large set of stateful entities on a set of systems?
* How do I ensure that entities in the cluster are distributed properly so that load is properly balanced across the machines?
* How do I ensure migrating entities from a crashed system without losing state?
* How do I ensure that an entity does not exist on multiple systems at the same time and hence kept consistent?

### Cluster Singleton

A common (in fact, a bit too common) use case in distributed systems is to have a single entity responsible 
for a given task which is shared among other members of the cluster and migrated if the host system fails. 
While this undeniably introduces a common bottleneck for the whole cluster that limits scaling,
there are scenarios where the use of this pattern is unavoidable. Cluster singleton allows a cluster to elect an 
actor system which will host a particular actor while other systems can always access said service independently from 
where it is.

The Singleton module can be used to solve these problems:

* How do I ensure that only one instance is running of a service in the whole cluster?
* How do I ensure that the service is up even if the system hosting it currently crashes or shut down during the process of scaling down?
* How do I reach this instance from any member of the cluster assuming that it can migrate to other systems over time?  

### Cluster Publish-Subscribe

For coordination among systems it is often necessary to distribute messages to all, or one system of a set of 
interested systems in a cluster. This pattern is usually called publish-subscribe and this module solves this exact 
problem. It is possible to subscribe to topics and receive messages published to that topic and it is also possible 
to broadcast or anycast messages to subscribers of that topic.

* How do I broadcast messages to an interested set of parties in a cluster?
* How do I anycast messages to a member from an interested set of parties in a cluster?
* How to subscribe and unsubscribe for events of a certain topic in the cluster?

### Persistence

Just like objects in OOP actors keep their state in volatile memory. Once the system is shut down, gracefully or
because of a crash, all data that was in memory is lost. Persistence provide patterns to enable actors to persist 
events that lead to their current state. Upon startup events can be replayed to restore the state of the entity hosted 
by the actor. The event stream can be queried and fed into additional processing pipelines (an external Big Data 
cluster for example) or alternate views (like reports).
 
Persistence tackles the following problems:
 
* How do I restore the state of an entity/actor when system restarts or crashes?
* How do I implement a [CQRS system](https://msdn.microsoft.com/en-us/library/jj591573.aspx)?
* How do I ensure reliable delivery of messages in face of network errors and system crashes?
* How do I introspect domain events that has lead an entity to its current state?

### Streams

Actors are a fundamental model for concurrency, but there are common patterns where their use requires the user 
to implement the same pattern over and over. Very common is the scenario where a chain (or graph) of actors need to 
process a potentially large (or infinite) stream of sequential events and properly coordinate resource usage so that 
faster processing stages don’t overwhelm slower ones in the chain (or graph). Streams provide a higher-level 
abstraction on top of actors that simplifies writing such processing networks, handling all the fine details in the 
background and providing a safe programming model. Streams is also an implementation 
of the [Reactive Streams standard](http://www.reactive-streams.org) which enables integration with all 3rd 
party implementations of that standard.

* How do I handle streams of events or large datasets with high performance, exploiting concurrency and keep resource usage tight?
* How do I assemble reusable pieces of event/data processing into flexible pipelines?
* How do I connect asynchronous services in a flexible way to each other, and have good performance?
* How do I provide or consume Reactive Streams compliant interfaces to interface with a 3rd party library?

### HTTP

The de facto standard for providing APIs remotely (internal or external) is HTTP. Akka provides a library to provide or 
consume such HTTP services by giving a set of tools to create HTTP services (and serve them) and a client that can be 
used to consume other services. These tools are particularly suited to streaming in and out large set of data or real 
time events by leveraging the underlying model of Akka Streams.

* How do I expose services of my system or cluster of systems to the external world via an HTTP API in a performant way?
* How do I stream large datasets in and out of a system using HTTP?
* How do I stream live events in and out of a system using HTTP?

***

This is an incomplete list of all the available modules, but it gives a nice overview of the landscape of modules 
and the level of sophistication you can reach when you start building systems on top of Akka. All these modules 
integrate with together seamlessly. For example, take a large set of stateful business objects 
(a document, a shopping cart, etc) that is accessed by on-line users of your website. Model these as sharded 
entities using Sharding and Persistence to keep them balanced across a cluster that you can scale out on-demand 
(for example during an advertising campaign before holidays) and keep them available even if some systems crash. 
Take the real-time stream of domain events of your business objects with Persistence Query and use Streams to pipe 
it into a streaming BigData engine. Take the output of that engine as a Stream, manipulate it using Akka Streams 
operators and expose it as websocket connections served by a load balanced set of HTTP servers hosted by your cluster 
to power your real-time business analytics tool.
 
Got you interested?
