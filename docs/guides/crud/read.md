# CRUD - Read - Java SDK
## Read Operations
You can read back the data that you have
stored in Realm.
The standard data access pattern across Realm
SDKs is to find, filter, and sort objects, in that order. To
get the best performance from Realm as your app grows and
your queries become more complex, design your app's data
access patterns around a solid understanding of Realm
read characteristics.

### Read Characteristics
When you design your app's data access patterns around the
following three key characteristics of reads in Realm,
you can be confident you are reading data as
efficiently as possible.

### Results Are Not Copies
Results to a query are not copies of your data: modifying
the results of a query will modify the data on disk
directly. This memory mapping also means that results are
**live**: that is, they always reflect the current state on
disk.

### Results Are Lazy
Realm defers execution of a query until you access the
results. You can chain several filter and sort operations
without requiring extra work to process the intermediate
state.

### References Are Retained
One benefit of Realm's object model is that
Realm automatically retains all of an object's
relationships as
direct references, so you can traverse your graph of
relationships directly through the results of a query.

A **direct reference**, or pointer, allows you to access a
related object's properties directly through the reference.

Other databases typically copy objects from database storage
into application memory when you need to work with them
directly. Because application objects contain direct
references, you are left with a choice: copy the object
referred to by each direct reference out of the database in
case it's needed, or just copy the foreign key for each
object and query for the object with that key if it's
accessed. If you choose to copy referenced objects into
application memory, you can use up a lot of resources for
objects that are never accessed, but if you choose to only
copy the foreign key, referenced object lookups can cause
your application to slow down.

Realm bypasses all of this using zero-copy
live objects. Realm object accessors point directly into
database storage using memory mapping, so there is no distinction
between the objects in Realm and the results of your query in
application memory. Because of this, you can traverse direct references
across an entire realm from any query result.

## About the Examples on this Page
The examples on this page use the data model of a project
management app that has two Realm object types: `Project`
and `Task`. A `Project` has zero or more `Tasks`.

See the schema for these two classes, `Project` and
`Task`, below:

#### Java

```java

import org.bson.types.ObjectId;

import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.RealmClass;
import io.realm.annotations.Required;

public class ProjectTask extends RealmObject {
    @PrimaryKey
    public ObjectId _id;
    @Required
    public String name;
    public String assignee;
    public int progressMinutes;
    public boolean isComplete;
    public int priority;
    @Required
    public String _partition;
}

```

```java

import org.bson.types.ObjectId;

import io.realm.RealmList;
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.RealmClass;
import io.realm.annotations.Required;

public class Project extends RealmObject {
    @PrimaryKey
    public ObjectId _id;
    @Required
    public String name;
    public RealmList<ProjectTask> tasks = new RealmList<>();
}

```

#### Kotlin

```kotlin
import io.realm.RealmObject
import io.realm.annotations.PrimaryKey
import io.realm.annotations.Required
import org.bson.types.ObjectId

open class ProjectTask(
    @PrimaryKey
    var _id: ObjectId = ObjectId(),
    @Required
    var name: String = "",
    var assignee: String? = null,
    var progressMinutes: Int = 0,
    var isComplete: Boolean = false,
    var priority: Int = 0,
    var _partition: String = ""
): RealmObject()

```

```kotlin
import io.realm.RealmList
import io.realm.RealmObject
import io.realm.annotations.PrimaryKey
import io.realm.annotations.Required
import org.bson.types.ObjectId

open class Project(
    @PrimaryKey
    var _id: ObjectId = ObjectId(),
    @Required
    var name: String = "",
    var tasks: RealmList<ProjectTask> = RealmList(),
): RealmObject()

```

## Read from Realm
A read from a realm generally consists of the following
steps:

- Get all objects of a certain type from the realm.
- Optionally, filter the results using the query engine.
- Optionally, sort the results.

All query, filter, and sort operations return a
results collection. The results
collections are live, meaning they always contain the latest
results of the associated query.

> Important:
> By default, you can only read or write to a realm in your
application's UI thread using
asynchronous transactions. That is,
you can only use `Realm` methods whose name ends with the word
`Async` in the main thread of your Android application unless you
explicitly allow the use of synchronous methods.
>
> This restriction exists for the benefit of your application users:
performing read and write operations on the UI thread can lead to
unresponsive or slow UI interactions, so it's usually best to handle
these operations either asynchronously or in a background thread.

### Find a Specific Object by Primary Key
To find an object with a specific primary key value, open a realm
and query the primary key field for the desired primary key value
using the `RealmQuery.equalTo()` method:

#### Java

