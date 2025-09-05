# Threading - Java SDK
To make your Android apps fast and responsive, you must
balance the computing time needed to lay out the visuals and
handle user interactions with the time needed to process
your data and run your business logic. Typically, app
developers spread this work across multiple threads: the
main or UI thread for all of the user interface-related
work, and one or more background threads to compute heavier
workloads before sending it to the UI thread for
presentation. By offloading heavy work to background
threads, the UI thread can remain highly responsive
regardless of the size of the workload.

## Three Rules to Keep in Mind
Realm enables simple and safe multithreaded code when you
follow these three rules:

You can write to a
realm from any thread, but there can be only one
writer at a time. Consequently, write transactions block
each other. A write on the UI thread may result in your
app appearing unresponsive while it waits for a write on a
background thread to complete.

Live objects, collections, and realm instances are
**thread-confined**: that is, they are only valid on the
thread on which they were created. Practically speaking,
this means you cannot pass live instances to other
threads. However, Realm offers several mechanisms for
sharing objects across threads.

Realm's Multiversion Concurrency Control (MVCC) architecture eliminates the need to lock for read operations. The
values you read will never be corrupted or in a
partially-modified state. You can freely read from realms
on any thread without the need for locks or mutexes.
Unnecessarily locking would be a performance bottleneck
since each thread might need to wait its turn before
reading.

## Communication Across Threads
Live objects, collections, and realms are **thread-confined**. If
you need to work with the same data across multiple threads, you should
open the same realm on multiple threads as separate realm
instances. The Java SDK consolidates underlying connections across
threads where possible to make this pattern
more efficient.

When you need to communicate across threads, you have
several options depending on your use case:

- To modify the data on two threads, query
for the object on both threads using a
primary key.
- To send a fast, read-only view of an object to other threads,
freeze the object.
- To keep and share many read-only views of the object in your app, copy
the object from the realm.
- To react to changes made on any thread, use
notifications.
- To see changes from other threads in the realm on the current
thread, refresh your realm
instance (event loop threads refresh automatically).

### Intents
Managed `RealmObject` instances are
not thread-safe or `Parcelable`, so you cannot pass them between
activities or threads via an `Intent`. Instead, you can pass an object
identifier, like a primary key,
in the `Intent` extras bundle, and then open a new realm instance
in the separate thread to query for that identifier. Alternatively, you
can freeze Realm objects.

