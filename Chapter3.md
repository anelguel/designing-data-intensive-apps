# Chapter 3: Storage and Retrieval

## Data Structures That Power Your Database

The general idea behind an *index* is to keep some additional metadata on the side, which acts as a signpost and helps you  locate the data you want.

An index is an additional structure that is derived from the primary data. Many databases allow you to add and remove indexes, and this doesn't affect the contents of the database; it only affects the preformance of the queries.

This is an important trade-off in storage systems: well chosen indexes speed up read queries, but every index slows down writes.

### Hash Indexes 