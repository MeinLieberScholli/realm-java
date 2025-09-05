# Change an Object Model - Java SDK
#### Local

The following examples demonstrate how to add, delete, and modify
properties in a schema. First, make the required schema change.
Then, increment the schema version. Finally, if the change is
breaking (destructive) create a corresponding migration function to move data from the original schema
to the updated schema.

> Note:
> Assume that each schema change shown in the following example
occurs after the application has used the existing schema. The
new schema version numbers apply only after you open the
realm and explicitly specify the new version number.
In other words, you can't specify version 3 without previously
specifying and using versions 0, 1, and 2.
>

A realm using schema version `0` has a `Person` object type:

#### Java

```java
public class Person extends RealmObject { // Realm schema version 0
    @Required
    public String firstName;
    @Required
    public int age;
}

```

#### Kotlin

```kotlin
class Person: RealmObject { // Realm schema version 0
    var firstName: String = ""
    var age: int = 0
}

```

### A. Add a Property
The following example adds a `lastName` property to the
original Person schema:

#### Java

```java
public class Person extends RealmObject { // Realm schema version 1
    @Required
    public String firstName;
    @Required
    public String lastName;
    @Required
    public int age;
}
```

#### Kotlin

```kotlin
class Person: RealmObject { // Realm schema version 1
    var firstName: String = ""
    var lastName: String = ""
    var age: int = 0
}

```

### B. Delete a Property
The following example uses a combined
`fullName` property instead of the separate `firstName` and
`lastName` property in the original Person schema:

#### Java

```java
public class Person extends RealmObject { // Realm schema version 2
    @Required
    public String fullName;
    @Required
    public int age;
}

```

#### Kotlin

```kotlin
class Person: RealmObject { // Realm schema version 2
    var fullName: String = ""
    var age: int = 0
}

```

### C. Modify a Property Type or Rename a Property
The following example modifies the `age` property in the
original Person schema by
renaming it to `birthday` and changing the type to `Date`:

#### Java

```java
public class Person extends RealmObject { // Realm schema version 3
    @Required
    public String fullName;
    @Required
    public Date birthday = new Date();
}

```

#### Kotlin

```kotlin
class Person: RealmObject { // Realm schema version 3
    var fullName: String = ""
    var birthday: Date = Date()
}

```

### D. Migration Functions
To migrate the realm to conform to the updated
`Person` schema, set the realm's
schema version to `3`
and define a migration function to set the value of
`fullName` based on the existing `firstName` and
`lastName` properties and the value of `birthday` based on
`age`:

#### Java

```java
public class Migration implements RealmMigration {
  @Override
  public void migrate(DynamicRealm realm, long oldVersion, long newVersion) {
     Long version = oldVersion;

     // DynamicRealm exposes an editable schema
     RealmSchema schema = realm.getSchema();

     // Changes from version 0 to 1: Adding lastName.
     // All properties will be initialized with the default value "".
     if (version == 0L) {
        schema.get("Person")
            .addField("lastName", String.class, FieldAttribute.REQUIRED);
        version++;
     }

     // Changes from version 1 to 2: combine firstName/lastName into fullName
     if (version == 1L) {
        schema.get("Person")
            .addField("fullName", String.class, FieldAttribute.REQUIRED)
            .transform( DynamicRealmObject obj -> {
                String name = "${obj.getString("firstName")} ${obj.getString("lastName")}";
                obj.setString("fullName", name);
            })
            .removeField("firstName")
            .removeField("lastName");
        version++;
     }

     // Changes from version 2 to 3: replace age with birthday
     if (version == 2L) {
        schema.get("Person")
            .addField("birthday", Date::class.java, FieldAttribute.REQUIRED)
            .transform(DynamicRealmObject obj -> {
                Int birthYear = Date().year - obj.getInt("age");
                obj.setDate("birthday", Date(birthYear, 1, 1));
            })
            .removeField("age");
        version++;
     }
  }
};

@RealmModule(classes = { Person.class })
public class Module {}

RealmConfiguration config = new RealmConfiguration.Builder()
    .modules(new Module())
    .schemaVersion(3) // Must be bumped when the schema changes
    .migration(new Migration()) // Migration to run instead of throwing an exception
    .build();

```

#### Kotlin

```kotlin
val migration = object: RealmMigration {
    override fun migrate(realm: DynamicRealm, oldVersion: Long, newVersion: Long) {
        var version: Long = oldVersion

        // DynamicRealm exposes an editable schema
        val schema: RealmSchema = realm.schema

        // Changes from version 0 to 1: Adding lastName.
        // All properties will be initialized with the default value "".
        if (version == 0L) {
            schema.get("Person")!!
                    .addField("lastName", String::class.java, FieldAttribute.REQUIRED)
            version++
        }

        // Changes from version 1 to 2: Combining firstName/lastName into fullName
        if (version == 1L) {
            schema.get("Person")!!
                    .addField("fullName", String::class.java, FieldAttribute.REQUIRED)
                    .transform { obj: DynamicRealmObject ->
                        val name = "${obj.getString("firstName")} ${obj.getString("lastName")}"
                        obj.setString("fullName", name)
                    }
                    .removeField("firstName")
                    .removeField("lastName")
            version++
        }

        // Changes from version 2 to 3: Replace age with birthday
        if (version == 2L) {
            schema.get("Person")!!
                    .addField("birthday", Date::class.java, FieldAttribute.REQUIRED)
                    .transform { obj: DynamicRealmObject ->
                        var birthYear = Date().year - obj.getInt("age")
                        obj.setDate("birthday", Date(birthYear, 1, 1))
                    }
                    .removeField("age")
            version++
        }
    }
}

@RealmModule(classes = { Person::class.java })
class Module

val config = RealmConfiguration.Builder()
    .schemaVersion(3) // Must be bumped when the schema changes
    .migration(migration) // Migration to run instead of throwing an exception
    .build()
```
