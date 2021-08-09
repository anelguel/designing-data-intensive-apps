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

XML is criticized forbeing too verbose and unecessarily complicated. JSON's popularity is mainly due to its built-in support in web browsers (by virtue of being a subset of Javascript) and simplcity relative to XML. CSV is another popular language-independent format, although less powerful.

JSON, XML and CSV are textual formats, and thus somewhat human-readable. Some subtle problems when them include:

* A lot of ambiguity around the encoding of numbers.

* JSON and XML have good support for Unicode character strings (i.e. human-readable text), but they don't support binary strings (sequences of bytes without a character encoding).