# Chapter 4: Encoding and Evolution
In large applications, code changes cannot happen instantaneously: 

* With server-side applications you may want to perform a *rolling upgrade* (also known as a *staged rollout*), deplying the new version to a gew nodes at a time, cheching whether the new version is running smoothly, and gradually working your way through all the nodes. This allows new versions to be deployed without service downtime, and thus encourages more frequent releases and better evolvability. 

* With client-sode applications you're at the mercy of the user, who may or may not install the update for some time.

That means that old and new versions of the code and new data formats, may potentially coexist in the system at the same time. In order for the system to continue running smoothly, we need to maintain compatibility in both directions:

*Backward compatibility*: newer code can read data that was written by older code

*Forward compatibility*: Older code can read data that was written by newer code

## Formats for Encoding Data

Programs usually work with data in (at least) two different representations:

* In memory, data is kept in objects, structs, lists, arrays, hash tables, and so on. These data structures are optimized for efficient access and manipulation by the CPU (typically using pointers).

* When you want to write data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (for example, a JSON document). Since a pointer wouldn't make sense to any other process, this sequence-of-bytes representation looks quite different from the data structures that are normally used in memory.

Thus, we need some kind of translation between the two representations. This translation from the in-memory representation to a byte sequence is called *encoding* (also known as serialization or marshalling) and the reverse is called decoding (parsing, deserialization, unmarshalling). A more common term than encoding is serialization.

## Language-Specific Formats

Many programming languages come with built-in support for encoding in-memory objects into byte sequences (Ruby has Marshal, Python has pickle, etc.) And there also exists third-party libraries like Kryo for Java.

These encoding libraries are convienent, because they allow in-memory objects to be saved and restored with minimal additional code. However, there are some limitations, including:

* The encoding is ogten tied to a particular programming language, and reading the data in another language is difficult.

* In order to restore data in the same object types, the decoding process needs to be able to instantiate arbitrary classes. This is often a source of security problems.

* Versioning data is often an afterthought in these libraries: as they are intended for quick and easy encoding of data, they often neglect the inconvient problems of forward and backward compatibility.

* Effieciency (CPU time taken to encode and decode, and the size of the encoded structure) is often and afterthought.

For these reasons, it's generally a bad idea to use your language's built in encoding for anything other than very transient purposes.

## JSON, XML, and Binary Variants
Standarized encodings that can be written and read by many programming languages, JSON and XML are the obvious contenders. They are widely known, supported -- and also widely disliked. 

XML is criticized for being too verbose and unecessarily complicated. JSON's popularity is mainly due to its built-in support in web browsers (by virtue of being a subset of Javascript) and simplcity relative to XML. CSV is another popular language-independent format, although less powerful.

JSON, XML and CSV are textual formats, and thus somewhat human-readable. Some subtle problems when them include:

* A lot of ambiguity around the encoding of numbers.

* JSON and XML have good support for Unicode character strings (i.e. human-readable text), but they don't support binary strings (sequences of bytes without a character encoding).

* There is optional schema support for both XML and JSON. These schema languages are quite powerful, and thus quite complicated to learn and implement. 

* CSV does not have any schema, so it is up to the application to define the meaning of each row and column.

Despite these flaws, it's likely that these three standardized encodings will remain popular, especially as  data interchange formats (i.e. for sending data from one organization to another).

### Binary encoding

When it comes to formats that are faster, JSON is less verbose than XML, but both still use a lot of space compared to binary formats.

This led to development of a bunch of binary encodings from JSON (MessagePack, BSON, BJSON, Smile) and XML (WBXML and Fast Infoset). These have been adopted for niches (like a company that uses large datasets and can agree on one format),  but none of them are as widely adopted as JSON and XML. Also, the reduction in spaced used is very small so it would only be practical to adopt a binary encoding if you're working at a very large scale.

### Thrift and Protocol Buffers

