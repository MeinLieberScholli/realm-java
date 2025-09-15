# RealmDictionary - Java SDK
> Version added: 10.6.0

You can use the `RealmDictionary` data type to manage a collection of
unique `String` keys paired with values. `RealmDictionary`
implements Java's `Map` interface, so it works just like the built-in
`HashMap` class, except managed `RealmDictionary` instances persist
their contents to a realm. `RealmDictionary` instances that
contain Realm objects store references to those objects.
When you delete a Realm object from a realm, any
references to that object in a `RealmDictionary` become `null`
values.

## Usage
To create a field of type `RealmDictionary`, define an object property
of type `RealmDictionary<T>`, where `T` defines the values you would
like to store in your `RealmDictionary`. Currently, `RealmDictionary`
instances can only use keys of type `String`.

The following table shows which methods you can use to complete common
collection tasks with `RealmDictionary`:

|Task|Method|
| --- | --- |
|Add an object to a `RealmDictionary`|`put()` (or the `[]` operator in Kotlin)|
|Add multiple objects to a `RealmDictionary`|`putAll()`|
|Check if the dictionary contains an specific key|`containsKey()`|
|Check if the dictionary contains a specific value|`containsValue()`|

#### Java

```java
import io.realm.RealmDictionary;
import io.realm.RealmObject;

public class Frog extends RealmObject {
    String name;
    RealmDictionary<Frog> nicknamesToFriends;
    // realm-required empty constructor
    public Frog() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public RealmDictionary<Frog> getNicknamesToFriends() { return nicknamesToFriends; }
    public void setNicknamesToFriends(RealmDictionary<Frog> nicknamesToFriends) { this.nicknamesToFriends = nicknamesToFriends; }
}

```

```java
Frog frog = realm.createObject(Frog.class);
frog.setName("George Washington");

// get the RealmDictionary field from the object we just created
RealmDictionary<Frog> dictionary = frog.getNicknamesToFriends();

// add key/value to the dictionary
Frog wirt = realm.createObject(Frog.class);
wirt.setName("Wirt");
dictionary.put("tall frog", wirt);

// add multiple keys/values to the dictionary
Frog greg = realm.createObject(Frog.class);
greg.setName("Greg");
Frog beatrice = realm.createObject(Frog.class);
beatrice.setName("Beatrice");
dictionary.putAll(Map.of("small frog", greg, "feathered frog", beatrice));

// check for the presence of a key
Assert.assertTrue(dictionary.containsKey("small frog"));

// check for the presence of a value
Assert.assertTrue(dictionary.containsValue(greg));

// remove a key
dictionary.remove("feathered frog");
Assert.assertFalse(dictionary.containsKey("feathered frog"));

// deleting a Realm object does NOT remove it from the dictionary
int sizeOfDictionaryBeforeDelete = dictionary.size();
greg.deleteFromRealm();
// deleting greg object did not reduce the size of the dictionary
Assert.assertEquals(sizeOfDictionaryBeforeDelete, dictionary.size());
// but greg object IS now null:
Assert.assertEquals(dictionary.get("small frog"), null);

```

#### Kotlin

```kotlin
import io.realm.RealmDictionary
import io.realm.RealmObject

open class Frog
    : RealmObject() {
    var name: String? = null
    var nicknamesToFriends: RealmDictionary<Frog> = RealmDictionary<Frog>()
}

```