```java
ProjectTask task = realm.where(ProjectTask.class).equalTo("_id", PRIMARY_KEY_VALUE.get()).findFirst();
Log.v("EXAMPLE", "Fetched object by primary key: " + task);

```

#### Kotlin

```kotlin
val task = realm.where(ProjectTask::class.java)
    .equalTo("_id", ObjectId.get()).findFirst()
Log.v("EXAMPLE", "Fetched object by primary key: $task")

```

### Query All Objects of a Given Type
The first step of any read is to **get all objects** of a
certain type in a realm. With this results collection, you
can operate on all instances on a type or filter and sort to
refine the results.

In order to access all instances of `ProjectTask` and `Project`, use
the `where()` method
to specify a class:

#### Java

```java
RealmQuery<ProjectTask> tasksQuery = realm.where(ProjectTask.class);
RealmQuery<Project> projectsQuery = realm.where(Project.class);

```

#### Kotlin

```kotlin
val tasksQuery = realm.where(ProjectTask::class.java)
val projectsQuery = realm.where(Project::class.java)

```

### Filter Queries Based on Object Properties
A **filter** selects a subset of results based on the
value(s) of one or more object properties. Realm provides a
full-featured query engine you
can use to define filters. The most common use case is to
find objects where a certain property matches a certain
value. Additionally, you can compare strings, aggregate over
collections of numbers, and use logical operators to build
up complex queries.

In the following example, we use the query
engine's comparison operators to:

- Find high priority tasks by comparing the value of the `priority` property value with a threshold number, above which priority can be considered high.
- Find just-started or short-running tasks by seeing if the `progressMinutes` property falls within a certain range.
- Find unassigned tasks by finding tasks where the `assignee` property is equal to null.
- Find tasks assigned to specific teammates Ali or Jamie by seeing if the `assignee` property is in a list of names.

#### Java

```java
RealmQuery<ProjectTask> tasksQuery = realm.where(ProjectTask.class);
Log.i("EXAMPLE", "High priority tasks: " + tasksQuery.greaterThan("priority", 5).count());
Log.i("EXAMPLE", "Just-started or short tasks: " + tasksQuery.between("progressMinutes", 1, 10).count());
Log.i("EXAMPLE", "Unassigned tasks: " + tasksQuery.isNull("assignee").count());
Log.i("EXAMPLE", "Ali or Jamie's tasks: " + tasksQuery.in("assignee", new String[]{"Ali", "Jamie"}).count());

```

#### Kotlin

```kotlin
val tasksQuery = realm.where(ProjectTask::class.java)
Log.i(
    "EXAMPLE", "High priority tasks: " + tasksQuery.greaterThan(
        "priority",
        5
    ).count()
)
Log.i(
    "EXAMPLE", "Just-started or short tasks: " + tasksQuery.between(
        "progressMinutes",
        1,
        10
    ).count()
)
Log.i(
    "EXAMPLE",
    "Unassigned tasks: " + tasksQuery.isNull("assignee").count()
)
Log.i(
    "EXAMPLE", "Ali or Jamie's tasks: " + tasksQuery.`in`(
        "assignee", arrayOf(
            "Ali",
            "Jamie"
        )
    ).count()
)

```

### Sort Query Results
A **sort** operation allows you to configure the order in
which Realm returns queried objects. You can sort based on
one or more properties of the objects in the results
collection.

Realm only guarantees a consistent order of results when the
results are sorted.

The following code sorts the projects by name in reverse
alphabetical order (i.e. "descending" order).

#### Java

```java
RealmQuery<Project> projectsQuery = realm.where(Project.class);
RealmResults<Project> results = projectsQuery.sort("name", Sort.DESCENDING).findAll();

```

#### Kotlin

```kotlin
val projectsQuery = realm.where(Project::class.java)
val results = projectsQuery.sort("name", Sort.DESCENDING).findAll()

```

### Query a Relationship
#### Java

Consider the following relationship between classes `Human` and
`Cat`. This arrangement allows each human to own a single cat:

```java
import org.bson.types.ObjectId;

import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;

public class Human extends RealmObject {
    @PrimaryKey
    private ObjectId _id = new ObjectId();
    private String name;
    private Cat cat; 

    public Human(String name) {
        this.name = name;
    }

    public Human() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public ObjectId get_id() {
        return _id;
    }
}

```

