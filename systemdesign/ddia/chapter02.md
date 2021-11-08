# Chapter 2: Data Models and Query Languages

---

- [Data Models](#data-models)
  - [Model Types](#model-types)
    - [Relational Model (SQL)](#relational-model--sql-)
    - [Nonrelational Model (NoSQL)](#nonrelational-model--nosql-)
      - [Document Model](#document-model)
    - [Hierarchical Model](#hierarchical-model)
    - [Network Model](#network-model)
  - [Object-Relational Mismatch](#object-relational-mismatch)
  - [Many-to-One and Many-to-Many Relationships](#many-to-one-and-many-to-many-relationships)
  - [Schemas](#schemas)
  - [Data locality](#data-locality)
- [Query Languages for Data](#query-languages-for-data)

---

## Data Models

- Affect not only how software is written but also how we think about the problem
- Applications are layers of data models

### Model Types

#### Relational Model (SQL)

- Data is organized into _relations_ (_tables_ in SQL) where each relation is an unordered collection of _tuples_ (_rows_ in SQL).
- Roots primarily in business data processing
  - _Transactions processing_: sales/banking transactions, airline reservations, stock-keeping
  - _Batch processing_: customer invoicing, payroll, reporting
- Various competing approaches (_network model_, _hierarchical model_)
- Good support for joins
- Good support for many-to-ony and many-to-many relationships
- Data is simple, no complicated access paths
- Query optimizer automatically forms the "access path" so developer rarely needs to think about it

#### Nonrelational Model (NoSQL)

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

### Object-Relational Mismatch

_If data is stored in relational tables, an awkward translation layer is required between the objects in the application code and the database model of tables, rows, and columns._

- AKA: **impedance mismatch**
- **Object-relational mapping (ORM)** frameworks reduce some boilerplate but can't completely hide the differences between the two models.
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

Normalizing data requires _many-to-one_ relationships which relational model supports better than document model (need to emulate joins in application code).

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
