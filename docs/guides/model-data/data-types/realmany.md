# RealmAny - Java SDK
> Version added: 10.6.0

You can use the `RealmAny` data type to create
Realm object fields that can contain any of several
underlying types. You can store multiple `RealmAny` instances in
`RealmList`, `RealmDictionary`, or `RealmSet` fields. To change
the value of a `RealmAny` field, assign a new `RealmAny` instance
with a different underlying value. `RealmAny` fields are indexable, but
cannot be used as primary keys.

> Note:
> `RealmAny` objects can refer to any
supported field type
*except*:
>
> - `RealmAny`
> - `RealmList`
> - `RealmSet`
> - `RealmDictionary`
>

## Usage
To create a `RealmAny` instance, use the
`RealmAny.valueOf()` method
to assign an initial value or `RealmAny.nullValue()` to assign no
value. `RealmAny` instances are immutable just like `String` or
`Integer` instances; if you want to assign a new value to a
`RealmAny` field, you must create a new `RealmAny` instance.

> Warning:
> `RealmAny` instances are always nullable. Additionally, instances can contain a value
of type `RealmAny.Type.NULL`.
>

#### Java

```java
import com.mongodb.realm.examples.model.kotlin.Person;

import io.realm.RealmAny;
import io.realm.RealmObject;

public class Frog extends RealmObject {
    String name;
    RealmAny bestFriend;
    // realm-required empty constructor
    public Frog() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public RealmAny getBestFriend() { return bestFriend; }
    public void setBestFriend(RealmAny bestFriend) { this.bestFriend = bestFriend; }
    public String bestFriendToString() {
        switch(bestFriend.getType()) {
            case NULL: {
                return "no best friend";
            }
            case STRING: {
                return bestFriend.asString();
            }
            case OBJECT: {
                if (bestFriend.getValueClass().equals(Person.class)) {
                    Person person = bestFriend.asRealmModel(Person.class);
                    return person.getName();
                }
            }
            default: {
                return "unknown type";
            }
        }
    }
}

```

```java
  Frog frog = realm.createObject(Frog.class);
  frog.setName("Jonathan Livingston Applesauce");

  // set RealmAny field to a null value
  frog.setBestFriend(RealmAny.nullValue());
  Log.v("EXAMPLE", "Best friend: " + frog.bestFriendToString());

  // possible types for RealmAny are defined in RealmAny.Type
  Assert.assertTrue(frog.getBestFriend().getType() == RealmAny.Type.NULL);

  // set RealmAny field to a string with RealmAny.valueOf a string value
  frog.setBestFriend(RealmAny.valueOf("Greg"));
  Log.v("EXAMPLE", "Best friend: " + frog.bestFriendToString());

  // RealmAny instances change type as you reassign to different values
  Assert.assertTrue(frog.getBestFriend().getType() == RealmAny.Type.STRING);

  // set RealmAny field to a realm object, also with valueOf
  Person person = new Person("Jason Funderburker");

  frog.setBestFriend(RealmAny.valueOf(person));
  Log.v("EXAMPLE", "Best friend: " + frog.bestFriendToString());

  // You can also extract underlying Realm Objects from RealmAny with asRealmModel
  Person bestFriendObject = frog.getBestFriend().asRealmModel(Person.class);
  Log.v("EXAMPLE", "Best friend: " + bestFriendObject.getName());

  // RealmAny fields referring to any Realm Object use the OBJECT type
  Assert.assertTrue(frog.getBestFriend().getType() == RealmAny.Type.OBJECT);

  // you can't put a RealmList in a RealmAny field directly,
  // ...but you can set a RealmAny field to a RealmObject that contains a list
  GroupOfPeople persons = new GroupOfPeople();
  // GroupOfPeople contains a RealmList of people
  persons.getPeople().add("Rand");
  persons.getPeople().add("Perrin");
  persons.getPeople().add("Mat");

  frog.setBestFriend(RealmAny.valueOf(persons));
  Log.v("EXAMPLE", "Best friend: " +
          frog.getBestFriend().asRealmModel(GroupOfPeople.class).getPeople().toString());

```

#### Kotlin

```kotlin
import io.realm.RealmAny
import io.realm.RealmObject

open class Frog(var bestFriend: RealmAny? = RealmAny.nullValue()) : RealmObject() {
    var name: String? = null
    open fun bestFriendToString(): String {
        if (bestFriend == null) {
            return "null"
        }
        return when (bestFriend!!.type) {
            RealmAny.Type.NULL -> {
                "no best friend"
            }
            RealmAny.Type.STRING -> {
                bestFriend!!.asString()
            }
            RealmAny.Type.OBJECT -> {
                if (bestFriend!!.valueClass == Person::class.java) {
                    val person = bestFriend!!.asRealmModel(Person::class.java)
                    person.name
                }
                "unknown type"
            }
            else -> {
                "unknown type"
            }
        }
    }
}

```

