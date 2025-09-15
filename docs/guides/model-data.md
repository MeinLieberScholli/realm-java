# Model Data - Java SDK

An **object schema** is a configuration object that defines the fields and
relationships of a Realm object type. Android
Realm applications define object schemas with Java or Kotlin
classes using Realm Schemas.

Object schemas specify constraints on object fields such as the data
type of each field, whether a
field is required, and default field values. Schemas can also define
relationships between object types in
a realm.

Modifying your application's Realm Schema requires you to
migrate data from older
versions of your Realm Schema to the new version.

## Realm Apps
Every App has a Realm Schema
composed of a list of object schemas for each type of object that the
realms in that application may contain.

Realm guarantees that all objects in a realm conform to the
schema for their object type and validates objects whenever they're
created, modified, or deleted.

## Relationships
You can model **one-to-one** relationships in realm with
`RealmObject` fields.
You can model **one-to-many** and **many-to-one** relationships
`RealmList` fields.
Inverse relationships are the opposite end of a **one-to-many** or
**many-to-one** relationship.
You can make **inverse** relationships traversable with the
`@LinkingObjects`
annotation on a `RealmResults`
field. In an instance of a `RealmObject`, inverse relationship fields
contain the set of Realm objects that point to that object
instance through the described relationship. You can find the same set
of Realm objects with a manual query, but the inverse
relationship field reduces boilerplate query code and capacity for error.

## Realm Objects
Unlike normal Java objects, which contain their own data, a
Realm object doesn't contain data. Instead,
Realm objects read and write properties directly to
Realm.

Instances of Realm objects can be either **managed** or **unmanaged**.

- **Managed** objects are: persisted in Realmalways up to datethread-confinedgenerally more lightweight than the unmanaged version, as they take
up less space on the Java heap.
- **Unmanaged** objects are just like ordinary Java objects, since
they are not persisted and never update automatically.
You can move unmanaged objects freely across threads.

You can convert between the two states using
`realm.copyToRealm()`
and `realm.copyFromRealm()`.

### RealmProxy
The `RealmProxy` classes are the Realm SDK's way of
ensuring that Realm objects don't contain any data
themselves. Instead, each class's `RealmProxy` accesses data directly
in the database.

For every model class in your project, the Realm annotation
processor generates a corresponding `RealmProxy` class. This class
extends your model class and is returned when you call
`Realm.createObject()`. In your code, this object works just like your
model class.

### Realm Object Limitations
Realm objects:

- cannot contain fields that use the `final` or `volatile` modifiers
(except for inverse relationship
fields).
- cannot extend any object other than `RealmObject`.
- must contain an empty constructor (if your class does not include any
constructor, the automatically generated empty constructor will suffice)

Naming limitations:

- Class names cannot exceed 57 characters.
- Class names must be unique within realm modules
- Field names cannot exceed 63 characters.

Size limitations:

- `String` or `byte[]` fields cannot exceed 16 MB.

Usage limitations:

- Because Realm objects are live and can change at any time,
their `hashCode()` value can change over time. As a result, you
should not use `RealmObject` instances as a key in any map or set.

## Incremental Builds
The bytecode transformer used by Realm supports incremental
builds, but your application requires a full rebuild when adding or
removing the following from a Realm object field:

- an `@Ignore` annotation
- the `static` keyword
- the `transient` keyword

You can perform a full rebuild with Build > Clean Project
and Build > Rebuild Project in these cases.

## Schema Version
A **schema version** identifies the state of a Realm Schema at some point in time. Realm tracks the schema
version of each realm and uses it to map the objects in each realm
to the correct schema.

Schema versions are integers that you may include
in the realm configuration when you open a realm. If a client
application does not specify a version number when it opens a realm then
the realm defaults to version `0`.

> Important:
> Migrations must update a realm to a
higher schema version. Realm throws an error if a client
application opens a realm with a schema version that is lower than
the realm's current version or if the specified schema version is the
same as the realm's current version but includes different
object schemas.
>

## Migrations
A **local migration** is a migration for a realm with
another realm. Local migrations have access to the existing
Realm Schema, version, and objects and define logic that
incrementally updates the realm to its new schema version.
To perform a local migration you must specify a new schema
version that is higher than the current version and provide
a migration function when you open the out-of-date realm.

With the SDK, you can update underlying data to reflect schema changes
using manual migrations. During such a manual migration, you can
define new and deleted properties when they are added or removed from
your schema. The editable schema exposed via a
`DynamicRealm` provides
convenience functions for renaming fields. This gives you full control
over the behavior of your data during complex schema migrations.

> Tip:
> During development of an application, `RealmObject` classes can
change frequently. You can use `Realm.deleteRealm()`to
delete the database file and eliminate the need to write a full
migration for testing data.
>
