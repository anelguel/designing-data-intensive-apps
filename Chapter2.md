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