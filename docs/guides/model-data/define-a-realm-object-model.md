# Define a Realm Object Model - Java SDK
## Define a Realm Object
To define a Realm object in your application,
create a subclass of `RealmObject`
or implement `RealmModel`.

> Important:
> - All Realm objects must provide an empty constructor.
> - All Realm objects must use the `public` visibility modifier in Java
or the `open` visibility modifier in Kotlin.
>

> Note:
> Class names are limited to a maximum of 57 UTF-8 characters.
>

### Extend RealmObject
The following code block shows a Realm object that
describes a Frog. This Frog class can be stored in
Realm because it `extends` the `RealmObject` class.

#### Java

```java
import io.realm.RealmObject;

// To add an object to your Realm Schema, extend RealmObject
public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
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

// providing default values for each constructor parameter
// fulfills the need for an empty constructor
open class Frog(
    var name: String? = null,
    var age: Int = 0,
    var species: String? = null,
    var owner: String? = null
) : RealmObject() // To add an object to your Realm Schema, extend RealmObject
```

### Implement RealmModel
The following code block shows a Realm object that
describes a Frog. This Frog class can
be stored in Realm because it `implements` the
`RealmModel` class and uses the `@RealmClass` annotation:

#### Java

```java
import io.realm.RealmModel;
import io.realm.annotations.RealmClass;

@RealmClass
public class Frog implements RealmModel {
    private String name;
    private int age;
    private String species;
    private String owner;
    public Frog(String name, int age, String species, String owner) {
        this.name = name;
        this.age = age;
        this.species = species;
        this.owner = owner;
    }
    public Frog() {} // RealmObject subclasses must provide an empty constructor

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

> Important:
> All Realm objects must use the `public`
visibility modifier.
>

#### Kotlin

```kotlin
import io.realm.RealmModel
import io.realm.annotations.RealmClass

