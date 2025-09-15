# Work with Realm Files - Java SDK
A **realm** is a set of related objects that conform to a pre-defined
schema. Realms may contain more than one type of data as long as a
schema exists for each type.

Every realm stores data in a separate realm file that
contains a binary encoding of each object in the realm. You can
automatically synchronize realm across multiple
devices and set up reactive
event handlers that call a
function any time an object in a realm is created,
modified, or deleted.

## The Realm Lifecycle
Every realm instance consumes a significant amount of resources.
Opening and closing a realm are both expensive operations, but
keeping a realm open also incurs significant resource overhead. To
maximize the performance of your application, you should minimize the
number of open realms at any given time and limit the number of
open and close operations used.

However, opening a realm is not always consistently expensive.
If the realm is already open within the same process or thread,
opening an additional instance requires fewer resources:

- If the realm is not open within the same process, opening the
realm is expensive.
- If the realm is already open on a different thread within the
same process, opening the realm is less expensive, but still
nontrivial.
- If the realm is already open on the same thread within the same
process, opening the realm requires minimal additional resources.

When you open a realm for the first time, Realm
performs the memory-mapping and schema validation required to read and
write data to the realm. Additional instances of that
realm on the same thread use the same underlying resources.
Instances of that realm on separate threads use some of the same
underlying resources.

When all connections to a realm are closed in
a thread, Realm frees the thread resources used to
connect to that realm. When all connections to a realm are
closed in a process, Realm frees all resources used to
connect to that realm.

As a best practice, we recommend tying the realm instance
lifecycle to the lifecycles of the views that observe the realm. For
instance, consider a `RecyclerView` that displays `RealmResults`
data via a `Fragment`. You could:

- Open a single realm that contains the data for that view
in the `Fragment.onCreateView()` lifecycle method.
- Close that same realm in the `Fragment.onDestroyView()`
lifecycle method.

> Note:
> If your realm is especially large, fetching a realm instance
in `Fragment.onCreateView()` may briefly block rendering. If
opening your realm in `onCreateView()` causes performance
issues, consider managing the realm from `Fragment.onStart()`
and `Fragment.onStop()` instead.
>

If multiple `Fragment` instances require access to the same dataset,
you could manage a single realm in the enclosing `Activity`:

- Open the realm in the `Activity.onCreate()` lifecycle method.
- Close the realm in the `Activity.onDestroy()` lifecycle method.

## Multi-process
You cannot access encrypted or
[delete me]s
simultaneously from different processes. However, local realms
function normally across processes, so you can read, write, and
receive notifications from multiple APKs.

## Realm Schema
A **Realm Schema** is a list of valid object schemas that each define an object type that an App
may persist. All objects in a realm must conform to the Realm Schema.

By default, the SDK automatically adds all classes in your project
that derive from `RealmObject` to the
realm schema.

Client applications provide a Realm Schema when they open a
realm. If a realm already contains data, then Realm
validates each existing object to ensure that an object schema was
provided for its type and that it meets all of the constraints specified
in the schema.

> Example:
> A realm that contains basic data about books in libraries might use a
schema like the following:
>
> ```json
> [
>   {
>     "type": "Library",
>     "properties": {
>       "address": "string",
>       "books": "Book[]"
>     }
>   },
>   {
>     "type": "Book",
>     "primaryKey": "isbn",
>     "properties": {
>       "isbn": "string",
>       "title": "string",
>       "author": "string",
>       "numberOwned": { "type": "int?", "default": 0 },
>       "numberLoaned": { "type": "int?", "default": 0 }
>     }
>   }
> ]
> ```
>

## Find Your Realm File
Realm stores a binary encoded version of every object
and type in a realm in a single `.realm` file.

The filesystem used by Android emulators is not directly accessible
from the machine running Realm Studio. You must download the file
from the emulator before you can access it.

First, find the path of the file on the emulator:

```java
// Run this on the device to find the path on the emulator
Realm realm = Realm.getDefaultInstance();
Log.i("Realm", realm.getPath());
```

Then, download the file using ADB. You can do this while the app
is running.

```java
> adb pull <path>
```

You can also upload the modified file again using ADB, but only
when the app isn't running. Uploading a modified file while the
app is running can corrupt the file.

```java
> adb push <file> <path>
```

> Seealso:
> Realm creates additional files for each realm.
To learn more about these files, see Realm Internals.
>

## Realm File Size
Realm usually takes up less space on disk than an
equivalent SQLite database. However, in order to give you a consistent
view of your data, Realm operates on multiple versions of a
realm. If many versions of a realm are opened simultaneously,
the realm file can require additional space on disk.

These versions take up an amount of space dependent on the amount of
changes in each transaction. Many small transactions have the same
overhead as a small number of large transactions.

Unexpected file size growth usually happens for one of three reasons:

1. *You open a realm on a background thread and forget to close it
again.* As a result, Realm retains a reference to the
older version of data on the background thread. Because
Realm automatically updates realms to the most
recent version on threads with loopers, the UI thread and other
Looper threads do not have this problem.
2. *You hold references to too many versions of frozen objects.*
Frozen objects preserve the version of a realm that existed when
the object was first frozen. If you need to freeze a large number of
objects, consider using `Realm.copyFromRealm()` instead to only preserve the
data you need.
3. *You read some data from a realm. Then, you block the thread with
a long-running operation. Meanwhile, you write many times to the
realm on other threads.* This causes Realm to
create many intermediate versions. You can avoid this by: batching the
writes, avoiding leaving the realm open while otherwise blocking the
background thread.

### Limit the Maximum Number of Active Versions
You can set `maxNumberOfActiveVersions()`
when building your `RealmConfiguration` to throw an
`IllegalStateException` if your application opens more versions of
a realm than the permitted number. Versions are created when
executing a write transaction.

Realm automatically removes older versions of data once
they are no longer used by your application. However,
Realm does not free the space used by older versions of
data; instead, that space is used for new writes to the realm.

### Compact a Realm
You can remove unused space by **compacting** the realm file:

- Manually: call `compactRealm()`
- Automatically: specify the `compactOnLaunch()`
builder option when opening the first connection to a realm in your
Android application

> Important:
> Every production application should implement compacting to
periodically reduce realm file size.
>

## Backup and Restore Realms
Realm persists realms to disk using files on your
Android device. To back up a realm, find your realm file and copy it to a safe location. You should close
all instances of the realm before copying it.

Alternatively, you can also use `realm.writeCopyTo()` to write a compacted
version of a realm to a destination file.

> Seealso:
> If you want to back up a realm to an external location like
Google Drive, see the following article series: ([Part 1](https://medium.com/glucosio-project/example-class-to-export-import-a-realm-database-on-java-c429ade2b4ed#.80ibsc7wm),
[Part 2](https://medium.com/glucosio-project/backup-restore-a-realm-database-on-google-drive-with-drive-api-c238515a5975#.qbuugb322),
[Part 3](https://medium.com/glucosio-project/build-a-nice-ux-to-backup-and-sync-your-app-data-on-google-drive-3-3-a3b598cab68b#.5mjk4w4se)).
>

## Modules
Realm Modules describe the set of Realm objects
that can be stored in a realm. By default, Realm
automatically creates a Realm Module that contains all
Realm objects defined in your application.
You can define a `RealmModule`
to restrict a realm to a subset of classes defined in an application.
If you produce a library that uses Realm, you can use a
Realm Module to explicitly include only the Realm
objects defined in your library in your realm. This allows
applications that include your library to also use Realm
without managing object name conflicts and migrations with your library's
defined Realm objects.