Apache Thrift and Protocol Buffers are binary encoding libraries based on binary encoding. Protocol Buffers was developed at Google and Thrift for Facebook. They were both made open-source in 2007-2008.

Both these languages come with a code generation tool that takes a schema definition and produces classes that implement the schema in various programming languages.

### Avro

Apache Avro is another binary encoding format that also uses a schema to specify the structure of the data being encoded. It evolved in 2009 by Hadoop as a reaction to Thrift not being a good fit for Hadoop's use cases. 

It uses a schema to specify the structure of the data being encoded. It has two schema languages (one intended for human editing, and one based on JSON).

## Modes of Dataflow

Whenever you want to send some data to another process with which you don't share memory, you need to encode it as a sequence of bytes (JSON, XML, etc.) 

The ways that data can flow from one process to another are diverse, here are three that will be discussed:

* Via databases

* Via service calls

* Via asynchronous message passing

### Dataflow Through Databases

In a database, the process that writes the database encodes it, and the process that reads from the database decodes it.

In general, it's common for several different processes to be accessing a database at the same time. Those processes might be several different applications or services, or they may simply be several instances of the same service (running in parallel, etc.) Either way, in an enivornment where the application is changing, it is likely that some processes accessing the database will be running newer code and some will be running older code, so some instances have been updated while others haven't yet.

This means that a value in the database may be written by a newer version of the code, and subsequently read by an older version of the code that is still running. Thus, forward compatibility is also often required for databases.


### Different values written at different times
*Data outlives code* explains the observation that when you deploy a new version of your application code, you may replace an old version with a new version within minutes. That is not the case with data, where the data contents remain the same.

Rewriting (*migrating*) data into a new schema is certainly possible, but it's an expensive thing to do on a large dataset, so most databases avoid it is possible. Most relational databases allow schema changes, without rewriting existing data. When an old data row is read, the databse fills in nulls for any columns that are missing from the encoded data on disk.

Schema evolution thus allows the entire database to appear as if it was encoded with a single schema, even though the underlying storage may contain records encoded with various historical versions of the schema.


## Data Through Services: REST and RPC

When you have processes that need to communicate over a network, there are a few different ways of arranging that communication. The most common arrangement is to have two roles: clients and servers. The servers expose an API over the network, and the clients can connect to the servers to make requests to that API. The API exposed by the server is known as a service.

### Web services
When HTTP is used as the underlying protocol for talking to the service, it is called a *web service*. This is perhaps a slight misnomer, because webb services are not only used on the web, but in serveral different contexts, for example:

1. A client application running on a user's device making requests to a service over HTTP. These requests typically go over the public internet.

2. One service making requests to another service owned by the same organization, often located within the same datacenter, as part of a service-oriented/microservices architecture. Software the supposers this kind of use case is sometimes called *middleware*.

3. One service making requests to a service owned by a different organization, usually via the internet.

**There are two popular approaches to web services: REST and SOAP.**

REST is not a protocol, but rather a design philosophy that builds upon the principles of HTTP. It emphasizes simple data formats, using URLs for identifying resources and using HTTP features for cache control, authentication, and content type negotiation. REST has been gaining popularity compared to SOAP, at least in the context of cross-organizational service integration, and is often associated with microservices. An API designed according to the principles of REST is called RESTful. 

SOAP is an XML-based protocol for making network API requests. Although it is most commonly used over HTTP, it aims to be independent from HTTP and avoids using most HTTP features. The API of SOAP web service is described using an XML-based language called the Web Services Description Language, or WSDL.

WSDL enables code generation so that a client can access a remote service using local classes and method calls. WSDL is not designed to be human readable and SOAP messages are often too complex to construct manually, therefore a lot of SOAP users rely heavily on tool support, code generation and IDEs.

RESTful APIs tend to favor simpler approaches, typically involving less code gneration and automated tooling. A definition formar such as OpenAPI, also known as Swagger, can be used to describe RESTful APIs and produce documentation.