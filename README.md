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
If the data in your application has a document-like structure (i.e., a tree of one-to- many relationships, where 
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
The Resource Description Framework (RDF) was intended as a mechanism for different web‐ sites to publish data in
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

Because RDF doesn’t distinguish between properties and edges but just uses predi‐ cates for both, you can use the 
same syntax for matching properties.

#### The Foundation: Datalog
Datalog’s data model is similar to the triple-store model. Instead of writing a triple as (subject, predicate, object), 
we write it as predicate(subject, object). In Datalog, we define rules that tell the database about new predicates. 
These predicates aren't triples stored in the database, but instead they are derived from data or from other rules. 
Rules can refer to other rules.

## Chapter 3: Storage and Retrieval<a name="Chapter3"></a>
