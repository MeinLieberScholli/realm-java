# Relationships - Java SDK
## Relationships
Realm allows you to define explicit relationships between the types of
objects in an App. A relationship is an object property that references
another Realm object rather than one of the primitive data types. You
define relationships by setting the type of an object property to
another object type in the property schema.

Relationships are direct references to other objects in a realm, which
means that you don't need bridge tables or explicit joins to define a
relationship like you would in a relational database. Instead you can
access related objects by reading and writing to the property that
defines the relationship. Realm executes read operations
lazily as they come in, so querying a relationship is just as performant
as reading a regular property.

There are three primary types of relationships between objects:

- One-to-One Relationship
- One-to-Many Relationship
- Inverse Relationship

You can define relationships, collections, and embedded objects in your
object schema using the following types:

- `RealmObject`
- `RealmList <? extends RealmObject>`

Use annotations to indicate whether a given field represents a foreign
key relationship or an embedded object relationship. For more
information, see Relationship Annotations.

### To-One Relationship
A **to-one** relationship means that an object is related in a specific
way to no more than one other object. You define a to-one relationship
for an object type in its object schema by
specifying a property where the type is the related Realm object type.

Setting a relationship field to null removes the connection between
objects, but Realm does not delete the referenced object
unless that object is embedded.

### To-Many Relationship
A **to-many** relationship means that an object is related in a specific
way to multiple objects. You can create a relationship between one object
and any number of objects using a field of type `RealmList<T>`
where `T` is a Realm object in your application:

### Inverse Relationship
An **inverse relationship** links an object back to any other objects that refer
to it in a defined to-one or to-many relationship. Relationship definitions are
unidirectional, so you must explicitly define a property in the object's model
as an inverse relationship.

For example, the to-many relationship "User has many Tasks" does not
automatically create the inverse relationship "Task belongs to User". If you
don't specify the inverse relationship in the object model, you would need to
run a separate query to look up the user that is assigned to a given task.

To define an inverse relationship, define a `LinkingObjects` property in your
object model. The `LinkingObjects` definition specifies the object type and
property name of the relationship that it inverts.

Realm automatically updates implicit relationships whenever an
object is added or removed in the specified relationship. You cannot manually
set the value of an inverse relationship property.

Fields annotated with `@LinkingObjects` must be:

- marked `final`
- of type `RealmResults<T>` where `T` is the type at the opposite
end of the relationship

Since relationships are many-to-one or many-to-many, following inverse
relationships can result in zero, one, or many objects.

Like any other `RealmResults` set, you can
query an inverse relationship.

## Define a Relationship Field

> Warning:
> Realm objects use getters and setters to persist updated
field values to your realms. Always use getters and setters for
updates.
>

### Many-to-One
To set up a many-to-one or one-to-one relationship, create a field
whose type is a Realm object in your application:

#### Java

```java
import io.realm.RealmObject;

public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
    private Frog bestFriend;
    public Frog(String name, int age, String species, String owner, Frog bestFriend) {
        this.name = name;
        this.age = age;
        this.species = species;
        this.owner = owner;
        this.bestFriend = bestFriend;
    }
    public Frog(){} // RealmObject subclasses must provide an empty constructor

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public String getSpecies() { return species; }
    public void setSpecies(String species) { this.species = species; }
    public String getOwner() { return owner; }
    public void setOwner(String owner) { this.owner = owner; }
    public Frog getBestFriend() { return bestFriend; }
    public void setBestFriend(Frog bestFriend) { this.bestFriend = bestFriend; }
}
```

#### Kotlin

```kotlin
import io.realm.RealmObject

open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null
    var bestFriend: Frog? = null

    constructor(
        name: String?,
        age: Int,
        species: String?,
        owner: String?,
        bestFriend: Frog?
    ) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
        this.bestFriend = bestFriend
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

> Important:
> When you declare a to-one relationship in your object model, it must
be an optional property. If you try to make a to-one relationship
required, Realm throws an exception at runtime.
>

Each `Frog` references either zero `Frog` instances or one other `Frog` instance. Nothing
prevents multiple `Frog` instances from referencing the same `Frog`
as a best friend; the distinction between a many-to-one and a one-to-one
relationship is up to your application.

### Many-to-Many
#### Java

```java
import io.realm.RealmList;
import io.realm.RealmObject;

public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
    private RealmList<Frog> bestFriends;
    public Frog(String name, int age, String species, String owner, RealmList<Frog> bestFriends) {
        this.name = name;
        this.age = age;
        this.species = species;
        this.owner = owner;
        this.bestFriends = bestFriends;
    }
    public Frog(){} // RealmObject subclasses must provide an empty constructor

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public String getSpecies() { return species; }
    public void setSpecies(String species) { this.species = species; }
    public String getOwner() { return owner; }
    public void setOwner(String owner) { this.owner = owner; }
    public RealmList<Frog> getBestFriends() { return bestFriends; }
    public void setBestFriends(RealmList<Frog> bestFriends) { this.bestFriends = bestFriends; }
}
```

#### Kotlin

```kotlin
import io.realm.RealmList
import io.realm.RealmObject

open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null
    var bestFriends: RealmList<Frog>? = null

    constructor(
        name: String?,
        age: Int,
        species: String?,
        owner: String?,
        bestFriends: RealmList<Frog>?
    ) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
        this.bestFriends = bestFriends
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

`RealmList` s are containers of `RealmObject` s, but otherwise behave
like a regular collection. You can use the same object in multiple
`RealmList` s.

### Inverse Relationships
By default, Realm relationships are unidirectional. You
can follow a link from one class to a referenced class, but not in the
opposite direction. Consider the following class defining a `Toad` with
a list of `frogFriends`:

#### Java

```java
import io.realm.RealmList;
import io.realm.RealmObject;

public class Toad extends RealmObject {
    private RealmList<Frog> frogFriends;
    public Toad(RealmList<Frog> frogFriends) {
        this.frogFriends = frogFriends;
    }
    public Toad() {}

    public RealmList<Frog> getFrogFriends() { return frogFriends; }
    public void setFrogFriends(RealmList<Frog> frogFriends) { this.frogFriends = frogFriends; }
}
```

#### Kotlin

```kotlin
import io.realm.RealmList
import io.realm.RealmObject

open class Toad : RealmObject {
    var frogFriends: RealmList<Frog>? = null

    constructor(frogFriends: RealmList<Frog>?) {
        this.frogFriends = frogFriends
    }

    constructor() {}
}
```

You can provide a link in the opposite direction, from `Frog` to `Toad`,
with the `@LinkingObjects`
annotation on a `final` (in Java) or `val` (in Kotlin) field of type
`RealmResults<T>`:

#### Java

```java
import io.realm.RealmObject;
import io.realm.RealmResults;
import io.realm.annotations.LinkingObjects;

public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
    @LinkingObjects("frogFriends")
    private final RealmResults<Toad> toadFriends = null;

    public Frog(String name, int age, String species, String owner) {
        this.name = name;
        this.age = age;
        this.species = species;
        this.owner = owner;
    }
    public Frog(){} // RealmObject subclasses must provide an empty constructor

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
    public String getSpecies() { return species; }
    public void setSpecies(String species) { this.species = species; }
    public String getOwner() { return owner; }
    public void setOwner(String owner) { this.owner = owner; }
}
```

#### Kotlin

```kotlin
import io.realm.RealmObject
import io.realm.RealmResults
import io.realm.annotations.LinkingObjects

open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null
    @LinkingObjects("frogFriends")
    private val toadFriends: RealmResults<Toad>? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

> Important:
> Inverse relationship fields must be marked `final`.
>
