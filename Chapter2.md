# Chapter 2: Data Models and Query Languages

## Relational Data Model

The best-known data model today is probably that of SQL, based on the relational model propsed by Edgar Codd in 1970: data is organized into *relations* (called *tables* in SQL), where each relation is an unordered collection of *tuples* (*rows* in SQL).

## The Birth of NoSQL

There are several driving forces behind the adoption of *Not Only SQL* databases, including:

* A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput
* A widespread preference for free and open source software over commercial database products
* Specialized query operations that are not well supported by the relational model
* Frustration with restrictiveness of relational schemas, and a desire for a more dynamic and expressive data model 

## The Object-Relational Mismatch

Most application development today is done in object-oriented programming languages, which leads to a common criticism of the SQL data model: if data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns.

## Many-to-One and Many-to-Many Relationships

There are time when you'll have many-to-one or many-to-many relationships. The book used LinkedIn profiles as an example. Bill Gate's location, "Seattle" and industry,"Philanthropy" can be saved in a database as standarized lists instead of plain-text strings. This is up for the developer to decide. One is never intrinsically better than the other, rather they vary on a case by case basis.

Some pros to having a seperate `location` and `industry_id` table include:

* Consistent style and spelling across profiles
* Avoiding ambiguity (e.g., if there are several cities with the same name)
* Ease of updating - the name is stored in only one place, so it is easy to update across the board if it ever needs to be changed (e.g. change of city name due to political events)
* Localization support - when the site is translated into other languages, the standardized lists can be localized, so the region and industry can be displayed in the viewer's language
* Better search - e.g., a search for philanthropists in the state of Washington can match this profile, because the list of regions can encode the fact that Seattle is on Washington (which is not apparent from the string "Greater Seattle Area")

## Are Document Databases Repeating History?

Various solutions were proposed to solve the limitations of the hierarchical model where joins aren't supported.

### The network model

Also known as the CODASYL (Conference on Data Systems Languages) model, this model initally had a large following, but eventually faded into obscurity. 

It is a gernalization of the hierarchical model. In the tree structure of the hierarchical model, every record has exactly one parent; in the network model, a record could have multiple parents. 

For example, there could be one record for the "Greater Seattle Area" region, and every user who lived in that region could be linked to it.

### The relational model

The relational model became SQL, and took over the world of data!

What the relational data model did, by contrast to the network model, was to lay out all the data in the open: a relation (table) is simply a collection of tuples (rows), and that's it.

Querying happens automatically by the query optimizer, not by the application developer. Also, you can declare indexes

### Comparison to document databases

Both relational and document databases refer to have a unique identifer, which is called a foreign key in relational databases and a document reference in document databases.

## Relational Versus Document Databses Today

The main arguments in favor of the divument data model are schema flexibility, better performabe due to locality and that for some applications it is closer to the data structures used by the application.

The relational model counters by providing better sipport got joins, and many-to-one and many-to-many relationships.

### Which data model leads to simpler application code?

Document model:
* Good for data that has document-like structure (i.e. a tree of one-to-many relationships, where typically the entire tree is loaded at once).
    * Limitations include: 
        * You cannot refer directly to a nested item within a document but instead you need to say something like, "the second item in the list of positions fo ruser 251"
        * Poor support for joins. If joins are involved, you pretty much want to go with the relational data model.

### Schema flexibility in the document model

Most document databases, and the JSON support in relational databases, do not enforce any schema on the data in documents. No schema means that arbitrary keys and values can be added to a document, and when reading, clients have no guarantee as to what fields the dicuments may contain.

Document databases are sometimes called schemaless, but that is misleading, as the code tht reads the data usually assumes some kind of strcture - i.e. there is an implict schema, but ut us not enforced by the database. The more accurate term is schema-on-read.

### Data locality for queries

If data is split across multiple tables (like the LinkedIn example), muliple index lookups are required to retreive it all, which may require more disk seeks and take more time.

The locality advantage only applies if you need large parts of the document at the same time. The database typically needs to load the entire document, even if you one access a small portion of it, which can be wasteful on large documents.

### Convergence of document and relational databases

