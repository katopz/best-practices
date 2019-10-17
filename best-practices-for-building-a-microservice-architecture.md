> Credit : 
http://www.vinaysahni.com/best-practices-for-building-a-microservice-architecture

# Best Practices for Building a Microservice Architecture

<a id="top"></a>

Complexity has managed to creep in to your product. It's become increasingly difficult to evolve it at the pace you once could. It's time to look for a better way of doing things. Microservice architectures promise to keep your teams moving fast... but also come with a new set of challenges.

In building out a microservice architecture for [Enchant](http://www.enchant.com), I wanted to document a set of pragmatic practices that fit well with modern web and cloud technologies. I've tried to learn from those that have gone down this path before (i.e. Netflix, Soundcloud, Google, Amazon, Spotify, etc) in order to get more right than wrong.

## TL;DR

A microservice architecture shifts around complexity. Instead of a single complex system, you have a bunch of simple services with complex interactions. Our [goal](#requirements) is to keep the complexity in check.

**The Platform**

*   [Your platform is a set of standards combined with supporting tools](#the-platform)

**Service Essentials**

*   [Independently develop & deploy services](#identifying-the-key-requirements)
*   [Services should have their own private data](#private-data-ownership)
*   [Keep Services small enough to stay focused and big enough to add value](#identifying-service-boundaries)
*   [Store data in databases, not ephemeral service instances](#stateless-service-instances)
*   [Eventual consistency is your friend](#eventual-consistency)
*   [Offload work to asynchronous workers whenever possible](#asynchronous-workers)
*   [Keep helpful documentation for all services in a common place](#documentation)
*   [Distribute work with load balancers](#load-balancers)
*   [Aggregation services on network boundaries can translate for the outside world](#aggregation-services)
*   [Layer your security and don't write your own crypto code!](#security)

**Service Interactions**

*   [Transport data over HTTP, serialized using JSON or protobuf](#communication)
*   [For HTTP services, 500 series errors or timeouts mean the service is unhealthy](#unhealthy-service)
*   [APIs should be simple and effective](#api-design)
*   [A service discovery mechanism makes it easy for services to find each other](#service-discovery)
*   [Prefer decentralized interactions over centralized orchestrators](#decentralized-interactions)
*   [Version all APIs, colocating multiple versions within the same service instances](#versioning)
*   [Use limits on resources to fail fast before a service gets overloaded](#limit-everything)
*   [Connection pools can reduce downstream impact of sudden request spikes](#connection-pools)
*   [Timouts minimize impact from downstream delays and failures](#timeouts)
*   [Be tolerant of unrelated downstream API changes](#tolerate-changes)
*   [Circuit breakers give downstream services a break during tough times](#circuit-breakers)
*   [Correlation IDs help you track requests across service logs](#correlation-ids)
*   [Make sure you can guarantee eventual consistency](#distributed-consistency)
*   [Authenticating all API calls provides a clearer picture of usage patterns](#authentication)
*   [Auto retry failed requests with random retry intervals](#auto-retry)
*   [Only talk to a services through exposed and documented APIs](#exposed-apis)
*   [Economic forces encourage efficient usage of available resources](#economic-forces)
*   [Client libraries can handle all the basics, so you can focus on what matters](#client-libraries)

**Development**

*   [Use a common source control platform for all services](#source-control)
*   [Either mimic prod in dev or use isolated cloud based dev environments](#development-environments)
*   [Push working code to mainline often](#continuous-integration)
*   [Release less, release it faster](#continuous-delivery)
*   [Warning: shared libraries are painful to update](#risks-of-shared-libraries)
*   [Your service templates should cover the fundamentals out of the box](#service-templates)
*   [Simple services are also easy to replace](#replaceability)

**Deployment**

*   [Use a system image for a deployment package](#deployment-package)
*   [Have a way to automatically deploy any version of any service to any environment](#automated-deployment)
*   [Feature flags decouple code deployment from feature deployment](#feature-flags)
*   [Configuration should be managed outside of the deployment package](#configuration-management)

**Operations**

*   [Manage all logs in one place](#logging)
*   [Use a common monitoring platform for all services](#monitoring)
*   [Stateless services are easy to auto scale](#auto-scaling)
*   [Dependent services that don't run on your platform also need automation](#external-services)

**People**

*   [Service teams develop, deploy & operate their own services](#full-lifecycle-ownership)
*   [Teams should be autonomous in daily operations](#full-stack)

## Identifying The Key Requirements

A microservice architecture adds its own complexities. Instead of a few systems to operate, you now have many. Logs are all over the place. Consistency is hard in a distributed environment. This list can easily go on. Our goal is to get a state of **simplified complexity** - know that the complexity is there, but have tools and processes to keep it in check.

Stating some requirements I'm going to strive for:

*   **Maximize team autonomy**: Create an environment where teams can get more done without having to coordinate with other teams.
*   **Optimize for development speed**: Hardware is cheap, people are not. Empower teams to build powerful services easily and quickly.
*   **Focus on automation**: People make mistakes. More systems to operate also means more things that can go wrong. Automate everything.
*   **Provide flexibility without compromising consistency**: Give teams the freedom to do what's right for their services, but have a set of standardized building blocks to keep things sane in the long run.
*   **Built for resilience**: Systems can fail for a number of reasons. A distributed system introduces a whole set of new failure scenarios. Ensure measures are in place to minimize impact.
*   **Simplified maintenance**: Instead of one codebase, you'll have many. Have guidelines and tools in place to ensure consistency.

## The Platform

Service teams need freedom to build what's necessary. At the same time, you need to set some standards to ensure consistency and to manage operational complexity. This means standardizing communications, logging, monitoring and deployment, among others.

Your platform is a set of standards combined with tools that makes it easy to create and operate services that meet the standards.

**.. it needs a control plane**

How will your teams interact with the platform? It's typical to have many different web interfaces for continuous integration, monitoring, logging and documentation. Your teams will need a dashboard as a starting point for all of this. This could be something simple that lists all the services and links to the various internal tools. Ideally, it would also collect data from the internal tools and provide additional value at a quick glance.

For organizations that already use team chat solutions, one popular option is to use a custom bot to bring common tasks right into the chat interface. This can be useful for triggering tests and deploys, requesting quick stats about a running service, etc. The chat logs also become an audit trail of past actions.

## Service Essentials

Within your platform, you'll be running many services. Depending on your size, _many_ can mean tens, hundreds or even thousands. Each service encapsulates a piece of business capability into an independent package. You need to build services that are small enough to keep them focused on a single purpose and big enough to minimize interactions with other services.

## Independently Developed & Deployed

### Service Essentials

Each service should be independently developed and deployed. No coordination should be needed with other service teams if no breaking API changes have been made. Each service is effectively it's own product with it's own codebase and lifecycle.

If you find the need to deploy services together, you're doing something wrong.
If you have a single codebase for all your services, you're doing something wrong.
If you have to send out a warning before each deploy of a service, you're doing something wrong.

**Watch out for shared libraries!** If changes to a shared library require all services be updated simultaneously, then you have a point of tight coupling across services. Carefully understand the implications of any shared library you're introducing.

## Private Data Ownership

### Service Essentials

Once multiple services are directly reading and writing the same tables in a database, any changes to those tables requires coordinated deployment of all those services services. This goes against our prime directive of service independence. Sharing data storage is a sneaky source of coupling. Each service should have it's own private data.

Private data also has the advantage of letting you to select the right database technology based on the use cases of the service.

**Does each service needs it's own data server?**

Not necessarily. Each service needs it's own _database_, possibly colocated within a shared _data server_. The key point it that the services should have no knowledge of each other's underlying database. This way, you can start out with a shared data server and separate things out in the future with just a config change.

However, sharing a data server does have it's own complications. Firstly, it becomes a single point of failure that can take down a bunch of services together. This isn't something to take lightly. Secondly, you've also made it possible for one service to unintentionally impact others by hogging too many resources.

## Identifying Service Boundaries

### Service Essentials

This is a complex one to grasp. Each service should be an autonomous unit that implements a business capability.

A service should be **loosely coupled**. It should have minimal dependence on other services. Any communication it does with other services should be over the exposed public interfaces (API, events, etc). Those interfaces also need to be designed to not expose internal details.

A service should have **high cohesion**. Closely related functionality should stay together in the same service. This minimizes chattiness between services.

A service should cover a **single bounded context**. A bounded context encapsulates internal details of a domain, including any domain specific models.

Ideally, you understand your product and business well enough to have identified natural service boundaries. Even if you get it wrong the first time around, loose coupling makes it easy to refactor (i.e. combine, split or restructure) services in the future.

**Wait, about about shared models?**

Let's dig a little deeper into bounded contexts. You want to avoid creating dumb [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) services, as that would just result in tight coupling and poor cohesion. [Domain driven design](https://en.wikipedia.org/wiki/Domain-driven_design) introduces the concept of a bounded context that can help us identify sensible service boundaries. A bounded context encapsulates related pieces of a domain together (i.e. into a service, in our case). Multiple bounded contexts communicate over well defined interfaces (i.e. the APIs, in our case). Although some models may be completely encapsulated within a bounded context, others may have different use cases (and associated attributes) spread across multiple bounded contexts. In this case, each bounded context should own it's attributes related to the model.

This needs a concrete example. Consider [Enchant](http://www.enchant.com), a help desk solution. The core model of the system is a ticket, which represents a customer support request. The ticketing service manages the ticket lifecycle and owns primary attributes. Additionally, there's a reporting service which precalculates and stores statistics that are associated with specific tickets. There are two approaches to storing the ticket specific reporting statistics:

*   Store the statistics **in the ticketing service** since it ultimately owns the ticket models and the ticket lifecycle. With this approach, the reporting service would need to talk to the ticketing service whenever it wants to do anything with the data. This tightly couples the services together and makes them extremely chatty.
*   Store the statistics **in the reporting service** since it's responsible for reporting related data. With this approach, both services will have a ticket model, just each stores different attributes. This keeps the data close to where it's actually used. It also enables the reporting persistence to be optimized for reporting use cases. However, now the reporting service needs to get notified when a new ticket is created or when changes happen to existing tickets.

Storing the statistics in the reporting service better meets the service requirements - loose coupling, high cohesion and each service responsible for it's own bounded context. However, this approach adds some complexity. The reporting service needs to be notified about changes to tickets. This is accomplished by having the reporting service subscribe to an event stream coming out of the ticketing service, keeping coupling between the services to a minimum.

**But how big can a service be?**

_Micro_ in microservice has nothing to do with the physical size or lines of code, it's about minimizing complexity. A service should be _small enough_ that it serves a focused purpose. At the same time, it should _big enough_ that it minimizes interservice communication.

There's no hard set rule that a service can only be one process, one virtual machine, or one container. A service consists of what it needs to autonomously implement a business capability. This includes [external services](#external-services) like data servers for persistence, job queues for asynchronous workers or even caches to keep things fast.

## Stateless Service Instances

### Service Essentials

Instances of a stateless service don't store any information related to previous requests. An incoming request can be sent to any instance of the service. The primary benefit here is simplified operations and scaling. You can run the service behind a simple [load balancer](#load-balancers). Then, it's easy to add or remove instances as request volumes change. It's also easy to replace a failing instance.

That said, many of your services do need to store data of some kind. This data should be pushed into [external services](#external-services) like disk bound database servers or memory bound caches.

## Eventual Consistency

### Service Essentials

No matter how you look at it, consistency is hard in a distributed system. Rather than fight it, a better approach to take for a distributed system is [eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency). In an eventually consistent system, although services may have a divergent view of the data at any point in time, they'll eventually converge to having a consistent view.

If you've modelled your services well (i.e. loose coupling & high cohesion), you'll find that eventual consistency is a good enough default for most use cases. Creating eventually consistent distributed systems is also in line with creating loosely coupled systems. They tend to communicate asynchronously and inherently shield themselves from failures of downstream services.

Let's look at an example. For [Enchant](http://www.enchant.com), there's a ticketing service (that manages customer support requests) and a reporting service (that calculates ticket statistics). The reporting service gets an asynchronous feed of updates from the ticketing service. This means that whenever an update happens in the ticketing service, the reporting service only finds out a few seconds later. During those few seconds, both services have a divergent view of the underlying customer requests. For the reporting use cases, this few second lag is acceptable. As an added advantage, this approach also shields the ticketing service from failures in the reporting service.

## Asynchronous Workers

### Service Essentials

As you embrace eventual consistency, you'll find that not everything needs to be done while the request is blocked waiting for a response. Anything that can wait (and is resource or time intensive) should be passed as jobs to asynchronous workers.

This approach:

1.  **Speeds up the primary request path**. Since you're only doing a portion of the total work that needs to get done as part of the request.
2.  **Spreads loads to easy to scale worker processes**. Perfect for an auto-scaling setup, where the number of workers changes dynamically based on available work to be done.
3.  **Reduces error scenarios** on the primary service API. When jobs running in async workers fail, they can be retried behind the scenes without forcing the requesting service to wait.

**Understanding idempotency**

I mentioned that jobs can be retried when they fail. The challenge with automatically retrying jobs is that you may not know if the failing job completed its work before it failed or not. To keep things operationally simple, you really want your jobs to be be [idempotent](https://en.wikipedia.org/wiki/Idempotence). For our context, This means that there should be no negative impact of a job running more than once. The end result should be the same whether the job ran once or more than once.

## Documentation

### Service Essentials

A service (and it's API) is only as good as it's documentation. It's critical to have clear and easy to approach usage documentation for each service. Ideally, usage documentation for all services should be in a common place. Service teams shouldn't have to think too hard about where documentation is for a service they're using.

**What should happen when the API changes?**

Notification about changes to documented endpoints need to go out to owners of other dependent services. The notification system should have knowledge of who the _current_ owners are, accounting for any team or ownership changes. This is information that can be tracked and made available by the [platform](#platform).

## Load Balancers

### Service Essentials

One of the advantages of stateless services is that you can bring up multiple instances of your service. Since there's no per-instance state, any instance can handle any request. This is a natural fit for load balancers, which can be leveraged to help scale the service. They're also readily available on cloud platforms.

Traditional load balancers are on the receiving end, where the client only knows of one target. That target receives the request and distributes it among multiple (hidden) internal service instances. An alternate approach is a client side load balancer, such as what [Netflix has done with Ribbon](http://techblog.netflix.com/2013/01/announcing-ribbon-tying-netflix-mid.html). With a client side load balancer, the client is aware of the multiple possible targets and chooses a target based on a policy, such as preferring a target that's in the same data center to reduce request latency. A combined approach can also work well: start with traditional load balancers, and add in client side load balancers when you need the advanced routing.

Client side load balancers would be included as part of the [client libraries](#client-libraries).

## Aggregation Services on Network Boundaries

### Service Essentials

A number of additional requirements come in to play when data is crossing private network boundaries for communications with external clients. Data needs to be encoded in certain ways, you want to minimize round trips, you need to have heightened security to ensure these clients can only access what they need to, etc.

An aggregation service takes on the role of collecting data from other services. It handles any special encoding or compression requirements and inherently simplifies security efforts as the client would only need to communicate with a single service.

Depending on your needs, you may find it practical to build a multiple aggregation services, one for each use case (public API, mobile client, desktop client, etc). If your needs are simpler, a single service may do the trick.

**Limit the amount of business logic in an aggregation service**

As aggregation services work with data from many other services, it becomes easy to for accidentally sneak business logic into them and reduce cohesion of your services. Watch out for this! Any business logic related to a service should belong to it. Aggregation services area meant to be thin glue layers between external clients and internal services.

**What if one of the internal services is down?**

Answering this is very much dependent on your specific context. Some questions to ask:

*   Can the functionality be gracefully removed or does the endpoint need to throw an error?
*   Is the availability of the service critical enough that the whole aggregation service needs to be taken down?
*   If gracefully removed from the endpoint, how would the client show the failure to the user?

The nature of an aggregation service is that it's dependent on (and deeply coupled to) one or more other services. Accordingly, it's impacted by failures in any of the services... and things will fail. You need to understand the failure scenarios and have a plan of action in place.

## Security

### Service Essentials

Consider the security needs of a service based on the data it's housing or it's role in the grand scheme of things. You may need data security in transit or at rest. You may need network security at the service perimeter or at the perimeter of your private network. Good security is hard. Here are some principles worth thinking about:

*   **Layer your security**: Also known as [defence in depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)). Rather than assuming a network perimeter firewall is _good enough_, continue to add multiple layers of security _where it matters most_. This adds redundancy to your security and also helps slow down an attacker when one layer of security fails or a vulnerability is identified.
*   **Use automatic security updates**: In many cases, the benefit of automatic security updates outweighs the possibility of a service failure as a result of it. Combine automatic updates with automated testing, and you'll be able to roll out security updates with much higher confidence.
*   **Harden your base operating system**: Services typically need minimal access to the base operating system. Accordingly, the operating system can place strong limits on what a service can and cannot do. This helps contain a vulnerability if found in a service.
*   **Do not write your own crypto code**: It's very hard to get it right and very tempting to think you have something that works. Always use well known & widely used implementations.

## Service Interactions

A microservice architecture promotes having many small but focused services communicating with each other. This raises a bunch of questions: how should services find each other? Are they all talking a common protocol? What happens when one fails to communicate to another? These are some of the topics that we'll cover as we discuss service interactions.

## Communication Protocols

### Service Interactions

As you build more services, it becomes critical to have standardized methods of communication between them. Since services don't all have to be written in the same language, the chosen communication protocols must be language and platform independent. We'll need to account for both synchronous and asynchronous communications.

**First, the transport protocol**

HTTP is a great choice for **synchronous communications**. HTTP clients are already available in all languages. HTTP load balancers are built into cloud platforms. The protocol has built in mechanisms for caching, persistent connections, compression, authentication and encryption. Most importantly, there's an ecosystem of robust and mature tools that can be leveraged: caching servers, load balancers, excellent browser based debuggers and even proxies that can replay requests.

The one negative of HTTP is that it's a verbose protocol as plain text headers are repeatedly sent and connections are repeatedly created and torn down. Although I could argue that this is a reasonable tradeoff given the significant value that already comes with the HTTP ecosystem, we already have a better option on the horizon: [HTTP/2](https://tools.ietf.org/html/rfc7540). It effectively solves the verbosity problem by using compressed headers and multiplexing requests over persistent connections. It does all that while maintaining backwards compatibility with older clients. **HTTP is here today and will serve well into the future**.

That said, if you've achieved enough scale where saving internal transport overhead can make a notable difference to the bottom line, then other transport options may be a better fit.

For **asynchronous communications**, we'll need to implement the [publish subscribe](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) pattern. There are two major approaches here:

*   Use a **message broker**: All services push events to the broker. Other services subscribe to the events they need. In this scenario, the message broker defines it's own transport protocol. Since a centralized broker can easily become a single point of failure, it's important to ensure such a system is fault tolerant and horizontally scalable.
*   Use **webhooks delivered by the services**: A service exposes an endpoint from which other services can subscribe to events. It delivers those events as [webhooks](https://en.wikipedia.org/wiki/Webhook) (i.e. an HTTP POST with a serialized message in the body) to a target destination provided at time of subscription. Such webhook deliveries should be sent by [asynchronous workers](#asynchronous-workers) that the service manages. This approach avoids introducing a single point of failure and is inherently horizontally scalable. This functionality can be built right into a [service template](#service-templates).

**What about an Enterprise Service Bus (ESB) or a messaging fabric?**

The main problem with heavyweight messaging fabrics is they encourage pushing business logic out of the services and into the messaging layer. This lowers service cohesion and adds another layer where complexity builds up accidentally over time. Any business logic related to a service should belong to the service and be managed by the service teams. I strongly recommend sticking to **smart services with dumb pipes**. This ensures continued autonomy of the teams.

**Now let's talk about the serialization format**

There are two popular contenders here:

*   **JSON**: A plain text format defined in [RFC 7159](https://tools.ietf.org/html/rfc7159).
*   **Protocol Buffers**: an [IDL](https://en.wikipedia.org/wiki/Interface_description_language) with a binary wire format created by Google.

JSON is a stable and widely used serialization format. It's natively parsed in browsers. Debuggers built into the browsers also display it well. Nothing other than a JSON parser/serializer is required, which are readily available in all languages. The main negative about using JSON is that the attribute names get repeated in every message, resulting in an inefficient use of the transport. Compression on the transport protocol can significantly mitigate this.

Protocol buffers are efficient to parse, efficient over the wire and heavily battle tested at Google. However, they do require language specific parser/serializer generators based on a message definition files. Language support isn't as wide as JSON, though most modern languages are covered. Servers must also share the message definition files with clients in advance.

JSON is easier to get started with and more universal. Protocol buffers keep things leaner and faster but come with a little additional development overhead in sharing and compiling .proto files. Both are good options. Pick one and use it consistently.

## Definition Of Unhealthy Service

### Service Interactions

As we'll need automated monitoring and alerting, it's a good idea for all services to agree on what it means for a service to be unhealthy.

For an HTTP transport protocol, this one is pretty easy. Services are expected to generate 200, 300 & 400 series HTTP status codes. Any 500 error codes or timeouts can be assumed to be a result of a service failure. This is also inline with reverse proxies and load balancers, which will throw a 502 (Bad Gateway) or 503 (Service Unavailable) if they're unable to communicate with the backend instance.

## API Design

### Service Interactions

A good API is easy to use and understand. It provides _enough_ to get the job done without exposing underlying implementation details. It evolves with minimal impact to existing consumers. API design is as much an art as it is a science.

We've already chosen HTTP as our transport protocol. To unlock the full potential of HTTP, you'll need to combine HTTP with [REST](https://en.wikipedia.org/wiki/Representational_state_transfer). A RESTful API provides resourceful endpoints which can be operated on using verbs like GET, POST and PATCH. I've written a post on [RESTful API design](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api) which covers the design of public facing APIs in depth. Much of that post is also relevant to microservice API design.

**But why should service APIs be resource oriented?**

It leads to **consistency and clarity** across your service APIs. There's an obvious way to retrieve or search for things. Instead of trying to find the method that modifies a specific attribute of a resource, it's always just a PATCH (a partial update) to the resource. This leads to fewer endpoints on the API, which helps in further reducing complexity.

Since most modern public APIs are RESTful, there's also a healthy **ecosystem of tools** that can be used. This includes client libraries, test automation tools and introspecting proxies.

## Service Discovery

### Service Interactions

In an environment where service instances come and go, hard coding IP addresses isn't going to work. You will need a discovery mechanism that services can use to find each other. This means having a source of truth for what services are available. You'll also need some way to utilize that source of truth to discover and balance communication to the services instances.

**The service registry**

This is your source of truth. It contains information about the available services and their network locations. Given the critical nature of this service (it's a single point of failure), it needs to be extremely fault tolerant.

There are a two approaches to getting your services registered with the service registry:

*   **Self registration**: a service can register itself during startup and send updates as it goes through different lifecycle phases (initializing, accepting requests, shutting down, etc). It will also need to send regular heartbeats to the registry to let it know that it's still available. The registry can then automatically mark the service as down if it doesn't get a heartbeat. This is a good candidate for inclusion into a [service template](#service-templates).
*   **Externally monitored**: an external service keeps an eye on service health and updates the registry accordingly. This is approach adopted by many microservice platforms, which typically take on the role of service lifecycle management.

In the greater scheme of things, the service registry can also be the source of state used by the monitoring system or system visualization tools.

**Discovery and load balancing**

Having a working registry is only half of the problem. You also need to actually use it for services can discover each other dynamically! There are two main approaches here:

*   **Smart servers**: The client makes a request to a known load balancer, which has knowledge of instances that it has retrieved through the registry. This is the traditional approach, but does mean all traffic runs through load balancer endpoints. Server side load balancers come standard on cloud platforms.
*   **Smart clients**: The client discovers a list of instances via the service registry and decides which to connect to. This removes the need for a load balancer altogether and has the added benefit of spreading out network traffic more evenly. Netflix has taken this approach with [Ribbon](http://techblog.netflix.com/2013/01/announcing-ribbon-tying-netflix-mid.html) which also handles advanced policy based routing. To utilize this approach, you'll need the discovery and balancing functionality in your language specific [client libraries](#client-libraries).

**A simplified discovery mechanism using load balancers and DNS**

An easy way to get a rudimentary service discovery setup going on most cloud platforms is to use a DNS entry for each service that points to a load balancer. The load balancer's list of registered instances becomes your service registry and the DNS lookup becomes your service discovery mechanism. Unhealthy instances are automatically removed by the load balancer and re-added when they're healthy again.

## Decentralized Interactions

### Service Interactions

There are two main approaches for implementing complex workflows where multiple services need to coordinate together: using a centralized orchestrator or using decentralized interactions.

With **a centralized orchestrator**, a process coordinates with multiple services to complete a larger workflow. The services have no knowledge of the workflow or their specific involvement in it. The orchestrator takes care of the complexities. For example, enforcing the order in which services complete their work or retrying if a request to a service fails. To ensure the orchestrator knows what's going on, communications tend to be synchronous. The challenge with an orchestrator is that business logic will build up in a central place.

With **decentralized interactions**, each service takes full responsibility for its role in the greater workflow. It will listen for events from other services, complete it's work as soon as possible, retry if a failure occurs and send out events upon completion. Here, communications tend to be asynchronous and business logic stays within the related services. The challenge with this approach is tracking progress of the workflow as a whole.

Decentralized interactions meet our requirements better: loose coupling, high cohestion and each service responsible for it's own bounded context. All of this ultimately improves team autonomy. A service that monitors events coming out of all the coordinating services can passively track the state of the workflow as a whole.

## Versioning

### Service Interactions

Change is inevitable. What's important is how well the change is managed. Versioning your API and supporting multiple versions simultaneously goes a long way to minimizing impact to other service teams. This gives them time to update their code on their own schedule. Every API should be versioned!

That said, maintaining old versions indefinitely can be challenging. Old versions should be supported for a few weeks to a few months at most, whatever is reasonable for your organization. This gives other teams the time they need without further impacting your own development speed.

**How about maintaining multiple versions as separate services?**

Although this sounds like a good idea, it really isn't. An entirely new service also comes with it's own overhead. You'll have more things to monitor and more things that can fail. Bugs found in old versions will likely need to be fixed in new versions too.

It gets even more complicated if all versions of the service need a shared view of the underlying data. You could have them all talking to the same database, but that would be another bad idea! They would all be strongly coupled to the persistence schema. Any changes to the schema in any version can cause unintended breakage in other versions. You end up having to keep multiple code bases in sync.

**So how should multiple versions be maintained?**

All supported versions should co-exist in the same codebase and the same service instances. Use a [versioning scheme](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#versioning) to identify which version a request is for. When possible, old endpoints should be updated to relay modified requests to the equivalent new endpoints. Although having multiple versions co-exist in the same service doesn't eliminate the complexity, it avoids accidental complexity and tackles the inherent complexity head on.

## Limit Everything

### Service Interactions

A service that fails cleanly and quickly is better than one that's slowing everybody down because it's overloaded. All types of requests should have consumer specific limits in place. You'll also need a way to increase the limits for specific consumers as needed. This ensures stability of a service as it's team will have an opportunity to plan for large usage increases.

While such limits are most important for services that can't rapidly auto-scale, it's still a good idea for those that can. You don't want to be learning about the limits of your design decisions by surprise! That said, limits for auto-scaling services can be quite liberal.

Limit management interfaces could be built into [service templates](#service-templates) or provided as a centralized service at the platform level to enable self-service management by service teams.

## Connection Pools

### Service Interactions

Sudden spikes in request volume can make a service hammer another downstream service and pass the pain down the chain. Connection pools help smooth out the impact from sudden short term spikes in request volumes. A reasonable pool size limits the number of requests you'll make to the downstream service at any time.

Have a separate connection pool for each service you need to communicate with. This will isolate a fault in a downstream service to a specific part of your system.

**.. and remember to fail fast**

If a connection from the pool can't be acquired, it's better to fail fast rather than blocking indefinitely. This limits how long other services are waiting on yours. The failures will alert the team and raise some useful questions: Is it time to add capacity? Is the downstream service experiencing an outage?

## Short Timeouts

### Service Interactions

Imagine this scenario: One service gets overloaded with requests and slows down. As a result, all services calling it slow down. This pain continues to trickle upwards and eventually the user interfaces are lagging. The users aren't seeing the responses they expect and start clicking erratically in an attempt to fix things (sadly, this really happens) which only compounds the pain. This is a [cascading failure](https://en.wikipedia.org/wiki/Cascading_failure). Many services failing and raising alerts at the same time. You really don't want to experience this first hand, trust me.

With multiple services backing up and failing, identifying the source of the problem becomes a challenge. Is a service having an internal problem or is it a result of a downstream service? Using short timeouts on downstream API calls can help in this scenario. Timeouts prevent multiple services from just slowing down. Instead, you'll have one service really failing and other services failing fast and pointing to it.

Now, it's not good enough to just have a default 30 second timeout. You need a timeout that tightly covers what's reasonably expected of a downstream service. For example, if you expect a service to respond within 10 - 50 milliseconds, than any timeout over 500 milliseconds is already more than enough.

## Tolerate Unrelated Changes

### Service Interactions

Service APIs will evolve over time. A change that requires coordination with API consumers is slower to release than one that requires no coordination. To minimize coupling, services should be able to tolerate unrelated changes to responses of services they communicate with. This just means they shouldn't break if a field is added or an unused field is changed/removed.

If all services tolerated unrelated changes, additive API changes could be made without any coordination. Unrelated breaking changes would just require consuming service teams to run through their test suite to verify everything is still working.

## Circuit Breakers

### Service Interactions

Every attempt to communicate with a failing resource has a cost. It uses resources on the consumer side to try to make a request, it uses up network resources and it uses up resources on the target side.

A circuit breaker prevents requests that are doomed to fail from even being attempted. Implementing this is straight forward: if requests to a service are resulting in a high number of failures, flip a flag and stop trying to send requests to service for a period of time. Periodically allow a single request through to see if the service is back online, so you can flip the flag back.

Circuit breaking logic should be wrapped up in a [client library](#client-libraries) that's included as part of the [service template](#service-templates).

## Correlation IDs

### Service Interactions

A single user request can result in activity occurring across many services, which makes things difficult when trying to debug the impact of a specific request. One way to make things simpler is to include a correlation ID in service requests. A correlation ID is a unique identifier for the originating request that is passed by each service to any downstream requests. When combined with a [centralized logging](#logging) layer, this makes it really easy to see a request make it's way through your infrastructure.

The IDs are generated by either user facing [aggregation service](#aggregation-services) or by any service that needs to make a request that's not an immediate side effect of an incoming request. Any sufficiently random string (like a [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)) would do the trick.

## Maintaining Distributed Consistency

### Service Interactions

In an eventually consistent world, a service can synchronize data with another service by subscribing to its feed of events.

While this sounds easy enough, the devil is in the details. Your database and event stream are likely two different systems, making it extremely difficult to atomically write to both systems and thus hard to guarantee eventual consistency.

You could use a local database transaction to wrap the database operation and write to an event table at the same time. Then, the event publisher would just read from the event table. But not all databases support such transactions. You could have your event publisher read from your database's commit log. But not all databases expose such logs.

**... or you could just allow the inconsistency and fix it later**

Consistency is very hard in a distributed system. Even database systems for which distributed consistency is a core feature struggle to get it right. Rather than fight an uphill battle, you could just have a best effort synchronization solution combined with a process which identifies and corrects inconsistencies after the fact.

This approach is still eventually consistent. It's just that the _window of inconsistency_ may be a little longer than if you had taken on the complexity of guaranteeing cross system (database & event event stream) consistency.

**Every piece of data should have a single source of truth**

Even if you have to duplicate some data across multiple services, one service should always be the source of truth for any piece of data. All updates should go through the source of truth. This also becomes the originating source against which consistency verification can be done in the future.

**What if we need some services to be strongly consistent?**

First, I'd double check that you've got the [service boundaries](#service-boundaries) right. When services need to be strongly consistent, it usually also makes sense to colocate the data into a single service (and a single database), making it simpler to provide [transactional guarantees](https://en.wikipedia.org/wiki/ACID).

If you're sure you have the right service boundaries but still need strong consistency, then you'll need to look at [distributed transactions](https://en.wikipedia.org/wiki/Distributed_transaction), which are difficult to implement correctly and would also strongly couple the two services together. This should be your last resort.

## Authentication

### Service Interactions

All API requests should be authenticated. This helps service teams better analyse usage patterns and provides an identifier which can be used to manage [consumer specific limits](#limit-everything).

The identifiers would be unique API keys provided by the service team to consumers who use the service. You'll need some way to issue and revoke these API keys. This could either be built into the [service template](#service-templates) or be provided as a centralized authentication service at the platform level to enable self-service management of keys by service teams.

## Auto-Retry

### Service Interactions

When you're failing fast, it makes sense to automatically retry certain kinds of requests. This is especially the case for asynchronous communications.

A service that's down can easily get hammered when it comes back online if a bunch of other services were retrying at the same retry window. This is also known as a [thundering herd](https://en.wikipedia.org/wiki/Thundering_herd_problem), which can be easily avoided this by using randomized retry windows. If your infrastructure doesn't implement [circuit breakers](#circuit-breakers), I recommend combining randomized retry windows with an [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff) to further spread out requests.

**What if there's a permanent failure?**

Sometimes the failure is a result of a malformed request and not just the target service being down. In such a situation, no matter how many times you retry, the request isn't going to succeed. Such requests should be sent to a _dead queue_ for investigation after a number of failed retries.

## Communicate Only Via Exposed APIs

### Service Interactions

Communication between services should only happen through [established communication protocols](#communication). No exceptions. If you find you have a service talking directly to the database of another service, you're doing something very wrong.

As an added bonus: when you can make universal assumptions about how services communicate, it becomes easy to secure the rest of the service components behind strong firewalls.

## Economic Forces

### Service Interactions

When a team is using a service provided by another team, they tend to assume it's free. While it may be free for them to use, there are real costs to the other team and to the organization. In order to make effective use of the available resources, teams need to understand the cost of a service.

One powerful way to pull this off is to have services invoice other services for usage. Not using a made up points system. Invoice using real cash. A service should pass on the cost of development and operations to it's consumers. The true cost of a service includes development costs, infrastructure costs, and costs to use other services. This can all be flattened into a simple per-request price that's adjusted periodically (once or twice a year) as request volumes and costs change.

When the cost of using a service is transparent, developers are better equipped to see what's right for their service and the organization.

## Client Libraries

### Service Interactions

There are a lot of little things that need to be managed when talking to other services. For example: [discovery](#service-discovery), [authentication](#authentication), [circuit breaking](#circuit-breakers), [connection pools](#connection-pools) and [timeouts](#timeouts). Rather than each team rewrite this stuff from scratch, it should be packaged up into a client library with sensible defaults.

Client libraries shouldn't include any service specific business logic. It's scope should be limited to auxiliary concerns like connectivity, transport, logging and monitoring. Also, be aware of the [risks of shared libraries.](#risks-of-shared-libraries)

## Development

## Source Control

### Development

Each service should have it's own repository. This keeps checkouts small, source control logs clean and enables granular access control. You're [not deploying services together](#independent) and shouldn't be colocating their code either.

Additionally, standardize on a source control technology. This will keep things simple for your teams and make your [continuous integration](#continuous-integration) and [continuous delivery](#continuous-delivery) efforts easier.

## Development Environments

### Development

Developers need to be able to work quickly on their computers. To ensure a consistent environment on any operating system, the development environments should be packaged as virtual machines.

However, given the complexity and number of services involved in a microservice approach, it may not be practical to bring everything up on a single developer machine. In that case, the service that is developed and running locally could be combined with an _isolated_ environment running in the cloud. The developer would be able to quickly iterate in their development environment while testing against other services running in the cloud. Note that isolation is critical for such cloud environments. Shared environments between developers will only cause confusion as a result of unexpected changes.

## Continuous Integration

### Development

Integrate working code to the mainline branch as soon as possible. Updates to the mainline branch should trigger automated builds on the continuous integration system. The builds should trigger automated tests to verify that the build is in good shape.

Since these automated tests aren't running on the developer's computer, it becomes feasible to run more complex and time consuming tests on the continuous integration system. Popular solutions in the space keep things running quickly by executing tests in parallel across a cluster of machines.

If all the tests pass, the continuous integration system should release the [deployment package](#deployment-package) to the [automated deployment](#automated-deployment) system.

So what do you gain:

*   With code getting integrated quickly, everybody has visibility of changes that are being made. Any conflicts created as a result of multiple people changing the same code are identified quickly and are resolved sooner.
*   With full test suites running often, bugs are identified sooner.
*   Most importantly, when there are few changes per iteration of integration, one can be much more confident of the correctness of those changes.

Continuous integration speeds up your team's ability to deliver quality software.

## Continuous Delivery

### Development

The goal of continuous delivery is to release smaller sets of changes faster. Rather than tackle a large piece of work in one go, break it down into smaller chunks that can be completed and released one after another. You want to keep the system in a working state along the way.

Small releases are great. They are easy to test. They simplify code review efforts. It's much easier to confidently release and deploy small sets of changes.

To pull off continuous delivery, you will need to rapidly run through build, test and development cycles. This means you'll need to build out solid [continuous itegration](#continuous-integration) and [automated deployment](#automated-deployment) pipelines.

**But won't end users then see incomplete features?**

With [feature flags](#feature-flags) you'll be able to release features to specific sets of users when you're ready. This lets you deploy changes in smaller chunks without the user seeing incomplete features.

## Risks Of Shared Libraries

### Development

The biggest challenge with shared libraries is that you have little control of when updates will get deployed across the services that use them. It could take days or weeks before other teams deploy the updated library. In an environment where services are [independently developed and deployed](#independent), any change that requires all services to be simultaneously updated is just not practical.

The best you can do is post a deprecation schedule and coordinate with the service teams to ensure the updates get applied in a timely manner. As a result, any changes to shared libraries also need to be backwards compatible.

If it's not already obvious: shared libraries are ideal for managing auxiliary concerns like connectivity, transport, logging and monitoring. Service specific business logic should also stay out of shared libraries.

## Service Templates

### Development

In addition to their core business logic, services need to manage a number of additional supplementary tasks. Some examples include: service registration, monitoring, client side load balancing, limit management and circuit breaking. Teams should be provided with templates which can be used to quickly bootstrap services which handle all these common tasks and integrate well into your [platform](#platform).

**Should using a template be required?**

The templates should exist to speed teams up and not to enforce structure. However, certain _behaviours should be required_, like those to enable registration, monitoring and logging. Leave it to the team to decide if building something from scratch that meets behavioural requirements makes more sense than using the readymade template.

**So we can create a template for every popular tech stack?**

Although, microservices enable a [polyglot architecture](https://en.wikipedia.org/wiki/Polyglot_(computing)), it's important to not get carried away. There are a number of benefits of only supporting a limited set of technologies:

*   Your teams won't need to reimplement the tools for each stack, making it easier to focus on building robust standardized tools.
*   It enables cross team code reviews.
*   Most importantly, it makes it easy for developers to move between teams.

You should have templates available for each _supported_ stack.

## Service Replaceability

### Development

As the usage of a service grows, you'll eventually hit limits of your architectural design. By then, you should have learned enough about the specific needs and patterns of the service to be able to implement a more scalable solution than last time around. The great thing about simple and focused services is that they're also easy to replace.

Perhaps you'd like to switch to a specialized databased, or perhaps to a different language stack. As long as you maintain the documented interfaces (APIs and event streams), you can swap out a complete implementation without impacting other services.

.. or perhaps you want to change everything, including the API! In that case, you would create a new service altogether. Have any consumers of the existing service migrate over and remove the existing service when it's not being used anymore.

## Deployment

## Deployment Package

### Deployment

A standardized deployment package is an important building block for an [automated deployment](#automated-deployment) pipeline.

Your deployment package should have the following properties:

*   **Can be deployed anywhere**: the same package, without changes, should be deployable to any environment. Development, staging or production.
*   **External configuration/secrets**: Configuration and secrets shouldn't be stored within the package. Instead, they should be provided to (or retrieved by) the package at startup.
*   **Isolated deployments**: If multiple services share the same resources, it's easy for a service to accidentally consume an unfair amount of resources and cause unintentional impact on other services. Isolating each deployed service minimizes such impact.

A system image fits these requirements well. A versioned system image would be created for each service. Every update to that service would result in a new image. The system image could be for a physical machine, a virtual machine or a container. They all have the ability to limit and monitor resources (memory, cpu, network, etc) that the system consumes, which gives us what's needed to provide a certain level of isolation between the services. You're effectively running a single service per host.

**Immutable infrastructure for the win**

When your deployment package is a system image, you never update a running system in place. It's just replaced by a system built from a newer image. This approach improves confidence and reliability as you test the exact the same image that you'll be deploying to production. It also avoids configuration drift as a result of direct changes to production environments.

## Automated Deployment

### Deployment

Developers should have a common way to trigger automated deployments for any version of any service to any environment. Keeping deployment fully automated and simple makes it easy to confidently deploy small changes often.

**Aim for zero-downtime updates**

If a service has to be taken down to apply an update, then every update would send little shock waves across other services. To avoid such mini disruptions (which would discourage frequent deployment), you need a way to gracefully update a service with no downtime.

One approach is to use a rolling restart, which would update and restart one instance at a time behind a load balancer. Although a sound approach, you effectively need to run through a full rolling restart again if a problem is found and a rollback is needed.

A more robust approach is one where instances running the new version are brought up beside the original version, but serving no requests. The load balancers are then switched over to the instances running the new version, while keeping the existing version instances around for a period of time in case a quick rollback is needed. This is a powerful approach made possible on cloud environments where additional resources can be used temporarily.

## Feature Flags

### Deployment

A feature flag is code that lets you turn on or off specific features at run time, effectively decoupling code deployment from feature deployment. This enables you to _deploy_ the code for a feature incrementally over a period of time. Then, you can _release_ the feature to the users when you're ready.

Your service teams will need an interface to view and manage feature flags on the [platform](#platform). The code to lookup the flags can be included in a shared library.

**Incremental feature releases**

Feature flags make it possible to release features to sets of users in phases. Perhaps to 10% of your users at first or perhaps only to users in a specific region. The advantage here is that you'll have an opportunity to identify problems before they impact a large percentage of your users. It also enables quick rollback of the feature by turning off the flag.

**The flags should be short lived**

Feature flags should exist only until the feature is successfully deployed. Long running flags are a bad idea: they make it harder to support the users (as they'll be experiencing different behaviors), harder to test the system (with many code paths) and harder to debug the system. A flag should be scheduled to be removed soon after the feature is fully deployed.

**Only wrap the flag around the entry point**

The point of a feature flag is to decouple feature deployment from code deployment. For this, you just need to wrap the flag around the entry point to the feature, not all the code paths related to it. As an example, for a user interface visible feature, a flag could just hide the link/button in the interface to get to the feature.

## Configuration Management

### Deployment

A deployment package that's deployable anywhere shouldn't contain environment specific options or secrets. For that, we need a separate solution. The teams need the ability to manage the configuration and securely get them to the services on startup. Microservice platforms typically have built in solutions that can be leveraged for this.

Popular approaches for delivering the configuration are:

*   **Environment variables**: Load configuration into the environment variables of the service.
*   **Filesystem volume**: Mount a filesystem with the secrets and configuration into the service.
*   **Shared key/value store**: Have the service talk to a shared key/value store.

If you're using environment variables, one thing to watch out for is that they tend to be leaky by default. Exception handlers will grab and ship the environment to a logging platform. Child processes also duplicate the parent's environment on startup. As a result, it's possible to accidentally leak secrets. You can work around this by scrubbing the environment after reading the variables, but that's just an extra step that could be missed.

## Operations

## Centralized Logging

### Operations

Each instance of a service will generate logs. With a [system image for a deployment package](#deployment-package), those instances will get replaced every time a new release is deployed. You can't really store any logs on those instances, they would just get lost on the next deployment.

A centralized logging system should be provided to the service teams by the platform. All services should ship their logs to the same logging system in a standardized log format. This approach provides the service teams with the most flexibility - ability to search across all services, within a specific service, or within an instance of a service. All from the same place.

The code that ships logs to the centralized logging system could be included in shared libraries or be provided as part of the [service templates](#service-templates).

**But how do you track the impact of a request across multiple other services?**

There is where [correlation IDs](#correlation-ids) come in. Pass a correlation ID when communicating with any service and have the services include them into their log entries. Now, when you search across all services for the correlation ID, you're able to see the timeline of side effects from the original request across all services.

## Centralized Monitoring

### Operations

When failures happen, tools that can help quickly understand the scope and source of the problem are invaluable. Centralized monitoring should be a core component of your [platform](#platform). It provides your team with a much needed big picture and is especially helpful if you're experiencing cascading failures.

For high availability, you will almost always be running more than one instance of a service behind a [load balancer](#load-balancers). Your monitoring solution should have the ability to aggregate metrics across instances. Additionally, you need to be able to quickly drilldown on those aggregated metrics to see their components in detail. All of this helps quickly assess if an identified failure is occurring service wide or is isolated to a specific instance of a service.

**What kind of metrics should be monitored?**

This can be broken down into a few different types:

*   **Infrastructure**: Data you can gather at the OS level. Filesystem operations, filesystem latencies, network operations, memory usage, CPU usage.
*   **General**: Inbound requests to the service. Request count, request latency, error count (total and broken down per error code).
*   **Integrations**: Downstream requests made by the service to other services. Request count, request latency, error count (total and broken down per error code).
*   **External Services**: Communications with third party hosted services or other systems managed outside of the microservice platform.
*   **Service Specific**: Any other metrics specific to the service.

Everything except service specific metrics can be captured automatically by code in the [service templates](#service-templates) or shared libraries. With automatic capturing in place, you'll also be able to provide the service teams with a useful initial configuration for monitoring their services.

**Distributed tracing to connect the dots**

Although monitoring solutions do a good job at identifying what's happening in and around a specific service, it's still hard to connect the dots across the services and understand the big picture.

A distributed tracing system tracks requests as they break down into additional requests across your services. All this data is then visualized as a timeline. You get a ton of insight into how certain requests flow across your services and are able to quickly identify bottlenecks.

Distributed tracing is to monitoring what [correlation IDs](#correlation-ids) are to logging. The two are similar enough that ID used to identify the request by the tracing system could also double as a correlation ID.

## Auto Scaling

### Operations

Services that are [stateless](#stateless) are inherently easy to scale. Just add more instances as needed behind your [load balancer](#load-balancers). The information needed to make a scaling decision (cpu/memory usage, etc) can be retrieved from the [monitoring](#monitoring) platform.

Many microservice platforms have declarative interfaces to handle instance count, which can be quite handy. You tell it how many instances you need, it makes it happen. All you really need to implement auto-scaling on such a platform is a way to update the "required instance count" programmatically. As an added bonus, the same process also takes care of failing instances by adding a new one whenever an existing one fails.

## External Services

### Operations

Your services will also need to talk to systems that are not created by your teams. Some examples are: databases, caches, message queues, email delivery systems. These systems can be made available to your teams as hosted services provided by a third party or custom services managed within your organization. Either way, given the large number of services and environments that may need their own instances of these systems, it's important to ensure you have automation around the provisioning and management of these systems.

**What about just wrapping them as services on the platform?**

It's definitely possible to provide a database system with persistent storage and integrate it into your [logging](#logging) and [monitoring](#monitoring) systems. However, this may not always be practical. Some systems have special infrastructure requirements, especially when considering high availability configurations. Some may not be in a position to be automatically restarted after a failure. You'll need to assess these on a case by case basis.

**What about having multiple services just share the systems?**

This works as long as you take care to ensure that each service isn't aware of another service's configuration or data. For example, multiple services could share a common data server, each with their own database. They have no knowledge of any other databases on the shared data server. When a particular service needs to scale faster than the others, it's database can be extracted into a dedicated data server.

The caveat with this approach, however, is that shared resources can be harder to independently isolate and monitor. For example, in a shared data server, it may be possible for one service to use an excessive amount of resources and unknowingly impact the performance of other services. If monitoring weren't granular enough, it would also take time to identify the problematic service.

## People

## Full Lifecycle Ownership

### People

Service teams should own, operate and evolve the services they build. Their work is done when the service is retired, not when it's shipped.

With this approach, those who feel the pain of poor architectural decisions are also able to fix them. The additional operational knowledge they gain is valuable input when deciding how to best evolve the service to meet future growth requirements. All of this encourages operational simplicity which ultimately results in improved stability of the services.

## Autonomous Full Stack Teams

### People

When you're building a number of small services, each team member will be part owner of multiple services. It's important that the team that owns the service has the skills and tools necessary to develop, deploy and operate the service. They should be fully autonomous in their daily operations so they're able to react quickly to changing business requirements.

**Managing team turnover**

People quit from time to time. When that happens, you need to ensure no service goes ownerless. Even a service that's been running without fuss for a long time needs someone responsible for it when things go wrong.

People also move around within an organization. Consistency in development, deployment and operations practices across your microservices can minimize the learning curve when service ownership changes hands.

**How big should a team be?**

Communication gets harder as a team gets bigger. Teams should be big enough that they can get stuff done autonomously without wasting too much time in the _processes_ that enable them to communicate. Amazon, for example, is famously known for their two-pizza teams. Those are teams that can be fed with two pizzas.

## References

I've done my best to learn from those who've gone down the path of microservices before:

*   Netflix: [Migrating to Microservices](https://www.infoq.com/presentations/migration-cloud-native), [Deployment](https://www.infoq.com/news/2013/06/netflix), [Builds](http://techblog.netflix.com/2016/03/how-we-build-code-at-netflix.html)
*   Gilt: [Scaling Microservices](https://www.infoq.com/news/2015/04/scaling-microservices-gilt), [The Essentials](https://www.infoq.com/presentations/microservice-arch-gilt), [Making It Work](http://tech.gilt.com/2014/11/14/making-architecture-work-in-microservice), [Scaling Microservices](https://www.youtube.com/watch?v=ZxE_wLWu1x4)
*   SoundCloud: [Microservices](http://philcalcado.com/2015/09/08/how_we_ended_up_with_microservices.html), [Dealing With the Monolith](https://developers.soundcloud.com/blog/building-products-at-soundcloud-part-1-dealing-with-the-monolith), [Breaking the Monolith](https://developers.soundcloud.com/blog/building-products-at-soundcloud-part-2-breaking-the-monolith), [Standardizing on a Stack](https://developers.soundcloud.com/blog/building-products-at-soundcloud-part-3-microservices-in-scala-and-finagle)
*   Google: [Monolith to Microservices](http://www.ustream.tv/recorded/61479577), [Deep Lessons](http://highscalability.com/blog/2015/12/1/deep-lessons-from-google-and-ebay-on-building-ecosystems-of.html)
*   [Amazon](https://queue.acm.org/detail.cfm?id=1142065)
*   [eBay](http://www.addsimplicity.com/downloads/eBaySDForum2006-11-29.pdf)
*   [Yelp](http://engineeringblog.yelp.com/2015/03/using-services-to-break-down-monoliths.html)
*   [Spotify](https://www.youtube.com/watch?v=7LGPeBgNFuU)
*   [REA Group](https://yow.eventer.com/yow-2014-1222/the-odyssey-from-monoliths-to-microservices-at-realestate-com-au-by-beth-skurrie-and-evan-bottcher-and-jon-eaves-1751)
*   [Otto](https://dev.otto.de/2015/09/30/on-monoliths-and-microservices/)
*   [AutoScout24](https://www.infoq.com/news/2016/02/autoscout-microservices)
*   [Martin Fowler](http://martinfowler.com/articles/microservices.html)
*   [12 factor app](http://12factor.net/)
*   [SOA without the ESB](https://www.infoq.com/presentations/soa-without-esb)
*   [Building Microservices](http://shop.oreilly.com/product/0636920033158.do)
*   [Microservice Architecture](http://shop.oreilly.com/product/0636920050308.do)
