# CRUD - Update - Java SDK
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

## Modify an Object
Within a transaction, you can update a Realm object the same
way you would update any other object in your language of
choice. Just assign a new value to the property or update
the property.

The following example changes the turtle's name to "Archibald" and
sets Archibald's age to 101 by assigning new values to properties:

#### Java

```java
realm.executeTransaction(r -> {
    // Get a turtle to update.
    Turtle turtle = r.where(Turtle.class).findFirst();
    // Update properties on the instance.
    // This change is saved to the realm.
    turtle.setName("Archibald");
    turtle.setAge(101);
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    // Get a turtle to update.
    val turtle = r.where(Turtle::class.java).findFirst()
    // Update properties on the instance.
    // This change is saved to the realm.
    turtle!!.name = "Archibald"
    turtle.age = 101
}

```

## Upsert an Object
An **upsert** is a write operation that either inserts a new object
with a given primary key or updates an existing object that already has
that primary key. We call this an upsert because it is an "**update** or
**insert**" operation. This is useful when an object may or may not
already exist, such as when bulk importing a dataset into an existing
realm. Upserting is an elegant way to update existing entries while
adding any new entries.

The following example demonstrates how to upsert an object with
realm. We create a new turtle enthusiast named "Drew" and then
update their name to "Andy" using `insertOrUpdate()`:

#### Java

```java
realm.executeTransaction(r -> {
    ObjectId id = new ObjectId();
    TurtleEnthusiast drew = new TurtleEnthusiast();
    drew.set_id(id);
    drew.setName("Drew");
    drew.setAge(25);
    // Add a new turtle enthusiast to the realm. Since nobody with this id
    // has been added yet, this adds the instance to the realm.
    r.insertOrUpdate(drew);
    TurtleEnthusiast andy = new TurtleEnthusiast();
    andy.set_id(id);
    andy.setName("Andy");
    andy.setAge(56);
    // Judging by the ID, it's the same turtle enthusiast, just with a different name.
    // As a result, you overwrite the original entry, renaming "Drew" to "Andy".
    r.insertOrUpdate(andy);
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    val id = ObjectId()
    val drew = TurtleEnthusiast()
    drew._id = id
    drew.name = "Drew"
    drew.age = 25
    // Add a new turtle enthusiast to the realm. Since nobody with this id
    // has been added yet, this adds the instance to the realm.
    r.insertOrUpdate(drew)
    val andy = TurtleEnthusiast()
    andy._id = id
    andy.name = "Andy"
    andy.age = 56
    // Judging by the ID, it's the same turtle enthusiast, just with a different name.
    // As a result, you overwrite the original entry, renaming "Drew" to "Andy".
    r.insertOrUpdate(andy)
}

```

You can also use `copyToRealmOrUpdate()` to
either create a new object based on a supplied object or update an
existing object with the same primary key value. Use the
`CHECK_SAME_VALUES_BEFORE_SET`
`ImportFlag` to only update fields
that are different in the supplied object:

The following example demonstrates how to insert an object or, if an object already
exists with the same primary key, update only those fields that differ:

#### Java

```java
realm.executeTransaction(r -> {
    ObjectId id = new ObjectId();
    TurtleEnthusiast drew = new TurtleEnthusiast();
    drew.set_id(id);
    drew.setName("Drew");
    drew.setAge(25);
    // Add a new turtle enthusiast to the realm. Since nobody with this id
    // has been added yet, this adds the instance to the realm.
    r.insertOrUpdate(drew);
    TurtleEnthusiast andy = new TurtleEnthusiast();
    andy.set_id(id);
    andy.setName("Andy");
    // Judging by the ID, it's the same turtle enthusiast, just with a different name.
    // As a result, you overwrite the original entry, renaming "Drew" to "Andy".
    // the flag passed ensures that we only write the updated name field to the db
    r.copyToRealmOrUpdate(andy, ImportFlag.CHECK_SAME_VALUES_BEFORE_SET);
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    val id = ObjectId()
    val drew = TurtleEnthusiast()
    drew._id = id
    drew.name = "Drew"
    drew.age = 25
    // Add a new turtle enthusiast to the realm. Since nobody with this id
    // has been added yet, this adds the instance to the realm.
    r.insertOrUpdate(drew)
    val andy = TurtleEnthusiast()
    andy._id = id
    andy.name = "Andy"
    // Judging by the ID, it's the same turtle enthusiast, just with a different name.
    // As a result, you overwrite the original entry, renaming "Drew" to "Andy".
    r.copyToRealmOrUpdate(andy,
        ImportFlag.CHECK_SAME_VALUES_BEFORE_SET)
}

```

