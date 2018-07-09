# Designing Data Intensive Applications

## Table of contents:
1. [Chapter 1: Reliable, Scalable, and Maintainable Applications](#Chapter1)
2. [Chapter 2: Data Models and Query Languages](#Chapter2)
3. [Chapter 3: Storage and Retrieval](#Chapter3)
4. [Chapter 4: Encoding and Evolution](#Chapter4)
5. [Chapter 5: Replication](#Chapter5)
6. [Chapter 6: Partitioning](#Chapter6)
7. [Chapter 7: Transactions](#Chapter7)
8. [Chapter 8: The Trouble with Distributed Systems](#Chapter8)
9. [Chapter 9: Consistency and Consensus](#Chapter9)
10. [Chapter 10: Batch Processing](#Chapter10)
11. [Chapter 11: Stream Processing](#Chapter11)
12. [Chapter 12: The Future of Data Systems](#Chapter12)

## Chapter 1: Reliable, Scalable, and Maintainable Applications<a name="Chapter1"></a>
### Thinking About Data Systems
The three concerns that are important in most software systems are:

    * Reliability: The system should continue to work correctly (even performant) even in the face of adversity 
    * Scalability: Scalability is the term we use to describe a system’s ability to cope with increased load
    * Maintainability: Different people should be able to work productively on the system
    
### Reliability
The things that can go wrong are called faults, and systems that anticipate faults and can cope with them are called
 fault-tolerant or resilient, although it only makes sense to talk about tolerating certain types of faults. A fault 
 is usually defined as one component of the system deviating from its spec, whereas a failure is when the system as
  a whole stops providing the required service to the user. 

#### Hardware Faults
Typical approach to this problem is to add redundancy to the individual hardware components in order to reduce the 
failure rate of the system.

#### Human Errors
To alleviate this problem, design systems in a way that minimizes opportunities for error decouple the places where 
people make the most mistakes from the places where they can cause failures, test thoroughly at all levels, set up 
detailed and clear monitoring and implement good management practices and training.

### Scalability
#### Describing Load
Load can be described with a few numbers which we call load parameters (request per second, reads/writes ration, 
concurrent users...)

#### Describing Performance
Look at performance in two ways:

    * When a load parameter changes, how is the performance affected?
    * How much do you need to increase the resources to keep performance unchanged if you increase a load parameter?
    
Service level objectives (SLOs) and service level agreements (SLAs) are often defined through the performance 
percentiles around request time.

#### Approaches for Coping with Load
There is two ways to cope with increasing load: scaling up (more powerful machines) and scaling out (adding more 
nodes to the cluster).

### Maintainability
Software can be designed in such a way that it  minimizes maintenance pain. We will pay particular attention to three 
design principles for software systems:

    * Operability: Make it easy for operations teams to keep the system running smoothly
    * Simplicity: Make it easy for new engineers to understand the system, by removing as much complexity as possible 
      from the system
    * Evolvability: Make it easy for engineers to make changes to the system in the future, adapting it for 
    unanticipated use cases as requirements change
    
#### Operability: Making Life Easy for Operations
An operations team typically is responsible for the monitorization of the service, tracking down the cause of problems,
keeping the software and platforms updated, keeping tabs on how different systems affect each other, doing preventive
 work, setting best practices and tools, setting security, defining process and documenting all of the above. Good 
 operability means making routine tasks easy, allowing the operations team to focus on other activities

#### Simplicity: Managing Complexity
Reducing complexity greatly improves the maintainability of software, and thus simplicity should be a key goal for 
the systems we build. Abstraction removes unnecesary complexity: it can hide a great deal of implementation detail 
behind a clean, simple-to-understand façade.

#### Evolvability: Making Change Easy
The ease with which you can modify a data system, and adapt it to changing requirements, is closely  linked to its 
simplicity and its abstractions: simple and easy-to-understand systems are usually easier to modify than complex ones. 


## Chapter 2: Data Models and Query Languages<a name="Chapter2"></a>
### Relational Model Versus Document Model
In SQL data is organized into relations called tables, where each relation is an unordered collection of tuples (rows).

#### The Birth of NoSQL
The adoption of NoSQL databases is driven by:

    * A need for greater scalability than RDBMS, including very large datasets or very high write throughput
    * A widespread preference for free and open source software
    * Specialized query operations that are not well supported by the relational model
    * Frustration with the restrictiveness of relational schemas, desire for a more dynamic and expressive data model
    
#### The Object-Relational Mismatch
The impedance mismatch is the disconnection between the relational model and the object model, which can be partially
 solved by the ORM frameworks. Later versions of SQL support structured datatypes and XML within a single row. JSON 
 format can partially solve the impedance mismatch, it has better locality than multi-table schema as the data is 
 stored as a tree structure, but the lack of schema can be either an advantage or a disadvantage.
 
#### Many-to-One and Many-to-Many Relationships
Normalized data requires many-to-one relantionships which don't fit very well in the document model: support for 
joins in this model is often weak and you may need to perform this join in application code. 

#### Are Document Databases Repeating History?
The hierarchichal model (data is represented as a tree of records nested within records) has limitations that the 
relational and network model tries to solve.

##### The network model
Also named CODASYL model, is a generalization of the hierarchical model, although in this model, a record could have 
multiple parents. The links between records in the network model were not foreign keys, but pointers in a programming 
language (stored on disk). The only way of accessing a record was to follow a path (called access path) from a root 
record along these chains of links.
A query in CODASYL was performed by moving a cursor through the database by iterating over lists of records and 
 following access paths. If a record had multiple parents, the application code had to keep track of all the various 
 relationships. This makes updates difficult and queries complicated to write.
 
##### The relational model
In a relational database, the query optimizer automatically decides which parts of the query to execute in which 
order, and which indexes to use. This model makes querying more efficient, but adds complexity on the code optimizer.

##### Comparison to document databases
Document databases are similar to the networking model in regards to storing nested records, but when it comes to 
representing many-to-one and many-to-many relation‐ships, relational and document databases references the joined table 
by a unique identifier, which is called a foreign key in the RBDMS and a document reference in the document model.

#### Relational Versus Document Databases Today
##### Which data model leads to simpler application code?
If the data in your application has a document-like structure (i.e., a tree of one-to-many relationships, where 
typically the entire tree is loaded at once), use a document model. The document model still needs to have access 
patterns for nested structures. If many-to-many relationships are needed, you are most likely to use a relational model.
Usually highly interconnected data is more efficiently represented with the relational model and graph models.

##### Schema flexibility in the document model
No schema means that arbitrary keys and values can be added to a document, and when reading, clients have no guarantees
 as to what fields the documents may contain. Document databases has some kind of implicit schema, but it is not 
 enforced by the database (schema-on-read).
The schema-on-read approach is advantageous if the items in the collection don’t all have the same structure, which 
could happen due to have many different types of objects (and hence is not practical to put each type in its own 
table) or the structure of the table is determined by external systems.

##### Data locality for queries
If your application often needs to access an entire document, there is a performance advantage to this storage locality.
The locality advantage only applies if you need large parts of the document at the same time. Updates usually requires
 to write the whole document (only modifications that don’t change the encoded size of a document can easily be 
 performed in place), it is generally recommended that you keep documents fairly small and avoid writes that increase 
 the size of a document.
 
##### Convergence of document and relational databases
Most relational database systems has support for XML, including functions to make local modifications to XML documents 
and the ability to index and query inside XML documents (same applies to JSON). On the document database side support 
for relational-like joins (performing a client-side joins) have been added, although this is likely to be slower than a 
join performed in the database since it requires additional network round-trips and is less optimized.

### Query Languages for Data
SQL is a declarative query language, whereas IMS and CODASYL queried the database using imperative code. The fact 
that SQL is more limited in functionality gives the database much more room for automatic optimizations. Declarative 
languages are easier to parallelize.

#### MapReduce Querying
MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between: the 
logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework. Some 
document databases like MongoDB supports a limited form of map reduce.
The map and reduce functions are somewhat restricted in what they are allowed to do. They must be pure functions,
 which means they only use the data that is passed to them as input, they cannot perform additional database queries,
 and they must not have any side effects. 
 
### Graph-Like Data Models
A graph consists of vertices (also known as nodes or entities) and edges (also known as relationships or arcs). Many
 kinds of data can be modeled as a graph:
 
    * Social graphs: Vertices are people, and edges indicate which people know each other
    * The web graph: Vertices are web pages, and edges indicate HTML links to other pages
    * Road or rail networks: Vertices are junctions, and edges represent the roads or railway lines between them
    
An equally powerful use of graphs is to provide a consistent way of storing completely different types of objects 
in a single datastore.

#### Property Graphs
In the property graph model, each vertex consists of:

    * A unique identifier
    * A set of outgoing edges
    * A set of incoming edges
    * A collection of properties (key-value pairs)

Each edge consists of:

    * A unique identifier
    * The vertex at which the edge starts (the tail vertex)
    * The vertex at which the edge ends (the head vertex)
    * A label to describe the kind of relationship between the two vertices
    * A collection of properties (key-value pairs)
    
Modelling this into a relational model would be a two table model with:

```sql
CREATE TABLE vertices (vertex_id integerPRIMARYKEY, properties json);

CREATE TABLE edges (edge_id integer PRIMARY KEY,properties json, label text,
tail_vertex integer REFERENCES vertices (vertex_id), head_vertex integer REFERENCES vertices (vertex_id));
```

Worth notice the many to many relationship established between the vertices through the edges, and how having labels 
for different kind of relationships allows for the storage of different kinds of information in a single graph. 
Graphs can be easily extended to accommodate changes in the application's data structures.

#### The Cypher Query Language
Cypher is a declarative query language for property graphs. The following example creates 3 vertices and two edges 
between them each one with the label 'WITHIN':

```text
CREATE
  (NAmerica:Location {name:'North America', type:'continent'}),
  (USA:Location      {name:'United States', type:'country'  }),
  (Idaho:Location    {name:'Idaho',         type:'state'    }),
  (Idaho) -[:WITHIN]->  (USA)  -[:WITHIN]-> (NAmerica)
```

Which allow us to traverse the graph to ask questions like "give me all the people that were born in Idaho". As it is a
 declarative query language, you don’t need to specifyexecution details when writing the query (chosen by the query 
 optimizer). An example of that type of query is:
 
 ```text
MATCH
(person) -[:BORN_IN]->  () -[:WITHIN*0..]-> (us:Location {name:'United States'}),
(person) -[:LIVES_IN]-> () -[:WITHIN*0..]-> (eu:Location {name:'Europe'}) RETURN person.name
```

The `-[:WITHIN*0..]->` expresses "follow a WITHIN edge, zero or more times." It is like the * operator in a regular 
expression.
As suggested before,  we can also query graph data by using SQL if we put it in a relational structure. Although in 
SQL you usually know in advance which joins you need in your query while in graph databases is not.

#### Triple-Stores and SPARQL
In a triple-store, all information is stored in the form of very simple three-part statements: (subject, predicate,
object). The subject here is similar to a vertex in a graph, the object is either a primitive value (string, number) 
equivalent to the key and value of a property on the subject vertex, or another vertex in the graph (tail vertex). 

##### The semantic web
The Resource Description Framework (RDF) was intended as a mechanism for different websites to publish data in
 a consistent format, allowing data from different websites to be automatically combined into a web of data—a kind 
 of internet-wide "database of everything.". Triples can be a good internal data model to represent it.
 
##### The RDF data model
On the RDF data model, the subject, predicate, and object of a triple are often URIs (like <http://my-company
.com/namespace#lives_in> instead of WITHIN). The reasoning behind this  design is that you should be able to combine 
your data with someone else’s data, and if they attach a different meaning to the word within or lives_in, you won’t
 get a conflict because their predicates are. Something like avoiding collisions by using different namespaces.
 
##### The SPARQL query language
SPARQL is a query language for triple-stores using the RDF data model (similar to Cypher):

```text
PREFIX : <urn:example:>
SELECT ?personName WHERE {
  ?person :name ?personName.
  ?person :bornIn  / :within* / :name "United States".
  ?person :livesIn / :within* / :name "Europe".
}
```

Because RDF doesn’t distinguish between properties and edges but just uses predicates for both, you can use the 
same syntax for matching properties.

#### The Foundation: Datalog
Datalog’s data model is similar to the triple-store model. Instead of writing a triple as (subject, predicate, object), 
we write it as predicate(subject, object). In Datalog, we define rules that tell the database about new predicates. 
These predicates aren't triples stored in the database, but instead they are derived from data or from other rules. 
Rules can refer to other rules.


## Chapter 3: Storage and Retrieval<a name="Chapter3"></a>
### Data Structures That Power Your Database
Many databases internally use a log, which is an append-only data file. In order to efficiently find the value for a
 particular key in the database, we need a different data structure: an index. If you want to search the same data 
 in several different ways, you may need several different indexes on different parts of the data. There is an 
 important trade-off in storage systems: well-chosen indexes speed up read queries, but every index slows down writes.

#### Hash Indexes
A simple possible strategy to implement indexes for key-value data is to keep an in-memory hash map where every key 
is mapped to a byte offset in the data file. Bitcask is a storage engine that does this, stores all the keys in 
memory and values are either in memory (cache) or on disk, only a disk seek has to be done to load a value. This is 
well suited for frequently updated keys. The log can be break into segments of certain size (segments are closed when
 they reach certain size), once a segment is closed, compaction (throwing away duplicate keys keeping the most up to 
 date value) is performed an a new segment file is opened. After compaction, the segments can be merge if they are 
 small enough (segments are not modified so they are written to a new file), this is done in the background by a 
 separate thread. 
Some performance considerations are:
 
    * File format: binary preferred
    * Deleting records: a new append must be done to indicate the deletion and would be deleted on next merge
    * Crash recovery: Data needs to be reloaded in memory which can take time if segments are large
    * Partially written records: Checksums for records prevent corrupted parts of the logs being used
    * Concurrency control: A common implementation is to have a single writer thread per segment

Append only allows for multiple concurrent reads, is usually faster and merging segments avoids data file 
fragmentation over time. Although it is limited by the memory and range queries are not efficient.

#### SSTables and LSM-Trees
If instead of having the key-value pairs in the log in the order they were written we have the keys sorted by keys, 
we have what is called a Sorted String Table (SSTable). Keys must appear only once within each merged segment file. 
This is beneficial due to:

    * Merging files is simpler and efficient, mergesort algorithm (taking several segment and writting the lowest and
     most recent key to the segment in each step) can be used
    * No need for indexing all keys (you can jump to another close position and scan from there), sparse key indexes 
    can be used
    * Since read request needs to scan over several key-value pairs for request, it is possible to group and compress
     those records in a block before writting it, which reduces I/O bandwidth 
     
##### Constructing and maintaining SSTables
Maintaining a sorted structure on disk is possible, but maintaining it in memory is much easier. The storage engine 
can work as follows:

    * When a write comes in, add it to the in-memory balanced tree (called memtable)
    * If memtable size > threshold, write it to disk, while this happens new writes are written to a new memtable 
    instance
    * On request, tru to find the key in the memtable, if it fails go to the most recent disk segment, if it fails go
     to the second, and so on
    * Run merging an compaction in the background regularly
    
The only problem with this is that it may led to data lost if the server goes down and memtable has not been written 
to disk (which can be solved by appending to an unsorted separate log every write as it happens). This log can be 
discarded every time the memtable is written to disk.

##### Making an LSM-tree out of SSTables
Some key-value storage engines are designed to be embedded into other applications. This index structure is called 
Log-Structured Merge-Tree or (LSM-Tree). A similar concept is used for full text search in Lucene.

##### Performance optimizations
the LSM-tree algorithm can be slow if the key does not exists (as it does a full scan). Storage engines often use 
_Bloom filters_, which are used to tell you if a key does not appear in the database. Other concerns are when and how
 the compaction happens, the algorithm used are size-tiered(newer and smaller SSTables are merged into older and 
 larger ones), and leveled compaction (key range is split up into smaller SSTables and older data is moved into 
 separate levels). 
 
#### B-Trees
The most widely used indexing structure is the B-tree, which keep key-value pairs sorted by key, allowing for efficient 
key-value lookups and range queries. B-trees break the database down into fixed-size blocks or pages, traditionally 4
KB in size (sometimes bigger), and read or write one page at a time. Each page can be identified using an address 
or location, which allows one page to refer to another—similar to a pointer, but on disk instead of in memory and 
this structure can be use to construct a tree of pages.
One page is designated as the root of the B-tree, which is used to start looking for a key. The page contains 
several keys and references to child pages. Each child is responsible for a continuous range of keys, and the keys 
between the references indicate where the boundaries between those ranges lie. A leaf page contains individual keys, 
which contains either the value for the key or the reference to the page where the value is. 
The number of references to child pages in one page of the B-tree is called the branching factor.

##### Making B-trees reliable
The basic write operation is to overwrite pages on disk. This can fail, for example if you split a page because an 
insertion caused it to be overfull, you need to write the two pages that were split, and also overwrite their parent
page to update the references to the two child pages (can be solved partially by adding a append only write-ahead log
 named WAL). Pages has tipically latches (lightweight locks), to deal with concurrency problems. 

##### B-tree optimizations
Some optimizations can be:

    * Use a copy-on-write scheme instead of WAL: A modified page is written to a different location, and a new 
    version of the parent pages in the tree is created, pointing at the new location
    * Abreviatting the keys to save space, specially in interior pages which only needs to act as boundaries
    * Many B-tree implementations therefore try to lay out the tree so that leaf pages appear in sequential order on disk
    * Additional pointer to sibling pages might be added to speed scanning
    * Fractal trees variants
    
#### Comparing B-Trees and LSM-Trees
LSM-trees are typically faster for writes, whereas B-trees are thought to be faster for reads, although you need to 
test systems with your particular workload in order to make a valid comparison.

##### Advantages of LSM-trees
A B-tree index must write every piece of data at least twice: once to the write-ahead log, and once to the tree page 
itself. Log-structured indexes also rewrite data multiple times due to repeated compaction and merging of SSTables. 
This multiple write is called _write amplification_, LSM-trees are typically able to sustain higher write throughput 
than B-trees, partly because they sometimes have lower write amplification.
LSM-trees can be compressed better, and thus often produce smaller files on disk than B-trees

##### Downsides of LSM-trees
A downside of log-structured storage is that the compaction process can sometimes interfere with the performance of 
ongoing reads and writes. Also, the disk’s finite write bandwidth needs to be shared between the initial write (logging 
and flushing a memtable to disk) and the compaction threads running in the background, the bigger the database gets, 
the more disk bandwidth is required for compaction.  
In B-trees, each key exists in exactly one place in the index, whereas a log-structured storage engine may have 
multiple copies of the same key in different segments (advantage on strong transactional storages).

#### Other Indexing Structures
Appart of the key-value (like PK for a RDBMS or a document ID for a document database...), there are other structures
 like FK or secondary indexes. The main difference between the primary and secondary indexes is that in a secondary 
 index, the indexed values are not necessarily unique; that is, there might be many rows (documents, vertices) under
 the same index entry.
  
##### Storing values within the index
The value that a key points to, can be one of two things: it could be the actual row, or it could be a reference to 
the row stored elsewhere (the place where rows are stored is known as a heap file and it stores unordered data).
The heap file avoids duplicating data when multiple secondary indexes are present: each index just references a 
location in the heap file, and the actual data is kept in one place.
In some cases, it can be desirable to store the indexed row directly within an index, known as a clustered index 
(storing all row data within the index).
_Covering index_ or _index with included columns_ stores some of a table’s columns within the index, and some are 
references to other locations.

##### Multi-column indexes
Multi-dimensional indexes are a more general way of querying several columns at once, the most common being the 
concatenated index, which combines several fields into one key by appending one column to another.

##### Full-text search and fuzzy indexes
Full-text search engines commonly allow a search for one word to be expanded to include synonyms of the word, to 
ignore grammatical variations of words, and to search for occurrences of words near each other in the same document,
 and support various other features that depend on linguistic analysis of the text.

##### Keeping everything in memory
Many datasets are simply not that big, so it’s quite feasible to keep them entirely in memory, potentially 
distributed across several machines. When an in-memory database is restarted, it needs to reload its state, either 
from disk or over the network from a replica. The performance advantage of in-memory databases is due to not having 
to deal with overheads of encoding in-memory data structures in a form that can be written to disk.

### Transaction Processing or Analytics?
A transaction in DBs is a group of reads and writes that form a logical unit. The typical access pattern became known
as online transaction processing (OLTP) and usually involves a small number of records per query, fetched by key. A 
different access patter is used for analytics, usually an analytic query needs to scan over a huge number of 
records, only reading a few columns per record, and calculates aggregate statistics. The pattern for analytics is 
called online analytic processing (OLAP).

#### Data Warehousing
A data warehouse is a database that analysts can query without affecting OLTP operations, it contains a read-only 
copy of the data in all the various OLTP systems in a company or organization. Data is extracted from OLTP databases
(using periodic data dump or a continuous stream of updates), transformed into an analysis-friendly schema, cleaned 
up, and then loaded into the data warehouse. This is known as Extract–Transform–Load (ETL).
Data warehouses and a relational OLTP database both have a SQL query interface but they are optimized for very different
 query patterns.
 
#### Stars and Snowflakes: Schemas for Analytics
The star schema or dimensional modeling is usually the formula used in analytics. It contains a _fact table_ to 
which _dimension tables_ (which stores events) points to. The event tables represents an event that occurred at a 
particular time. Facts are 
captured as individual events, because this allows maximum flexibility of analysis later. Some of the columns in the
 fact table are attributes and others are foreign key references to the dimension tables. As each row in the fact 
 table represents an event, the dimensions represent the _who_, _what_, _where_, _when_, _how_, and _why_ of the event.
A variation of this template is known as the snowflake schema, where dimensions are further broken down into 
subdimensions. Typical data warehouse tables are often very wide: fact tables often have several hundred columns. 

### Column-Oriented Storage
A typical data warehouse query usually accesses 4 or 5 columns at one time (`SELECT * ...` is not usually required 
for analytics). In most OLTP databases, storage is laid out in a row-oriented fashion: all the values from one row 
of a table are stored next to each other. 
In _column-oriented_ storage, stores all the values from each column together insteadThe column-oriented storage 
layout relies on each column file containing the rows in the same order. 

#### Column Compression
Column-oriented storage often lends itself very well to compression. One technique that is particularly effective 
in data warehouses is bitmap encoding (often, the number of distinct values in a column is small compared to the 
number of rows).

#### Sort Order in Column Storage
In a column store it usually doesn't make a difference the order of the data (it is the insertion order), but we can 
impose an order. The administrator of the database can choose the columns by which the table should be sorted, which 
is usually dependant on the query pattern most commonly used (i.e. by date and by country). This can help with 
compression of columns

##### Several different sort orders
Different queries benefit from different sort orders, storing the same data sorted in several different ways is 
sometimes used. Having multiple sort orders in a column-oriented store is similar to multiple secondary indexes in a
 row-oriented store, although row-oriented store keeps every row in one place (in the heap file or a clustered 
 index), and secondary indexes just contain pointers to the matching rows.
 
#### Writing to Column-Oriented Storage 
An update-in-place approach, like B-trees use, is not possible with compressed columns. If you wanted to insert a 
row in the middle of a sorted table, you would most likely have to rewrite all the column files. As rows are 
identified by their position within a column, the insertion has to update all columns consistently. An option is to 
use LSM-trees. All writes first go to an in-memory store, where they are added to a sorted structure and prepared 
for writing to disk. It doesn’t matter whether the in-memory store is row-oriented or column-oriented. When enough 
writes have accumulated, they are merged with the column files on disk and written t o new files in bulk. Queries 
need to examine both the column data on disk and the recent writes in memory, and combine the two (done by the query 
optimizer).

#### Aggregation: Data Cubes and Materialized Views
Data warehouse queries often involve an aggregate function (SUM, AVG...), which if are used often, might have sense 
to cache the results of such functions. One way of creating such a cache is a _materialized view_: similar to virtual
 views (data as result of a query), but written to disk. When the underlying data changes, a materialized view needs
 to be updated, because it is a denormalized copy of the data. A common special case of a materialized view is 
 known as a data cube or OLAP cube which is a grid of aggregates grouped by different dimensions. In this cubes, 
 performance is gain at the cost of flexibility in querying (therefore this type of aggregation is often limited to 
 some particular queries).


## Chapter 4: Encoding and Evolution<a name="Chapter4"></a>
### Formats for Encoding Data
Programs usually work with data in (at least) two different representations:

    * In memory: optimized for efficient access and manipulation by the CPU (lists, hashmaps, sets...)
    * Encoded in a particular sequence of bytes: There are no pointers (woldn't make sense), JSON, text...
    
The translation from the in-memory representation to a byte sequence is called encoding and the reverse is called 
decoding.

#### Language-Specific Formats
Many libraries came with a built-in support for in-memory encoding-decoding, but at the cost of:

    * Often, the encoding is tied to a particular programming language
    * In order to restore data in the same object types, the decoding process needs to be able to instantiate 
    arbitrary classes, which might lead to security vulnerabilities
    * Versioning data can be a source of problems
    * CPU efficiency can be a source of performance problems
    
#### JSON, XML, and Binary Variants
JSON, XML, and CSV are textual formats, that comes with some remarkable problems:

    * Lot of ambiguity around the encoding of numbers: XML doesn't distinguish between strings containing numbers and
     numbers, and JSON can't specify the precission of numbers
    * JSON and XML have good support for Unicode character strings, but they don’t support binary strings. These 
    strings are usually encoded using Base64, which increases the data size
    * There is optional schema support for XML and JSON, but it is complicated to implement
    * CSV does not have any schema, up to the application to interpret the data. Additions are complicated to handle 

##### Binary encoding
Data encoding matters for very big datasets due to performance reasons. Binary encoding of JSON or XML exists, 
although the gains are not very big in terms of space, so might not worth the loss of human readability.

#### Thrift and Protocol Buffers
Both Thrift and Protocol Buffers require a schema for any data that is encoded. Thrift and Protocol Buffers each 
come with a code generation tool that takes a schema definition, and produces classes that implement the schema in 
various programming languages. Thrift has two different binary encoding formats,iii called BinaryProtocol and 
CompactProtocol. 
In the thrift binary protocol, the data contains type, length of data, the data itself, and field 
tags (instead of field names), which are numbers identified the field names in the schema definition (like aliases). 
In the thrift compact protocol, the field type and tag numbers are combined in one, and fields are encoded using 
variable length integers (the top bytes are used to indicate whether there are still more bytes to come).
Protocol Buffers is very similar to Thrift’s CompactProtocol.

##### Field tags and schema evolution
In Thrift and protocol buffers each field is identified by its tag number and annotated with a datatype, if a field 
value is not set, it is simply omitted from the encoded record. You can change the name of a field in the schema, 
but you cannot change a field’s tag. You can add new fields to the schema, provided that you give each field a new 
tag number (Old code would skip a field if it has a tag that doesn't recognize). For backwards compatibility, if a 
new field is added, can't be made mandatory as it would invalidate the old schemas (or have a default value).

##### Datatypes and schema evolution
In data type changes, there is a risk that values will lose precision or get truncated.

#### Avro
Avro also uses a schema to specify the structure of the data being encoded. It has two schema languages: one (Avro 
IDL) intended for human editing, and one (based on JSON) that is more easily machine-readable. Values are concatenated 
together, to parse the binary data, you go through the fields in the order that they appear in the schema and use the
 schema to tell you the datatype of each field (schema mismatch would result in invalid data read).
 
##### The writer’s schema and the reader’s schema
With Avro, when an application wants to encode some data, it encodes the data using whatever version of the schema 
it knows about, which might be compiled into the application. This is known as the writer’s schema. On data decoding,
 it needs the data to be in some schema, known as reader's schema, which don't have to be the same than the writer's 
 schema (but needs to be compatible). The Avro library resolves the differences by looking at the writer’s schema and 
 the reader’s schema side by side and translating the data from the writer’s schema into the reader’s schema. 
 
##### Schema evolution rules
You may only add or remove a field that has a default value. In Avro, if you want to allow a field to be null, you 
have to use a _union type_: _union { null, long, string } field_; indicates that _field_ can be a number, or a string, 
or null. Changing the datatype of a field is possible, provided that Avro can convert the type.

##### But what is the writer’s schema?
The schema of an Avro file can't be included in every record, therefore:

    * Large file with lots of records: Usually the schema is at the beginning of the file. Avro specifies a file 
    format (object container file) to do this
    * Database with individually written records: different records may be written at different points in time using
     different writer’s schemas. A version number indicating the schema is included at the beginning of each record
    * Sending records over a network connection: Two endpoints in a communication can negotiate the schema version on
     connection setup (like in the Avro RPC protocol)

##### Dynamically generated schemas
Avro doesn't contain tag numbers like in Protocol Buffers and Thrift which is friendlier to the dynamically generated 
schemas, with Thrift or Protocol Buffers for this purpose, the field tags would likely have to be assigned by hand 
every time the schema changes (from DB columns to field tags).

##### Code generation and dynamically typed languages
Protocol Buffers and Thrift allows efficient in-memory structures to be used for decoded data after a schema has been
 defined. There is no point on doing this for dinamically typed languages. Avro provided optional code generation for
  statically typed languages.
  
#### The Merits of Schemas
Binary encodings based on schemas have a number of nice properties:

    * They can be much more compact than the various “binary JSON” variants, as they can omit field names
    * The schema is a valuable form of documentation, which is always up to date
    * Keeping a database of schemas allows you to check forward and backward compatibility of schema changes
    * Code generation from the schema enables type checking at compile time
    
### Modes of Dataflow
Compatibility is a relationship between one process that encodes the data, and another process that decodes it.

#### Dataflow Through Databases
In an environment where the application is changing, it is likely that some processes accessing the database will be 
running newer code and some will be running older code, therefore forward compatibility is also often required for 
databases. Older clients usually left newly added fields untouched.

##### Different values written at different times
When you deploy a new version of your application, you may entirely replace the old version with the new version 
which is not true of database contents, this observation is sometimes summed up as _data outlives code_. Rewriting 
data into a new schema is certainly possible, but it’s an expensive thing to do on a large dataset, simple schema 
changes, such as adding a new column with a null default value are supported in most relational databases.

##### Archival Storage
While archiving, the data dump will typically be encoded using the latest schema, even if the original encoding in the 
source database contained a mixture of schema versions from different eras.

#### Dataflow Through Services: REST and RPC
When you have processes that need to communicate over a network, the most common arrangement is to have two roles:
clients and servers. The servers expose an API over the network, and the clients make requests to that API (known as 
service). In some ways, services are similar to databases: they typically allow clients to submit and query data. As 
the querys a client can do are limited by the API, the services can impose fine grained restrictions to what a client
 can do.
A key design goal of a service-oriented/microservices architecture is to make the application easier to change and
maintain by making services independently deployable and evolvable.

##### Web Services
When HTTP is used as the underlying protocol for talking to the service, it is called a web service. There are two 
popular approaches:

    * REST: emphasizes simple data formats, using URLs for identifying resources and using HTTP features for cache 
    control, authentication, and content type negotiation
    * SOAP: XML-based protocol for making network API requests, the API of a SOAP web service is described using an 
    XML-based language called the Web Services Description Language, or WSDL
    
##### The problems with remote procedure calls (RPCs)
 The remote procedure call (RPC) model tries to make a request to a remote network service look the same as calling 
 a function or method in your programming language, within the same process. A network request is very different from 
 a local function call:

    * Local calls are predictable (either succeeds or not), remote calls are not because involves factors out of our 
    control like network problems
    * Local calls returns a result, an exception or never returns (infinite loop). A network call might return 
    nothing due to timeouts, not knowing what happened in the other end
    * Retrying a call might lead to unexpected results for non idempotent calls if the bit lost was the response from 
    the server
    * Response time in local calls are almost always the same, a network call adds latency to it
    * You can pass references in a local call (pointers), all parameters needs to be encoded in the network call
    * Client and services might be implemented using different languages, with potential data type impedance
    
##### Current directions for RPC
The new generation of RPC frameworks is more explicit about the fact that a remote request is different from a local
 function call, including _futures_ to encapsulate asynchronous calls, or streams, which are a series of requests and
 responses over time. REST seems to be the predominant style for public APIs. 
 
##### Data encoding and evolution for RPC
Regarding evolvability it is reasonable to assume that all the servers will be updated first, and all the clients 
second. Thus, you only need backward compatibility on requests, and forward compatibility on responses. Compatibility
needs to be maintained for a long time, perhaps indefinitely. For RESTful APIs, common approaches are to use a 
version number in the URL or in the HTTP Accept header.

#### Message-Passing Dataflow
Asynchronous message-passing systems are similar to RPC in that a client’s request is delivered to another process 
with low latency and similar to databases in that the message is not sent via a direct network connection, but goes 
via an intermediary called a message broker, which stores the message temporarily. This brings some advantages:

    * It can act as a buffer if the recipient is unavailable or overloaded, improving system reliability
    * It can automatically redeliver messages to a process that has crashed, preventing messages from being lost
    * It avoids the sender needing to know the IP address and port number of the recipient
    * It allows one message to be sent to several recipients.
    * It logically decouples the sender from the recipient. 
    
The sender doesn't usually expect to receive a reply.

##### Message Brokers
Message brokers are used as follows: one process sends a message to a named queue or topic, and the broker ensures 
that the message is delivered to one or more consumers of or subscribers to that queue or topic. A consumer may 
itself publish messages to another topic or to a reply queue that is consumed by the sender of the original message. 
Message brokers typically don’t enforce any particular data model

##### Distributed actor frameworks
The actor model is a programming model for concurrency in a single process. The logic is encapsulated in the actors, 
which might have some local non-shared state, and communicates with other actors by sending and receiving 
asynchronous messages, with message delivery not guaranteed.
A distributed actor framework essentially integrates a message broker and the actor programming model into a single 
 framework.
 
 
## Chapter 5: Replication<a name="Chapter5"></a>
If your dataset can fit in a single machine, all of the difficulty in replication lies in handling changes to replicated
data.

### Leaders and Followers
The replicas of datastores needs to be updated to stay in sync with the latest data. The most common solution for 
this is called leader-based replication:

    1. One replica is the leader (master or primary), which receives client request and writes new data to local storage
    2. The rest of replicas (followers, read replicas, slaves or secondaries), receives the data change from the 
    leader as part of a replication log or change stream and updates their local copy of the data by applying the 
    writes in order
    3. A client can then query the leader or the replicas, although writes are only accepted in the leader
    
This architecture is used in RDBMS (MySQL, Oracle, PostgreSQL...) and NoSQL DB (MongoDB, Espresso...) and even 
message brokers (Kafka, RabbitMQ)

#### Synchronous Versus Asynchronous Replication
Replication can happen synchronously or asynchronously (often configurable in RDBMS). In synchronous replication the 
leader waits until the follower have confirmed the write before reporting success to the client and before making the
 write visible to other clients (guarantee of availability if the leader fails at the cost of latency), in asynchronous 
 replication, the leader sends the write to the replica and does not wait (leading to potential data lost). Usually for 
 sync replication, at least one follower needs to be in sync (this is called semi-synchronous replication).
 
#### Setting Up New Followers
The process of setting up a new follower looks like this:

    1. Take a consistent snapshot of the leader’s database at some point in time
    2. Copy the snapshot to the new follower node
    3. After the copy, request all the changes since the snapshot was taken (snapshot is associated with an exact 
    position on the replication log)
    4. One the follower has copied all the changes we say it has caught up

#### Handling Node Outages
##### Follower failure: Catch-up recovery
The follower gets the last transaction it has processed and request to the leader all the changes since then. After 
the caught up, it can continue receiving a stream of data changes as before.

##### Leader failure: Failover
One of the followers needs to be promoted to be the new leader, clients needs to send the writes to it and the other 
followers need to start consuming data changes from it. This process is called failover. An automatic failover steps 
are:

    1. Determining that the leader has failed: Most of systems use a timeout, if a node does not reply in X seconds, 
    then it is assumed to be dead
    2. Elect a new leader: Either by election or be appointed by a previously elected controller node
    3. Reconfiguring the system to use the new leader: Clients needs to send request to the new leader, if the old 
    leader comes back, he might still thinks it is the leader
    
Things that can go wrong:

    * In async replication, the new leader might not be up to date with all the writes. If the old leader comes back,
     those not sync writes are usually discarded
    * Discarded writes are problematic for outsiders to the system, and a potential source of problems
    * In fault scenarios, two nodes might believe they are the leaders (this is called split brain), both accepting 
    writes, leading to inconsistency
    * Choosing the right timeout is problematic as it is a trade-off between availability and unnecessary failovers

##### Implementation of Replication Logs
There are several leader-based replication methods:
##### Statement-based replication
The leader logs every write request (statement) that it executes and sends that statement log to its followers. It 
is now being replaced due to:

    * If there is a call to a non-deterministic function like NOW() or RAND(), which would generate unsync data
    * If statements use an autoincrementing column or they depend on existing data, the statements needs to be 
    executed in order
    * Statements with side effects might result in different results in the replicas
    
##### Write-ahead log (WAL) shipping
Either for log structured storage engines or for B-trees, the log is an append-only sequence of bytes containing all 
writes to the DB. This log can be used to create another replica. The main disadvantage is that the log describes 
the data on a very low level: a WAL contains details of which bytes were changed in which disk blocks, leading to 
problems with version upgrading.

##### Logical (row-based) log replication
An alternative is to use different log formats for replication and for the storage engine, which allows the 
replication log to be decoupled from the storage engine internals (this is called logical log). This log:

    * For an inserted row, the log contains the new values of all columns
    * For a deleted row, the log contains enough information to uniquely identify the row that was deleted (PK or
    all columns need to be logged)
    * For an updated row, the log contains enough information to uniquely identify the updated row (same than before)
    
Logical log replication is decoupled from the underlying database software, and it is easier for an external 
application to parse.

##### Trigger-based replication
Replication might be needed to be moved up to the application layer. There are tools to make this like Oracle 
GoldenGate but there are other features to do this: _triggers_ and _stored procedures_.
A trigger lets you register custom application code that is automatically executed when a data change occurs in a 
database system, but is more limited and prone to bugs.

#### Problems with Replication Lag
For workloads that consist of mostly reads and only a small percentage of writes create many followers, and 
distribute the read requests across those followers. This approach only realistically works with asynchronous 
replication, but reads from asynchronous followers, may see outdated information if the follower has fallen behind. 
If you stop writing to the database and wait a while, the followers will eventually catch up and become consistent
with the leader (known as _eventual consistency_). The delay between a write happening on the leader and being 
reflected on a follower (the _replication lag_), can cause problems if the lag is big.

##### Reading Your Own Writes
When a user needs to submit data and see the results, _read-after-write consistency_ is needed, also known as 
_read-your-writes consistency_. This guarantees that if the user reloads the page, they will always see any updates they
submitted themselves, but makes no promises about other users. There are various possible techniques:

    * When reading something the user has modified, read it from the leader if possible, if not from one follower. 
    There should be a way to query certain type of information from the leader: i.e. the user profile
    * If most of things in the application are editable, you can select to read stuff that has been write since a 
    period of time (i.e. one minute) from the leader
    * The client can remember the timestamp of its most recent write, the system can ensure that the replica serving
     any reads for that user reflects updates at least until that timestamp (can be a logic timestamp, like an offset)
    * If the replicas are splitted across datacenters, requests that needs to be served by the leader must be routed 
    to the datacenter that contains the leader
    
If the user access the application from many devices, you need _cross-device read-after-write consistency_, consider:

    * Metadata needs to be centralized if the timestamp is used to retrieve the updates (clocks of the devices might 
    differ)
    * For distributed replicas, we can't assume that requests would be routed to the same datacenter (devices 
    might be in different networks), you need to ensure this programatically.

##### Monotonic Reads
When a user makes several reads from different replicas, user might see things moving backward in time. _Monotonic 
reads_ guarantees that this doesn't happen by making sure that each user always makes their reads from the same replica.

##### Consistent Prefix Reads
If a user querys a replica that is far behind the leader, he might get information from a more updated one, making it
looks like he received the answer before the query. _Consistent prefix reads_ guarantees that if a sequence of 
writes happens in a certain order, then anyone reading those writes will see them appear in the same order. 
However, in many distribute systems different partitions operate independently, so there is no global ordering of 
writes. One solution is to make sure that any writes that are causally related to each other are written to the 
same partition.
 
##### Solutions for Replication Lag
Transactions exists in databases to provide stronger guarantees so that the application can be simpler, but this is 
easer to achieve in single node system than in distributed ones.

#### Multi-Leader Replication
Leader-based application are limited by the fact that there is a single leader. In _multi-leader_ configuration (or 
_master–master_ or _active/active_ replication), each leader simultaneously acts as a follower to the other leaders. 

#### Use Cases for Multi-Leader Replication
This configuration rarely makes sense within a single datacenter, but it might make sense in certain situations:

##### Multi-datacenter operation
In a multi-leader configuration, you can have a leader in each datacenter: each datacenter’s leader replicates its 
changes to the leaders in other datacenters. _Single-leader_ and _multi-leader_ configurations in multidatacenter 
deployment differs in:

    * Performance: In single-leader configuration every write must go over the internet to the datacenter with the 
    leader, adding latency. In multi-leader configuration, every write can be processed in the local datacenter and
    is replicated asynchronously to the other datacenters, hidding the write lag to the final users
    * Tolerance of datacenter outages: Promoting a leader in a different datacenter has to happen in single leader 
    configuration, in multi leader configuration, each datacenter can continue operating independently
    * Tolerance of network problems: multi leader configuration handles better network problems

Multi-leadership is usually implemented with external tools, and one of the issues is that it has to solve the 
problem of concurrent writing to the same data in different datacenters. Other problems that might arise are 
autoincrementing keys, triggers, and integrity constraints.

##### Clients with offline operation
In the case where you need changes made by an application (possibly in multiple devices) when being offline, and you 
need the changes to be on sync in every device next time the device gets a connection, usually a local database that 
acts as a leader is maintain, and then a multi-leader replication process happens when the connection is established 
(each device is a datacenter).

##### Collaborative editing
In _Real-time collaborative editing_ (like in Google docs), you need to guarantee no editing conflicts, so a lock on 
the document must be obtained first, other users have to wait to get the lock to make modifications on the document.
 This collaboration model is equivalent to single-leader replication with transactions on the leader. For faster 
 collaboration, you may want to make the unit of change very small (e.g., a single keystroke) and avoid locking.
 
#### Handling Write Conflicts
##### Synchronous versus asynchronous conflict detection
In a multi-leader setup, conflicts are detected asynchronously unless you make it synchronous by waiting for the 
rest of the leader replicas to write the data, but this adds lag. 

##### Conflict avoidance
If the application can ensure that all writes for a particular record go through the same leader, then conflicts 
cannot occur. Sometimes you might want to change the designated leader for a record, so conflict avoidance breaks down,
and you have to deal with the possibility of concurrent writes on different leaders.

##### Converging toward a consistent state
In a single-leader database, if there are several updates to the same field, the last write determines the final 
value of the field. In multi-leader writes, the order of writes is not defined from datacenter to datacenter, thus 
conflicts must be solver in a convergent way. Possible solutions are:

    * Give each write a unique ID, pick the write with the highest ID as the winner, and throw away the other writes.
     (last write wins or LWW). This approach is prone to data loss
    * Give each replica a unique ID, and let writes that originated at a higher-numbered replica always take 
    precedence over writes that originated at a lower-numbered replica
    * Somehow merge the values together (application logic)
    * Record the conflict in an explicit data structure that preserves all information, and write application code 
     that resolves the conflict at some later time.

##### Custom conflict resolution logic
Two approaches:

    * On write : Conflict handler is called as soon as the database system detects a conflict.
    * On read: When a conflict is detected, all the conflicting writes are stored. The next time the data is read, 
    these multiple versions of the data are returned to the application. The application may prompt the user or
    automatically resolve the conflict, and write the result back to the database
    
Each conflict is considered separately for conflict resolution even if they are in the same transaction.

#### Multi-Leader Replication Topologies
A _replication topology_ describes the communication paths along which writes are propagated from one node to another. 
With two leaders, each write from one leader has to be replicated to the other. With more than two, there are various
 options:
 
    * All to all: Most common, every leader sents its writes to every other leader
    * Circular: Each node receives and setnds writes from and to a single node, making a ring (MySQL uses this)
    * Star topology: One designated root node forwards writes to all of the other nodes (can be generalized to a tree)

To prevent infinite replication loos in star and circular topologies, each node is given a unique identifier, and in
 the replication log, each write is tagged with the identifiers of all the nodes it has passed through. In these 
 topologies, if a node fails can cause problem to other nodes, although the topology can be reconfigured to ignore 
 the failed node. All to all topologies can have problems with the different speed of the network links, therefore 
 some writes may overtake others. To order write events correctly, _version vectors_ can be used.
 
### Leaderless Replication
Some data storage systems abandons the concept of a leader and allows replicas to directly accept writes from clients.
In some implementations, the client directly sends its writes to several replicas, in others, a coordinator node does
 this on behalf of the client, that coordinator does not enforce a particular ordering of writes.
 
#### Writing to the Database When a Node Is Down
In a leaderless configuration, failover does not exist (client might choose to ignore a failure in writing to a 
replica). The client send read request to several nodes in parallel to avoid getting stale values in case a replica 
couldn't get a write in the past. Version numbers are used to determine which value is newer.

##### Read repair and anti-entropy
The replication scheme should ensure that eventually all the data is copied to every replica:

    * Read repair: The client detect stale values and writes back the newer value to that replica
    * Anti-entropy process: A background process constantly looks for differences in replicas and copies missing data
    
##### Quorums for reading and writing
If there are n replicas (usually an odd number), every write must be confirmed by w nodes to be considered 
successful, and we must query at least r nodes for each read. As long as w + r > n, we expect to get an up-to-date 
value when reading, because at least one of the r nodes we're reading from must be up to date. Reads and writes 
that follows this rule are called _quorum_ reads and writes:

    * If w < n, we can still process writes if a node is unavailable
    * If r < n, we can still process reads if a node is unavailable
    * If fewer than the required w or r nodes are available, writes or reads return an error

#### Limitations of Quorum Consistency
w and r might be set to smaller numbers, so that w + r ≤ n: You are more likely to read stale values, but this allows 
lower latency and higher availability. Even with w + r > n, there are edge cases where stale values are returned:

    * In sloppy quorums, w writes may end up on different nodes than the r reads, there is no guarantee of overlap
    * If two writes occur concurrently (not clear which happened first), the only option is to merge the concurrent 
    writes. If a winner is picked based on a timestamp (LWW), writes can be lost due to clock skew
    * If a write happens concurrently with a read, the write may be reflected on only some of the replicas
    * If a write succeeded on some replicas but failed on others and overall succeeded on fewer than w replicas, it 
    is not rolled back on the replicas where it succeeded. This means that if a write was reported as failed, 
    subsequent reads may or may not return the value from that write
    * If a node carrying a new value fails, and its data is restored from a replica carrying an old value, the 
    number of replicas storing the new value may fall below w, breaking the quorum condition.
    * Unlucky timing can cause stale values to be read
    
Due to all of the above, stronger guarantees generally require transactions or consensus.

##### Monitoring staleness
For leader-based replication, writes are applied to the leader and to followers in the same order, and each node has
 a position in the replication log. By subtracting a follower’s current position from the leader’s current position,
 you can measure the amount of replication lag.
In systems with leaderless replication, if the database only uses read repair (no anti-entropy), there is no limit to
 how old a value might be.

#### Sloppy Quorums and Hinted Handoff
Databases with leader‐less replication are appealing for use cases that require high availability and low latency, and 
that can tolerate occasional stale reads. In a large cluster it’s likely that a client can connect to some database 
nodes during a network interruption, just not to the nodes that it needs to assemble a quorum for a particular value.
 In that case, database designers face a trade-off:
   
    * Is it better to return errors to all requests for which we cannot reach a quorum of w or r nodes
    * It accepts writes anyway, and write them to some nodes that are reachable but aren’t among the n nodes
     on which the value usually lives
     
The latter is called _sloppy quorum_, once the network interruption is fixed, any writes that one node temporarily 
accepted on behalf of another node are sent to the appropriate “home” nodes (called _hinted handoff_). 
A sloppy quorum is only an assurance of durability, namely that the data is stored on w nodes somewhere. There is no
 guarantee that a read of r nodes will see it until the hinted handoff has completed.
 
##### Multi-datacenter operation
Leaderless replication is also suitable for multi-datacenter operation, with replication between database clusters 
happening asynchronously in the background, in a style that is similar to multi-leader replication.

#### Detecting Concurrent Writes
Events may arrive in a different order at different nodes, due to variable network delays and partial failures in 
multi-leader replication, read repair or hinted handoff. Several approaches exist for achieving eventual convergence

##### Last write wins (discarding concurrent writes)
Newer values replaces older ones in each replica, therefore a way of unambiguously determining which write is more 
recent is needed. LWW can be used but is tricky cause even if they were reported as successful to the client 
(because they were written to w replicas), only one will survive and the others will be discarded silently.
The only safe way of using a database with LWW is to ensure that a key is only written once and treated as 
immutable, avoiding any concurrent updates to the same key (i.e. giving each write operation a unique key).

##### The “happens-before” relationship and concurrency
A value B is _causally dependent_ on value A if B depends on A and should therefore happen after. In the other hand 
we say that two operations are concurrent if neither happens before the other. A mechanism should exist to determine 
concurrency: two operations concurrent if they are both unaware of each other, regardless of the physical time.

##### Capturing the happens-before relationship
In order to determine if a concurrent write has happened, we can use the following algorithm:

    * The server maintains a version number for every key, increments it every time that key is written, and stores 
    the new value along with the value written
    * When a client reads a key, the server returns all values that have not been over‐written, as well as the 
    latest version number. A client must read a key before writing
    * When a client writes a key, it must include the version number from the prior read, and it must merge together 
    all values that it received in the prior read
    * When the server receives a write with a particular version number, it can over‐write all values with that 
    version number or below, but it must keep all values with a higher version number
    
##### Merging concurrently written values
The previous algorithm ensures that no data is silently dropped, but adds complexity in the clients which has to merge 
written values. These values are sometimes called _siblings_. Merging values has similarities with multi-leader write
 merging, and can be done with LWW, or using the union of the two writes which can lead to problems in case of 
 removing elements. For that purpose, usually a marker of deletion (called tombstone) with the version number is 
 stored.
 
##### Version vectors
If more replicas without leader where added to the previous scenario, we need to use a version number per replica and
 per key, the collection of version numbers from all the replicas is called a _version vector_, and allows the 
 database to distinguish between overwrites and concurrent writes.
 
 
 ## Chapter 6: Partitioning<a name="Chapter6"></a>
Partitions are like a small databases, but supports operations that touch multiple partitions at the same time. 
Partitioning is usually combined with replication so that copies of each partition are stored on multiple nodes.

### Partitioning of Key-Value Data
Our goal with partitioning is to spread the data and the query load evenly across nodes (if the partitioning is 
unfair so that some partitions have more data or queries than others, we call it skewed). The simplest approach for 
avoiding _hot spots_ would be to assign records to nodes randomly.

#### Partitioning by Key Range
Partitions are assigned (manually or automatically) as a continuous range of keys (like a dictionary index). Ranges 
need not to be evenly spaced if the data is not evenly distributed. This approach is used by MongoDB, BigTable, HBase...
Keys can be ordered within partitions. This approach can lead to hotspots

#### Partitioning by Hash of Key
A hash function is used to determine the partition for a given key. The hash function need not be cryptographically 
strong, you can assign each partition a range of hashes, and every key whose hash falls within a partition’s range 
will be stored in that partition.
A table in Cassandra can be declared with a compound primary key consisting of several columns. Only the first part 
of that key is hashed to determine the partition, but the other columns are used as a concatenated index for sorting
 the data in Cassandra’s SSTables
 
#### Skewed Workloads and Relieving Hot Spots
In the extreme case where all reads and writes are for the same key, you still end up with all requests being routed 
to the same partition (for example a celebrity twitter account). A common technique to solve this is if one key is 
known to be very hot, a simple technique is to add a random number to the beginning or end of the key. Extra work 
would have to be done around combining those keys and keeping track which keys were splitted in that way.

### Partitioning and Secondary Indexes
A secondary index usually doesn’t identify a record uniquely but rather is a way of searching for occurrences of a 
particular value. The problem with secondary indexes is that they don’t map neatly to partitions.

#### Partitioning Secondary Indexes by Document
If you have a list of documents distributed in partitions representing cars, and you want to allow users of that 
database to search for things like color or brand, each partition would have to have secondary indexes to map this 
features, covering only the cars in that partition. To do CRUD over a document, you have to interact with a single 
partition. A document-partitioned index is also known as a _local index_. Care has to be taken to avoid putting all 
potential documents that would be returned in a query (such as all the red cars) in a single partition. Querys are 
sent to every partition and combined back (known as _scatter/gather_), and they are prone to _tail latency_.

#### Partitioning Secondary Indexes by Term
We can also construct a global index that covers data in all partitions (namely _term index_). A global index must
 also be partitioned, but it can be partitioned differently from the primary key index: the term we’re looking for 
 determines the partition of the index, we can partition the index by the term itself, or using a hash of the term.
 It is more efficient than the scatter/gather approach as only one partition is queried, the downside of a global 
 index is that writes are slower and more complicated (a write on a document needs to modify several partitions 
 of the indexes). Updates in secondary indexes are usually asynchronous for this reason.
 
### Rebalancing Partitions
The process of moving load from one node in the cluster to another is called rebalancing. Rebalancing should at least:
    
    * Distribute the data load fairly between the nodes in the cluster
    * While rebalancing is happening, the database should continue accepting reads and writes
    * No more data than necessary should be moved between nodes
    
#### Strategies for Rebalancing
##### How not to do it: hash mod N
The problem with the mod N approach is that if the number of nodes N changes, most of the keys will need to be moved
 from one node to another.

##### Fixed number of partitions
A simple solution is to create more partitions than nodes, so if new nodes are added, whole partitions can be moved 
to that node, more powerful nodes can get more partitions to get more load. In this configuration, the number of 
partitions is usually fixed when the database is first set up and not changed afterward. It is hard to get the number
 of partitions right for very variable datasets.
 
##### Dynamic partitioning
Fixed partition number is not the most indicate solution for key range partitioning, therefore they usually allocate 
partitions dynamically: When a partition grows to exceed a configured size, it is split into two partitions so that 
approximately half of the data ends up on each side of the split (similar to a B tree).
An empty database starts with a single partition,so all writes have to be processed by a single node while the other 
nodes sit idle until the dataset is first split. Some databases allow an initial set of partitions to be configured 
on an empty database (_pre-splitting_). But pre-splitting requires that you already know what the key distribution 
is going to look like.

##### Partitioning proportionally to nodes
Both in fixed and dynamic partitioning, the number of partitions is independent of the number of nodes in the cluster.
Another approach is to make the number of partitions proportional to the number of nodes, this is to have a fixed 
number of partitions per node. When a new node joins the cluster, it randomly chooses a fixed number of existing 
partitions to split, and then takes ownership of one half of each of those split partitions while leaving the other 
half of each partition in place. Picking partition boundaries randomly requires that hash-based partitioning is used.

#### Operations: Automatic or Manual Rebalancing
Fully automated rebalancing require less operational work to do for normal maintenance but it can be unpredictable. 
Such automation can be dangerous in combination with automatic failure detection. 

### Request Routing
As partitions are rebalanced, the assignment of partitions to nodes changes and request have to be issued to the new 
holders of partitions (This is an instance of a more general problem called _service discovery_). There are different
 approaches to the problem:
 
    * Allow clients to contact any node, if the node has the partitions it responds to the query otherwise it 
    provides the address to the node
    * Send all requests from clients to a routing tier first (load balancer), which replies with the address of the node
    * Require that clients be aware of the partitioning and the assignment of partitions to nodes
    
Many distributed data systems rely on a separate coordination service such as ZooKeeper to keep track of this cluster
 metadata, which maintains the authoritative mapping of partitions to nodes. Other approaches use a gossip protocol 
 among the nodes to disseminate any changes in cluster state.
 
#### Parallel Query Execution
Massively parallel processing (MPP) relational database products,relies on query optimizer to break query complexty 
into a number of execution stages and partitions, many of which can be executed in parallel on different nodes of 
the database cluster.


## Chapter 7: Transactions<a name="Chapter7"></a>
A transaction is a way for an application to group several reads and writes together into a logical unit. Either the 
entire transaction succeeds (commit) or it fails (abort, rollback).

### The Slippery Concept of a Transaction
#### The Meaning of ACID
ACID stands for Atomicity, Consistency, Isolation, and Durability. Systems that do not meet the ACID criteria are 
sometimes called BASE, which stands for Basically Available, Soft state, and Eventual consistency.

##### Atomicity
Refers to something that cannot be broken down into smaller parts. If the writes are grouped together into an atomic 
transaction, and the transaction cannot be completed (committed) due to a fault, then the transaction is aborted and
 the database must discard or undo any writes it has made so far in that transaction. 
 
##### Consistency
Consistency ensures that you have certain statements about your data (invariants) that must always be true. Some 
specific invariants that can be checked by the database are using foreign key constraints or uniqueness constraints.
 
##### Isolation
Isolation means that concurrently executing transactions are isolated from each other. The database ensures that 
when the transactions have committed, the result is the same as if they had run serially.

##### Durability
Durability is the promise that once a transaction has committed successfully, any data it has written will not be 
forgotten, even if there is a hardware fault or the database crashes. 

#### Single-Object and Multi-Object Operations
Multi-object transactions require a way of determining which read and write operations belong to the same transaction. 
Everything between a BEGIN TRANSACTION and a COMMIT statement is considered to be part of the same transaction.

##### Single-object writes
Atomicity can be implemented using a log for crash recovery  and isolation using a lock on each object.

##### The need for multi-object transactions
Many distributed datastores have abandoned multi-object transactions because they are difficult to implement across 
partitions, and they can get in the way in some scenarios where very high availability or performance is required.
But multi-objects makes total sense in updating rows it FK, several documents in a denormalized document database or 
databases with secondary indexes on value changes.

##### Handling errors and aborts
ACID transactions can be retried if aborted, but in leaderless replication it follows the best effort which can be 
translated as "I do as much as I can, but on error I won't undo something done". Retrying a failed transaction is not
 perfect, it can lead to duplicates if the transaction succeeded and there were a network problem, can make things 
 worst if it was due to an overloaded node, can be pointless if it was not a transient error like a deadlock, can 
 have side effects even if it is aborted (like sending an email).
 
### Weak Isolation Levels
If two transactions don’t touch the same data, they can safely be run in parallel, because neither depends on the other.
Serializable isolation means that the database guarantees that transactions have the same effect as if they ran serially
 but this has a performance cost, therefore some systems offers a _weaker_ form of isolation.
 
#### Read Committed
This level of isolation makes two guarantees:
    
    1. When reading from the database, you will only see data that has been committed (no dirty reads)
    2. When writing to the database, you will only overwrite data that has been committed (no dirty writes)

Dirty read refers to the act of reading an uncommitted transaction, which should be prevented if the transaction 
happens at read committed.
If two transactions concurrently try to update the same object in a database, with the earlier write being part of a 
transaction that has not yet committed, if the later write overwrites an uncommitted value we call this a dirty write.
Read committed does not prevent the race condition between two counter increments
Databases prevent dirty writes by using row-level locks: when a transaction wants to modify a particular object (row 
or document), it must first acquire a lock on that object and hold it until the transaction is committed.
To prevent dirty reds, instead of acquiring the lock on the object which would slow down the reads on long running 
writes, while the transaction is ongoing any other transactions that read the object are simply given the old value.

#### Snapshot Isolation and Repeatable Read
_nonrepeatable read or read skew_ is a temporary inconsistency in a DB due to unfortunate timing during a read 
operation. It is acceptable on Read Committed, but might not be if:

    * Backups: if a backup that takes some time is performed, and meanwhile writes happens, some part of the backup 
    would end up with outdated data
    * Analytic queries and integrity checks: On queries that scans large parts of the database, the query is likely 
    to observe parts of the database at different points in time
    
The solution to this problem is _Snapshot Isolation_, which guarantees that the transaction sees all the data that 
was committed in the database at the start of the transaction.

##### Implementing snapshot isolation
Implementations of snapshot isolation typically use write locks to prevent dirty writes, reads do not require any 
locks. From a performance point of view, a key principle of snapshot isolation is readers never block writers, and 
viceversa. The database must potentially keep several different committed versions of an object, this technique is 
known as multi-version concurrency control (MVCC). Each transaction has an always increasing transaction ID that is 
used for recovering, writing or deleting purposes.

##### Visibility rules for observing a consistent snapshot
Transaction IDs are used to decide which objects it can see and which are invisible.

    1. At the start of each transaction, the database makes a list of all the other transactions in progress. Any 
    writes that those transactions have made are ignored, even if the transactions subsequently commit
    2. Any writes made by aborted transactions are ignored
    3. Any writes made by transactions with a later transaction ID are ignored, regardless of whether those 
    transactions have committed
    4. All other writes are visible to the application's queries
    
A transaction is visible if at the time when the reader’s transaction started, the transaction that created the object 
had already committed or the object is not marked for deletion (the transaction that requested deletion 
had not yet committed at the time when the reader’s transaction started).

##### Indexes and snapshot isolation
Several implementations to solve this problem exists, from indexes simply point to all versions of an object to 
B-trees and use an append-only/copy-on-write variant that does not overwrite pages of the tree when they are updated.

#### Preventing Lost Updates
Aside of the dirty write case, other problems might arise on concurrent writes. The lost update problem can occur if
 an application reads some value from the database, modifies it, and writes back the modified value.

##### Atomic write operations
Many databases provide atomic update operations, which remove the need to implement read-modify-write cycles in 
application code, in situations where atomic operations can be used, they are usually the best choice.
Atomic operations are usually implemented by taking an exclusive lock on the object when it is read so that no other 
transaction can read it until the update has been applied (called cursor stability), other option is to force 
all atomic operations to be executed on a single thread.

##### Explicit locking
If the database’s built-in atomic operations don’t provide the necessary functionality, is for the application to 
 explicitly lock objects that are going to be updated. This is done by:
 
```sql
BEGIN TRANSACTION;
--FOR UPDATE indicates that the database should take a lock on all rows returned by this query.
SELECT * FROM figures WHERE name = 'robot' AND game_id = 222 FOR UPDATE; 
UPDATE figures SET position = 'c4' WHERE id = 1234;
COMMIT;
```

##### Compare-and-set
Some DBs offers _compare-and-set_ operations to avoid lost updates by allowing an update to happen only if the value
 has not changed since you last read it.

##### Conflict resolution and replication
For multi-leader or leaderless replication, the locks and compare-and-set techniques are not valid cause there is 
several copies of the data distributed in several machines. A common approach is to allow concurrent writes to create 
several conflicting versions of a value (siblings) and merge it with special data structures or application code.
Last write wins (LWW) conflict resolution method is prone to lost updates

#### Write Skew and Phantoms
There are more race conditions apart of the dirty writes and lost updates. If there are two concurrent writes that 
check on a condition (for example a minimum number doctors on a call shift) in a database that has snapshot isolation
it can happen that the condition for an update is meet given the snapshot value, but then after both writes are 
committed the condition is not valid anymore. This anomaly is called _write skew_.

##### Characterizing write skew
In a write skew, the two transactions are updating two different objects and it is a generalization of the lost 
update problem: two transactions read the same objects, and then update some of those objects (if they update the same 
object, you get a dirty write or lost update anomaly). Write skews can't be prevented wit atomic writes, automatic 
detection of lost updates, using DB constrains (because several objects needs to be checked). They are usually 
prevented with serializable isolation level or by locking the rows the transaction depends on:

```sql
BEGIN TRANSACTION;
SELECT * FROM doctors WHERE on_call = true AND shift_id = 1234 FOR UPDATE;
UPDATE doctors SET on_call = false WHERE name = 'Alice' AND shift_id = 1234;
COMMIT;
``` 

##### Phantoms causing write skew
The usual pattern for write skew is as follows:

    1. A SELECT query checks whether some requirement is satisfied by searching for rows that match a search condition 
    2. Depending on the result of the first query, the application code decides how to continue
    3. If the condition is met, the application makes a write to the db and commits the transaction
    4. The effect of this write changes the precondition of the decision of step 2
    
The problem is that if the query in step 1 doesn't return any rows cause we are checking for the absence of them, 
SELECT FOR UPDATE can’t attach locks to anything (calling _phantoms_).

##### Materializing conflicts
A possible solution to avoid phantoms we can artificially introduce a lock object into the database. For example 
create a table with all possible combinations that can appear (for example rooms and slots) and make a select for 
update on those rows (this is called _materializing conflicts_).

### Serializability
The different scenarios covered before shows how difficult is to account of all the possibilities that might produce 
a race condition, therefore the usual recommendation has been to use the serializable isolation, which is considered 
the strongest isolation level. It guarantees that even though transactions may execute in parallel, the end result 
is the same as if they had executed one at a time, serially, without any concurrency. This is achieved through one of
 these techniques:
 
    * Literally executing transactions in a serial order
    * Two-phase locking
    * Optimistic concurrency control techniques such as serializable snapshot isolation
    
#### Actual Serial Execution
The simplest solution to this problem is to execute only one transaction at a time, in serial order, on a single thread.
This is now possible cause with increase and cheaper RAM memory you can have the whole dataset in memory and OLTP 
transactions are usually short and only make a small number of reads and writes.

##### Encapsulating transactions in stored procedures
If a transaction flow for an application depends on a input to progress depending on certain conditions, the 
data base needs to support a potentially huge number of concurrent transactions, most of them idle. An interactive 
style of application (sending request back and forth depending on the answer of the previous query), it not feasible
 for single threaded solutions unless the entire transaction is processed at once in a _stored procedure_.
 
##### Pros and cons of stored procedures
Stored procedures are vendor dependant (PL/SQL, T-SQL, PL/pgSQL), difficult to manage (debugging, code checking, 
deploying...) and are performance sensitive (since is a sink for a lof of applications).

##### Partitioning
For applications with high write throughput, the single-threaded transaction processor can become a serious 
bottleneck. Partition the dataset can be an option to scale, but the database must coordinate the transactions, 
making the cross-partition speed slower due to the necessary overhead, and it is very dataset dependant.

##### Summary of serial execution
Transactions must be small and fast, limited active dataset that can fit in memory, write throughput must be low, 
cross-partition transactions are possible but there is a hard limit to the extent to which they can be used.

#### Two-Phase Locking (2PL)
Two-phase allows several transactions to concurrently read the same object as long as nobody is writing to it, but 
as soon as anyone wants to write an object, exclusive access is required. Writers don’t just block other writers they
 also block readers and vice versa.
 
##### Implementation of two-phase locking
The blocking of readers and writers is implemented by a having a lock on each object in the database. The lock can 
either be in shared mode or in exclusive mode:
    
    * If a transaction wants to read an object, it must acquire the lock in shared mode. Several transactions can hold 
    the lock in shared mode simultaneously, but if another transaction has an exclusive lock on the object, these 
    transactions must wait
    * If a transaction wants to write to an object, it must acquire the lock in exclusive mode. No other 
    transaction may hold the lock at the same time
    * If a transaction first reads and then writes an object, it may upgrade its shared lock to an exclusive lock. The 
    upgrade works the same as getting an exclusive lock directly
    * After a transaction has acquired the lock, it must continue to hold the lock until the end of the transaction. 
    This is where the name “two-phase” comes from: the first phase (while the transaction is executing) is when the
    locks are acquired, and the second phase (at the end of the transaction) is when all the locks are released
    
The database automatically detects deadlocks between transactions and aborts one of them so that the others can make
 progress. The aborted transaction needs to be retried by the application.
 
##### Performance of two-phase locking
Performance is much worst in 2PL than in weak isolation, mostly due to reduce concurrency. If one transaction has to
 wait on another(s), there is no limit on how long it may have to wait. Deadlocks can be frequent also.
 
##### Predicate locks
_Predicate locks_ (like the ones described in the _phantoms_), are acquired as follows in 2PL:

    * A transaction that wants to read objects matching a condition must acquire a shared mode on the conditions of the 
    query (not the resulting rows)
    * A transaction that wants to insert or modify an object must check if either the old or the new value matches 
    any existing predicate lock before.
    
Predicate lock applies even to objects that do not yet exist in the database (hence the lock in the condition itself).

##### Index-range locks
Due to performance reasons, many 2PL databases implements _index-range locking_. Which are like a broader case of 
predicate locking. Instead of looking object room 1 between 1 and 3pm, you lock on room 1 for all times. An 
approximation of the search condition would be attached to one of the search indexes

#### Serializable Snapshot Isolation (SSI)
2PL don’t perform well and serial execution don’t scale well, _serializable snapshot isolation (SSI)_ provides full 
serializability, but has only a small performance penalty compared to snapshot isolation.

##### Pessimistic versus optimistic concurrency control
SSI is an optimistic concurrency control technique, wich doesn't block any object and allows transactions to 
continue, only  when a transaction wants to commit, the database checks whether anything bad happened, only transactions
 that executed serializably are allowed to commit. This performs badly if many transactions trying to access the same 
 objects, but then to perform better in if there is enough spare capacity.
 
##### Decisions based on an outdated premise
On decisions based on the output of a query on SSI, the transaction is taking an action based on a premise (a fact that 
was true at the beginning of the transaction. To be safe, the database needs to assume that any change in the query 
result (the premise) means that writes in that transaction may be invalid.

##### Detecting stale multi-version concurrency control (MVCC) reads
When a transaction reads from a consistent snapshot in an MVCC database, it ignores writes that were made by any 
other transactions that hadn’t yet committed at the time when the snapshot was taken. When the transaction wants 
to commit, the database checks whether any of the ignored writes have now been committed. If so, the transaction 
must be aborted.

##### Detecting writes that affect prior reads
Another case to consider is when another transaction modifies data after it has been read. When a transaction writes to
 the database, it must look in the indexes for any other transactions that have recently read the affected data. 
This process is similar to acquiring a write lock on the affected key range, but rather than blocking until the 
 readers have committed, the lock acts as a tripwire: it simply notifies the transactions that the data they read may
  no longer be up to date.
  
##### Performance of serializable snapshot isolation
SSI is very appealing for read-heavy workloads, SSI is less sensitive to slow transactions than 2PL or serial execution.


## Chapter 8: The Trouble with Distributed Systems<a name="Chapter8"></a>
## Faults and Partial Failures
A failure in a distributed system is usually non-deterministic, resulting in partial failures.

###Cloud Computing and Supercomputing
Supercomputers usually behaves like a big machine with specialized hardware, usually shares the same network and if a 
fail occurs they are usually shut down for mainteinance. Clod computing is different because the nodes are built 
from commodity machines, can't go offline because they usually provide high availability services. We need to assume 
that software is likely to fail and we should still provide a reliable service in that scenario (Building a Reliable
 System from Unreliable Components). 

## Unreliable Networks
Shared nothing architectures where machines are connected using internet and each machines has their own memory 
and disc are the most common way of building systems. In such systems if a sender sends a message the problems are 
indistinguishable from asynchronous network problems, and the usual approach is to raise a timeout (which doesn't 
mean that the operation didn't went through). 

### Network Faults in Practice
There is no way around a network fault, whichever the reason is, you do need to know how your software reacts to 
network problems and ensure that the system can recover from them.

### Detecting Faults
Even if your system can handle network failure like not routing to a dead node or electing a new leader in case of 
replicated systems, there is still uncertaintity on how much data the failing node actually processed: if you want to
 be sure that a request was successful, you need a positive response from the application itself.
 
### Timeouts and Unbounded Delays
Setting the appropriate timeouts can be problematic, if a node has simply slow down, then declared it dead might 
result on performing the same operation twice. Asynchronous networks have unbounded delays. That is, they try to 
deliver packets as quickly as possible, but there is no upper limit on the time it may take for a packet to arrive.

#### Network congestion and queueing
Network congestion slows down the rate of package sending. This can happen due to several reasons like multiple nodes
 trying to use the same communication link at the same time, if destination machine has all CPU cores busy (the 
 package will be queued), round robin of CPU cycles in virtualized environments, the TCP own flow control (congestion
 avoidance or backpressure)... 
In public clouds, resource utilization and network delays can be highly variable specially in noisy clusters. In that
 case the timeout is usually chosen experimentally. Even more, timeouts can be configured dinamically by measuring 
 response times and adjusting it automatically (jitter).
 
### Synchronous Versus Asynchronous Networks
In Synchronous networks like the telephone, the bandwidth is reserved and a circuit is established, guaranteeing 
almost no delay. The packets of a TCP connection in the other hand, opportunistically use whatever network bandwidth is 
available, but no network is used if a TCP link is idle, this is because they are optimized for bursty traffic. 

## Unreliable Clocks
In a distributed system, time is a tricky business, because communication is not instantaneous, each machine on the 
network has its own clock, which is an actual hardware device. The most commonly used mechanism to synchronize clocks
 is the Network Time Protocol (NTP).
 
### Monotonic Versus Time-of-Day Clocks
Modern computers have at least two different kinds of clocks: a time-of-day clock and a monotonic clock.

    * A time-of-day clock returns the current date and time according to some calendar, they are usually synchronized 
    with NTP which can result in odd time back travels if they are far away from the NTP server
    * A monotonic clock is suitable for measuring a duration, they are guaranteed to always move forward, but the 
    absolute value of the clock is meaningless. NTP may adjust the frequency at which the monotonic clock moves forward, 
    but cannot cause the monotonic clock to jump forward or backward
    
In a distributed system, using a monotonic clock for measuring elapsed time (e.g., timeouts) is usually fine, because it 
 doesn't assume any synchronization between different nodes’ clocks and is not sensitive to slight inaccuracies of 
 measurement.
 
### Clock Synchronization and Accuracy
The Time of day clock needs to be synchronized with the NTP server, a computer might reject this synchronization if 
the clock is too away from the NTP server. More problems can arise if the firewall does not allow this 
synchronization to happen or if there is network delays, leap seconds can mess up timing assumptions in systems. It 
is possible to achieve a more accurate synchronization using precision time protocol (PTP).

### Relying on Synchronized Clocks
If you use software that requires synchronized clocks, it is essential that you also carefully monitor the clock 
offsets between all the machines.

#### Timestamps for ordering events
If two nodes are not synchronized, the events they generate (for example a counter increment), can lead to 
incorrectly ordering them, which might cause a drop of a value if the strategy followed is a LRW. logical clocks, 
which are based on incrementing counters rather than an oscillating quartz crystal, are a safer for ordering events.

#### Clock readings have a confidence interval
Even if a clock has a lot of resolution (even nanoseconds), its accuracy depends on variable drift, reading a clock 
is more like a range of times, within a confidence interval.

#### Synchronized clocks for global snapshots
Spanshot isolation allows read-only transactions to see the database in a consistent state at a particular point 
in time, without locking and interfering with read-write transactions. A monotonically increasing transaction ID is 
the usual approach to do this, but in a distributed database this requires coordination. If we have two confidence 
intervals, each consisting of an earliest and latest possible timestamp, and those two intervals do not overlap, then B 
definitely happened after A. Only if the intervals overlap are we unsure in which order A and B happened.

### Process Pauses
In an scenario of a single leader partition where a node needs to know if it is still a leader to accept writes, a 
lease (a lock with a timeout) can be granted, the node accepts writes until the lease is expired and needs to be 
renewed. This scenario can be problematic if the expiry date on the lease is compared with the internal clock value. 
In these situations, a process pause can mess up the clock checking, for example a pause after checking if the lease 
is valid and before accepting the write. This pause can be due to garbage collection, process pause in virtualized 
environments, I/O pause, disks swaps...
A node in a distributed system must assume that its execution can be paused for a significant length of time at any 
 point, even in the middle of a function. 
 
#### Response time guarantees
The pauses described before can be eliminated on _hard real-time systems_ like airplane control systems, those in were 
if no response is given in a certain amount of time, it can provoke a system failure. This is achieved through real-time
 operating system (RTOS) which allows processes to be scheduled with a guaranteed allocation of CPU time in specified
 intervals is needed. These systems are limited to critical software cases due to cost.
 
#### Limiting the impact of garbage collection
An emerging idea is to treat GC pauses like brief planned outages of a node, and to let other nodes handle requests 
from clients while one node is collecting its garbage. If the runtime can warn the application that a node soon 
requires a GC pause, the application can stop sending new requests to that node, wait for it to finish processing 
outstanding requests, or planned restart of process (and thus cleaning of long live objects) can be scheduled.

## Knowledge, Truth, and Lies
A node in the network can only make guesses based on the messages it receives (or doesn’t receive) via the network. 
A node can only find out what state another node is in by exchanging messages with it. If a remote node doesn’t 
respond, there is no way of knowing what state it is in. In a distributed system, we can state the assumptions we 
are making about the behavior (the system model) and design the actual system in such a way that it meets those 
assumptions. 

### The Truth Is Defined by the Majority
If a node can receive messages but can't send them, or after a long GC pause, it can be declared dead, thus we 
shouldn't rely on single node, but in a _quorum_ of nodes. A majority has to take the decision, to avoid conflicts.

#### The leader and the lock
Frequently, a system requires there to be only one of some thing (single node processing write requests, or hold a 
lock). Implementing this in a distributed system requires care an quorum.

#### Fencing tokens
When using a lock or lease to protect access to some resource, we need to ensure that a node that is under a false 
belief of being “the chosen one” cannot disrupt the rest of the system. A simple technique that achieves this goal is
 called fencing. A fencing token is given every time a lock is granted (an increasing number), which is included on 
 the write requests. If a write request is processed by the storage system, it will reject writes with a lower token 
 number.
 
### Byzantine Faults
If a node deliberately wanted to subvert the system’s guarantees, it can do so by sending messages with a fake fencing
 token. A Byzantine fault occurs when a node claim to have received a particular message when in fact it didn't and 
 systems can be Byzantine fault-tolerant. Most Byzantine fault-tolerant algorithms require a supermajority of more than 
 two-thirds of the nodes to be functioning correctly.
 
#### Weak forms of lying
It can be worth adding mechanisms to software that guard against weak forms of “lying” like invalid messages due to 
hardware issues, software bugs, and misconfiguration. This usually includes checksums, input sanitization, or 
multiserver configuration to determine quorums.

### System Model and Reality
Algorithms need to be written in a way that does not depend too heavily on the details of the hardware and software 
 configuration on which they are run. With regard to timing assumptions, three system models are in common use:
 
    * Synchronous model: assumes bounded network delay, bounded process pau‐ ses, and bounded clock error
    * Partially synchronous model: assumes that systems behaves like a synchronous system most of the time
    * Asynchronous model: an algorithm is not allowed to make any timing assumptions (it does not even have a clock)
    
The three most common system models for nodes are:

    * Crash-stop faults: an algorithm may assume that a node can fail in only one way, namely by crashing
    * Crash-recovery faults: nodes may crash at any moment, and perhaps start responding again after some unknown time. 
    Nodes are assumed to have stable storage that is preserved across crashes, in-memory state is assumed to be lost
    * Byzantine (arbitrary) faults: Nodes may do absolutely anything, including trying to trick and deceive other nodes
    
For modeling real systems, the partially synchronous model with crash-recovery is generally the most useful model.

#### Correctness of an algorithm
To define what it means for an algorithm to be correct, we can describe its properties. An algorithm is correct in 
some system model if it always satisfies its properties in all situations that we assume may occur in that system model.

#### Safety and liveness
The properties mentioned before can be of two types:

    * safety: defined as nothing bad happens, if they are violated we can point at a particular point in time at 
    which it was broken (i.e. uniqueness), the violation can't be undone as the damage is already done
    * liveness: defined as something good eventually happens, they may not hold at some point in time (i.e. server 
    availability)
    
For distributed algorithms, it is common to require that safety properties always hold, in all possible situations 
 of a system model while with liveness properties we are allowed to make caveats.
 
#### Mapping system models to the real world
The system model is a simplified abstraction of reality, a real implementation may still have to include code to 
handle the case where something happens that was assumed to be impossible. Theoretical analysis and empirical testing 
are equally important.


## Chapter 9: Consistency and Consensus<a name="Chapter9"></a>
The best way of building fault-tolerant systems is to find some general-purpose abstractions with useful guarantees, 
one of the most useful ones is _consensus_.

### Consistency Guarantees
Most replicated databases provide at least eventual consistency (also named _convergence_), which means that if you 
stop writing to the database and wait for some unspecified length of time, then eventually all read requests will return
 the same value. This is a weak for of guarantee, systems with stronger guarantees may have worse performance or be 
 less fault-tolerant than systems with weaker guarantees.
 
### Linearizability
The idea behind linearizability ( or _atomic consistency_, _strong consistency_, _immediate consistency_, or 
_external consistency_) is that the database gives the illusion that there is only one replica so the client would have 
the same view of the data. This means guaranteeing that the value read is the most recent, up-to-date value, and 
doesn't come from a stale cache or replica.

#### What Makes a System Linearizable?
In a linearizable system we imagine that there must be some point in time (between the start and end of the write 
operation) at which the value written atomically flips from one value to another. Thus, if one client’s read returns 
the new value, all subsequent reads must also return the new value, even if the write operation has not yet completed.
Once a new value has been written or read, all subsequent reads see the value that was written, until it is 
overwritten again.

#### Relying on Linearizability

    * Locking and leader election: A system that uses single-leader replication needs to ensure that there is indeed 
    only one leader, not several (split brain). This can be implemented with locks, every node that starts up tries 
    to acquire the lock, and the one that succeeds becomes the leader
    * Constraints and uniqueness guarantees: If you want to enforce the uniqueness constraint as the data is written, 
    you need linearizability
    * Cross-channel timing dependencies: If a system needs two different communication channels to do some task (like
     a background resizing process that needs to read from an external storage), you need linearizability 

#### Implementing Linearizable Systems
The most common approach to making a system fault-tolerant is to use replication. But if we compare whether they can be 
made linearizable:

    * Single-leader replication (potentially linearizable): Using the leader for reads relies on the assumption that 
    you know for sure who the leader is (not guaranteed with asynchronous replication)
    * Consensus algorithms (linearizable): these protocols contain measures to prevent split brain and stale replicas
    * Multi-leader replication (not linearizable): because concurrently process writes on multiple nodes and 
    asynchronously replicate them to other nodes
    * Leaderless replication (probably not linearizable): Depending on the exact configuration of the quorums, and 
    depending on how you define strong consistency, linearizability might be broken
    
##### Linearizability and quorums
It is possible to make Dynamo-style quorums linearizable at the cost of reduced performance: a reader must perform 
read repair synchronously, before returning results to the application, and a writer must read the latest state of a
 quorum of nodes before sending its writes. It is safest to assume that a leaderless system with Dynamo-style 
 replication does not provide linearizability.

#### The Cost of Linearizability
Consider what happens if there is a network interruption between the two datacenters. With a multi-leader database, 
each datacenter can continue operating normally: since writes from one datacenter are asynchronously replicated to 
the other, the writes are simply queued up and exchanged when network connectivity is restored. If single-leader 
replication is used,any writes and any linearizable reads must be sent to the leader, for any clients connected
 to a follower datacenter, those read and write requests must be sent synchronously over the network to the leader 
 datacenter.

##### The CAP theorem
Important considerations:

    * If your application requires linearizability, and some replicas are disconnected from the other replicas due 
    to a network problem, then some replicas cannot process requests while they are disconnected
    * If your application does not require linearizability, then it can be written in a way that each replica can 
    process requests independently, even if it is disconnected from other replicas

The CAP theorem only considers one consistency model (linearizability) and one kind of fault (network partitions or 
nodes that are alive but disconnected from each other), so it has little practical value for designing systems.

##### Linearizability and network delays
Even in multi core CPUs, reads are not guaranteed to get the value written by another core thread, since there are 
intermediate caches and buffers. In this case, the reason for dropping linearizability is performance, not fault 
tolerance, which is applicable to other databases. Weaker consistency models can be much faster than linear ones.

### Ordering Guarantees
Linearizability requires some guarantee of ordering, there are deep connections between ordering, linearizability, 
and consensus.

#### Ordering and Causality
Order preserves causality, some examples of casuality are: 

    * Causal dependency between the question and the answer (prefix reads)
    * A row should exists to be able to modify it (read overtake)
    * A happened before relationship is another expression of causality (concurrent write detection)
    * In snapshot isolation, transaction reads from a consistent snapshot. Reading stale data violates casuality
    * Write skew between transactions, serializable snapshot isolation detects write skew by track‐ ing the causal 
    dependencies between transactions

The chains of causally dependent operations define the causal order in the system, what happened before what. If a 
system obeys the ordering imposed by causality, we say that it is causally consistent.

##### The causal order is not a total order
A total order allows any two elements to be compared (i.e natural numbers). The difference between a total order and
 a partial order is reflected in different database consistency models:
 
    * Linearizability: There is a total order of operations, we can always say which operation happened first
    * Causality: Two events are ordered if they are causally related, but they are incomparable if they are 
    concurrent. Casualty defines a partial order (some elements are incomparable)
    
According to this definition, there are no concurrent operations in a linearizable datastore.

##### Linearizability is stronger than causal consistency
Linearizability implies causality: any system that is linearizable will preserve causality correctly. Linearizability is
 not the only way of preserving causality, causal consistency is the strongest possible consistency model that does 
 not slow down due to network delays, and remains available in the face of network failures.
 
##### Capturing causal dependencies
In order to maintain causality, you need to know which operation happened before which other operation. Causal 
consistency techniques are similar to the ones for detecting concurrent writes, but it needs to track causal 
dependencies across the entire database, not just for a single key. Hence these databases needs to know which version
 of the data was read by the application.
 
#### Sequence Number Ordering
Sequence numbers helps to guarantee casuality and keep tracks of dependencies. Timestamps for these numbers can come 
from a _logical clock_, which is an algorithm to generate a sequence of numbers to identify operations, typically 
using counters that are incremented for every operation.
In a database with single-leader replication, the replication log defines a total order of write operations that is 
consistent with causality.

##### Noncausal sequence number generators
For non single-leader databases is a bit less clear how to generate these sequence numbers. There are several methods:

    * Each node can generate its own independent set of sequence numbers
    * You can attach a timestamp from a time-of-day clock to each operation (used in LWR)
    * You can preallocate blocks of sequence numbers
    
These operations are more performant but the sequence numbers they generate are not consistent with causality.

##### Lamport timestamps
There is a simple method for generating sequence numbers that is consistent with causality, Called _Lamport timestamp_.
The Lamport timestamp is simply a pair of operation counter and node identifier attached to the operation. This provides
total ordering: if you have two timestamps, the one with a greater counter value is the greater timestamp and if the 
counter values are the same, the one with the greater node ID is the greater timestamp.
This is consistent with casuality because every node and every client keeps track of the maximum counter value it 
has seen so far, and includes that maximum on every request. When a node receives a request or response with a 
maximum counter value greater than its own counter value, it immediately increases its own counter to that maximum.

##### Timestamp ordering is not sufficient
In the case where a node needs to make a decission on the moment, without gathering the results from other nodes to 
determine if it is possible (for example creation of a unique username), timestamp ordering is not sufficient, you 
also need to know when that order is finalized. This idea of knowing when your total order is finalized is known as 
_total order broadcast_.

#### Total Order Broadcast
Total order broadcast is a protocol for exchanging messages between nodes. It requires two safety properties:

    * Reliable delivery: No messages are lost (if a message is delivered to one node, it is delivered to all nodes)
    * Totally ordered delivery: Messages are delivered to every node in the same order
    
##### Using total order broadcast
Consensus services (_ZooKeeper_ and _etcd_) implements total order broadcast. Total order broadcast is needed for 
database replication: if every message represents a write to the database, and every replica processes the same 
writes in the same order, then the replicas will remain consistent with each other (known as state machine replication).
In total order broadcast the order is fixed at the time the messages are delivered, also useful for implementing a 
 lock service that provides fencing tokens.

##### Implementing linearizable storage using total order broadcast
Total order broadcast is asynchronous: messages are guaranteed to be delivered reliably in a fixed order, but there 
is no guarantee about when a message will be delivered. Linearizability is a recency guarantee: a read is guaranteed 
to see the latest value written. You can build linearizable storage on top of total order broadcast, with the example
 of the unique username:
     
    * Append a message to the log, tentatively indicating the username you want to claim
    * Read the log, and wait for the message you appended to be delivered back to you
    * Check for any messages claiming the username that you want. If the first message for your desired username is 
    your own message, then you are successful (perhaps commmit it by appending another message to the log) and 
    acknowledge it to the client
    
The procedure described provides sequential consistency, also known as _timeline consistency_, a slightly weaker 
guarantee than linearizability).

##### Implementing total order broadcast using linearizable storage
The easiest way to build total order broadcast using linearizable storage is to assume you have a linearizable register
 that stores an integer and that has an atomic increment-and-get operation. Alternatively, an atomic compare-and-set
 operation would also do the job.
 
### Distributed Transactions and Consensus
The goal of consensus is simply to get sev‐ eral nodes to agree on something. Consensus is important in _leader 
election_ or _Atomic commit_ (in distributed transactions either all nodes commits or rolls back).

#### Atomic Commit and Two-Phase Commit (2PC)
##### From single-node to distributed atomic commit
On a single node, transaction commitment crucially depends on the order in which data is durably written to disk: 
first the data, then the commit record to indicate a successful transaction. In distributed transactions it is not 
sufficient to simply send a commit request to all of the nodes and independently commit the transaction on each one, 
as problems with communications, uniqueness of keys or even crashes might happen in any node making the transaction 
inconsistent.

##### Introduction to two-phase commit
Two-phase commit is an algorithm for achieving atomic transaction commit across multiple nodes. 2PC uses a component
that does not normally appear in single-node transactions named _coordinator_ or _transaction manager_. A 2PC  
transaction begins with the application reading and writing data on multiple database nodes (called participants in 
the transaction). In phase 1, the coordinator sends a _prepare_ request to each of the nodes, asking them whether 
they are able to commit: If all participants reply “yes”, the coordinator sends out a commit request in phase 2, and
 the commit actually takes place, if any of the participants replies “no” the coordinator sends an abort request to 
 all nodes in phase 2.

##### A system of promises
To guarantee that the previous 2PC actually works, the process requires:

    * When the application wants to begin a distributed transaction, it requests a transaction ID from the coordinator 
    (globally unique)
    * The application begins a single-node transaction on each of the participants with the transaction ID
    * When the application is ready to commit, the coordinator sends a prepare request to all participants, tagged with 
    the global transaction ID (or an abort request if any node fails)
    * When a participant receives the prepare request, it makes sure that it can definitely commit the transaction 
    under all circumstances. This includes writing all transaction data to disk, and checking for any conflicts or 
    constraint violations. The participant surrenders the right to abort the transaction, but without committing it
    * When the coordinator has received responses to all prepare requests, it makes a definitive decision. The 
    coordinator must write that decision to its transaction log on disk (called the commit point)
    * Once the coordinator’s decision has been written to disk, the commit or abort request is sent to all 
    participants. If this request fails or times out, the coordinator must retry forever until it succeed
    
##### Coordinator failure
If a participant has received a prepare request and voted “yes” it can no longer abort unilaterally a transaction. If 
the coordinator crashes or the network fails at this point, the participant can do nothing but wait (this state is 
called in doubt or uncertain). The only way 2PC can complete is by waiting for the coordinator to recover (polling 
other participants is not part of the protocol).

#### Distributed Transactions in Practice
Many cloud services choose not to implement distributed transactions due to the operational problems they engender. 
Two quite different types of distributed transactions are often conflated:

    * Database-internal distributed transactions: all the nodes participating in the transaction are running the same
     database software
    * Heterogeneous distributed transactions: the participants are two or more different technologies
    
##### Exactly-once message processing
Heterogeneous distributed transactions allow diverse systems to be integrated, a distributed transaction is only 
possible if all systems affected by the transaction are able to use the same atomic commit protocol. If all side 
effects of processing a message are rolled back on transaction abort, then the processing step can safely be retried
 as if nothing had happened.
 
##### XA transactions
X/Open XA (short for eXtended Architecture) is a standard for implementing two- phase commit across heterogeneous 
technologies (it is a C API for interfacing with a transaction coordinator). XA assumes that your application uses a
 network driver or client library to communicate with the participant databases or messaging services.
 
##### Holding locks while in doubt
Database transactions usually take a row-level exclusive lock on any rows they modify, to prevent dirty writes. The 
database cannot release those locks until the transaction commits or aborts.

##### Recovering from coordinator failure
In practice, orphaned in-doubt transactions (transactions for which the coordinator cannot decide the outcome for 
whatever reason) do occur, sitting forever in the database, holding locks and blocking other transactions. The only 
way out is for an administrator to manually decide whether to commit or roll back the transactions. Many XA 
implementations have an emergency escape hatch called heuristic decisions: allowing a participant to unilaterally 
decide to abort or commit an in-doubt transaction without a definitive decision from the coordinator.

##### Limitations of distributed transactions
XA transactions introduces major operational problems:

    * If the coordinator is not replicated but runs only on a single machine, it is a single point of failure for the 
    entire system
    * Many server-side applications are developed in a stateless model, with all persistent state stored in a database, 
    since the coordinator logs are required in order to recover in-doubt transactions after a crash, those application 
    servers are no longer stateless.
    * Since XA needs to be compatible with a wide range of data systems, it is necessarily a lowest common denominator
    * Distributed transactions thus have a tendency of amplifying failures, if any part of the system is broken, the 
    transaction also fails
    
#### Fault-Tolerant Consensus
A consensus algorithm must satisfy the following properties:

    * Uniform agreement: No two nodes decide differently
    * Integrity: No node decides twice
    * Validity: If a node decides value v, then v was proposed by some node
    * Termination: Every node that does not crash eventually decides some value

Here, _termination_ is a liveness property, whereas the other three are safety properties (2PC does not meet the 
requirements for termination), although any consensus algorithm requires at least a majority of nodes to be 
functioning correctly in order to assure termination.

##### Consensus algorithms and total order broadcast
Most of consensus algorithms don’t directly use the formal model described before. Instead, they decide on a 
sequence of values, which makes them total order broadcast algorithms. Total order broadcast is equivalent to 
repeated rounds of consensus.

##### Epoch numbering and quorums
As stated before, single-leader replication needs consensus to avoid the split-brain problem, but it seems that in 
order to elect a leader, we first need a leader (so we need to solve consensus). All of the consensus protocols 
discussed don’t guarantee that the leader is unique. Instead, they can make a weaker guarantee: the protocols define
 an epoch number and guarantee that within each epoch, the leader is unique.
If there is a conflict between two different leaders in two different epochs, then the leader with the higher epoch
 number prevails. For every decision that a leader wants to make, it must send the proposed value to the other nodes
 and wait for a quorum of nodes to respond in favor of the proposal.

##### Limitations of consensus
Consensus algorithms are not used everywhere, because the benefits come at the cost of performance (they are a kind of 
synchronous replication), always require a strict majority to operate, assume a fixed set of nodes that participate 
in voting (you can't just add or remove nodes), they generally rely on timeouts to detect failed nodes (producing 
false positive detections and affecting performance) and some consensus algorithms are particularly affected by 
network problems.

#### Membership and Coordination Services
Projects like ZooKeeper or etcd are often described as “distributed key-value stores”, holding small amounts of data 
that can fit entirely in memory, which is replicated across all the nodes using a fault-tolerant total order 
broadcast algorithm. Zookeeper provides features that are particularly useful when building distributed systems:

    * Linearizable atomic operations: Using an atomic compare-and-set operation, you can implement a lock (usually 
    implemented as a lease)
    * Total ordering of operations: fencing tokens are needed to prevent clients from conflicting with each other in 
    the case of a process pause. ZooKeeper provides this by totally ordering all operations and giving each 
    operation a monotonically increasing transaction ID (zxid) and version number (cversion)
    * Failure detection: Clients maintain a long-lived session on ZooKeeper servers, and the client and server 
    periodically exchange heartbeats to check that the other node is still alive
    * Change notifications: A client can read locks and values that were created by another client and watch them for
    changes
    
##### Service discovery
ZooKeeper, etcd, and Consul are also often used for service discovery—that is, to find out which IP address you need
to connect to in order to reach a particular service. Although service discovery does not require consensus, leader
election does. For this purpose, some consensus systems support read-only caching replicas. These replicas 
asynchronously receive the log of all decisions of the consensus algorithm, but do not actively participate in voting.

##### Membership services
Zookeeper and other similar systems can be seen as a _membership service_, which determines which nodes are currently 
active and live members of a cluster. If you couple failure detection with consensus, nodes can come to an agreement 
about which nodes should be considered alive or not. 

## Chapter 10: Batch Processing<a name="Chapter10"></a>
