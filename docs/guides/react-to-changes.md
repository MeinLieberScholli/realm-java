# React to Changes - Java SDK
Objects in Realm clients are **live objects** that
update automatically to reflect data changes and emit
notification events that you
can subscribe to whenever their underlying data changes.

Any modern app should be able to react when data changes,
regardless of where that change originated. When a user adds
a new item to a list, you may want to update the UI, show a
notification, or log a message. When someone updates that
item, you may want to change its visual state or fire off a
network request. Finally, when someone deletes the item, you
probably want to remove it from the UI. Realm's notification
system allows you to watch for and react to changes in your
data, independent of the writes that caused the changes.

Realm emits three kinds of notifications:

- Realm notifications whenever a specific realm commits a write transaction.
- Collection notifications whenever any Realm object in a collection changes, including inserts, updates, and deletes.
- Object notifications whenever a specific Realm object changes, including updates and deletes.

## Auto-Refresh
Realm objects accessed on a thread associated with a
[Looper](https://developer.android.com/reference/android/os/Looper.html) automatically
update periodically to reflect changes to underlying data.

The Android UI thread always contains a `Looper` instance. If you need
to keep Realm objects around for long periods of time on
any other thread, you should configure a `Looper` for that thread.

> Warning:
> Realms on a thread without a [Looper](https://developer.android.com/reference/android/os/Looper)
do not automatically advance their version. This can increase the size of the
realm in memory and on disk. Avoid using realm instances on
non-Looper threads when possible. If you *do* open a realm on a non-Looper
thread, close the realm when you're done using it.
>

## Register a Realm Change Listener
You can register a notification handler on an entire realm.
Realm calls the notification handler whenever any write
transaction involving that realm is committed. The
handler receives no information about the change.

This is useful when you want to know that there has been a
change but do not care to know specifically what changed.
For example, proof of concept apps often use this
notification type and simply refresh the entire UI when
anything changes. As the app becomes more sophisticated and
performance-sensitive, the app developers shift to more
granular notifications.

> Example:
> Suppose you are writing a real-time collaborative app. To
give the sense that your app is buzzing with collaborative
activity, you want to have an indicator that lights up when
any change is made. In that case, a realm notification
handler would be a great way to drive the code that controls
the indicator. The following code shows how to observe a realm
for changes with with `addChangeListener()`:
>
> #### Java
>
> ```java
> public class MyActivity extends Activity {
>     private Realm realm;
>     private RealmChangeListener realmListener;
>
>     @Override
>     protected void onCreate(Bundle savedInstanceState) {
>         super.onCreate(savedInstanceState);
>         realm = Realm.getDefaultInstance();
>         realmListener = new RealmChangeListener<Realm>() {
>             @Override
>             public void onChange(Realm realm) {
>               // ... do something with the updates (UI, etc.) ...
>             }
>           };
>         // Observe realm notifications.
>         realm.addChangeListener(realmListener);
>     }
>
>     @Override
>     protected void onDestroy() {
>         super.onDestroy();
>         // Remove the listener.
>         realm.removeChangeListener(realmListener);
>         // Close the Realm instance.
>         realm.close();
>     }
> }
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> class MyActivity : Activity() {
>     private lateinit var realm: Realm
>     private lateinit var realmListener: RealmChangeListener<Realm>
>
>     override fun onCreate(savedInstanceState: Bundle?) {
>         super.onCreate(savedInstanceState)
>         realm = Realm.getDefaultInstance()
>         realmListener = RealmChangeListener {
>             // ... do something with the updates (UI, etc.) ...
>         }
>         // Observe realm notifications.
>         realm.addChangeListener(realmListener)
>     }
>
>     override fun onDestroy() {
>         super.onDestroy()
>         // Remove the listener.
>         realm.removeChangeListener(realmListener)
>         // Close the Realm instance.
>         realm.close()
>     }
> }
>
> ```
>
>

> Important:
> All threads that contain a `Looper` automatically refresh
`RealmObject` and `RealmResult` instances when new changes are
written to the realm. As a result, it isn't necessary to fetch
those objects again when reacting to a `RealmChangeListener`, since
those objects are already updated and ready to be redrawn to the
screen.
>

## Register a Collection Change Listener
You can register a notification handler on a specific
collection within a realm. The handler receives a
description of changes since the last notification.
Specifically, this description consists of three lists of
indices:

- The indices of the objects that were deleted.
- The indices of the objects that were inserted.
- The indices of the objects that were modified.

Stop notification delivery by calling the `removeChangeListener()` or
`removeAllChangeListeners()` methods. Notifications also stop if:

- the object on which the listener is registered gets garbage collected.
- the realm instance closes.

Keep a strong reference to the object you're listening to
for as long as you need the notifications.

> Important:
> In collection notification handlers, always apply changes
in the following order: deletions, insertions, then
modifications. Handling insertions before deletions may
result in unexpected behavior.
>

Realm emits an initial notification after retrieving the
collection. After that, Realm delivers collection
notifications asynchronously whenever a write transaction
adds, changes, or removes objects in the collection.

Unlike realm notifications, collection notifications contain
detailed information about the change. This enables
sophisticated and selective reactions to changes. Collection
notifications provide all the information needed to manage a
list or other view that represents the collection in the UI.

The following code shows how to observe a collection for
changes with `addChangeListener()`:

#### Java

```java
RealmResults<Dog> dogs = realm.where(Dog.class).findAll();
// Set up the collection notification handler.
OrderedRealmCollectionChangeListener<RealmResults<Dog>> changeListener = (collection, changeSet) -> {
    // For deletions, notify the UI in reverse order if removing elements the UI
    OrderedCollectionChangeSet.Range[] deletions = changeSet.getDeletionRanges();
    for (int i = deletions.length - 1; i >= 0; i--) {
        OrderedCollectionChangeSet.Range range = deletions[i];
        Log.v("EXAMPLE", range.length + " dogs deleted at " + range.startIndex);
    }
    OrderedCollectionChangeSet.Range[] insertions = changeSet.getInsertionRanges();
    for (OrderedCollectionChangeSet.Range range : insertions) {
        Log.v("EXAMPLE", range.length + " dogs inserted at " + range.startIndex);
    }
    OrderedCollectionChangeSet.Range[] modifications = changeSet.getChangeRanges();
    for (OrderedCollectionChangeSet.Range range : modifications) {
        Log.v("EXAMPLE", range.length + " dogs modified at " + range.startIndex);
    }
};
// Observe collection notifications.
dogs.addChangeListener(changeListener);

```

#### Kotlin

```kotlin
val dogs = realm.where(Dog::class.java).findAll()
// Set up the collection notification handler.
val changeListener =
    OrderedRealmCollectionChangeListener { collection: RealmResults<Dog>?, changeSet: OrderedCollectionChangeSet ->
        // For deletions, notify the UI in reverse order if removing elements the UI
        val deletions = changeSet.deletionRanges
        for (i in deletions.indices.reversed()) {
            val range = deletions[i]
            Log.v("EXAMPLE", "${range.length} dogs deleted at ${range.startIndex}")
        }
        val insertions = changeSet.insertionRanges
        for (range in insertions) {
            Log.v("EXAMPLE", "${range.length} dogs inserted at ${range.startIndex}")
        }
        val modifications = changeSet.changeRanges
        for (range in modifications) {
            Log.v("EXAMPLE", "${range.length} dogs modified at ${range.startIndex}")
        }
    }
// Observe collection notifications.
dogs.addChangeListener(changeListener)

```

## Register an Object Change Listener
You can register a notification handler on a specific object
within a realm. Realm notifies your handler:

- When the object is deleted.
- When any of the object's properties change.

The handler receives information about what fields changed
and whether the object was deleted.

Stop notification delivery by calling the `removeChangeListener()` or
`removeAllChangeListeners()` methods. Notifications also stop if:

- the object on which the listener is registered gets garbage collected.
- the realm instance closes.

Keep a strong reference of the object you're listening to
for as long as you need the notifications.

The following code shows how create a new instance of a class
in a realm and observe that instance for changes with
`addChangeListener()`:

#### Java

```java
// Create a dog in the realm.
AtomicReference<Dog> dog = new AtomicReference<Dog>();
realm.executeTransaction(transactionRealm -> {
    dog.set(transactionRealm.createObject(Dog.class, new ObjectId()));
    dog.get().setName("Max");
});

// Set up the listener.
RealmObjectChangeListener<Dog> listener = (changedDog, changeSet) -> {
    if (changeSet.isDeleted()) {
        Log.i("EXAMPLE", "The dog was deleted");
        return;
    }
    for (String fieldName : changeSet.getChangedFields()) {
        Log.i("EXAMPLE", "Field '" + fieldName + "' changed.");
    }
};

// Observe object notifications.
dog.get().addChangeListener(listener);

// Update the dog to see the effect.
realm.executeTransaction(r -> {
    dog.get().setName("Wolfie"); // -> "Field 'name' was changed."
});

```

#### Kotlin

```kotlin
// Create a dog in the realm.
var dog = Dog()
realm.executeTransaction { transactionRealm ->
    dog = transactionRealm.createObject(Dog::class.java, ObjectId())
    dog.name = "Max"
}

// Set up the listener.
val listener = RealmObjectChangeListener { changedDog: Dog?, changeSet: ObjectChangeSet? ->
    if (changeSet!!.isDeleted) {
        Log.i("EXAMPLE", "The dog was deleted")
    } else {
        for (fieldName in changeSet.changedFields) {
            Log.i(
                "EXAMPLE",
                "Field '$fieldName' changed."
            )
        }
    }
}

// Observe object notifications.
dog.addChangeListener(listener)

// Update the dog to see the effect.
realm.executeTransaction { r: Realm? ->
    dog.name = "Wolfie" // -> "Field 'name' was changed."
}

```

## Unregister a Change Listener
You can unregister a change listener by passing your change listener to
`Realm.removeChangeListener()`.
You can unregister all change listeners currently subscribed to changes
in a realm or any of its linked objects or collections with
`Realm.removeAllChangeListeners()`.

## Use Realm in System Apps on Custom ROMs
Realm uses named pipes in order to support notifications and
access to the realm file from multiple processes. While this is
allowed by default for normal user apps, it is disallowed for system
apps.

You can define a system apps by setting
`android:sharedUserId="android.uid.system"` in the Android manifest.
When working with a system app, you may see a security violation in
Logcat that looks something like this:

```
05-24 14:08:08.984  6921  6921 W .realmsystemapp: type=1400 audit(0.0:99): avc: denied { write } for name="realm.testapp.com.realmsystemapp-Bfqpnjj4mUvxWtfMcOXBCA==" dev="vdc" ino=14660 scontext=u:r:system_app:s0 tcontext=u:object_r:apk_data_file:s0 tclass=dir permissive=0
05-24 14:08:08.984  6921  6921 W .realmsystemapp: type=1400 audit(0.0:100): avc: denied { write } for name="realm.testapp.com.realmsystemapp-Bfqpnjj4mUvxWtfMcOXBCA==" dev="vdc" ino=14660 scontext=u:r:system_app:s0 tcontext=u:object_r:apk_data_file:s0 tclass=dir permissive=0
```

In order to fix this you need to adjust the SELinux security rules in
the ROM. This can be done by using the tool `audit2allow`, which ships
as part of AOSP:

1. Pull the current policy from the device: `adb pull /sys/fs/selinux/policy`
2. Copy the SELinux error inside a text file called input.txt.
3. Run the `audit2allow` tool: `audit2allow -p policy -i input.txt`
4. The tool should output a rule you can add to your existing policy
to enable the use of Realm.

An example of such a policy is provided below:

```
# Allow system_app to create named pipes required by Realm
# Credit: https://github.com/mikalackis/platform_vendor_ariel/blob/master_oreo/sepolicy/system_app.te
allow system_app fuse:fifo_file create;
allow system_app system_app_data_file:fifo_file create;
allow system_app system_app_data_file:fifo_file { read write };
allow system_app system_app_data_file:fifo_file open;
```

> Seealso:
> `audit2allow` is produced when compiling AOSP/ROM and only runs on
Linux. You can read more about it [here](https://source.android.com/security/selinux/validate#using_audit2allow).
>

> Note:
> Since Android Oreo, Google changed the way it configures SELinux.
The default security policies are now much more modularized.
Read more about that
[here](https://source.android.com/security/selinux/images/SELinux_Treble.pdf).
>

## Change Notification Limits
Changes in nested documents deeper than four levels down do not trigger
change notifications.

If you have a data structure where you need to listen for changes five
levels down or deeper, workarounds include:

- Refactor the schema to reduce nesting.
- Add something like "push-to-refresh" to enable users to manually refresh data.