@RealmClass
open class Frog : RealmModel {
    var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null

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
> All Realm objects must use the `open`
visibility modifier.
>

> Tip:
> When you create a Realm object by extending the `RealmObject`
class, you can access `RealmObject` class methods dynamically on
instances of your Realm object. Realm objects
created by implementing `RealmModel` can access those same methods
statically through the `RealmObject` class:
>
> #### Java
>
> ```java
> // With RealmObject
> frogRealmObject.isValid();
> frogRealmObject.addChangeListener(listener);
>
> // With RealmModel
> RealmObject.isValid(frogRealmModel);
> RealmObject.addChangeListener(frogRealmModel, listener);
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> // With RealmObject
> frogRealmObject?.isValid
> frogRealmObject?.addChangeListener(listener)
>
> // With RealmModel
> RealmObject.isValid(frogRealmModel)
> RealmObject.addChangeListener(frogRealmModel, listener)
>
> ```
>
>

## Lists
Realm objects can contain lists of non-Realm-object data
types:

#### Java

Unlike lists of Realm objects, these lists can contain
null values. If null values shouldn't be allowed, use the
@Required annotation.

```java
import io.realm.RealmList;
import io.realm.RealmObject;

public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
    private RealmList<String> favoriteColors;
    public Frog(String name, int age, String species, String owner, RealmList<String> favoriteColors) {
        this.name = name;
        this.age = age;
        this.species = species;
        this.owner = owner;
        this.favoriteColors = favoriteColors;
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
    public RealmList<String> getFavoriteColors() { return favoriteColors; }
    public void setFavoriteColors(RealmList<String> favoriteColors) { this.favoriteColors = favoriteColors; }
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
    var favoriteColors : RealmList<String>? = null

    constructor(
        name: String?,
        age: Int,
        species: String?,
        owner: String?,
        favoriteColors: RealmList<String>?
    ) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
        this.favoriteColors = favoriteColors
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

> Seealso:
> Data Types: Lists
>

## Define an Embedded Object Field
Realm provides the ability to nest objects within other
objects. This has several advantages:

- When you delete an object that contains another object, the delete
operation removes both objects from the realm, so unused objects
don't accumulate in your realm file, taking up valuable space on
user's mobile devices.

To embed an object, set the `embedded` property of the
`@RealmClass`
annotation to `true` on the class that you'd like to nest within
another class:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.RealmClass;

@RealmClass(embedded=true)
public class Fly extends RealmObject {
    private String name;
    public Fly(String name) {
        this.name = name;
    }
    public Fly() {} // RealmObject subclasses must provide an empty constructor
}
```

#### Kotlin

```kotlin
import io.realm.RealmObject
import io.realm.annotations.RealmClass

@RealmClass(embedded = true)
open class Fly : RealmObject {
    private var name: String? = null

    constructor(name: String?) {
        this.name = name
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

Then, any time you reference that class from another class,
Realm will embed the referenced class within the enclosing
class, as in the following example:

#### Java

```java
import io.realm.RealmObject;

public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
    private Fly lastMeal;
    public Frog(String name, int age, String species, String owner, Fly lastMeal) {
        this.name = name;
        this.age = age;
        this.species = species;
        this.owner = owner;
        this.lastMeal = lastMeal;
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
    public Fly getLastMeal() { return lastMeal; }
    public void setLastMeal(Fly lastMeal) { this.lastMeal = lastMeal; }
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
    var lastMeal: Fly? = null

    constructor(
        name: String?,
        age: Int,
        species: String?,
        owner: String?,
        lastMeal: Fly?
    ) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
        this.lastMeal = lastMeal
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

> Seealso:
> Data Types: Embedded Objects
>

## Annotations
Use annotations to customize your Realm object models.

### Primary Key
> Version added: 10.6.0
> Realm automatically indexes
primary key fields. Previously, Realm only indexed `String` primary
keys automatically.
>

Realm treats fields marked with the
`@PrimaryKey` annotation
as primary keys for their corresponding object schema. Primary keys are
subject to the following limitations:

- You can define only one primary key per object schema.
- Primary key values must be unique across all instances of an object
in a realm. Attempting to insert a duplicate primary key value
results in a `RealmPrimaryKeyConstraintException`.
- Primary key values are immutable. To change the primary key value of
an object, you must delete the original object and insert a new object
with a different primary key value.
- Embedded objects cannot define a
primary key.

You can create a primary key with any of the following types:

- `String`
- `UUID`
- `ObjectId`
- `Integer` or `int`
- `Long` or `long`
- `Short` or `short`
- `Byte` or `byte[]`

Non-primitive types can contain a value of `null` as a primary key
value, but only for one object of a particular type, since each primary
key value must be unique. Attempting to insert an object with an existing
primary key into a realm will result in a
`[RealmPrimaryKeyConstraintException`.

Realm automatically indexes
primary key fields, which allows you to efficiently read and modify
objects based on their primary key.

You cannot change the primary key field for an object type after adding
any object of that type to a realm.

Embedded objects cannot contain primary keys.

You may optionally define a primary key for an object type as part of
the object schema with the
`@PrimaryKey` annotation:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;

public class Frog extends RealmObject {
    @PrimaryKey private String name;
    private int age;
    private String species;
    private String owner;
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
import io.realm.annotations.PrimaryKey

open class Frog : RealmObject {
    @PrimaryKey var name : String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

### Required Fields
#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.Required;

public class Frog extends RealmObject {
    @Required private String name;
    private int age;
    private String species;
    private String owner;
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
import io.realm.annotations.Required

open class Frog : RealmObject {
    @Required var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

### Optional Fields
Fields marked with Java object types and Kotlin nullable types
(ending with `?`) are nullable by default. All other types
(primitives, non-nullable Kotlin object types) are required by default.
You can mark a nullable field with the `@Required`
annotation to prevent that field from holding a null value.
`RealmLists` are never nullable, but
you can use the `@Required` annotation to prevent objects in a list
from holding a null value, even if the base type would otherwise allow it.
You cannot mark a `RealmList` of `RealmObject` subtypes as required.

You can make any of the following types required:

- `String`
- `UUID`
- `ObjectId`
- `Integer`
- `Long`
- `Short`
- `Byte` or `byte[]`
- `Boolean`
- `Float`
- `Double`
- `Date`
- `RealmList`

Primitive types such as `int` and the `RealmList` type are
implicitly required. Fields with the `RealmObject` type are always
nullable, and cannot be made required.

> Important:
> In Kotlin, types are non-nullable by default unless you explicitly
add a `?` suffix to the type. You can only annotate
nullable types. Using the
`@Required` annotation on non-nullable types will fail compilation.
>

#### Java

Nullable fields are optional by default in Realm, unless
otherwise specified with the @Required
annotation. The following types are nullable:

- `String`
- `Date`
- `UUID`
- `ObjectId`
- `Integer`
- `Long`
- `Short`
- `Byte` or `byte[]`
- `Boolean`
- `Float`
- `Double`

Primitive types like `int` and `long` are non-nullable by
default and cannot be made nullable, as they cannot be set to a
null value.

#### Kotlin

In Kotlin, fields are considered nullable only if a field is
marked nullable with the Kotlin [? operator](https://kotlinlang.org/docs/reference/null-safety.html) except
for the following types:

- `String`
- `Date`
- `UUID`
- `ObjectId`
- `Decimal128`
- `RealmAny`

You can require any type that ends with the Kotlin `?`
operator, such as `Int?`.

The `RealmList` type is non-nullable by default and cannot be
made nullable.

### Default Field Values
To assign a default value to a field, use the built-in language features
to assign default values.

#### Java

Use the class constructor(s) to assign default values:

```java
import io.realm.RealmObject;

public class Frog extends RealmObject {
    private String name = "Kitty";
    private int age;
    private String species;
    private String owner;
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

Assign default values in the field declaration:

```kotlin
import io.realm.RealmObject

open class Frog : RealmObject {
    var name = "Kitty"
    var age = 0
    var species: String? = null
    var owner: String? = null

    constructor(name: String, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

> Note:
> While default values ensure that a newly created object cannot contain
a value of `null` (unless you specify a default value of `null`),
they do not impact the nullability of a field. To make a field
non-nullable, see Required Fields.
>

### Index a Field
**Indexes** support the efficient execution of queries in
Realm. Without indexes, Realm must perform a
*collection scan*, i.e. scan every document in a collection, to select
those documents that match a query. If an appropriate index exists for a
query, Realm can use the index to limit the number of
documents that it must inspect.

Indexes are special data structures that store a small portion of a
realm's data in an easy to traverse form. The index stores the value
of a specific field ordered by the value of the field. The ordering of
the index entries supports efficient equality matches and range-based
query operations.

Adding an index can speed up some queries at the cost of slightly slower write
times and additional storage and memory overhead. Indexes require space in your
realm file, so adding an index to a property will increase disk space consumed
by your realm file. Each index entry is a minimum of 12 bytes.

You can index fields with the following types:

- `String`
- `UUID`
- `ObjectId`
- `Integer` or `int`
- `Long` or `long`
- `Short` or `short`
- `Byte` or `byte[]`
- `Boolean` or `bool`
- `Date`
- `RealmAny`

Realm creates indexes for fields annotated with
`@Index`.

To index a field, use the `@Index`
annotation:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.Index;

public class Frog extends RealmObject {
    private String name;
    private int age;
    @Index private String species;
    private String owner;
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
import io.realm.annotations.Index

open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    @Index var species : String? = null
    var owner: String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

### Ignore a Field
If you don't want to save a field in your model to a realm, you can
ignore a field.

Ignore a field from a Realm object model with the
`@Ignore` annotation:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.Ignore;

public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    // can you ever really own a frog persistently?
    @Ignore private String owner;
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
import io.realm.annotations.Ignore

open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    var species: String? = null
    // can you ever really own a frog persistently?
    @Ignore var owner  : String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

> Note:
> Fields marked `static` or `transient` are always ignored, and do
not need the `@Ignore` annotation.
>

### Rename a Field
By default, Realm uses the name defined in the model class
to represent fields internally. In some cases you might want to change
this behavior:

- To make it easier to work across platforms, since naming conventions differ.
- To change a field name in Kotlin without forcing a migration.

Choosing an internal name that differs from the name used in model classes
has the following implications:

- Migrations must use the internal name when creating classes and fields.
- Schema errors reported will use the internal name.

Use the `@RealmField`
annotation to rename a field:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.RealmField;

public class Frog extends RealmObject {
    private String name;
    private int age;
    @RealmField("latinName") private String species;
    private String owner;
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
import io.realm.annotations.RealmField

open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    @RealmField("latinName") var species: String? = null
    var owner: String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

Alternatively, you can also assign a naming policy at the module or
class levels to change the way that Realm interprets field
names.

You can define a
`naming policy`
at the module level,
which will affect all classes included in the module:

#### Java

```java
import io.realm.annotations.RealmModule;
import io.realm.annotations.RealmNamingPolicy;

@RealmModule(
        allClasses = true,
        classNamingPolicy = RealmNamingPolicy.LOWER_CASE_WITH_UNDERSCORES,
        fieldNamingPolicy = RealmNamingPolicy.LOWER_CASE_WITH_UNDERSCORES
)
public class MyModule {
}
```

#### Kotlin

```kotlin
import io.realm.annotations.RealmModule
import io.realm.annotations.RealmNamingPolicy

@RealmModule(
    allClasses = true,
    classNamingPolicy = RealmNamingPolicy.LOWER_CASE_WITH_UNDERSCORES,
    fieldNamingPolicy = RealmNamingPolicy.LOWER_CASE_WITH_UNDERSCORES
)
open class MyModule
```

You can also define a
`naming policy`
at the class level, which overrides module level settings:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.RealmClass;
import io.realm.annotations.RealmNamingPolicy;

@RealmClass(fieldNamingPolicy = RealmNamingPolicy.PASCAL_CASE)
public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
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
import io.realm.annotations.RealmClass
import io.realm.annotations.RealmNamingPolicy

@RealmClass(fieldNamingPolicy = RealmNamingPolicy.PASCAL_CASE)
open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

### Rename a Class
By default, Realm uses the name defined in the model class
to represent classes internally. In some cases you might want to change
this behavior:

- To support multiple model classes with the same simple name in different packages.
- To make it easier to work across platforms, since naming conventions differ.
- To use a class name that is longer than the 57 character limit enforced by Realm.
- To change a class name in Kotlin without forcing a migration.

Use the `@RealmClass`
annotation to rename a class:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.RealmClass;

@RealmClass(name = "ShortBodiedTaillessAmphibian")
public class Frog extends RealmObject {
    private String name;
    private int age;
    private String species;
    private String owner;
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
import io.realm.annotations.RealmClass

@RealmClass(name = "Short_Bodied_Tailless_Amphibian")
open class Frog : RealmObject {
    var name: String? = null
    var age = 0
    var species: String? = null
    var owner: String? = null

    constructor(name: String?, age: Int, species: String?, owner: String?) {
        this.name = name
        this.age = age
        this.species = species
        this.owner = owner
    }

    constructor() {} // RealmObject subclasses must provide an empty constructor
}
```

## Omit Classes from your Realm Schema
By default, your application's Realm Schema includes all
classes that extend `RealmObject`. If you only want to include a
subset of classes that extend `RealmObject` in your Realm
Schema, you can include that subset of classes in a module and open
your realm using that module:

#### Java

```java
import io.realm.annotations.RealmModule;

@RealmModule(classes = { Frog.class, Fly.class })
public class MyModule {
}
```

#### Kotlin

```kotlin
import io.realm.annotations.RealmModule

@RealmModule(classes = [Frog::class, Fly::class])
open class MyModule
```