```kotlin
val frog = realm.createObject(Frog::class.java)
frog.name = "George Washington"

// set RealmAny field to a null value

// set RealmAny field to a null value
frog.bestFriend = RealmAny.nullValue()
Log.v("EXAMPLE", "Best friend: " + frog.bestFriendToString())

// possible types for RealmAny are defined in RealmAny.Type
Assert.assertEquals(frog.bestFriend?.type, RealmAny.Type.NULL)

// set RealmAny field to a string with RealmAny.valueOf a string value
frog.bestFriend = RealmAny.valueOf("Greg")
Log.v("EXAMPLE", "Best friend: " + frog.bestFriendToString())

// RealmAny instances change type as you reassign to different values
Assert.assertEquals(frog.bestFriend?.type, RealmAny.Type.STRING)

// set RealmAny field to a realm object, also with valueOf
val person = Person("Jason Funderburker")

frog.bestFriend = RealmAny.valueOf(person)
Log.v("EXAMPLE", "Best friend: " + frog.bestFriendToString())

// You can also extract underlying Realm Objects from RealmAny with asRealmModel
val bestFriendObject = frog.bestFriend?.asRealmModel(Person::class.java)
Log.v("EXAMPLE", "Best friend: " + bestFriendObject?.name)

// RealmAny fields referring to any Realm Object use the OBJECT type
Assert.assertEquals(frog.bestFriend?.type, RealmAny.Type.OBJECT)

// you can't put a RealmList in a RealmAny field directly,
// ...but you can set a RealmAny field to a RealmObject that contains a list
val persons = GroupOfPeople()
// GroupOfPeople contains a RealmList of people
persons.people.add("Rand")
persons.people.add("Perrin")
persons.people.add("Mat")

frog.bestFriend = RealmAny.valueOf(persons)
Log.v("EXAMPLE", "Best friend: " +
        frog.bestFriend?.asRealmModel(GroupOfPeople::class.java)
                ?.people.toString())

```

## Queries
You can query a `RealmAny` field just like any other data type.
Operators that only work with certain types, such as string
operators and arithmetic operators, ignore
values that do not contain that type. Negating such operators matches
values that do not contain the type. Type queries match the underlying
type, rather than `RealmAny`. Arithmetic operators convert numeric
values implicitly to compare across types.

## Notifications
To subscribe to changes to a `RealmAny` field, use the
`RealmObject.addChangeListener`
method of the enclosing object. You can use the
`ObjectChangeSet`
parameter to determine if the `RealmAny` field changed.

#### Java

```java
AtomicReference<Frog> frog = new AtomicReference<Frog>();
realm.executeTransaction(r -> {
        frog.set(realm.createObject(Frog.class));
        frog.get().setName("Jonathan Livingston Applesauce");
});

RealmObjectChangeListener<Frog> objectChangeListener =
        new RealmObjectChangeListener<Frog>() {
    @Override
    public void onChange(@NotNull Frog frog, @Nullable ObjectChangeSet changeSet) {
        if (changeSet != null) {
            Log.v("EXAMPLE", "Changes to fields: " +
                    Arrays.toString(changeSet.getChangedFields()));
            if (changeSet.isFieldChanged("best_friend")) {
                Log.v("EXAMPLE", "RealmAny best friend field changed to : " +
                        frog.bestFriendToString());
            }
        }
    }
};

frog.get().addChangeListener(objectChangeListener);

realm.executeTransaction(r -> {
    // set RealmAny field to a null value
    frog.get().setBestFriend(RealmAny.nullValue());
    Log.v("EXAMPLE", "Best friend: " + frog.get().bestFriendToString());

    // set RealmAny field to a string with RealmAny.valueOf a string value
    frog.get().setBestFriend(RealmAny.valueOf("Greg"));

});

```

#### Kotlin

```kotlin
var frog: Frog? = null

realm.executeTransaction { r: Realm? ->
    frog = realm.createObject(Frog::class.java)
    frog?.name = "Jonathan Livingston Applesauce"
}

val objectChangeListener
        = RealmObjectChangeListener<Frog> { frog, changeSet ->
    if (changeSet != null) {
        Log.v("EXAMPLE", "Changes to fields: " +
                changeSet.changedFields)
        if (changeSet.isFieldChanged("best_friend")) {
            Log.v("EXAMPLE", "RealmAny best friend field changed to : " +
                    frog.bestFriendToString())
        }
    }
}

frog?.addChangeListener(objectChangeListener)

realm.executeTransaction { r: Realm? ->
    // set RealmAny field to a null value
    frog?.bestFriend = RealmAny.nullValue()
    Log.v("EXAMPLE", "Best friend: " + frog?.bestFriendToString())

    // set RealmAny field to a string with RealmAny.valueOf a string value
    frog?.bestFriend = RealmAny.valueOf("Greg")
}

```

