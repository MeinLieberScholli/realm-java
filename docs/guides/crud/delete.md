# CRUD - Delete - Java SDK
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

## Delete an Object
To delete an object from a realm, use either the dynamic or static
versions of the `deleteFromRealm()` method of a `RealmObject` subclass.

The following example shows how to delete one object from
its realm with `deleteFromRealm()`:

#### Java

```java
realm.executeTransaction(r -> {
    // Get a turtle named "Tony".
    Turtle tony = r.where(Turtle.class).equalTo("name", "Tony").findFirst();
    tony.deleteFromRealm();
    // discard the reference
    tony = null;
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    // Get a turtle named "Tony".
    var tony = r.where(Turtle::class.java)
        .equalTo("name", "Tony")
        .findFirst()
    tony!!.deleteFromRealm()
    // discard the reference
    tony = null
}

```

> Tip:
> The SDK throws an error if you try to use an object after
it has been deleted.
>

## Delete Multiple Objects
To delete an object from a realm, use the `deleteAllFromRealm()`
method of the `RealmResults`
instance that contains the objects you would like to delete. You can
filter the `RealmResults` down to a subset of objects using the
`where()` method.

The following example demonstrates how to delete a
collection from a realm with `deleteAllFromRealm()`:

#### Java

```java
realm.executeTransaction(r -> {
    // Find turtles older than 2 years old.
    RealmResults<Turtle> oldTurtles = r.where(Turtle.class).greaterThan("age", 2).findAll();
    oldTurtles.deleteAllFromRealm();
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    // Find turtles older than 2 years old.
    val oldTurtles = r.where(Turtle::class.java)
        .greaterThan("age", 2)
        .findAll()
    oldTurtles.deleteAllFromRealm()
}

```

## Delete an Object and its Dependent Objects
Sometimes, you have dependent objects that you want to delete when
you delete the parent object. We call this a **chaining
delete**. Realm does not delete the dependent
objects for you. If you do not delete the objects yourself,
they will remain orphaned in your realm. Whether or not
this is a problem depends on your application's needs.

Currently, the best way to delete dependent objects is to
iterate through the dependencies and delete them before
deleting the parent object.

The following example demonstrates how to perform a
chaining delete by first deleting all of Ali's turtles,
then deleting Ali:

#### Java

```java
realm.executeTransaction(r -> {
    // Find a turtle enthusiast named "Ali"
    TurtleEnthusiast ali = r.where(TurtleEnthusiast.class).equalTo("name", "Ali").findFirst();
    // Delete all of ali's turtles
    ali.getTurtles().deleteAllFromRealm();
    ali.deleteFromRealm();
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    // Find a turtle enthusiast named "Ali"
    val ali = r.where(TurtleEnthusiast::class.java)
        .equalTo("name", "Ali").findFirst()
    // Delete all of ali's turtles
    ali!!.turtles!!.deleteAllFromRealm()
    ali.deleteFromRealm()
}

```

## Delete All Objects of a Specific Type
Realm supports deleting all instances of a Realm type from a realm.

The following example demonstrates how to delete all
Turtle instances from a realm with `delete()`:

#### Java

```java
realm.executeTransaction(r -> {
    r.delete(Turtle.class);
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    r.delete(Turtle::class.java)
}

```

## Delete All Objects in a Realm
It is possible to delete all objects from the realm. This
does not affect the schema of the realm. This is useful for
quickly clearing out your realm while prototyping.

The following example demonstrates how to delete everything
from a realm with `deleteAll()`:

#### Java

```java
realm.executeTransaction(r -> {
    r.deleteAll();
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    r.deleteAll()
}

```

## Delete an Object Using an Iterator
Because realm collections always reflect the latest state, they
can appear, disappear, or change while you iterate over a collection.
To get a stable collection you can iterate over, you can create a
**snapshot** of a collection's data. A snapshot guarantees the order of
elements will not change, even if an element is deleted.

For an example, refer to Iteration.
