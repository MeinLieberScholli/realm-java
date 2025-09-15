# CRUD - Create - Java SDK
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

## Create a New Object
Use `realm.createObject()`
in a transaction to create a persistent instance of a Realm object in a
realm. You can then modify the returned object with other field values
using accessors and mutators.

The following example demonstrates how to create an object with
`createObject()`:

#### Java

```java
realm.executeTransaction(r -> {
    // Instantiate the class using the factory function.
    Turtle turtle = r.createObject(Turtle.class, new ObjectId());
    // Configure the instance.
    turtle.setName("Max");
    // Create a TurtleEnthusiast with a primary key.
    ObjectId primaryKeyValue = new ObjectId();
    TurtleEnthusiast turtleEnthusiast = r.createObject(TurtleEnthusiast.class, primaryKeyValue);
});

```

#### Kotlin

```kotlin
realm.executeTransaction { r: Realm ->
    // Instantiate the class using the factory function.
    val turtle = r.createObject(Turtle::class.java, ObjectId())
    // Configure the instance.
    turtle.name = "Max"
    // Create a TurtleEnthusiast with a primary key.
    val primaryKeyValue = ObjectId()
    val turtleEnthusiast = r.createObject(
        TurtleEnthusiast::class.java,
        primaryKeyValue
    )
}

```

You can also insert objects into a realm from JSON. Realm
supports creating objects from `String`,
[JSONObject](https://developer.android.com/reference/org/json/JSONObject.html), and
[InputStream](https://developer.android.com/reference/java/io/InputStream.html) types.
Realm ignores any properties present in the JSON that are
not defined in the Realm object schema.

The following example demonstrates how to create a single object from JSON with
`createObjectFromJson()`
or multiple objects from JSON with
`createAllFromJson()`:

#### Java

```java
// Insert from a string
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        realm.createObjectFromJson(Frog.class,
                "{ name: \"Doctor Cucumber\", age: 1, species: \"bullfrog\", owner: \"Wirt\" }");
    }
});

// Insert multiple items using an InputStream
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        try {
            InputStream inputStream = new FileInputStream(
                    new File("path_to_file"));
            realm.createAllFromJson(Frog.class, inputStream);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
});

```

#### Kotlin

```kotlin
// Insert from a string
realm.executeTransaction { realm ->
    realm.createObjectFromJson(
        Frog::class.java,
        "{ name: \"Doctor Cucumber\", age: 1, species: \"bullfrog\", owner: \"Wirt\" }"
    )
}

// Insert multiple items using an InputStream
realm.executeTransaction { realm ->
    try {
        val inputStream: InputStream =
            FileInputStream(File("path_to_file"))
        realm.createAllFromJson(Frog::class.java, inputStream)
    } catch (e: IOException) {
        throw RuntimeException(e)
    }
}

```