```java
import org.bson.types.ObjectId;

import io.realm.RealmObject;
import io.realm.RealmResults;
import io.realm.annotations.LinkingObjects;
import io.realm.annotations.PrimaryKey;

public class Cat extends RealmObject {
    @PrimaryKey
    private ObjectId _id = new ObjectId();
    private String name = null;
    @LinkingObjects("cat") 
    private final RealmResults<Human> owner = null; 
    public Cat(String name) {
        this.name = name;
    }
    public Cat() {
    }

    public ObjectId get_id() {
        return _id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public RealmResults<Human> getOwner() {
        return owner;
    }
}

```

To query this relationship, use dot notation in a
query to access any property
of the linked object.

#### Kotlin

Consider the following relationship between classes `Person` and
`Dog`. This arrangement allows each person to own a single dog:

```kotlin
import io.realm.RealmObject
import io.realm.annotations.PrimaryKey
import org.bson.types.ObjectId

open class Person(var name : String? = null) : RealmObject() {
    @PrimaryKey
    var _id : ObjectId = ObjectId()
    var dog: Dog? = null 
}

```

```kotlin
import io.realm.RealmObject
import io.realm.RealmResults
import io.realm.annotations.LinkingObjects
import io.realm.annotations.PrimaryKey
import org.bson.types.ObjectId

open class Dog(var name : String? = null): RealmObject() {
    @PrimaryKey
    var _id : ObjectId = ObjectId()
    @LinkingObjects("dog") 
    val owner: RealmResults<Person>? = null 
}

```

To query this relationship, use dot notation in a
query to access any property
of the linked object.

### Query an Inverse Relationship
#### Java

Consider the following relationship between classes `Cat` and
`Human`. In this example, all cats link to their human (or
multiple humans, if multiple human objects refer to the same cat).
Realm calculates the owners of each cat for you based on the field
name you provide to the `@LinkingObjects` annotation:

```java
import org.bson.types.ObjectId;

import io.realm.RealmObject;
import io.realm.RealmResults;
import io.realm.annotations.LinkingObjects;
import io.realm.annotations.PrimaryKey;

public class Cat extends RealmObject {
    @PrimaryKey
    private ObjectId _id = new ObjectId();
    private String name = null;
    @LinkingObjects("cat")
    private final RealmResults<Human> owner = null;
    public Cat(String name) {
        this.name = name;
    }
    public Cat() {
    }

    public ObjectId get_id() {
        return _id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public RealmResults<Human> getOwner() {
        return owner;
    }
}
```

```java
import org.bson.types.ObjectId;

import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;

public class Human extends RealmObject {
    @PrimaryKey
    private ObjectId _id = new ObjectId();
    private String name;
    private Cat cat;

    public Human(String name) {
        this.name = name;
    }

    public Human() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Cat getCat() {
        return cat;
    }

    public void setCat(Cat cat) {
        this.cat = cat;
    }

    public ObjectId get_id() {
        return _id;
    }
}
```

To query this relationship, use dot notation in a
query to access any property
of the linked object.

#### Kotlin

Consider the following relationship between classes `Dog` and
`Person`. In this example, all dogs link to their owner (or
multiple owners, if multiple person objects refer to the same dog).
Realm calculates the owners of each dog for you based on the field
name you provide to the `@LinkingObjects` annotation:

```kotlin
import io.realm.RealmObject
import io.realm.RealmResults
import io.realm.annotations.LinkingObjects
import io.realm.annotations.PrimaryKey
import org.bson.types.ObjectId

open class Dog(var name : String? = null): RealmObject() {
    @PrimaryKey
    var _id : ObjectId = ObjectId()
    @LinkingObjects("dog")
    val owner: RealmResults<Person>? = null
}
```

```kotlin
import io.realm.RealmObject
import io.realm.annotations.PrimaryKey
import org.bson.types.ObjectId

open class Person(var name : String? = null) : RealmObject() {
    @PrimaryKey
    var _id : ObjectId = ObjectId()
    var dog: Dog? = null
}
```

To query this relationship, use dot notation in a
query to access any property
of the linked object.

### Aggregate Data
#### Java

```java
RealmQuery<ProjectTask> tasksQuery = realm.where(ProjectTask.class);
/*
Aggregate operators do not support dot-notation, so you
cannot directly operate on a property of all of the objects
in a collection property.

You can operate on a numeric property of the top-level
object, however:
*/
Log.i("EXAMPLE", "Tasks average priority: " + tasksQuery.average("priority"));

```

#### Kotlin

```kotlin
val tasksQuery = realm.where(ProjectTask::class.java)
/*
Aggregate operators do not support dot-notation, so you
cannot directly operate on a property of all of the objects
in a collection property.

You can operate on a numeric property of the top-level
object, however:
*/Log.i("EXAMPLE", "Tasks average priority: " + tasksQuery.average("priority"))

```

