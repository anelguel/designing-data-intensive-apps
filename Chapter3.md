# Chapter 3: Storage and Retrieval

## Data Structures That Power Your Database

The general idea behind an *index* is to keep some additional metadata on the side, which acts as a signpost and helps you  locate the data you want.

An index is an additional structure that is derived from the primary data. Many databases allow you to add and remove indexes, and this doesn't affect the contents of the database; it only affects the preformance of the queries.

This is an important trade-off in storage systems: well chosen indexes speed up read queries, but every index slows down writes.

### Hash Indexes 

The textbook doesn't explain hash indexes and functions, but I retrieved the following from https://www.mssqltips.com/sqlservertip/3099/understanding-sql-server-memoryoptimized-tables-hash-indexes/ :

*What is a hash index?*

Basically, a hash index is an array of N buckets or slots, each one containing a pointer to a row. Hash indexes use a hash function F(K, N) in which given a key K and the number of buckets N , the function maps the key to the corresponding bucket of the hash index. I want to emphasize that the buckets do not store the keys or its hashed value. They only store the memory address in which the data is placed.

*What is a hash function?*

A hash function is any algorithm that maps data of variable length to data of a fixed length in a deterministic and close to random way. A very simple hash function would be a string that returns its length, so F("John") = 4 and F("Ed") = 2. If we define a hash index of 5 buckets using this function then the pointers that point to "John" and "Ed" are stored at buckets 4 and 2 respectively. The following image will help you understand.

![](images\chapter3\hashindexandfunc.jpg)

At this point everything seems clear, but what happens when we apply this simple hash function on "Bill"? The length of string "Bill" is 4, so F("Bill") = 4 which is the same as F("John") = 4. Don't get confused, the fact that two keys have the same Hash value doesn't mean the keys are the same. This is called a "collision" and is very common in hash functions. Of course the more collisions a function has the worse a function is because a large number of hash collisions can have a performance impact on read operations. Also collisions may be caused when the number of buckets is too small compared with the number of distinct keys.

Back to the text, which describes hash index similarily, as seen in the following picture:

![](images\chapter3\pg72.jpg)
*Here the hash function F(K,N) would be F("123456", 0) and F(42, 64), respectively. These key-value pairs are similar to a dictionary in most programming languages.*

*Compaction* means throwing away duplicate keys in the log, and keeping only the most recent update for each key: 
![](images\chapter3\pg73.jpg)
*Only the most recent value is retained, and addresses the optimizing disk space.*

Lastly, because compaction makes segments much smaller, you can also merge several segments together. Segments are never modified after they've been written, so the merged segment is written to a new file. The merging and compaction of frozen segments can be done in a background thread, and while it is going on, we can still continue to serve read and write requests as normal, using the old segment files. After the merging process is complete, we switch read requests to using the new merged segment instead of the old segments - and then the old segment files can simply be deleted as seen below:

![](images\chapter3\pg74.jpg)

This approach is good, but comes with its limitations, including: file format, deleting records, crash recovery, partially written records, and concurrency control.

## SSTables and LSM-Trees

An SSTable is shown below

![](images\chapter3\pg76.jpg)

Sorted Strings Table (SSTable) is a persistent file format used by Scylla, Apache Cassandra, and other NoSQL databases to take the in-memory data stored in memtables, order it for fast access, and store it on disk in a persistent, ordered, immutable set of files. Immutable means SSTables are never modified. They are later merged into new SSTables or deleted as data is updated. (from https://www.scylladb.com/glossary/sstable/)

LSM Tree A log-structured merge-tree (or LSM tree) is a data structure with performance characteristics that make it attractive for providing indexed access to files with high insert volume, such as transactional log data. LSM trees, like other search trees, maintain key-value pairs. LSM trees maintain data in two or more separate structures, each of which is optimized for its respective underlying storage medium; data is synchronized between the two structures efficiently, in batches. (from Wikipedia)

B Trees
A B-tree is a self-balancing tree data structure that maintains sorted data and allows searches, sequential access, insertions, and deletions in logarithmic time. The B-tree generalizes the binary search tree, allowing for nodes with more than two children. Unlike other self-balancing binary search trees, the B-tree is well suited for storage systems that read and write relatively large blocks of data, such as disks. It is commonly used in databases and file systems. (from Wikipedia)