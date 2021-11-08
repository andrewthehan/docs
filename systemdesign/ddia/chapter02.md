# Chapter 2: Data Models and Query Languages

---

- [Data Models](#data-models)
  - [Model Types](#model-types)
    - [Relational Model ("SQL")](#relational-model---sql--)
    - [Nonrelational Model ("NoSQL")](#nonrelational-model---nosql--)
      - [Document Model](#document-model)
    - [Hierarchical Model](#hierarchical-model)
    - [Network Model](#network-model)
    - [Other data models](#other-data-models)
  - [Object-Relational Mismatch](#object-relational-mismatch)
  - [Many-to-One and Many-to-Many Relationships](#many-to-one-and-many-to-many-relationships)
  - [Schemas](#schemas)
  - [Data Locality](#data-locality)
- [Query Languages for Data](#query-languages-for-data)
  - [Declarative](#declarative)
  - [Imperative](#imperative)
  - [Examples](#examples)
    - [Queries on the Web](#queries-on-the-web)
    - [MapReduce](#mapreduce)
- [Graph-Like Data Models](#graph-like-data-models)
  - [Comparison to Network Model](#comparison-to-network-model)
  - [Models](#models)
    - [Property Graph Model](#property-graph-model)
    - [Triple-store Model](#triple-store-model)
      - [Semantic Web](#semantic-web)
  - [Query Languages](#query-languages)
    - [Declarative](#declarative-1)
      - [Cypher](#cypher)
      - [SPARQL](#sparql)
      - [Datalog](#datalog)
- [Summary](#summary)

---

## Data Models

- Affect not only how software is written but also how we think about the problem
- Applications are layers of data models

### Model Types

#### Relational Model ("SQL")

- Data is organized into _relations_ (_tables_ in SQL) where each relation is an unordered collection of _tuples_ (_rows_ in SQL)
- Roots primarily in business data processing
  - _Transactions processing_: sales/banking transactions, airline reservations, stock-keeping
  - _Batch processing_: customer invoicing, payroll, reporting
- Good support for joins
- Good support for many-to-ony and many-to-many relationships
- Data is simple, no complicated access paths
- Query optimizer automatically forms the "access path" so developer rarely needs to think about it

#### Nonrelational Model ("NoSQL")

- Latest attempt at competing approach to relational model
- _NoSQL_ doesn't refer to any particular technology, originally a Twitter hashtag
- Driving forces behind adoption:
  - Need for greater scalability including large datasets and high write throughput
  - Preference for free and open source software over commercial products
  - Specialized query operations not well supported by relational model
  - Frustration with restrictiveness of relational schemas and a desire for a more dynamic and expressive data model
- **Polyglot persistence**: likely both relational and nonrelational datastores will coexist

##### Document Model

- Reduces the **impedance mismatch** between application code and the storage layer due to prevalence of OOP
- Better _locality_ than multi-tabled schema (all relevant information is in one place without a need to join)
- Schema flexibility
- Cannot refer directly to a nested item

#### Hierarchical Model

- **Information Management System (IMS)**
- Tree of records nested within records
- Made many-to-many relationships difficult
- Doesn't support joins

#### Network Model

- **Conference on Data Systems Languages (CODASYL)** committee standardized this model
- Generalization of the hierarchical model
  - Each record can have multiple parents whereas in the hierarchical model, each record has one parent
- Links between records are not foreign keys but pointers
- Only way to access record is to find the _access path_ from root record to desired record
- Made efficient use of limited hardware but queries were complex and application code was tightly coupled with data model
  - Queries are performed by moving a cursor by iterating over lists of records and following access paths
  - Application code had to keep track of all relationships

#### Other data models

- Genome data
  - Matching one very long string against a large database of strings that are similar, but not identical
- Large Hadron Collider
  - Custom solutions required when working with hundreds of petabytes to stop hardware costs from spiraling out of control
- Full-text search

### Object-Relational Mismatch

- If data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns
- Also known as **impedance mismatch**
- **Object-relational mapping (ORM)** frameworks reduce some boilerplate but can't completely hide the differences between the two models
- Various ways to manage one-to-many relationships in a relational database:
  - Normalized representation with a separate table storing mapped data with a foreign key reference to the primary table
  - Leveraging support for structured datatypes allowing multi-valued data to be stored within a single row, with support for querying and indexing
  - Encode mapped data (as JSON or XML) and store it on a text column (no support for querying values inside that encoded column)

### Many-to-One and Many-to-Many Relationships

Benefits of ID/enum over free-text fields:

- Consistent style and spelling
- Avoiding ambiguity
- Ease of updating
- Localization support
- Better search

IDs remove duplication:

- IDs have no meaning to humans so no need to change
- No overhead of updating duplicate copies
- No write overhead
- No risk for inconsistencies

**Normalization**: removing duplication in databases

- Normalizing data requires _many-to-one_ relationships which relational model supports better than document model (need to emulate joins in application code)

### Schemas

**Schemaless**: often used to describe document databases but is misleading

**Schema-on-read**:

- Structure of data is implicit, only interpreted when data is read
- Similar to dynamic (runtime) type checking
- To change data format, just write new documents and have code support both cases
- Advantageous if items in collection have different structures
  - Many different types of objects and not practical to put each type in its own table
  - Structure of data is determined by external systems

**Schema-on-write**

- Schema is explicit and database ensures all written data conforms to it
- Similar to static (compile-time) type checking
- To change data format, need to do a data _migration_

### Data Locality

- Performance advantage to storage locality
- Advantage only applies if you need large parts of document at the same time
- Advantage is not limited to document model
  - Google's Spanner database allows a table's rows to be interleaved within a parent table
  - Bigtable data model has _column-family_ concept

## Query Languages for Data

### Declarative

- Specifies the pattern of data you want (conditions the results must meet and how data should be transformed) but now how to achieve that goal
- SQL is a _declarative_ query language
- Follows structure of relational algebra
- Attractive because concise and easier to work with
- Hides implementation details of the database engine allowing future performance improvements without query changes
- Lends themselves to parallel execution

### Imperative

- Describes algorithm to get desired results
- IMS and CODASYL queries using _imperative_ code
- Hard to parallelize because specifies instructions that must be performed in a particular order

### Examples

#### Queries on the Web

- CSS is declarative
- JavaScript DOM API is imperative

#### MapReduce

- Programming model for processing large amounts of data in bulk across many machines
- Limited form is supported by some NoSQL datastores for performing read-only queries
- Neither declarative nor imperative
- Based on _map_ (`collect`) and _reduce_ (`fold` or `inject`) functions that exist in functional programming languages
- Both map and reduce functions:
  - Must be _pure_ (only use data that is input)
  - Cannot perform additional database queries
  - Must not have any side effects
- Allows database to
  - Run the functions anywhere
  - Run the functions in any order
  - Rerun the functions on failure
- Requires two carefully coordinated functions which is harder than writing a single query

## Graph-Like Data Models

- Natural model for data with many complex many-to-many relationships
- Consists of
  - **Vertices** / Nodes / Entities
  - **Edges** / Relationships / Arcs
- Typical examples of **homogeneous** data include:
  - Social graphs
    - **Vertices**: people
    - **Edges**: which people know each other
  - The web graph
    - **Vertices**: web pages
    - **Edges**: HTML links to other pages
  - Road or rail networks
    - **Vertices**: junctions
    - **Edges**: the roads or railway lines between them
- Well-known algorithms can operate on these graphs
  - _Shortest path between two points_ can be used for car navigation system
  - _PageRank_ can be used to determine the popularity of a web page and its ranking in search results
- Can also store different types of data
  - Facebook maintains a single graph with different types of vertices and edges
    - **Vertices**: people, locations, events, checkins, comments
    - **Edges**: which people are friends, which checkin happened in which location, who commented on which post, who attended which event

### Comparison to Network Model

| Graph                                                                                                      | Network                                                                                  |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Any vertex can have an edge to any other vertex                                                            | Has schema that specifies which record type can be nested within which other record type |
| Can refer directly to any vertex by its unique ID or use an index to find vertices with a particular value | Only way to reach a particular record is to traverse one of the access paths to it       |
| Vertices and edges are not ordered                                                                         | Children of a record are ordered so database has to maintain the ordering                |
| Supports both declarative and imperative queries                                                           | All queries are imperative                                                               |

### Models

#### Property Graph Model

- Implemented by Neo4j, Titan, InfiniteGraph
- Each vertex consists of
  - Unique identifier
  - Set of outgoing edges
  - Set of incoming edges
  - Collection of properties (key-value pairs)
- Each edge consists of
  - Unique identifier
  - **Tail vertex**: vertex at which the edge starts
  - **Head vertex**: vertex at which the edge ends
  - Label to describe the kind of relationship between the two vertices
  - Collection of properties (key-value pairs)
- Can consider graph stores as consisting of two relational tables, one for vertices and one for edges
- Important aspects:
  - Any vertex can have an edge connecting it with any other edge (schema cannot restrict these)
  - Given any vertex, you can efficiently find both its incoming and its outgoing edges, and thus traverse the graph
  - By using different labels for different kinds of relationships, you can store several different kinds of information in a single graph

#### Triple-store Model

- Implemented by Datomic, AllegroGraph
- Similar to property graph, uses different words to describe the same ideas
- All information is stored as three-part statements: (_subject_, _predicate_, _object_)
  - Subject == vertex
  - Predicate/Object == (key/value) or (edge/vertex)

##### Semantic Web

- _"Websites already publish information as text and pictures for humans to read, so why don't they also publish information as machine-readable data for computers to read?"_
- **Resource Description Framework (RDF)**: mechanism for different websites to publish data in a consistent format, allowing data from different websites to be automatically combined into a web of data
  - Because it is designed for internet-wide data exchange, the subject, predicate, and object are often URIs
    - URIs don't need to resolve to anything, just a namespace to avoid conflicts
- Lots of hype but no sign of being realized in practice

### Query Languages

#### Declarative

- Possible to query graph data represented in a relational database using SQL
  - Traversing a variable number of edges means number of joins is not fixed
  - Possible using _recursive common table expressions_ (`WITH RECURSIVE`)

##### Cypher

- For property graph model
- Created for the Neo4j graph database
- Named after a character in the movie _The Matrix_ and is not related to ciphers in cryptography

##### SPARQL

- For triple-store model
- Uses the RDF data model
- Acronym for **SPARQL Protocol and RDF Query Language** ("sparkle")

##### Datalog

- For Datomic
- Cascalog is Datalog implementation for Hadoop
- Similar to the triple-store model
  - Instead of `(subject, predicate, object)` -> `predicate(subject, object)`
- Define _rules_ for new predicates, similar to reusable functions

## Summary

- Started with data being represented as one big tree with **hierarchical model**
- Need for many-to-many relationships solved with **relational model**
- Some applications don't fit relational model:
  - **Document model**: use cases where data comes in self-contained documents and relationships are rare
  - **Graph model**: use cases where anything is potentially related to everything
- Each data model comes with its own query language
