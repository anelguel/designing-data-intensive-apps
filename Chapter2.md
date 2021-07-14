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

The network model
The relational model
Comparison to document databases

Relational Versus Document Databses Today
Which data model leads to simpler application code?
Schema flexibility in the document model
Data locality for queries
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