# Open & Close a Realm - Java SDK
Interacting with realms in an Android
application uses the following high-level series of steps:

1. Create a configuration for the realm you want to open.
2. Open the realm using the config.
3. Close the realm to free up resources when you're finished.

## The Default Realm
You can save any `RealmConfiguration`
as the default for your application using the
`setDefaultConfiguration()`
method:

#### Java

```java
RealmConfiguration config = new RealmConfiguration.Builder()
        .name("default-realm")
        .allowQueriesOnUiThread(true)
        .allowWritesOnUiThread(true)
        .compactOnLaunch()
        .inMemory()
        .build();
// set this config as the default realm
Realm.setDefaultConfiguration(config);
```

#### Kotlin

```kotlin
val config = RealmConfiguration.Builder()
    .name("default-realm")
    .allowQueriesOnUiThread(true)
    .allowWritesOnUiThread(true)
    .compactOnLaunch()
    .inMemory()
    .build()
// set this config as the default realm
Realm.setDefaultConfiguration(config)
```

You can then use
`getDefaultConfiguration()`
to access that configuration, or
`getDefaultInstance()`
to open a realm with that configuration:

#### Java

```java
Realm realm = Realm.getDefaultInstance();
Log.v("EXAMPLE","Successfully opened the default realm at: " + realm.getPath());
```

#### Kotlin

```kotlin
val realm = Realm.getDefaultInstance()
Log.v("EXAMPLE","Successfully opened the default realm at: ${realm.path}")
```

## Local Realms
Local realms store data only on the client device. You can customize
the settings for a local realm with `RealmConfiguration`.

### Local Realm Configuration
To configure settings for a realm, create a
`RealmConfiguration` with a
`RealmConfiguration.Builder`.
The following example configures a local realm with:

- the file name "alternate-realm"
- synchronous reads explicitly allowed on the UI thread
- synchronous writes explicitly allowed on the UI thread
- automatic compaction when launching the realm to save file space

#### Java

```java
RealmConfiguration config = new RealmConfiguration.Builder()
        .name("alternate-realm")
        .allowQueriesOnUiThread(true)
        .allowWritesOnUiThread(true)
        .compactOnLaunch()
        .build();

Realm realm = Realm.getInstance(config);
Log.v("EXAMPLE", "Successfully opened a realm at: " + realm.getPath());

```

#### Kotlin

```kotlin
val config = RealmConfiguration.Builder()
    .name("alternate-realm")
    .allowQueriesOnUiThread(true)
    .allowWritesOnUiThread(true)
    .compactOnLaunch()
    .build()
val realm = Realm.getInstance(config)
Log.v("EXAMPLE", "Successfully opened a realm at: ${realm.path}")

```

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

### Open a Local Realm
To open a realm, create a
`RealmConfiguration` with
`RealmConfiguration.Builder` and
pass the resulting `RealmConfiguration` to
`getInstance()`
or `getInstanceAsync()`:

#### Java

```java
RealmConfiguration config = new RealmConfiguration.Builder()
        .allowQueriesOnUiThread(true)
        .allowWritesOnUiThread(true)
        .build();

Realm realm;
try {
    realm = Realm.getInstance(config);
    Log.v("EXAMPLE", "Successfully opened a realm at: " + realm.getPath());
} catch (RealmFileException ex) {
    Log.v("EXAMPLE", "Error opening the realm.");
    Log.v("EXAMPLE", ex.toString());
}

```

#### Kotlin

```kotlin
val config = RealmConfiguration.Builder()
    .allowQueriesOnUiThread(true)
    .allowWritesOnUiThread(true)
    .build()

var realm: Realm
try {
    realm = Realm.getInstance(config)
    Log.v("EXAMPLE", "Successfully opened a realm at: ${realm.path}")
} catch(ex: RealmFileException) {
    Log.v("EXAMPLE", "Error opening the realm.")
    Log.v("EXAMPLE", ex.toString())
}

```

### Read-Only Realms
It's sometimes useful to ship a prepared realm file with your app
that contains shared data that does not frequently change. You can use
the `readOnly()`
method when configuring your realm to make it read-only. This can
prevent accidental writes to the realm and causes the realm to
throw an `IllegalStateException` if a write occurs.

> Warning:
> Read-only realms are only enforced as read-only in process.
The realm file itself is still writeable.
>

#### Java

```java
RealmConfiguration config = new RealmConfiguration.Builder()
        .assetFile("bundled.realm")
        .readOnly()
        .modules(new BundledRealmModule())
        .build();
```

#### Kotlin

```kotlin
val config = RealmConfiguration.Builder()
    .assetFile("readonly.realm")
    .readOnly()
    .modules(BundledRealmModule())
    .build()
```

