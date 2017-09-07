# Data
This document attempts to take a first principals approach to the topic of data in computer systems and provides opinionated ways to think about data systems.

The main contentions are that:
- complex data models are relational. i.e. a list of user_id's is a representation of a list of users
- a query makes a demand for specific data in a specific form
- to effectively construct this specific form, an expressive query language must understand/embrace the relation graph

In other words, ***your data is a graph, deal with it!***.

## Structure of the graph
Any data structure or database you can dream of can be represented via entities and relations between these entities (Set Theoretic basis). 

When I say graph I mean a directed, potentially cyclic graph of entities (nodes) and relations (links). Relations are of the form `some.namespace.for.relation/rname` or simply `rname` and for an entity `e` and a value `v`, we assert that `e` is related to `v` via the statement `e some.namespace.for.relation/name v`.

To convince you of this claim I've provided a few examples.

### Ex 1 - Scalar Values
The scalar values `{2.4, "two", 't', symbol, 04/04/95}` are just single entity graphs with no relations defined. Except perhaps `identity`.

### Ex 2 - Set (unsorted List)
For the list A:

| A |
|:-:|
| 1 |
| 2 |
| 3 |
| 4 |

We can represent this graph by constructing the relations `elementOf` and entities `A 1 2 3 4` and relating them as follows.
```
1 elementOf A
2 elementOf A
3 elementOf A
4 elementOf A
```

### Ex 3 - Tree and Sorted List
For a simple family tree 2 levels deep with one grandparent we can construct the `childOf` relation and entities `grandfather father daughter son`. And relating them:

```
son childOf father
daughter childOf father
father childOf grandfather
```

*Note: A sorted list (represented as a linked list) is a special case of a tree.*

### Ex 4 - Relational databases (see datomic database)
Given two SQL tables.

User:

| id |  name  | address_id |
|:--:|:------:|:----------:|
| u1 | Alexis |     a1     |
| u2 | Julien |     a1     |

Address:

| id |            home            |
|:--:|:--------------------------:|
| a1 | 4 Private Drive, Somewhere |

We can naively represent this graph by constructing the relations `user/id user/name user/address_id` and entities `eu1 eu2 ea1` and relating them as follows.
```
eu1 user/id u1
eu1 user/name "Alexis"
eu1 user/address_id a1

eu2 user/id u2
eu2 user/name "Julien"
eu2 user/address_id a1

ea1 address/id a1
ea1 address/home "4 Private Drive, Somewhere"
```

A better approch is to directly relate the user and address by constructing the relations `user/id user/name user/address` (notice `user/address_id -> user/address`) and to use the id's as the entity id `u1 u2 a1` and then relating them as follows:
```
u1 user/name "Alexis"
u1 user/address a1

u2 user/name "Julien"
u2 user/address a1

a1 address/home "4 Private Drive, Somewhere"
```

Additionally, when a table is a join table for a many to many relation of two tables `t` and `t'` you can ignore this table and simply make the direct relation between the entities from `t` and `t'`.

To add type information and uniqueness constrants etc, simply embue the relation with the information you require (via a relation of course).

Eg. The relation `user/name`, could have the following relations/attributes defined on it.
```
user/name  db/cardinality one     # every user has one name
user/name  db/unique      false   # users can have the same name
user/name  db/type        string  # a name is a string
```

This mapping from SQL databases to graph databases is (with standard schema) injective and homomorphic (some more consideration of all properties of SQL database semantics is required to make this claim). This suggests that there should also be an injective, homomorphic mapping from the SQL query language to a **good** graph query language. And conversely a mapping from a subset of the graph query language back to SQL. This is nice, since it means that any solution built for a graph database should be (with obvious limitations) automatically extendible to existing SQL databases.

## Asking questions of data (i.e. queries)
When we make a 'database' query, the purpose is to extract some immeadiately useful, specific information, derived from the data graph. Let's explore this a bit.

### Ex 1 - Chat Window
Imagine we have a chat window between two parties. Upon load we want to display to the user the following *compound* information:
- their name
- name of person they're talking to
- list of messages

What we would like to happen is for the data graph that contains this information to run through a transformation the gives us specifically the information above. Additionally we might have some UI component that expects to recieve the data in some structure, meaning the transformation should reduce the data graph to a specific form. This transformation then is then the underlying query.

### Ex 2 - Analytics
Imagine a music store owner wants to find out how many **Rolling Stones** albums have been sold in the past month. The answer in this case might simply be `7`.

### Give me what I want, I don't care how you get it
While these examples differ in terms of complexity, they share a purpose. In both cases we have an entity that needs some very specific information, in a very specific structure, which resides in some data graph. Neither cares (to some degree) the specific process taken to obtain or construct the information.

The query is then simply a description of the data needs of the entity requesting it and the *easier* it is to describe this query the better.

Under this conceptual understanding of a query, a query is nothing more then a function `q: data graph -> immeadiately usefull data graph`. And so our query language, whatever it may be, would ideally be able to precicely describe our data needs.

