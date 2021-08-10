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

* When you want to write data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (for example, a JSON document). Since a ponter wouldn't make sense to any other process, this sequence-of-bytes representation looks quite different from the data structures that are normally used in memory.

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

Apache Thrift and Protocol Buffers are binary encoding libraries based on binary encodingl Protocol Buffers was developed at Google and Thrift for Facebook. They were both made open-source in 2007-2008.

Both these languages come with a code generation tool that takes a schema definition and produces classes that implement the schema in various programming languages.

### Avro

Apache Avro is another binary encoding format that also uses a schema to specify the structure of the data being encoded. It evolved in 2009 by Hadoop as a reaction to Thrift not being a good fit for Hadoop's use cases. 

It uses a schema to specify the structure of the data being encoded. It has two schema languages (one intended for human editing, and one based on JSON).

## Modes of Dataflow

Whenever you want to send some data to another process with which you don't share memory, you need to encode it as a sequence of bytes (JSON, XML, etc.) 

The ways that data can flow from one process to another are diverse, here are three that will be discussed:

* Via databases

* Via service calls

* Via asynchronour message passing

### Dataflow Through Databases

In a database, the process that writes the database encodes it, and the process that reads from the database decodes it.

In general, it's common for several different processes to be accessing a database at the same time. Those processes might be several different applications or services, or they may simply be several instances of the same service (running in parallel, etc.) Either way, in an einvornment where the application is changing, it is likely that some processes accessing the database will be running newer code and some will be running older code, so some instances have been updated while others haven't yet.

This means that a value in the database may be written by a newer version of the code, and subsequently read by an older version of the code that is still running. Thus, forward compatibility is also often required for databases.

