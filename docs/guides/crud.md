# CRUD - Java SDK
## Write Operations
You can **create** objects in a realm,
**update** objects in a realm, and eventually **delete**
objects from a realm. Because these operations modify the
state of the realm, we call them writes.

Realm handles writes in terms of **transactions**. A
transaction is a list of read and write operations that
Realm treats as a single indivisible operation. In other
words, a transaction is *all or nothing*: either all of the
operations in the transaction succeed or none of the
operations in the transaction take effect.

> Note:
> All writes must happen in a transaction.
>

A realm allows only one open write transaction at a time. Realm
blocks other writes on other threads until the open
transaction is complete. Consequently, there is no race
condition when reading values from the realm within a
transaction.

When you are done with your transaction, Realm either
**commits** it or **cancels** it:

- When Realm **commits** a transaction, Realm writes
all changes to disk.
- When Realm **cancels** a write transaction or an operation in
the transaction causes an error, all changes are discarded
(or "rolled back").

> Tip:
> Whenever you create, update, or delete a Realm object,
your changes update the representation of that object in
Realm and emit
notifications to any subscribed
listeners. As a result, you should only write to Realm
objects when necessary to persist data.
>

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

## Managed Objects
**Managed objects** are live Realm objects that update
based on changes to underlying data in Realm. Managed
objects can only come from an open realm, and receive updates
as long as that realm remains open. Managed objects *cannot be passed
between threads*.

## Unmanaged objects
**Unmanaged objects** are instances of Realm objects that are
not live. You can get an unmanaged object by manually constructing a
Realm object yourself, or by calling
`[Realm.copyFromRealm()`.
Unmanaged objects *can be passed between threads*.

## Run a Transaction
Realm represents each transaction as a callback function
that contains zero or more read and write operations. To run
a transaction, define a transaction callback and pass it to
the realm's `write` method. Within this callback, you are
free to create, read, update, and delete on the realm. If
the code in the callback throws an exception when Realm runs
it, Realm cancels the transaction. Otherwise, Realm commits
the transaction immediately after the callback.

> Example:
> The following code shows how to run a transaction with
`executeTransaction()`
or `executeTransactionAsync()`.
If the code in the callback throws an exception, Realm
cancels the transaction. Otherwise, Realm commits the
transaction.
>
> #### Java
>
> ```java
> realm.executeTransaction(r -> {
>     // Create a turtle enthusiast named Ali.
>     TurtleEnthusiast ali = r.createObject(TurtleEnthusiast.class, new ObjectId());
>     ali.setName("Ali");
>     // Find turtles younger than 2 years old
>     RealmResults<Turtle> hatchlings = r.where(Turtle.class).lessThan("age", 2).findAll();
>     // Give all hatchlings to Ali.
>     hatchlings.setObject("owner", ali);
> });
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> realm.executeTransaction { r: Realm ->
>     // Create a turtle enthusiast named Ali.
>     val ali = r.createObject(TurtleEnthusiast::class.java, ObjectId())
>     ali.name = "Ali"
>     // Find turtles younger than 2 years old
>     val hatchlings =
>         r.where(Turtle::class.java).lessThan("age", 2).findAll()
>     // Give all hatchlings to Ali.
>     hatchlings.setObject("owner", ali)
> }
>
> ```
>
>