## Update a Collection
Realm supports collection-wide updates. A collection update
applies the same update to specific properties of several
objects in a collection at once.

The following example demonstrates how to update a
collection. Thanks to the implicit inverse
relationship between the Turtle's
`owner` property and the TurtleEnthusiast's `turtles` property,
Realm automatically updates Josephine's list of turtles
when you use `setObject()`
to update the "owner" property for all turtles in the collection.

#### Java

```java
realm.executeTransaction(r -> {
    // Create a turtle enthusiast named Josephine.
    TurtleEnthusiast josephine = r.createObject(TurtleEnthusiast.class, new ObjectId());
    josephine.setName("Josephine");

    // Get all turtles named "Pierogi".
    RealmResults<Turtle> turtles = r.where(Turtle.class).equalTo("name", "Pierogi").findAll();

    // Give all turtles named "Pierogi" to Josephine
    turtles.setObject("owner", josephine);
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    // Create a turtle enthusiast named Josephine.
    val josephine = realm.createObject(
        TurtleEnthusiast::class.java,
        ObjectId()
    )
    josephine.name = "Josephine"

    // Get all turtles named "Pierogi".
    val turtles = r.where(Turtle::class.java)
        .equalTo("name", "Pierogi")
        .findAll()

    // Give all turtles named "Pierogi" to Josephine
    turtles.setObject("owner", josephine)
}

```

## Iteration
Because realm collections always reflect the latest state, they
can appear, disappear, or change while you iterate over a collection.
To get a stable collection you can iterate over, you can create a
**snapshot** of a collection's data. A snapshot guarantees the order of
elements will not change, even if an element is deleted or modified.

`Iterator` objects created from `RealmResults` use snapshots
automatically. `Iterator` objects created from `RealmList`
instances do *not* use snapshots. Use
`RealmList.createSnapshot()`
or
`RealmResults.createSnapshot()`
to manually generate a snapshot you can iterate over manually:

The following example demonstrates how to iterate over a collection
safely using either an implicit snapshot created from a `RealmResults`
`Iterator` or a manual snapshot created from a `RealmList`:

#### Java

```java
RealmResults<Frog> frogs = realm.where(Frog.class)
        .equalTo("species", "bullfrog")
        .findAll();

// Use an iterator to rename the species of all bullfrogs
realm.executeTransaction(r -> {
    for (Frog frog : frogs) {
        frog.setSpecies("Lithobates catesbeiana");
    }
});

// Use a snapshot to rename the species of all bullfrogs
realm.executeTransaction(r -> {
    OrderedRealmCollectionSnapshot<Frog> frogsSnapshot = frogs.createSnapshot();
    for (int i = 0; i < frogsSnapshot.size(); i++) {
        frogsSnapshot.get(i).setSpecies("Lithobates catesbeiana");
    }
});

```

#### Kotlin

```kotlin
val frogs = realm.where(Frog::class.java)
    .equalTo("species", "bullfrog")
    .findAll()

// Use an iterator to rename the species of all bullfrogs
realm.executeTransaction {
    for (frog in frogs) {
        frog.species = "Lithobates catesbeiana"
    }
}

// Use a snapshot to rename the species of all bullfrogs
realm.executeTransaction {
    val frogsSnapshot = frogs.createSnapshot()
    for (i in frogsSnapshot.indices) {
        frogsSnapshot[i]!!.species = "Lithobates catesbeiana"
    }
}

```