```kotlin
val frog =
    realm.createObject(Frog::class.java)
frog.name = "George Washington"

// get the RealmDictionary field from the object we just created
val dictionary = frog.nicknamesToFriends

// add key/value to the dictionary
val wirt =
    realm.createObject(Frog::class.java)
wirt.name = "Wirt"
dictionary["tall frog"] = wirt

// add multiple keys/values to the dictionary
val greg =
    realm.createObject(Frog::class.java)
greg.name = "Greg"
val beatrice =
    realm.createObject(Frog::class.java)
beatrice.name = "Beatrice"
dictionary.putAll(mapOf<String, Frog>(
    Pair("small frog", greg),
    Pair("feathered frog", beatrice)))

// check for the presence of a key
Assert.assertTrue(dictionary.containsKey("small frog"))

// check for the presence of a value
Assert.assertTrue(dictionary.containsValue(greg))

// remove a key
dictionary.remove("feathered frog")
Assert.assertFalse(dictionary.containsKey("feathered frog"))

// deleting a Realm object does NOT remove it from the dictionary
val sizeOfDictionaryBeforeDelete = dictionary.size
greg.deleteFromRealm()
// deleting greg object did not reduce the size of the dictionary
Assert.assertEquals(
    sizeOfDictionaryBeforeDelete.toLong(),
    dictionary.size.toLong()
)
// but greg object IS now null:
Assert.assertEquals(dictionary["small frog"], null)

```

## Notifications
To subscribe to changes to a `RealmDictionary`, pass a
`MapChangeListener`
implementation to the `RealmSet.addChangeListener` method.
Your `MapChangeListener` implementation must define an
`onChange()` method, which accepts a reference to the changed `RealmDictionary`
and a set of changes as parameters. You can access the keys
added to the dictionary as well as the keys removed from the dictionary
through the `MapChangeSet` parameter.

#### Java

```java
AtomicReference<Frog> frog = new AtomicReference<Frog>();
realm.executeTransaction(r -> {
    frog.set(realm.createObject(Frog.class));
    frog.get().setName("Jonathan Livingston Applesauce");
});

MapChangeListener<String, Frog> mapChangeListener =
    new MapChangeListener<String, Frog>() {
        @Override
        public void onChange(RealmMap<String, Frog> map,
                             MapChangeSet<String> changes) {
            for (String insertion : changes.getInsertions()) {
                Log.v("EXAMPLE",
                        "Inserted key:  " + insertion +
                                ", Inserted value: " + map.get(insertion).getName());
            }
        }
    };

frog.get().getNicknamesToFriends().addChangeListener(mapChangeListener);

realm.executeTransaction(r -> {
    // get the RealmDictionary field from the object we just created
    RealmDictionary<Frog> dictionary = frog.get().getNicknamesToFriends();

    // add key/value to the dictionary
    Frog wirt = realm.createObject(Frog.class);
    wirt.setName("Wirt");
    dictionary.put("tall frog", wirt);

    // add multiple keys/values to the dictionary
    Frog greg = realm.createObject(Frog.class);
    greg.setName("Greg");
    Frog beatrice = realm.createObject(Frog.class);
    beatrice.setName("Beatrice");
    dictionary.putAll(Map.of("small frog", greg, "feathered frog", beatrice));

});

```

#### Kotlin

```kotlin
var frog: Frog? = null
realm.executeTransaction { r: Realm? ->
    frog = realm.createObject(Frog::class.java)
    frog?.name = "Jonathan Livingston Applesauce"
}

val mapChangeListener: MapChangeListener<String, Frog>
        = MapChangeListener<String, Frog> { map, changes ->
    for (insertion in changes.insertions) {
        Log.v("EXAMPLE",
                "Inserted key:  $insertion, Inserted value: ${map[insertion]!!.name}")
    }
}

frog?.nicknamesToFriends?.addChangeListener(mapChangeListener)

realm.executeTransaction { r: Realm? ->
    // get the RealmDictionary field from the object we just created
    val dictionary = frog!!.nicknamesToFriends

    // add key/value to the dictionary
    val wirt = realm.createObject(Frog::class.java)
    wirt.name = "Wirt"
    dictionary["tall frog"] = wirt

    // add multiple keys/values to the dictionary
    val greg = realm.createObject(Frog::class.java)
    greg.name = "Greg"
    val beatrice = realm.createObject(Frog::class.java)
    beatrice.name = "Beatrice"
    dictionary.putAll(mapOf<String, Frog>(
            Pair("small frog", greg),
            Pair("feathered frog", beatrice)))
}

```

