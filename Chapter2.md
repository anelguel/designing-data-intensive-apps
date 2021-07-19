# Chapter 2: Data Models and Query Languages

**Relational Data Model**

The best-known data model today is probably that of SQL, based on the relational model propsed by Edgar Codd in 1970: data is organized into *relations* (called *tables* in SQL), where each relation is an unordered collection of *tuples* (*rows* in SQL).

**The Birth of NoSQL**

There are several driving forces behind the adoption of *Not Only SQL* databases, including:

* A need for greater scalability than relational databases can easily achieve, including very large datasets or very high write throughput
* A widespread preference for free and open source software over commercial database products
* Specialized query operations that are not well supported by the relational model
* Frustration with restrictiveness of relational schemas, and a desire for a more dynamic and expressive data model 

**The Object-Relational Mismatch**

Most application development today is done in object-oriented programming languages, which leads to a common criticism of the SQL data model: if data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns.

**Many-to-One and Many-to-Many Relationships**

There are time when you'll have many-to-one or many-to-many relationships. The book used LinkedIn profiles as an example. Bill Gate's location, "Seattle" and industry,"Philanthropy" can be saved in a database as standarized lists instead of plain-text strings. This is up for the developer to decide. One is never intrinsically better than the other, rather they vary on a case by case basis.

Some pros to having a seperate `location` and `industry_id` table include:

* Consistent style and spelling across profiles
* Avoiding ambiguity (e.g., if there are several cities with the same name)
* Ease of updating - the name is stored in only one place, so it is easy to update across the board if it ever needs to be changed (e.g. change of city name due to political events)
* Localization support - when the site is translated into other languages, the standardized lists can be localized, so the region and industry can be displayed in the viewer's language
* Better search - e.g., a search for philanthropists in the state of Washington can match this profile, because the list of regions can encode the fact that Seattle is on Washington (which is not apparent from the string "Greater Seattle Area")

**Are Document Databases Repeating History?**
Various solutions were proposed to solve the limitations of the hierarchical model where joins aren't supported.

*The network model*

Also known as the CODASYL (Conference on Data Systems Languages) model, this model initally had a large following, but eventually faded into obscurity. 

It is a gernalization of the hierarchical model. In the tree structure of the hierarchical model, every record has exactly one parent; in the network model, a record could have multiple parents. 

For example, there could be one record for the "Greater Seattle Area" region, and every user who lived in that region could be linked to it.

*The relational model*

The relational model became SQL, and took over the world of data!

What the relational data model did, by contrast to the network model, was to lay out all the data in the open: a relation (table) is simply a collection of tuples (rows), and that's it.

Querying happens automatically by the query optimizer, not by the application developer. Also, you can declare indexes

*Comparison to document databases*

Both relational and document databases refer to have a unique identifer, which is called a foreign key in relational databases and a document reference in document databases.

**Relational Versus Document Databses Today**

The main arguments in favor of the divument data model are schema flexibility, better performabe due to locality and that for some applications it is closer to the data structures used by the application.

The relational model counters by providing better sipport got joins, and many-to-one and many-to-many relationships.

*Which data model leads to simpler application code?*

Document model:
* Good for data that has document-like structure (i.e. a tree of one-to-many relationships, where typically the entire tree is loaded at once).
    * Limitations include: 
        * You cannot refer directly to a nested item within a document but instead you need to say something like, "the second item in the list of positions fo ruser 251"
        * Poor support for joins. If joins are involved, you pretty much want to go with the relational data model.

Schema flexibility in the document model

*Data locality for queries*

If data is split across multiple tables (like the LinkedIn example), muliple index lookups are required to retreive it all, which may require more desk seeks and take more time.

The locality advantage only applies if you need large parts of the document at the same time. The database typically needs to load the entire document, even if you one access a small portion of it, which can be wasteful on large documents.

Convergence of document and relational databases

Query Languages for Data
Declarative Queries on the Web
MapReduce Querying

Graph-Like Data Models
Property Graphs
The Cypher Query Language
Graph Queries in SQL
Triple-Stores and SPARQL
The semantic web
The RDF data model
The SPARQL query language

The Foundation: Datalog