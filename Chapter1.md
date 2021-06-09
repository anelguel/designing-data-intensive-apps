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

Reliability issues usually relate to **hardware faults**, **software errors**, and *human errors**.

## Scalability
**Scalability** is the term we use to describe a system's ability to cope with increased load.

It's important to describe the current load on the system. 
