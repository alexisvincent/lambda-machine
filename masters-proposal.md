# Graph Filters

## Background
For some years now I've been frustrated with the speed of development and eventual quality of modern user facing software applications. In my experience, only 20% of the code produced is novel, domain specific and non-trivial. By leveraging simpler and more powerful abstractions, I believe there is still significant space for automation in this class of application.

## Overview
Modern frontend applications have complex data requirements and should be able to convey these requirements precisely to the server. While we could directly expose the database query language to the client, this leaves our system open for unauthorized data access. Graph Filters aim to fix this problem by automatically augmenting database queries with the access control rules of the entity performing the query. This enables a large amount of potentially buggy, access control logic to be omitted, and the indirection between the database and the client to fall away.

Furthermore, Graph Filters provide a mechanism for precisely determining when database inserts should trigger query invalidations, significantly simplifying development of data efficient realtime clients.

## Reach
This project is motivated by the search for a practical solution to the described problem, as such, reach and feasibility of an implementation are primary concerns of the author. The techniques described in this paper, will focus on datalog as a query language, as provided by the Datomic database, but should be extendible to other graph databases, and even existing SQL installations.

## Presuppositions
This paper presupposes basic understanding of
- The EAV data model
- Datomic
- Datalog as implemented by Datomic
- Replicated Immutable Log's

## Graphs and Graph Queries
A Graph, is an ordered list/log of 4-tuples of the form `(:db/assert | :db/retract , entity, attribute, value)` called facts. Where `:db/assert` | `:db/retract` indicates whether the fact is being asserted or retracted. This structure is immutable, where modifications are simply additions to the log.

For example, the following facts would represent 3 products in the database

```clojure
[:db/assert 1 :product/name "iPhone 4"]
[:db/assert 1 :product/manufacturer "Apple"]
[:db/assert 1 :product/sku "SKU-23445"]
[:db/assert 1 :product/sales 5004]
[:db/assert 1 :product/released true]

[:db/assert 2 :product/name "iPhone 5"]
[:db/assert 2 :product/manufacturer "Apple"]
[:db/assert 2 :product/sku "SKU-23443"]
[:db/assert 2 :product/sales 20100]
[:db/assert 2 :product/released true]

[:db/assert 3 :product/name "iPhone 12"]
[:db/assert 3 :product/manufacturer "Apple"]
[:db/assert 3 :product/sku "SKU-23422"]
[:db/assert 3 :product/sales 0]
[:db/assert 3 :product/released false]
```

A query for the name of all products manufactured by Apple with less then 10000 sales would look like this:

```clojure
[:find ?name
 :where
 [?e :product/manufacturer "Apple"]
 [?e :product ?sales]
 [(< ?sales 10000)]]
```
 which would return the following result:
 ```clojure
 (["iPhone 4"]
  ["iPhone 12"])
 ```

Now imagine an entity is only allowed to see products from Apple if they have been released, we want to represent this as an access control rule such as
```clojure
([?product :product/manufacturer "Apple"]
 [?product :product/released true])
```

and for the query engine to merge this with our original query

```clojure
[:find ?name
 :where

 [(bind ?product ?e)]
 [?product :product/manufacturer "Apple"]
 [?product :product/released true]

 [?e :product/manufacturer "Apple"]
 [?e :product ?sales]
 [(< ?sales 10000)]]
```

which should then be optimized to simply

```clojure
[:find ?name
 :where
 [?e :product/released true]
 [?e :product/manufacturer "Apple"]
 [?e :product ?sales]
 [(< ?sales 10000)]]
```

### Graph Filters
A Graph Filter then, can be viewed as a filter applied over the original graph containing authorized data. More generally, a Graph Filter is a function `F: Graph -> Graph`, restricting access to certain facts, and the merge operation is the composition of filters, where queries are viewed as a special case of a filter.

The goal of this research is to provide a well defined formal calculus for performing this merge operation not just for this simple case, but for the full breadth of the query language defined by Datomic's datalog.

#### Realtime Queries
The same pattern matching system can be used to determine if a new database insert such as
```clojure
[:db/retract 3 :released false]
[:db/assert  3 :released true]
```
will invalidate the query, or any of the access control rules. This makes for simple implementation of realtime queries.

#### Access Control on Writes
Additionally, Graph Filters can also be used to validate writes. This is achieved by preemptively applying the write to a temporary write log and checking if new facts invalidate any access control rules. If so, the transaction is marked as invalid and rejected by the system.

## Why this is cool
- We can remove a lot of access control code from our code bases
- Access to data is formally controlled by a system and safer due to decreased bug surface area
- Operators can write complex access control rules without needing to write code
- The query engine can optimize queries by heavily caching query fragments
- Removes much of the N+1 problem
- Application clients can express complex data requirements by directly writing database queries
- A database client can be directly embedded in the application, abstracting away the hard distributed database problems

## Influences
This work is inspired by:
- Nikita Prokopov - The Web after Tomorrow
- Rich Hickey - The Value of Values; Datomic; Clojure; Hammock Driven Development
- Tim Berners-Lee - The Semantic Web
- Martin Kleppmann - Turning the database inside out with Apache Samza
