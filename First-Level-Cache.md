Caching is nothing but some buffer where a record is stored when first time retrieved from the database. When second time needed the same record, Hibernate does not access the database and instead reads from the cache. This type of adjustment decreases the database hits. This is what Cache Hibernate is. Accessing cache is much faster than accessing the database.

1. First level cache is _associated with “session” object_ and _other session objects in application can not see it_.
2. The _scope of cache objects is of session. Once session is closed, cached objects are gone forever._
3. _First level cache is enabled by default and you can not disable it._
4. When we query an entity first time, it is retrieved from database and stored in first level cache associated with hibernate session.
5. If we query same object again with same session object, it will be loaded from cache and no sql query will be executed.
6. The loaded entity can be removed from session using _evict()_ method. The next loading of this entity will again make a database call if it has been removed using evict() method.
7. The whole session cache can be removed using _clear()_ method. It will remove all the entities stored in cache.

```
  //Open the hibernate session
    Session session = HibernateUtil.getSessionFactory().openSession();
    session.beginTransaction();

    Session sessionTemp = HibernateUtil.getSessionFactory().openSession();
    sessionTemp.beginTransaction();
    try
    {
        //fetch the department entity from database first time
        DepartmentEntity department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
        System.out.println(department.getName());

        //fetch the department entity again
        department = (DepartmentEntity) session.load(DepartmentEntity.class, new Integer(1));
        System.out.println(department.getName());

        department = (DepartmentEntity) sessionTemp.load(DepartmentEntity.class, new Integer(1));
        System.out.println(department.getName());
    }
    finally
    {
        session.getTransaction().commit();
        HibernateUtil.shutdown();

        sessionTemp.getTransaction().commit();
        HibernateUtil.shutdown();
    }

    Output:

    Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
    Human Resource
    Human Resource

    Hibernate: select department0_.ID as ID0_0_, department0_.NAME as NAME0_0_ from DEPARTMENT department0_ where department0_.ID=?
    Human Resource
```



### Order
If an entity or object is loaded by calling the get() method then Hibernate first checked the first level cache, if it doesn't found the object then it goes to the second level cache if configured. If the object is not found then it finally goes to the database and returns the object, if there is no corresponding row in the table then it return null. When an object is loaded from the database is put on both second level and first level cache, so that other session who request the same object can now get it from the second level cache.

In case if the object is not found in the first level cache but found in the second level cache because some other sessions have loaded the object before then it is not only returned from first level cache but also cached at first level cache, so that next time if your code request the same object, it should be returned from 1st level cache rather than going to the 2nd level cache