### In-Memory Realms
You can create a realm that runs entirely in memory without being written
to a file. When memory runs low on an Android device, in-memory realms
may [swap](https://en.wikipedia.org/wiki/Memory_paging#Terminology) temporarily from main
memory to disk space. The SDK deletes all files created by an in-memory
realm when:

- the realm closes
- all references to that realm fall out of scope

To create an in-memory realm, use `inMemory()`
when configuring your realm:

#### Java

```java
RealmConfiguration config = new RealmConfiguration.Builder()
        .inMemory()
        .name("java.transient.realm")
        .build();
Realm realm = Realm.getInstance(config);
```

#### Kotlin

```kotlin
val config = RealmConfiguration.Builder()
    .inMemory()
    .name("kt.transient.realm")
    .build()
val realm = Realm.getInstance(config)
```

### Dynamic Realms
Conventional realms define a schema using `RealmObject` subclasses
or the `RealmModel` interface. A
`DynamicRealm` uses strings to
define a schema at runtime. Opening a dynamic realm uses the same
configuration as a conventional realm, but dynamic realms ignore
all configured schema, migration, and schema versions.

Dynamic realms offer flexibility at the expense of type safety and
performance. As a result, only use dynamic realms when that
flexibility is required, such as during migrations, manual client
resets, and when working with string-based data like CSV files or JSON.

To open a Dynamic Realm with a mutable schema, use
`DynamicRealm`:

#### Java

```java
RealmConfiguration config = new RealmConfiguration.Builder()
        .allowWritesOnUiThread(true)
        .allowQueriesOnUiThread(true)
        .name("java.dynamic.realm")
        .build();
DynamicRealm dynamicRealm = DynamicRealm.getInstance(config);

// all objects in a DynamicRealm are DynamicRealmObjects
AtomicReference<DynamicRealmObject> frog = new AtomicReference<>();
dynamicRealm.executeTransaction(transactionDynamicRealm -> {
    // add type Frog to the schema with name and age fields
    dynamicRealm.getSchema()
            .create("Frog")
            .addField("name", String.class)
            .addField("age", int.class);
     frog.set(transactionDynamicRealm.createObject("Frog"));
     frog.get().set("name", "Wirt Jr.");
     frog.get().set("age", 42);
});

// access all fields in a DynamicRealm using strings
String name = frog.get().getString("name");
int age = frog.get().getInt("age");

// because an underlying schema still exists,
// accessing a field that does not exist throws an exception
try {
    frog.get().getString("doesn't exist");
} catch (IllegalArgumentException e) {
    Log.e("EXAMPLE", "That field doesn't exist.");
}

// Queries still work normally
RealmResults<DynamicRealmObject> frogs = dynamicRealm.where("Frog")
        .equalTo("name", "Wirt Jr.")
        .findAll();
```

#### Kotlin

```kotlin
val config = RealmConfiguration.Builder()
    .allowWritesOnUiThread(true)
    .allowQueriesOnUiThread(true)
    .name("kt.dynamic.realm")
    .build()
val dynamicRealm = DynamicRealm.getInstance(config)

// all objects in a DynamicRealm are DynamicRealmObjects
var frog: DynamicRealmObject? = null
dynamicRealm.executeTransaction { transactionDynamicRealm: DynamicRealm ->
    // add type Frog to the schema with name and age fields
    dynamicRealm.schema
        .create("Frog")
        .addField("name", String::class.java)
        .addField("age", Integer::class.java)
    frog = transactionDynamicRealm.createObject("Frog")
    frog?.set("name", "Wirt Jr.")
    frog?.set("age", 42)
}

// access all fields in a DynamicRealm using strings
val name = frog?.getString("name")
val age = frog?.getInt("age")

// because an underlying schema still exists,
// accessing a field that does not exist throws an exception
try {
    frog?.getString("doesn't exist")
} catch (e: IllegalArgumentException) {
    Log.e("EXAMPLE", "That field doesn't exist.")
}

// Queries still work normally
val frogs = dynamicRealm.where("Frog")
    .equalTo("name", "Wirt Jr.")
    .findAll()
```

## Close a Realm
It is important to remember to call the `close()` method when done with a
realm instance to free resources. Neglecting to close realms can lead to an
`OutOfMemoryError`.

#### Java

```java
realm.close();

```

#### Kotlin

```kotlin
realm.close()

```

## Configure Which Classes to Include in Your Realm Schema
Realm modules are collections of Realm object
models. Specify a module or modules when opening a realm to control
which classes Realm should include in your schema. If you
do not specify a module, Realm uses the default module,
which includes all Realm objects defined in your
application.

> Note:
> Libraries that include Realm must expose and use their
schema through a module. Doing so prevents the library from
generating the default `RealmModule`, which would conflict with
the default `RealmModule` used by any app that includes the library.
Apps using the library access library classes through the module.
>

