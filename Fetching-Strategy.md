Hibernate allows you to choose among four fetching strategies for any association,
in association metadata and at runtime:
■ Immediate fetching—The associated object is fetched immediately, using a
sequential database read (or cache lookup).
■ Lazy fetching—The associated object or collection is fetched “lazily,” when
it’s first accessed. This results in a new request to the database (unless the
associated object is cached).
■ Eager fetching—The associated object or collection is fetched together with
the owning object, using an SQL outer join, and no further database request
is required.
■ Batch fetching—This approach may be used to improve the performance of
lazy fetching by retrieving a batch of objects or collections when a lazy association
is accessed. (Batch fetching may also be used to improve the performance
of immediate fetching.)
Let’s look more closely at each fetching strategy.
Immediate fetching
Immediate association fetching occurs when you retrieve an entity from the database
and then immediately retrieve another associated entity or entities in a further
request to the database or cache. Immediate fetching isn’t usually an efficient
fetching strategy unless you expect the associated entities to almost always be
cached already
Lazy fetching
When a client requests an entity and its associated graph of objects from the database,
it isn’t usually necessary to retrieve the whole graph of every (indirectly) associated
object. You wouldn’t want to load the whole database into memory at once;
for example, loading a single Category shouldn’t trigger the loading of all Items in
that category. 
Lazy fetching lets you decide how much of the object graph is loaded in the first
database hit and which associations should be loaded only when they’re first
accessed. Lazy fetching is a foundational concept in object persistence and the
first step to attaining acceptable performance.
We recommend that, to start with, all associations be configured for lazy (or perhaps
batched lazy) fetching in the mapping file. This strategy may then be overridden
at runtime by queries that force eager fetching to occur.
Eager (outer join) fetching
Lazy association fetching can help reduce database load and is often a good
default strategy. However, it’s a bit like a blind guess as far as performance optimization
goes.
Eager fetching lets you explicitly specify which associated objects should be loaded
together with the referencing object. Hibernate can then return the associated
objects in a single database request, utilizing an SQL OUTER JOIN. Performance optimization
in Hibernate often involves judicious use of eager fetching for particular
transactions. Hence, even though default eager fetching may be declared in the
mapping file, it’s more common to specify the use of this strategy at runtime for a
particular HQL or criteria query. 
Batch fetching
Batch fetching isn’t strictly an association fetching strategy; it’s a technique that may
help improve the performance of lazy (or immediate) fetching. Usually, when you
load an object or collection, your SQL WHERE clause specifies the identifier of the
object or object that owns the collection. If batch fetching is enabled, Hibernate
looks to see what other proxied instances or uninitialized collections are referenced
in the current session and tries to load them at the same time by specifying
multiple identifier values in the WHERE clause.
We aren’t great fans of this approach; eager fetching is almost always faster.
Batch fetching is useful for inexperienced users who wish to achieve acceptable
performance in Hibernate without having to think too hard about the SQL that will
be executed. (Note that batch fetching may be familiar to you, since it’s used by
many EJB2 engines.)
