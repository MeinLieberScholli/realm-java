# Asynchronous API - Java SDK
The Java SDK lets you access network and disk
resources in two ways: **synchronously** and **asynchronously**. While
synchronous, or "sync", requests block execution until the request returns
success or failure, asynchronous, or "async", requests assign a
callback and proceed execution to the next line of code. When
the request returns, the callback runs to process results.
In the callback, you can check if the request executed
successfully and either access the returned results or the returned
error.

## Asynchronous Calls
Asynchronous API requests in the SDK end with the suffix "Async".
There are several different ways an asynchronous request can behave,
depending on which part of the SDK you're using.

### Realm.Callback
Asynchronous calls to open a realm,
use a final parameter of type `Realm.Callback`. To retrieve returned values after the
request completes, implement the `onSuccess()` method in the callback
object passed as the final parameter to these asynchronous methods. You
should also implement the `onError()` method to handle request failures,
but it is not required.

#### Java

```java
Realm.getInstanceAsync(config, new Realm.Callback() {
    @Override
    public void onSuccess(@NotNull Realm realm) {
        Log.v("EXAMPLE", "Successfully fetched realm instance.");
    }
    public void onError(Exception e) {
        Log.e("EXAMPLE", "Failed to get realm instance: " + e);
    }
});

```

#### Kotlin

```kotlin
Realm.getInstanceAsync(config, object : Realm.Callback() {
    override fun onSuccess(realm: Realm) {
        Log.v("EXAMPLE", "Successfully fetched realm instance.")
    }

    fun onError(e: java.lang.Exception) {
        Log.e("EXAMPLE", "Failed to get realm instance: $e")
    }
})

```

### RealmAsyncTask
Asynchronous calls to execute transactions on a realm return
an instance of `RealmAsyncTask`. You can optionally specify an error
handler or a
success notification for `RealmAsyncTask` by
passing additional parameters to the asynchronous call. Additionally,
you use the `cancel()`
method to stop a transaction from completing. The lambda function passed
to a `RealmAsyncTask` contains the write operations to include in the
transaction.

#### Java

```java
// transaction logic, success notification, error handler all via lambdas
realm.executeTransactionAsync(transactionRealm -> {
    Item item = transactionRealm.createObject(Item.class);
}, () -> {
    Log.v("EXAMPLE", "Successfully completed the transaction");
}, error -> {
    Log.e("EXAMPLE", "Failed the transaction: " + error);
});

// using class instances for transaction, success, error
realm.executeTransactionAsync(new Realm.Transaction() {
    @Override
    public void execute(Realm transactionRealm) {
        Item item = transactionRealm.createObject(Item.class);
    }
}, new Realm.Transaction.OnSuccess() {
    @Override
    public void onSuccess() {
        Log.v("EXAMPLE", "Successfully completed the transaction");
    }
}, new Realm.Transaction.OnError() {
    @Override
    public void onError(Throwable error) {
        Log.e("EXAMPLE", "Failed the transaction: " + error);
    }
});

```

#### Kotlin

```kotlin
// using class instances for transaction, success, error
realm.executeTransactionAsync(Realm.Transaction { transactionRealm ->
        val item: Item = transactionRealm.createObject<Item>()
}, Realm.Transaction.OnSuccess {
        Log.v("EXAMPLE", "Successfully completed the transaction")
}, Realm.Transaction.OnError { error ->
        Log.e("EXAMPLE", "Failed the transaction: $error")
})

// transaction logic, success notification, error handler all via lambdas
realm.executeTransactionAsync(
    { transactionRealm ->
        val item = transactionRealm.createObject<Item>()
    },
    { Log.v("EXAMPLE", "Successfully completed the transaction") },
    { error ->
        Log.e("EXAMPLE", "Failed the transaction: $error")
    })

```

### RealmResults
Asynchronous reads from a realm using `findAllAsync()` immediately return an empty
`[RealmResults` instance. The SDK
executes the query on a background thread and populates the
`RealmResults` instance with the results when the query completes. You
can register a listener with `addChangeListener()`
to receive a notification when the query completes.

#### Java

```java
RealmResults<Item> items = realm.where(Item.class).findAllAsync();
// length of items is zero when initially returned
items.addChangeListener(new RealmChangeListener<RealmResults<Item>>() {
    @Override
    public void onChange(RealmResults<Item> items) {
        Log.v("EXAMPLE", "Completed the query.");
        // items results now contains all matched objects (more than zero)
    }
});

```

#### Kotlin

```kotlin
val items = realm.where<Item>().findAllAsync()
// length of items is zero when initially returned
items.addChangeListener(RealmChangeListener {
    Log.v("EXAMPLE", "Completed the query.")
    // items results now contains all matched objects (more than zero)
})

```

### RealmResultTask
You can cancel `RealmResultTask` instances just like
`RealmAsyncTask`. To access the values returned by your query, you
can use:

- `get()` to
block until the operation completes
- `getAsync()`
to handle the result via an App.Callback
instance

#### Java

```java
Document queryFilter  = new Document("type", "perennial");
mongoCollection.findOne(queryFilter).getAsync(task -> {
    if (task.isSuccess()) {
        Plant result = task.get();
        Log.v("EXAMPLE", "successfully found a document: " + result);
    } else {
        Log.e("EXAMPLE", "failed to find document with: ", task.getError());
    }
});

```

#### Kotlin

```kotlin
val queryFilter = Document("type", "perennial")
mongoCollection.findOne(queryFilter)
    .getAsync { task ->
        if (task.isSuccess) {
            val result = task.get()
            Log.v("EXAMPLE", "successfully found a document: $result")
        } else {
            Log.e("EXAMPLE", "failed to find document with: ${task.error}")
        }
    }

```

## Coroutines
The SDK provides a set of Kotlin extensions to request
asynchronously using coroutines and flows instead of callbacks. You can
use these extensions to execute transactions, watch for changes, read,
and write.

```kotlin
// open a realm asynchronously
Realm.getInstanceAsync(config, object : Realm.Callback() {
    override fun onSuccess(realm: Realm) {
        Log.v("EXAMPLE", "Successfully fetched realm instance")

        CoroutineScope(Dispatchers.Main).launch {
            // asynchronous transaction
            realm.executeTransactionAwait(Dispatchers.IO) { transactionRealm: Realm ->
                if (isActive) {
                    val item = transactionRealm.createObject<Item>()
                }
            }
        }
        // asynchronous query
        val items: Flow<RealmResults<Item>> = realm.where<Item>().findAllAsync().toFlow()
    }

    fun onError(e: Exception) {
        Log.e("EXAMPLE", "Failed to get realm instance: $e")
    }
})

```

> Tip:
> The `toFlow()` extension method passes frozen Realm objects to safely
communicate between threads.
>

> Seealso:
> The SDK also includes Kotlin extensions that make specifying type
parameters for Realm reads and writes easier.
>
