# Embedded Objects - Java SDK
An embedded object is a special type of Realm object that models complex data about a specific object.
Embedded objects are similar to relationships, but they provide additional constraints and
map more naturally to the denormalized document model.

Realm enforces unique ownership constraints that treat each embedded
object as nested data inside of a single, specific parent object. An
embedded object inherits the lifecycle of its parent object and cannot
exist as an independent Realm object. Realm automatically deletes
embedded objects if their parent object is deleted or when overwritten
by a new embedded object instance.

> Warning:
> When you delete a Realm object, Realm automatically deletes any
embedded objects referenced by that object. Any objects that your
application must persist after the deletion of their parent object
should use relationships
instead.
>

## Embedded Object Data Models
You can define embedded object types using either Realm object models or
a server-side document schema. Embedded object types are reusable and
composable. You can use the same embedded object type in multiple parent
object types and you can embed objects inside of other embedded objects.

> Important:
> Embedded objects cannot have a primary key.
>

### Realm Object Models
To define an embedded object, derive a class from `RealmObject` and set the `embedded` property of the
`RealmClass` annotation
to `true`. You can reference an embedded object type from parent
object types in the same way as you would define a relationship:

#### Java

```java
// Define an embedded object
@RealmClass(embedded = true)
public class Address extends RealmObject {
    String street;
    String city;
    String country;
    String postalCode;

    public Address(String street, String city, String country, String postalCode) {
        this.street = street;
        this.city = city;
        this.country = country;
        this.postalCode = postalCode;
    }

    public Address() {}
}

// Define an object containing one embedded object
public class Contact extends RealmObject {
    @PrimaryKey
    private ObjectId _id = new ObjectId();
    String name = "";

    // Embed a single object.
    // Embedded object properties must be marked optional
    Address address;

    public Contact(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    public Contact() {}
}

// Define an object containing an array of embedded objects
public class Business extends RealmObject {
    @PrimaryKey
    private ObjectId _id = new ObjectId();
    String name = "";

    // Embed an array of objects
    RealmList<Address> addresses = new RealmList<Address>();

    public Business(String name, RealmList<Address> addresses) {
        this.name = name;
        this.addresses = addresses;
    }

    public Business() {}
}

```

#### Kotlin

```kotlin
// Define an embedded object
@RealmClass(embedded = true)
open class Address(
    var street: String? = null,
    var city: String? = null,
    var country: String? = null,
    var postalCode: String? = null
): RealmObject() {}

// Define an object containing one embedded object
open class Contact(_name: String = "", _address: Address? = null) : RealmObject() {
    @PrimaryKey var _id: ObjectId = ObjectId()
    var name: String = _name

    // Embed a single object.
    // Embedded object properties must be marked optional
    var address: Address? = _address
}

// Define an object containing an array of embedded objects
open class Business(_name: String = "", _addresses: RealmList<Address> = RealmList()) : RealmObject() {
    @PrimaryKey var _id: ObjectId = ObjectId()
    var name: String = _name

    // Embed an array of objects
    var addresses: RealmList<Address> = _addresses
}

```

### JSON Schema
Embedded objects map to embedded documents in the parent type's schema.

```json
{
  "title": "Contact",
  "bsonType": "object",
  "required": ["_id"],
  "properties": {
    "_id": { "bsonType": "objectId" },
    "name": { "bsonType": "string" },
    "address": {
      "title": "Address",
      "bsonType": "object",
      "properties": {
        "street": { "bsonType": "string" },
        "city": { "bsonType": "string" },
        "country": { "bsonType": "string" },
        "postalCode": { "bsonType": "string" }
      }
    }
  }
}
```

```json
{
  "title": "Business",
  "bsonType": "object",
  "required": ["_id", "name"],
  "properties": {
    "_id": { "bsonType": "objectId" },
    "name": { "bsonType": "string" },
    "addresses": {
      "bsonType": "array",
      "items": {
        "title": "Address",
        "bsonType": "object",
        "properties": {
          "street": { "bsonType": "string" },
          "city": { "bsonType": "string" },
          "country": { "bsonType": "string" },
          "postalCode": { "bsonType": "string" }
        }
      }
    }
  }
}
```

## Read and Write Embedded Objects
### Create an Embedded Object
To create an embedded object, assign an instance of the embedded object
to a parent object's property.

#### Java

```java
// open realm

Address address = new Address("123 Fake St.", "Springfield", "USA", "90710");
Contact contact = new Contact("Nick Riviera", address);

realm.executeTransaction(transactionRealm -> {
    transactionRealm.insert(contact);
});

realm.close();

```

#### Kotlin

```kotlin
// open realm

val address = Address("123 Fake St.", "Springfield", "USA", "90710")
val contact = Contact("Nick Riviera", address)

realm.executeTransaction { transactionRealm ->
    transactionRealm.insert(contact)
}

realm.close()

```

### Update an Embedded Object Property
To update a property in an embedded object, modify the property in a
write transaction:

#### Java

```java
// assumes that at least one contact already exists in this partition
Contact resultContact = realm.where(Contact.class).findFirst();

realm.executeTransaction(transactionRealm -> {
    resultContact.address.street = "Hollywood Upstairs Medical College";
    resultContact.address.city = "Los Angeles";
    resultContact.address.postalCode = "90210";
    Log.v("EXAMPLE", "Updated contact: " + resultContact);
});

realm.close();

```

#### Kotlin

```kotlin
// assumes that at least one contact already exists in this partition
val result = realm.where<Contact>().findFirst()!!

realm.executeTransaction { transactionRealm ->
    result.address?.street = "Hollywood Upstairs Medical College"
    result.address?.city = "Los Angeles"
    result.address?.postalCode = "90210"
    Log.v("EXAMPLE", "Updated contact: ${result.name}")
}

realm.close()

```

### Overwrite an Embedded Object
To overwrite an embedded object, reassign the embedded object property
of a party to a new instance in a write transaction:

#### Java

```java
// assumes that at least one contact already exists in this partition
Contact oldContact = realm.where(Contact.class).findFirst();

realm.executeTransaction(transactionRealm -> {
    Address newAddress = new Address(
        "Hollywood Upstairs Medical College",
        "Los Angeles",
        "USA"
        "90210"
        );
    oldContact.address = newAddress;
    Log.v("EXAMPLE", "Replaced contact: " + oldContact);
});

realm.close();

```

#### Kotlin

```kotlin
// assumes that at least one contact already exists
val oldContact = realm.where<Contact>().findFirst()!!

realm.executeTransaction { transactionRealm ->
    val newAddress = Address(
        "Hollywood Upstairs Medical College",
        "Los Angeles",
        "USA",
        "90210")
    oldContact.address = newAddress
    Log.v("EXAMPLE", "Updated contact: $oldContact")
}

realm.close()

```

### Query a Collection on Embedded Object Properties
Use dot notation to filter or sort a collection of objects based on an embedded object
property value:

> Note:
> It is not possible to query embedded objects directly. Instead,
access embedded objects through a query for the parent object type.
>

#### Java

```java
RealmResults<Contact> losAngelesContacts = realm.where(Contact.class)
        .equalTo("address.city", "Los Angeles")
        .sort("address.street").findAll();
Log.v("EXAMPLE", "Los Angeles contacts: " + losAngelesContacts);

```

#### Kotlin

```kotlin
val losAngelesContacts = realm.where<Contact>()
    .equalTo("address.city", "Los Angeles")
    .sort("address.street").findAll()
Log.v("EXAMPLE", "Los Angeles Contacts: $losAngelesContacts")

```

