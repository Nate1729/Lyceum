# Database
Organized collection of inter-related data that models some aspect of the real-world.

Databases are the core component of most computer applications

# Database Example
Create a database that models a digital music store to keep track
of artists and albumbs.

Things we need to store:
- Information about <u>Artists</u>
- What <u>Albums</u> those Artists released

## Flat File Strawman
Store our databse as comma-separated value (CSV) files that we manage in
our own code.
- Use a separate file per entity.
- The application has to parse the files each tiem they want to read/update
the records.

Example Artist file:
```
"Wu Tang Clan",1992,"USA"
"Nortorius BIG", 1992,"USA"
"Ice Cube",1989,"USA"
```
Example Album file:
```
"Enter the Wu Tang","Wu Tang Clan",1993
"St. Ides Mix Tap","Wu Tang Clan",1994
"AmeriKKKa's Most Wanted","Ice Cube",1990
```

### Trying to Find the Year that Ice Cube Went Solo
```python
for line in file:
    record = parse(line)
    if "Ice Cube" == record[0]:
        print(int(record[1]))
```

### Flat Files: Data Integrity
- How do we ensure that the artist is the same for each album entry?
  - What about spelling mistakes?
  - What if an artist changes their name?
- What if somebody overwrites the album year with an invalid string?
  - The file in just plain text stored on disc, someone could easily open it
  and modify it.
- How do we store the case where _one_ album has _many_ artists?

### Flat Files: Implementation
- How do you find a particular record?
  - Are we doing to parse the entire file _every_ time just to find one record?
- What if we now want to create a new application that uses the same database?
  - We wrote the query logic in python... now if we have an application that
  doesn't _use_ python, we must duplicate all the query logic into a new language.
    - This has to be done for _every_ langauge that wants to use this system
- What if two threads try to write to the same file at the same time?
  - Race Condition
  - No ability to run concurrently

### Flat Files: Durability
- What if the machine crashes while our program is updating a record?
  - How do fix the broken state? Can we fix the broken state?
- What if we want to replicate the database on multiple machines for high availability?
  - If one machine crashes, we can use an back-up machine while we fix the first one

# Database Management System
A solution to these problems

A **DBMS** is a software that allows applications to store and analyze information in a database.

A general-purpose DBMS is designed to allow the definition, creation, querying, update,
and administration of databases.

This allows the details of fixing the aforementioned problems to be delt with by this
system. Event the engineer using the system doesn't have concern themselves with
preventing a data race condition.

This software is ***re-useable***. We don't have to re-architect/invent the solutions to
these issues for every application. Rather every application can use this one system.

# Early DBMSs
Database applications were difficult to build and maintain.

Tight coupling between logical and physical layers.

You have to (roughly) know what queries your app would execute before you
deployed the database.

We have the advantage now of all the mistakes people made in the past with naive approaches.

Another issue (which is more of an issue now) is that _people_ are more expensive
than computers and computing power. Having to refactor your data storage for every project
is a huge ask, fiscally.

## The Relational Model
- Proposed in 1970 by Ted Codd.
- Database abstraction to avoid this maintenance:
  - Store databse in simple data structures
  - Access data through high-level language
  - Physical storage left up to implementation

Two major papers are typically cited when talking about Codd's new idea

>Derivability, Redundancy and Consistency of Relations Stored in Large Data Banks 

and

>A Relational Modal of Data for Large Shared Data Banks
  
# Data Models
A <u>data model</u> is a collection of concepts for describing the data in a database.

A <u>schema</u> is a description of a particular collection of data, using a given data
model.

## Examples of Data Models
Most Common and focus of this course:
- Relational

NoSQL Style Models:
- Key/Value
- Graph
- Document
- Column-family

Machine Learning:
- Array / Matrix

Rare, mostly obsolete:
- Hierarchical
- Network

# Back to the Relational Model
**Structure:** The definition of relations and their contents.

**Integrity:** Ensure the database's contents satisfy constraints.

**Manipulation:** How to access and modify a database's contents.

## Using the Relational Model with the Artist/Album Example
> A <u>relation</u> is an unordered set that contains the relationship of attributes that
represent entities.

> A <u>tuple</u> is a set of attribute values (also known as its <u>domain</u>) in the relation
> * Values are (normally) atomic/scalar.
>   * The atomic constraint was what Codd originally proposed
> * The special value **NULL** is a member of every domain

Example:
| name          | year | country |
| ------------- | ---- | ------- |
| Wu Tang Clan  | 1992 | USA     |
| Notorious BIG | 1992 | USA     |
| Ice Cube      | 1989 | USA     |

## Primary Keys
A relation's <u>primary key</u> uniquely identifies a single tuple.

Some DBMSs automatically create an internal primary key if you don't define one.

Auto-generation of unique integer primary keys:
* `SEQUENCE` (SQL:2003)
* `AUTO_INCREMENT` (MySQL)

