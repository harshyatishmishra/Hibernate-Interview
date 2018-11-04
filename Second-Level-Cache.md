_Hibernate second level cache_ uses a common cache for all the session object of a session factory. It is useful if you have multiple session objects from a session factory.

SessionFactory holds the second level cache data. It is global for all the session objects and not enabled by default.

Different vendors have provided the implementation of Second Level Cache.
 * EH Cache
 * OS Cache
 * Swarm Cache
 * JBoss Cache


1. Whenever hibernate session try to load an entity, the very first place it look for cached copy of entity in first level cache (associated with particular hibernate session).
2. If cached copy of entity is present in first level cache, it is returned as result of load method.
3. If there is no cached entity in first level cache, then second level cache is looked up for cached entity.
4. If second level cache has cached entity, it is returned as result of load method. But, before returning the entity, it is stored in first level cache also so that next invocation to load method for entity will return the entity from first level cache itself, and there will not be need to go to second level cache again.
5. If entity is not found in first level cache and second level cache also, then database query is executed and entity is stored in both cache levels, before returning as response of load() method.
6. Second level cache validate itself for modified entities, if modification has been done through hibernate session APIs.
7. If some user or process make changes directly in database, the there is no way that second level cache update itself until “timeToLiveSeconds” duration has passed for that cache region. In this case, it is good idea to invalidate whole cache and let hibernate build its cache once again.

```
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-ehcache</artifactId>
      <version>5.2.2.Final</version>
    </dependency>
```

```
    <properties>
        ...
        <property name="hibernate.cache.use_second_level_cache" value="true"/>
        <property name="hibernate.cache.region.factory_class"
          value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
        ...
    </properties>
```
```
    @Entity
    @Cacheable
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    @Table
    public class EntityClass {
```
For each entity class, Hibernate will use a separate cache region to store state of instances for that class.

#### Collections are not cached by default, and we need to explicitly mark them as cacheable. 
```
    @Cacheable
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    @OneToMany
    private Collection<Object> object;
```
#### Ehcache configuration to limit the maximum number of cached Foo instances to 1000:
```
    <ehcache>
        <cache name="org.EntityClass" maxElementsInMemory="1000" />
    </ehcache>
```
### Cache Concurrency Strategy

* _READ_ONLY_: Used only for entities that never change (exception is thrown if an attempt to update such an entity is made). It is very simple and performant. Very suitable for some static reference data that don’t change
* _NONSTRICT_READ_WRITE_: Cache is updated after a transaction that changed the affected data has been committed. Thus, strong consistency is not guaranteed and there is a small time window in which stale data may be obtained from cache. This kind of strategy is suitable for use cases that can tolerate eventual consistency
* _READ_WRITE_: This strategy guarantees strong consistency which it achieves by using ‘soft’ locks: When a cached entity is updated, a soft lock is stored in the cache for that entity as well, which is released after the transaction is committed. All concurrent transactions that access soft-locked entries will fetch the corresponding data directly from database
* _TRANSACTIONAL_: Cache changes are done in distributed XA transactions. A change in a cached entity is either committed or rolled back in both database and cache in the same XA transaction