Most relational database systems (other than MySQL) have supported XML since the mid-2000s. This includes function to make local modifications to XML documents and the ability to index and query inside XML documents, which allows applications to use data models very similar to what they would do when using a document database.

On the document database side, ReThinkDB spports relational-like joins in its query language, and some MogoDB drivers automaticall resolve database references (effectively preforming a client-side join).

It seems that document and relational databases are becoming more similar over time, and that's a good thing! A hybrid of the relational and document models is a good route for databases to take in the future.

### Query Languages for Data

SQL is a *declarative* query language, while IMS and CODAYSL is queried using *imperative* code.

Imperative language tells the computer to perform certain operations in a certain order. 

In declarative language, you specify the pattern of the data you want - what conditions the results must meet, and how you want the data to be transotmed - but not *how* to achieve that goal. It is up to the database system's query optimizer to decide which indexes and joins to use, and in which order.

### Declarative Queries on the Web

In a web browswer, using declarative CSS styling is much better than manipulating styles imperatively in JavaScript. Similarily, in databases, declarative query languages like SQL turned out to be much better than imperative query APIs.

### MapReduce Querying

MapReduce is a programming model for processing large amounts of data in bulk across many machines, popularized by Google.

MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework.

### Graph-Like Data Models

A graph consists of two kinds of objects: vertices (also known as nodes or entities) and edges (also known as relationships or arcs).

We will discuss the *property graph* model (implemented by Neo4j, Titan, and InfiniteGraph) and the *triple-store* model (implemented by Datomic, AllegroGraph, and others).

### Property Graphs

In the property graph model, each vertex (node)/entity consists of: 

* A unique identifier
* A set of outgoing edges
* A set on incoming edges
* A collection of properties (key-value pairs) 

Each edge (relationship/arc) consists of:
* A unique identifer
* The vertex at which the edge starts (the *tail vertex*)
* The vertex at which the edge ends (the *head vertex*)
* A label to describe the kind of relationship between the two vertices
* A collection of properties (key-value pairs)

You can think of a graph store as consisting of two relational tables, one for vertices and one for edges.

Some important aspects of this model are:

1. Any vertex can have an edge connecting it with any other vertex. There is no schema that restricts which kinds of things can or cannit be associated.

2. Given any vertex, you can efficiently find both its incoming and its outgoing edges, and thus *traverse* the graph - follow the path through a chain of vertices, both forward and backward.

3. By using different labels for different kinds of relationships, you can store several different kinds of information in a single graph, while still maintaining a clean data model.

Graphs are good for evolvability: as you add features to you application, a graph can easily be extended to accomodate changes in your application's data structures.

### The Cypher Query Language

Cypher is a declarative query language for property graphs, created for Neo4j graph database.

### Graph Queries in SQL

You can put graph data in a relational structure, and query it using SQL.

### Triple-Stores and SPARQL

In a triple-store, all information is stored in the form of a very simple three-part statements: (subject, predicate, object). For example, in the triple (Jim, likes, bananas), Jim is the subject, likes in the predicate (verb), and bananas is the object.

### The semantic web

The semantic web is a simple and reasonable idea: websites already publish information as text and pictures for humans to read, so why don't they also publish information is machine-readable data for computers to read? This would be like a *web of data* - a kind of internet-wide "database of everything."

This was hyped in the 2000s, but hasn't seen any real sign of it actually being put into practice.

### The RDF data model

RDF is a standard model for data interchange on the Web. RDF has features that facilitate data merging even if the underlying schemas differ, and it specifically supports the evolution of schemas over time without requiring all the data consumers to be changed.

RDF extends the linking structure of the Web to use URIs to name the relationship between things as well as the two ends of the link (this is usually referred to as a “triple”). Using this simple model, it allows structured and semi-structured data to be mixed, exposed, and shared across different applications.

### The SPARQL query language

SPARQL is a query language for triple-stores using the RDP data model, it predates Cypher.

### The Foundation: Datalog

Datalog is much older language than SPARQL or Cypher, but its important because it provides the foudation that query language built upon. It's similar to a triple-store, but is generalized. Instead of writing a triple as (subject, predicate, object), it is predicate (subject, object).