> Seealso:
> You can find working examples in the [Passing Objects](https://github.com/realm/realm-java/blob/master/examples/threadExample/src/main/java/io/realm/examples/threads/PassingObjectsFragment.java)
portion of the [Java SDK Threading Example](https://github.com/realm/realm-java/tree/master/examples/threadExample).
The example shows you how to pass IDs and retrieve a `RealmObject`
in common Android use cases.
>

### Frozen Objects
Live, thread-confined objects work fine in most cases.
However, some apps -- those based on reactive, event
stream-based architectures, for example -- need to send
immutable copies across threads. In this case,
you can **freeze** objects, collections, and realms.

Freezing creates an immutable view of a specific object,
collection, or realm that still exists on disk and does not
need to be deeply copied when passed around to other
threads. You can freely share a frozen object across threads
without concern for thread issues.

Frozen objects are not live and do not automatically update. They are
effectively snapshots of the object state at the time of freezing. When
you freeze a realm all child objects and collections also become
frozen. You can't modify frozen objects, but you can read the primary
key from a frozen object, query a live realm for the underlying
object, and then update that live object instance.

Frozen objects remain valid for as long as the realm that
spawned them stays open. Avoid closing realms that contain frozen
objects until all threads are done working with those frozen objects.

> Warning:
> When working with frozen objects, an attempt to do any of
the following throws an exception:
>
> - Opening a write transaction on a frozen realm.
> - Modifying a frozen object.
> - Adding a change listener to a frozen realm, collection, or object.
>

Once frozen, you cannot unfreeze an object. You
can use `isFrozen()` to check if an object is frozen.
This method is always thread-safe.

To freeze an object, collection, or realm, use the
`freeze()` method:

#### Java

```java
Realm realm = Realm.getInstance(config);

// Get an immutable copy of the realm that can be passed across threads
Realm frozenRealm = realm.freeze();
Assert.assertTrue(frozenRealm.isFrozen());

RealmResults<Frog> frogs = realm.where(Frog.class).findAll();
// You can freeze collections
RealmResults<Frog> frozenFrogs = frogs.freeze();
Assert.assertTrue(frozenFrogs.isFrozen());

// You can still read from frozen realms
RealmResults<Frog> frozenFrogs2 = frozenRealm.where(Frog.class).findAll();
Assert.assertTrue(frozenFrogs2.isFrozen());

Frog frog = frogs.first();
Assert.assertTrue(!frog.getRealm().isFrozen());

// You can freeze objects
Frog frozenFrog = frog.freeze();
Assert.assertTrue(frozenFrog.isFrozen());
// Frozen objects have a reference to a frozen realm
Assert.assertTrue(frozenFrog.getRealm().isFrozen());

```

#### Kotlin

```kotlin
val realm = Realm.getInstance(config)

// Get an immutable copy of the realm that can be passed across threads
val frozenRealm = realm.freeze()
Assert.assertTrue(frozenRealm.isFrozen)
val frogs = realm.where(Frog::class.java).findAll()
// You can freeze collections
val frozenFrogs = frogs.freeze()
Assert.assertTrue(frozenFrogs.isFrozen)

// You can still read from frozen realms
val frozenFrogs2 =
    frozenRealm.where(Frog::class.java).findAll()
Assert.assertTrue(frozenFrogs2.isFrozen)
val frog: Frog = frogs.first()!!
Assert.assertTrue(!frog.realm.isFrozen)

// You can freeze objects
val frozenFrog: Frog = frog.freeze()
Assert.assertTrue(frozenFrog.isFrozen)
Assert.assertTrue(frozenFrog.realm.isFrozen)

```

> Important:
> Frozen objects preserve an entire copy of the realm that contains
them at the moment they were frozen. As a result, freezing a large
number of objects can cause a realm to consume more memory and
storage than it might have without frozen objects. If you need to
separately freeze a large number of objects for long periods of time,
consider copying what you need out of the realm instead.
>

## Refreshing Realms
When you open a realm, it reflects the most recent successful write
commit and remains on that version until it is **refreshed**. This means
that the realm will not see changes that happened on another thread
until the next refresh. Realms on any event loop thread
(including the UI thread) automatically refresh themselves at the
beginning of that thread's loop. However, you must manually refresh
realm instances that are tied to non-looping threads or that have
auto-refresh disabled. To refresh a realm, call
`Realm.refresh()`:

#### Java

```java
if (!realm.isAutoRefresh()) {
    // manually refresh
    realm.refresh();
}

```

#### Kotlin

```kotlin
if (!realm.isAutoRefresh) {
    // manually refresh
    realm.refresh()
}

```

> Tip:
> Realms also automatically refresh after completing a write transaction.
>

## Realm's Threading Model in Depth
Realm provides safe, fast, lock-free, and concurrent access
across threads with its [Multiversion Concurrency
Control (MVCC)](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)
architecture.

### Compared and Contrasted with Git
If you are familiar with a distributed version control
system like [Git](https://git-scm.com/), you may already
have an intuitive understanding of MVCC. Two fundamental
elements of Git are:

- Commits, which are atomic writes.
- Branches, which are different versions of the commit history.

Similarly, Realm has atomically-committed writes in the form
of transactions. Realm also has many
different versions of the history at any given time, like
branches.

Unlike Git, which actively supports distribution and
divergence through forking, a realm only has one true latest
version at any given time and always writes to the head of
that latest version. Realm cannot write to a previous
version. This makes sense: your data should converge on one
latest version of the truth.

### Internal Structure
A realm is implemented using a [B+ tree](https://en.wikipedia.org/wiki/B%2B_tree) data structure. The top-level node represents a
version of the realm; child nodes are objects in that
version of the realm. The realm has a pointer to its latest
version, much like how Git has a pointer to its HEAD commit.

Realm uses a copy-on-write technique to ensure
[isolation](https://en.wikipedia.org/wiki/Isolation_(database_systems)) and
[durability](https://en.wikipedia.org/wiki/Durability_(database_systems)).
When you make changes, Realm copies the relevant part of the
tree for writing, then commits the changes in two phases:

- Write changes to disk and verify success.
- Set the latest version pointer to point to the newly-written version.

This two-step commit process guarantees that even if the
write failed partway, the original version is not corrupted
in any way because the changes were made to a copy of the
relevant part of the tree. Likewise, the realm's root
pointer will point to the original version until the new
version is guaranteed to be valid.

Realm uses zero-copy techniques
like memory mapping to handle data. When you read a value
from the realm, you are virtually looking at the value on
the actual disk, not a copy of it. This is the basis for
live objects. This is also why a realm
head pointer can be set to point to the new version after
the write to disk has been validated.

## Summary
- Realm enables simple and safe multithreaded code when you follow
these rules: Don't pass live objects to other threads, and don't lock to read.
- In order to see changes made on other threads in your realm
instance, you must manually **refresh** realm instances that do
not exist on "loop" threads or that have auto-refresh disabled.
- For apps based on reactive, event-stream-based architectures, you can
**freeze** objects, collections, and realms in order to pass
copies around efficiently to different threads for processing.
- Realm's multiversion concurrency control (MVCC)
architecture is similar to Git's. Unlike Git, Realm has
only one true latest version for each realm.
- Realm commits in two stages to guarantee isolation and
durability.
