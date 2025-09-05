# RealmSet - Java SDK
> Version added: 10.6.0

You can use the `RealmSet` data type
to manage a collection of unique keys. `RealmSet` implements Java's
`Set` interface, so it works just like the built-in `HashSet` class,
except managed `RealmSet` instances persist their contents to a
realm. `RealmSet` instances that contain Realm objects
actually only store references to those objects, so deleting a
Realm object from a realm also deletes that object from
any `RealmSet` instances that contain the object.

Because `RealmSet` implements `RealmCollection`, it has some useful
mathematical methods, such as `sum`, `min`, and `max`. For a complete
list of available `RealmSet` methods, see: [the RealmSet API
reference](https://www.mongodb.com/docs/realm-sdks/java/latest/io/realm/RealmSet.html).

## Method Limitations
You cannot use the following `Realm` methods on objects that contain
a field of type `RealmSet`:

- `Realm.insert()`
- `Realm.insertOrUpdate()`
- `Realm.createAllFromJson()`
- `Realm.createObjectFromJson()`
- `Realm.createOrUpdateAllFromJson()`
- `Realm.createOrUpdateObjectFromJson()`

## Usage
To create a field of type `RealmSet`, define an object property of
type `RealmSet<E>`, where `E` defines the keys you would like to
store in your `RealmSet`.

- Add an object to a `RealmSet` with
`RealmSet.add()`
- Add multiple objects with
`RealmSet.addAll()`
- Check if the set contains a specific object with
`RealmSet.contains()`
- Check if the set contains all of multiple objects with
`RealmSet.containsAll()`

#### Java

```java
import io.realm.RealmObject;
import io.realm.RealmSet;

public class Frog extends RealmObject {
    String name;
    RealmSet<Snack> favoriteSnacks;
    // realm-required empty constructor
    public Frog() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public RealmSet<Snack> getFavoriteSnacks() { return favoriteSnacks; }
    public void setFavoriteSnacks(RealmSet<Snack> favoriteSnacks) { this.favoriteSnacks = favoriteSnacks; }
}

```

```java
import io.realm.RealmObject;

public class Snack extends RealmObject {
    private String name;
    public Snack() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

```

```java
Frog frog = realm.createObject(Frog.class);
frog.setName("George Washington");

// get the RealmSet field from the object we just created
RealmSet<Snack> set = frog.getFavoriteSnacks();

// add value to the RealmSet
Snack flies = realm.createObject(Snack.class);
flies.setName("flies");
set.add(flies);

// add multiple values to the RealmSet
Snack water = realm.createObject(Snack.class);
water.setName("water");
Snack verySmallRocks = realm.createObject(Snack.class);
verySmallRocks.setName("verySmallRocks");
set.addAll(Arrays.asList(water, verySmallRocks));

// check for the presence of a key with contains
Assert.assertTrue(set.contains(flies));

// check for the presence of multiple keys with containsAll
Snack biscuits = realm.createObject(Snack.class);
biscuits.setName("biscuits");
Assert.assertTrue(set.containsAll(Arrays.asList(water, biscuits)) == false);

// remove string from a set
set.remove(verySmallRocks);

// set no longer contains that string
Assert.assertTrue(set.contains(verySmallRocks) == false);

// deleting a Realm object also removes it from any RealmSets
int sizeOfSetBeforeDelete = set.size();
flies.deleteFromRealm();
// deleting flies object reduced the size of the set by one
Assert.assertTrue(sizeOfSetBeforeDelete == set.size() + 1);

```

#### Kotlin

```kotlin
import io.realm.RealmObject
import io.realm.RealmSet

open class Frog
    : RealmObject() {
    var name: String = ""
    var favoriteSnacks: RealmSet<Snack> = RealmSet<Snack>();
}

```

```kotlin
import io.realm.RealmObject

open class Snack : RealmObject() {
    var name: String? = null
}

```

```kotlin
val frog = realm.createObject(Frog::class.java)
frog.name = "Jonathan Livingston Applesauce"

// get the RealmSet field from the object we just created
val set = frog.favoriteSnacks

// add value to the RealmSet
val flies = realm.createObject(Snack::class.java)
flies.name = "flies"
set.add(flies)

// add multiple values to the RealmSet
val water = realm.createObject(Snack::class.java)
water.name = "water"
val verySmallRocks = realm.createObject(Snack::class.java)
verySmallRocks.name = "verySmallRocks"
set.addAll(listOf(water, verySmallRocks))

// check for the presence of a key with contains
Assert.assertTrue(set.contains(flies))

// check for the presence of multiple keys with containsAll
val biscuits = realm.createObject(Snack::class.java)
biscuits.name = "biscuits"
Assert.assertTrue(set.containsAll(Arrays.asList(water, biscuits)) == false)

// remove string from a set
set.remove(verySmallRocks)

// set no longer contains that string
Assert.assertTrue(set.contains(verySmallRocks) == false)

// deleting a Realm object also removes it from any RealmSets
val sizeOfSetBeforeDelete = set.size
flies.deleteFromRealm()
// deleting flies object reduced the size of the set by one
Assert.assertTrue(sizeOfSetBeforeDelete == set.size + 1)

```

## Notifications
To subscribe to changes to a `RealmSet`, pass a
`SetChangeListener`
implementation to the `RealmSet.addChangeListener` method.
Your `SetChangeListener` implementation must define an
`onChange()` method, which accepts a reference to the changed `RealmSet`
and a set of changes as parameters. You can access the number of items
added to the set as well as the number of items removed from the set
through the `SetChangeSet` parameter.

#### Java

```java
AtomicReference<Frog> frog = new AtomicReference<Frog>();
realm.executeTransaction(r -> {
    frog.set(realm.createObject(Frog.class));
    frog.get().setName("Jonathan Livingston Applesauce");
});

SetChangeListener<Snack> setChangeListener = new SetChangeListener<Snack>() {
    @Override
    public void onChange(@NotNull RealmSet<Snack> set, SetChangeSet changes) {
        Log.v("EXAMPLE", "Set changed: " +
                changes.getNumberOfInsertions() + " new items, " +
                changes.getNumberOfDeletions() + " items removed.");
    }
};
frog.get().getFavoriteSnacks().addChangeListener(setChangeListener);

realm.executeTransaction(r -> {
    // get the RealmSet field from the object we just created
    RealmSet<Snack> set = frog.get().getFavoriteSnacks();

    // add value to the RealmSet
    Snack flies = realm.createObject(Snack.class);
    flies.setName("flies");
    set.add(flies);

    // add multiple values to the RealmSet
    Snack water = realm.createObject(Snack.class);
    water.setName("water");
    Snack verySmallRocks = realm.createObject(Snack.class);
    verySmallRocks.setName("verySmallRocks");
    set.addAll(Arrays.asList(water, verySmallRocks));

});

```

#### Kotlin

```kotlin
var frog :Frog? = null
realm.executeTransaction { r: Realm? ->
    frog = realm.createObject(Frog::class.java)
    frog?.name = "Jonathan Livingston Applesauce"
}

val setChangeListener: SetChangeListener<Snack>
        = SetChangeListener<Snack> { set, changes ->
    Log.v("EXAMPLE", "Set changed: " +
            changes.numberOfInsertions + " new items, " +
            changes.numberOfDeletions + " items removed.")
}
frog?.favoriteSnacks?.addChangeListener(setChangeListener)

realm.executeTransaction { r: Realm? ->
    // get the RealmSet field from the object we just created
    val set = frog!!.favoriteSnacks

    // add value to the RealmSet
    val flies = realm.createObject(Snack::class.java)
    flies.name = "flies"
    set.add(flies)

    // add multiple values to the RealmSet
    val water = realm.createObject(Snack::class.java)
    water.name = "water"
    val verySmallRocks = realm.createObject(Snack::class.java)
    verySmallRocks.name = "verySmallRocks"
    set.addAll(Arrays.asList(water, verySmallRocks))
}

```

