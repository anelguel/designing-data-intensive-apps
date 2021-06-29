# Chapter 1: Reliable, Scalable, and Maintainable Applications

**Data-Intensive Applications** usually need to:
* **Store data** so that they, or another application, can find it again later (_databases_)
* **Remember the result of an expensive operation**, to speed up reads (_caches_)
* **Allow users to search data** by keyword or filter it in various ways (_search indexes_)
* **Send a message to another process**, to be handled asynchronously (_stream processing_)
* Periodically **crunch a large amount of accumulated data** (_batch processing_)

The foundation of all data systems, whether running on a single machine or distributed across a cluster of machines is:

* **Reliability**: tolerating hardware and software faults, human error
* **Scalability**: Measuring load and performance, latency, percentiles and thoughput
* **Maintainability**: Operability, simplicity, and evolvability

## Reliability
Typical expectations of software reliability include:
* The application preforms the function that the user expected.
* It can tolerate the user making mistakes or using the software in unexpected ways.
* Its performance is good enough for the required use case, under the expected load and data volume.
* The system prevents any unauthorized access and abuse.
_In other words, we can understand **reliability** to mean "continuing to work correctly, even when things go wrong."

Reliability issues usually relate to **hardware faults**, **software errors**, and **human errors**.

## Scalability
**Scalability** is the term we use to describe a system's ability to cope with increased load.

### Describing Load
It's important to **describe the current load on the system**. Load can be described with a few numbers which we call *load parameters*. Load parameters can vary but may be requests per second to a web server, the ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache, or others.

### Describing Performance
Once load is described, you can **investigate performance** by seeing what happens when the load increases.

To investigate performance, you may ask yourself two questions:
* When a load parameter is increased and the system resources remain unchanged, how is the performance of the system affected?
* When a load parameter is increased, how much do you need to increase the resources if you want to keep performance unchanged?

In online systems, what's usually more important is the service's **response time** - that is, the time between a client sending a request and the receiving response.

**Response time** is what the client sees: besides the actual time to process the request (the service time), it includes network delays and queueing delays.

**Latency** is the duration that a request is waiting to be handled - during which it is *latent*, awaiting a service.

These terms are often used synonymously, but they are not the same.

## Maintainability
Maintanence of software includes fixing bugs, keeping its systems operational, investigating failures, adapting it to new platforms, modifying it for new use cases, repaying technical debt, and adding new features. These are so-called *legacy* systems.
 
We pay special attention to these principles for software systems:
* **Operability** - Make it easy for operations teams to keep the system running smoothly.
