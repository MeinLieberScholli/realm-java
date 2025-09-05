# Counters - Java SDK
Realm offers `MutableRealmInteger`, a wrapper around numeric values,
to help better synchronize numeric changes across multiple clients.

Typically, incrementing or decrementing a
`byte`, `short`, `int`, or `long` field of a Realm
object looks something like this:

1. Read the current value of the field.
2. Update that value in memory to a new value based on the increment or
decrement.
3. Write a new value back to the field.

When multiple distributed clients attempt this at the same time,
updates reaching clients in different orders can
result in different values on different clients. `MutableRealmInteger`
improves on this by translating numeric updates into sync operations
that can be executed in any order to converge to the same value.

`MutableRealmInteger` fields are backed by traditional numeric types,
so no migration is required when changing a field from `byte`, `short`,
`int` or `long` to `MutableRealmInteger`.

The following example demonstrates a `MutableRealmInteger` field that
counts the number of ghosts found in a haunted house:

#### Java

```java
import io.realm.MutableRealmInteger;
import io.realm.RealmObject;
import io.realm.annotations.Required;

public class HauntedHouse extends RealmObject {
    @Required
    private final MutableRealmInteger ghosts = MutableRealmInteger.valueOf(0);
    public HauntedHouse() {}
    public MutableRealmInteger getGhosts() { return ghosts; }
}

```

#### Kotlin

```kotlin
import io.realm.MutableRealmInteger
import io.realm.RealmObject
import io.realm.annotations.Required

open class HauntedHouse: RealmObject() {
    @Required
    val ghosts: MutableRealmInteger = MutableRealmInteger.valueOf(0)
}

```

> Important:
> `MutableRealmInteger` is a live object like `RealmObject`,
`RealmResults` and `RealmList`. This means the value contained
inside the `MutableRealmInteger` can change when a realm is
written to. For this reason `MutableRealmInteger` fields must be
marked final in Java and `val` in Kotlin.
>

## Usage
The `counter.increment()`
and `counter.decrement()`
operators ensure that increments and decrements from multiple distributed
clients are aggregated correctly.

To change a `MutableRealmInteger` value, call `increment()` or
`decrement()` within a write transaction:

#### Java

```java
HauntedHouse house = realm.where(HauntedHouse.class)
        .findFirst();
realm.executeTransaction(r -> {
    Log.v("EXAMPLE", "Number of ghosts: " + house.getGhosts().get()); // 0
    house.getGhosts().increment(1);
    Log.v("EXAMPLE", "Number of ghosts: " + house.getGhosts().get()); // 1
    house.getGhosts().increment(5);
    Log.v("EXAMPLE", "Number of ghosts: " + house.getGhosts().get()); // 6
    house.getGhosts().decrement(2);
    Log.v("EXAMPLE", "Number of ghosts: " + house.getGhosts().get()); // 4
});

```

#### Kotlin

```kotlin
val house = realm.where(HauntedHouse::class.java)
    .findFirst()!!
realm.executeTransaction {
    Log.v("EXAMPLE", "Number of ghosts: ${house.ghosts.get()}") // 0
    house.ghosts.increment(1)
    Log.v("EXAMPLE", "Number of ghosts: ${house.ghosts.get()}") // 1
    house.ghosts.increment(5)
    Log.v("EXAMPLE", "Number of ghosts: ${house.ghosts.get()}") // 6
    house.ghosts.decrement(2)
    Log.v("EXAMPLE", "Number of ghosts: ${house.ghosts.get()}") // 4
}

```

You can assign a `MutableRealmInteger` a new value with a call to
`counter.set()`
within a write transaction.

> Warning:
> Use the `set()` operator with extreme care. `set()` ignores
the effects of any prior calls to `increment()` or `decrement()`.
Although the value of a `MutableRealmInteger` always converges
across devices, the specific value on which it converges depends on
the actual order in which operations took place.
Mixing `set()` with `increment()` and `decrement()` is
not advised unless fuzzy counting is acceptable.
>

#### Java

```java
realm.executeTransaction(r -> {
    house.getGhosts().set(42);
});

```

#### Kotlin

```kotlin
realm.executeTransaction {
    house!!.ghosts.set(42)
}

```

Since `MutableRealmInteger` instances retain a reference to their
parent object, neither object can be garbage collected while you still
retain a reference to the `MutableRealmInteger`.
