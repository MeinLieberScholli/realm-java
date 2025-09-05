# Enumerations - Java SDK
Enumerations, also known as enums, are not supported natively in the
Java SDK. However, you can use Java and Kotlin enums in your
Realm objects if you follow these steps.

## Usage
To use an enum in a Realm object class, define a field
with a type matching the underlying data type of your enum. Create
getters and setters for the field that convert the field value between
the underlying value and the enum type. You can use the Java's built-in
[Enum.valueOf()](https://docs.oracle.com/javase/7/docs/api/java/lang/Enum.html#valueOf(java.lang.Class,%20java.lang.String))
method to convert from the underlying type to the enum type.

#### Java

```kotlin
public enum FrogState {
    TADPOLE("Tadpole"),
    FROG("Frog"),
    OLD_FROG("Old Frog");

    private String state;
    FrogState(String state) {
        this.state = state;
    }
    public String getState() {
        return state;
    }
}

```

```java
import io.realm.RealmObject;

public class Frog extends RealmObject {
    String name;
    String state = FrogState.TADPOLE.getState();
    // realm-required empty constructor
    public Frog() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public FrogState getState() {
        // because state is actually a String and another client could assign an invalid value,
        // default the state to "TADPOLE" if the state is unreadable
        FrogState currentState = null;
        try {
            // fetches the FrogState enum value associated with the current internal string value
            currentState = FrogState.valueOf(state);
        } catch (IllegalArgumentException e) {
            currentState = FrogState.TADPOLE;
        }
        return currentState;
    }
    public void setState(FrogState value) {
        // users set state using a FrogState, but it is saved as a string internally
        this.state = value.getState();
    }
}

```

```java
Frog frog = realm.createObject(Frog.class);
frog.setName("Jonathan Livingston Applesauce");
// set the state using the enum
frog.setState(FrogState.FROG);

// fetching the state returns an enum
FrogState currentJonathanState = frog.getState();

```

#### Kotlin

```kotlin
enum class FrogState(val state: String) {
    TADPOLE("Tadpole"),
    FROG("Frog"),
    OLD_FROG("Old Frog");
}

```

```kotlin
import io.realm.RealmObject
import java.lang.IllegalArgumentException

open class Frog  // realm-required empty constructor
    : RealmObject() {
    var name: String? = null
    private var state: String = FrogState.TADPOLE.state
    var stateEnum: FrogState
        get() {
            // because state is actually a String and another client could assign an invalid value,
            // default the state to "TADPOLE" if the state is unreadable
            return try {
                // fetches the FrogState enum value associated with the current internal string value
                FrogState.valueOf(state)
            } catch (e: IllegalArgumentException) {
                FrogState.TADPOLE
            }
        }
        set(value) {
            // users set state using a FrogState, but it is saved as a string internally
            state = value.state
        }
}

```

```kotlin
val frog = realm.createObject(Frog::class.java)
frog.name = "Jonathan Livingston Applesauce"
// set the state using the enum
frog.stateEnum = FrogState.FROG

// fetching the state returns an enum
val currentJonathanState: FrogState = frog.stateEnum

```

