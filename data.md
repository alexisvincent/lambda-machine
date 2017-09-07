# Data
Information (data) forms a graph. Period. All attempts to hide this are going to cause issues. So any 'universal' data solution must embrace this graph nature.

## Structure of the graph
Any data structure or database you can dream of can be represented via entities and relations between these entities (Set Theoretic basis). 

When I say graph I mean a directed, potentially cyclic graph of entities (nodes) and relations (links). Relations are of the form `some.namespace.for.relation/name` and for an entity `e` and a value `v`, we assert that `e` is related to `v` via the statement `e some.namespace.for.relation/name v`.

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
`1 elementOf A`
`2 elementOf A`
`3 elementOf A`
`4 elementOf A`

### Ex 3 - Tree and Sorted List
For a simple family tree 2 levels deep with one grandparent we can construct the `childOf` relation and entities `grandfather father  daughter son`. And relate them:

`son childOf father`
`daughter childOf father`
`father childOf grandfather`

Note: A sorted list (represented as a linked list) is a special case of a tree.

### Ex 4 - Relational databases (see datomic database)
A (semi... kind of) formal definition is given below, but to set the stage, imagine two SQL tables 

User:
| id |  name  | address_id |
|:--:|:------:|:----------:|
| u1 | Alexis |     a1     |
| u2 | Julien |     a1     |

Address:
| id |            home            |
|:--:|:--------------------------:|
| a1 | 4 Private Drive, Somewhere |

We can naively represent this graph as follows: we construct the relations `user/id user/name user/address_id` and entities `eu1 eu2 ea1` and relate them as follows.
`eu1 user/id u1`
`eu1 user/name "Alexis"`
`eu1 user/address_id a1`

`eu2 user/id u2`
`eu2 user/name "Julien"`
`eu2 user/address_id a1`

`ea1 address/id a1`
`ea1 address/home "4 Private Drive, Somewhere"`

A better approch is to use the id's as the entity id, and to directly relate the user and address as follows: we construct the relations `user/id user/name user/address` (notice `user/address_id -> user/address`) and entities `u1 u2 a1` and relate them as follows.
`u1 user/name "Alexis"`
`u1 user/address a1`

`u2 user/name "Julien"`
`u2 user/address a1`

`a1 address/home "4 Private Drive, Somewhere"`

Below example isn't very well constructed, it started as a semi formal example, before I gave the above example.
Given a set of tables `T = {a, b, c}` with columns `C = {x, y, z}` for each table, and rows `Rt = {t[r1], t[r2], t[r3]}` for every table t. Where `(t[ri], c)` is the value of the row i of the table t at some column c.

Construct the relations/attributes: `T x C = {a/x, b/x, c/x, a/y, b/y, c/y, a/z, b/z, c/z}`.

Construct the entities: `t[ri]` for every row `ri` in every table `t`.

Relate the entities to values (or each other):
For every row `t[ri]` and every column `c`:
Let `t[ri]` be related via `t/c` to the value (t[ri], c).

In the case where (t[ri], c) is the id of a row t'[rj] in another table t', you can directly relate t[ri] to t'[rj] via t/c, embedding the semantic meaning of the id relations directly into the relation.



Additionally, when t is a join table for a many to many relation of two tables t and t' you can ignore this table and simply make the direct relation.

Thus a graph is actually a better (more simple) representation of the information then a SQL database. 

To add type information and uniqueness constrants etc, simply embue the relation with the information you require (via a relation of course).

Eg. The relation user/name, could have the following relations/attributes defined on it.
```
user/name  - db/cardinality ->  one     # every user has one name
user/name  -    db/unique   ->  false   # users can have the same name
user/name  -     db/type    ->  string  # a name is a string
```

### Note
This mapping from SQL databases to graph databases is (with standard schema) injective and homomorphic (some more consideration of all properties of SQL database semantics is required to make this claim). This suggests that there should also be an injective, homomorphic mapping from the SQL query language to a 'good' graph query language. And conversely a mapping from a subset of the graph query language back to SQL. This is nice, since it means that any solution built for a graph database should be (with obvious limitations) automatically extendible to existing SQL databases.

## Asking questions of data (i.e. queries)
When we make a 'database' query, the purpose is to extract some immeadiately useful, specific information, derived from the data graph. This derived information is itself a graph, under the view that a)

Fundementally, a query is a function q: data graph -> data graph. 
That is to say, it really is just a function on data. And so if we are being pedantic a powerful query language should actually be a fully featured programming language... The reason we call it a query Query is  (generally reduced). This 'question' is ***ALWAYS*** constraint/relational in nature.

i.e. 