Using the primary key concept our new "table" becomes this 
| id  | name          | year | country |
| --- | ------------- | ---- | ------- |
| 123 | Wu Tang Clan  | 1992 | USA     |
| 456 | Notorious BIG | 1992 | USA     |
| 789 | Ice Cube      | 1989 | USA     |

## Foreign Keys
A <u>foreign key</u> specifies that an attribute from one relation has to map to a
tuple in another relation.

This let's create relationships like this:

| id  | name          | year | country |
| --- | ------------- | ---- | ------- |
| 123 | Wu Tang Clan  | 1992 | USA     |
| 456 | Notorious BIG | 1992 | USA     |
| 789 | Ice Cube      | 1989 | USA     |

| id | name                    | artist | year |
| -- | ----------------------- | --- | ---- |
| 11 | Enter the Wu Tang       | 123 | 1993 |
| 22 | St. Ides Mix Tape       | 456 | 1994 |
| 33 | AmeriKKKa's Most Wanted | 789 | 1990 |

But we can actually be a bit more clever by introducing a 3rd table
| artist_id | album_id |
| --- | -- |
| 123 | 11 |
| 123 | 22 |
| 789 | 22 |
| 456 | 22 |

# Data Manipulation Languages (DML)
How to store and receive information from a database.


**Procedural:**
* The query specifies the (hight-level) strategy
the DBMS should use to find the desired result.
  * Relational Algebra

**Non-Procedural:**
* The query specifies only what data is wanted and not how to find it.
  * Relational Calculus
  * This won't be as relevant in this course, but this is necessary for query
  optimization

# Relational Algebra
Fundamental operations to retrieve and manipulate tuples in a relation
* Based on set algebra

Each operator takes one or more relations as its inputs and outputs a new relation.
* We can "chain" operators together to create more complex operations.

These operators are: Select, Projection, Union, Intersection, Difference, Product, and
Join

## Select (σ)
Choose a subset of the tuples from a relation that satisfies a selection predicate
* Predicate acts as a filter to retain only tuples that fulfill its qualifying requirement
* Can combine multiple predicates using conjunctions / disjunctions

We say find the subset where each tuple meets the requirements in my predicate. In SQL
we would write something like this.
```SQL
SELECT * from R
  WHERE a_id='a2' AND b_id > 102;
```

## Projection (Π)
Generate a relation with tuples that contain only the specified attributes.
* Can rearrange attributes' ordering
* Can manipulate the values

Example in SQL
```SQL
SELECT b_id-100, a_id
  FROM R WHERE a_id='a2';
```

## Union (∪)
Generate a relation that contains all tuples that appear in either only one or both input
relations. Attributes must be identical! Otherwise the union doesn't make sense

Example in SQL
```SQL
(SELECT * FROM R)
    UNION ALL
(SELECT * FROM S);
```

## Intersection (∩)
Generate a relation that contains only the tuples that appear in both of the input
relations. Just like the union operator the attributes must be identical for the
intersection to be well defined.

Example in SQL
```SQL
(SELECT * FROM R)
    UNION ALL
(SELECT * FROM S);
```

## Difference (-)
Generate a relation that contains only the tuples that appear in the first and not the
second of the input relations. This is set subtraction. Give me the values in R that 
_don't_ lie in S.

Example in SQL
```SQL
(SELECT * FROM R)
    EXCEPT
(SELECT * FROM S);
```

## Product (x)
Generate a relation that contains all possible _combinations_ of tuples from the input
relations.

This is sometimes referred to as the cartesian product.

Example in SQL
```SQL
SELECT * FROM R CROSS JOIN S;

SELECT * FROM R, S;
```

## Join (⋈)
Generate a relation that contains all tuples that are a combination of two
tuples (one form each input relation) with a common value(s) for one or more 
attributes. This specific definition is called a _natural join_. Notice that unlike
the union operator we don't need to have identical attributes for this operation to
be well-defined.

Example in SQL
```SQL
SELECT * FROM R NATURAL JOIN S;
```

## Other Operators from Relational Algebra
* Rename (ρ)
* Assignment (R <- S)
* Duplicate Elimination (δ)
* Aggregation (γ)
* Sorting (τ)
* Division (÷)

# Observation
Relational algebra still defines the high-level steps of how to compute a query.
Select b_id=102 on (R join S) vs. (R join (Select b_id=102 on S)). If S is large and the
number of b_id=102 is small. Then the order of these two operations are significant
for performance.

A better approach is to state the high-level answer that you want the DBMS to compute.
* Retrieve the joined tuples from `R` and `S` where `b_id` equals `102`.

# Relational Model - Queries
The relational Model is independent of any query language implementation.

**SQL** is the _de facto_ standard.

# Conclusion
Databases are ubiquitous.

Relational algebra defines the primitives for processing queries on a relational database.

We will see relational algebra again when we talk about query optimization + execution.