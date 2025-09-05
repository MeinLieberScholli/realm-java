# Quick Start - Java SDK

This page contains information to quickly get Realm
integrated into your app. Before you begin, ensure you have:

- Installed the Java SDK

## Initialize Realm
Before you can use Realm in your app, you must
initialize the Realm library. Your application should
initialize Realm just once each time the application runs.

To initialize the Realm library, provide an Android
`context` to the `Realm.init()` static function. You can provide
an Activity, Fragment, or Application `context` for initialization with no
difference in behavior. You can initialize the Realm library
in the `onCreate()` method of an [application subclass](https://developer.android.com/reference/android/app/Application) to
ensure that you only initialize Realm once each time the
application runs.

#### Java

```java
Realm.init(this); // context, usually an Activity or Application

```

#### Kotlin

```kotlin
Realm.init(this) // context, usually an Activity or Application

```

> Tip:
> If you create your own `Application` subclass, you must add it to your
application's `AndroidManifest.xml` to execute your custom
application logic. Set the `android.name` property of your manifest's
application definition to ensure that Android instantiates your `Application`
subclass before any other class when a user launches your application.
>
> ```xml
> <?xml version="1.0" encoding="utf-8"?>
> <manifest xmlns:android="http://schemas.android.com/apk/res/android"
>    package="com.mongodb.example">
>
>    <application
>       android:name=".MyApplicationSubclass"
>       ...
>    />
> </manifest>
> ```
>

## Define Your Object Model
Your application's **data model** defines the structure of data
stored within Realm.
You can define your application's data model via Kotlin or
Java classes in your application code with
Realm Object Models.

To define your application's data model, add the following class
definitions to your application code:

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.Required;

public class Task extends RealmObject {
    @PrimaryKey private String name;
    @Required private String status = TaskStatus.Open.name();

    public void setStatus(TaskStatus status) { this.status = status.name(); }
    public String getStatus() { return this.status; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public Task(String _name) { this.name = _name; }
    public Task() {}
}

```

```java

public enum TaskStatus {
    Open("Open"),
    InProgress("In Progress"),
    Complete("Complete");

    String displayName;
    TaskStatus(String displayName) {
        this.displayName = displayName;
    }
}

```

#### Kotlin

```kotlin

enum class TaskStatus(val displayName: String) {
    Open("Open"),
    InProgress("In Progress"),
    Complete("Complete"),
}

open class Task() : RealmObject() {
    @PrimaryKey
    var name: String = "task"

    @Required
    var status: String = TaskStatus.Open.name
    var statusEnum: TaskStatus
        get() {
            // because status is actually a String and another client could assign an invalid value,
            // default the status to "Open" if the status is unreadable
            return try {
                TaskStatus.valueOf(status)
            } catch (e: IllegalArgumentException) {
                TaskStatus.Open
            }
        }
        set(value) { status = value.name }
}

```

## Open a Realm
Use `RealmConfiguration` to control the specifics of the realm you
would like to open, including the name or location of the realm,
whether to allow synchronous reads or writes to a realm on the UI
thread, and more.

#### Java

```java
String realmName = "My Project";
RealmConfiguration config = new RealmConfiguration.Builder().name(realmName).build();

Realm backgroundThreadRealm = Realm.getInstance(config);

```

#### Kotlin

```kotlin
val realmName: String = "My Project"
val config = RealmConfiguration.Builder().name(realmName).build()

val backgroundThreadRealm : Realm = Realm.getInstance(config)

```

## Create, Read, Update, and Delete Objects
Once you have opened a realm, you can modify the
objects within that realm in a
write transaction block.

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

To create a new `Task`, instantiate an instance of the
`Task` class and add it to the realm in a write block:

#### Java

```java
Task Task = new Task("New Task");
backgroundThreadRealm.executeTransaction (transactionRealm -> {
    transactionRealm.insert(Task);
});

```

#### Kotlin

```kotlin
val task : Task = Task()
task.name = "New Task"
backgroundThreadRealm.executeTransaction { transactionRealm ->
    transactionRealm.insert(task)
}

```

You can retrieve a live collection
of all items in the realm:

#### Java

```java
// all Tasks in the realm
RealmResults<Task> Tasks = backgroundThreadRealm.where(Task.class).findAll();

```

#### Kotlin

```kotlin
// all tasks in the realm
val tasks : RealmResults<Task> = backgroundThreadRealm.where<Task>().findAll()

```

You can also filter that collection using a filter:

#### Java

```java
// you can also filter a collection
RealmResults<Task> TasksThatBeginWithN = Tasks.where().beginsWith("name", "N").findAll();
RealmResults<Task> openTasks = Tasks.where().equalTo("status", TaskStatus.Open.name()).findAll();

```

#### Kotlin

```kotlin
// you can also filter a collection
val tasksThatBeginWithN : List<Task> = tasks.where().beginsWith("name", "N").findAll()
val openTasks : List<Task> = tasks.where().equalTo("status", TaskStatus.Open.name).findAll()

```

To modify a task, update its properties in a write transaction block:

#### Java

```java
Task otherTask = Tasks.get(0);

// all modifications to a realm must happen inside of a write block
backgroundThreadRealm.executeTransaction( transactionRealm -> {
    Task innerOtherTask = transactionRealm.where(Task.class).equalTo("_id", otherTask.getName()).findFirst();
    innerOtherTask.setStatus(TaskStatus.Complete);
});

```

#### Kotlin

```kotlin
val otherTask: Task = tasks[0]!!

// all modifications to a realm must happen inside of a write block
backgroundThreadRealm.executeTransaction { transactionRealm ->
    val innerOtherTask : Task = transactionRealm.where<Task>().equalTo("name", otherTask.name).findFirst()!!
    innerOtherTask.status = TaskStatus.Complete.name
}

```

Finally, you can delete a task by calling the `deleteFromRealm()`
method in a write transaction block:

#### Java

```java
Task yetAnotherTask = Tasks.get(0);
String yetAnotherTaskName = yetAnotherTask.getName();
// all modifications to a realm must happen inside of a write block
backgroundThreadRealm.executeTransaction( transactionRealm -> {
    Task innerYetAnotherTask = transactionRealm.where(Task.class).equalTo("_id", yetAnotherTaskName).findFirst();
    innerYetAnotherTask.deleteFromRealm();
});

```

#### Kotlin

```kotlin
val yetAnotherTask: Task = tasks.get(0)!!
val yetAnotherTaskName: String = yetAnotherTask.name
// all modifications to a realm must happen inside of a write block
backgroundThreadRealm.executeTransaction { transactionRealm ->
    val innerYetAnotherTask : Task = transactionRealm.where<Task>().equalTo("name", yetAnotherTaskName).findFirst()!!
    innerYetAnotherTask.deleteFromRealm()
}

```

## Watch for Changes
You can watch a realm, collection, or object for changes by attaching a custom
`OrderedRealmCollectionChangeListener` with the `addChangeListener()`
method:

#### Java

```java
// all Tasks in the realm
RealmResults<Task> Tasks = uiThreadRealm.where(Task.class).findAllAsync();

Tasks.addChangeListener(new OrderedRealmCollectionChangeListener<RealmResults<Task>>() {
    @Override
    public void onChange(RealmResults<Task> collection, OrderedCollectionChangeSet changeSet) {
        // process deletions in reverse order if maintaining parallel data structures so indices don't change as you iterate
        OrderedCollectionChangeSet.Range[] deletions = changeSet.getDeletionRanges();
        for (OrderedCollectionChangeSet.Range range : deletions) {
            Log.v("QUICKSTART", "Deleted range: " + range.startIndex + " to " + (range.startIndex + range.length - 1));
        }

        OrderedCollectionChangeSet.Range[] insertions = changeSet.getInsertionRanges();
        for (OrderedCollectionChangeSet.Range range : insertions) {
            Log.v("QUICKSTART", "Inserted range: " + range.startIndex + " to " + (range.startIndex + range.length - 1));                            }

        OrderedCollectionChangeSet.Range[] modifications = changeSet.getChangeRanges();
        for (OrderedCollectionChangeSet.Range range : modifications) {
            Log.v("QUICKSTART", "Updated range: " + range.startIndex + " to " + (range.startIndex + range.length - 1));                            }
    }
});

```

#### Kotlin

```kotlin
// all tasks in the realm
val tasks : RealmResults<Task> = realm.where<Task>().findAllAsync()

tasks.addChangeListener(OrderedRealmCollectionChangeListener<RealmResults<Task>> { collection, changeSet ->
    // process deletions in reverse order if maintaining parallel data structures so indices don't change as you iterate
    val deletions = changeSet.deletionRanges
    for (i in deletions.indices.reversed()) {
        val range = deletions[i]
        Log.v("QUICKSTART", "Deleted range: ${range.startIndex} to ${range.startIndex + range.length - 1}")
    }

    val insertions = changeSet.insertionRanges
    for (range in insertions) {
        Log.v("QUICKSTART", "Inserted range: ${range.startIndex} to ${range.startIndex + range.length - 1}")
    }

    val modifications = changeSet.changeRanges
    for (range in modifications) {
        Log.v("QUICKSTART", "Updated range: ${range.startIndex} to ${range.startIndex + range.length - 1}")
    }
})

```

## Complete Example
If you're running this project in a fresh Android Studio project, you can
copy and paste this file into your application's `MainActivity` -- just
remember to:

- use a package declaration at the top of the file for your own project
- update the `import` statements for `Task` and `TaskStatus` if
you're using java

#### Java

```java
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.Required;

public class Task extends RealmObject {
    @PrimaryKey private String name;
    @Required private String status = TaskStatus.Open.name();

    public void setStatus(TaskStatus status) { this.status = status.name(); }
    public String getStatus() { return this.status; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public Task(String _name) { this.name = _name; }
    public Task() {}
}

```

```java

public enum TaskStatus {
    Open("Open"),
    InProgress("In Progress"),
    Complete("Complete");

    String displayName;
    TaskStatus(String displayName) {
        this.displayName = displayName;
    }
}

```

```java
import io.realm.OrderedCollectionChangeSet;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;
import android.util.Log;

import io.realm.OrderedRealmCollectionChangeListener;

import io.realm.Realm;
import io.realm.RealmConfiguration;
import io.realm.RealmResults;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;

import com.mongodb.realm.examples.model.java.Task;
import com.mongodb.realm.examples.model.java.TaskStatus;

public class MainActivity extends AppCompatActivity {
    Realm uiThreadRealm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Realm.init(this); // context, usually an Activity or Application

        String realmName = "My Project";
        RealmConfiguration config = new RealmConfiguration.Builder().name(realmName).build();

        uiThreadRealm = Realm.getInstance(config);

        addChangeListenerToRealm(uiThreadRealm);

        FutureTask<String> Task = new FutureTask(new BackgroundQuickStart(), "test");
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        executorService.execute(Task);

    }

    private void addChangeListenerToRealm(Realm realm) {
        // all Tasks in the realm
        RealmResults<Task> Tasks = uiThreadRealm.where(Task.class).findAllAsync();

        Tasks.addChangeListener(new OrderedRealmCollectionChangeListener<RealmResults<Task>>() {
            @Override
            public void onChange(RealmResults<Task> collection, OrderedCollectionChangeSet changeSet) {
                // process deletions in reverse order if maintaining parallel data structures so indices don't change as you iterate
                OrderedCollectionChangeSet.Range[] deletions = changeSet.getDeletionRanges();
                for (OrderedCollectionChangeSet.Range range : deletions) {
                    Log.v("QUICKSTART", "Deleted range: " + range.startIndex + " to " + (range.startIndex + range.length - 1));
                }

                OrderedCollectionChangeSet.Range[] insertions = changeSet.getInsertionRanges();
                for (OrderedCollectionChangeSet.Range range : insertions) {
                    Log.v("QUICKSTART", "Inserted range: " + range.startIndex + " to " + (range.startIndex + range.length - 1));                            }

                OrderedCollectionChangeSet.Range[] modifications = changeSet.getChangeRanges();
                for (OrderedCollectionChangeSet.Range range : modifications) {
                    Log.v("QUICKSTART", "Updated range: " + range.startIndex + " to " + (range.startIndex + range.length - 1));                            }
            }
        });
    }

        @Override
    protected void onDestroy() {
        super.onDestroy();
        // the ui thread realm uses asynchronous transactions, so we can only safely close the realm
        // when the activity ends and we can safely assume that those transactions have completed
        uiThreadRealm.close();
    }

    public class BackgroundQuickStart implements Runnable {

        @Override
        public void run() {
            String realmName = "My Project";
            RealmConfiguration config = new RealmConfiguration.Builder().name(realmName).build();

            Realm backgroundThreadRealm = Realm.getInstance(config);

            Task Task = new Task("New Task");
            backgroundThreadRealm.executeTransaction (transactionRealm -> {
                transactionRealm.insert(Task);
            });

            // all Tasks in the realm
            RealmResults<Task> Tasks = backgroundThreadRealm.where(Task.class).findAll();

            // you can also filter a collection
            RealmResults<Task> TasksThatBeginWithN = Tasks.where().beginsWith("name", "N").findAll();
            RealmResults<Task> openTasks = Tasks.where().equalTo("status", TaskStatus.Open.name()).findAll();

            Task otherTask = Tasks.get(0);

            // all modifications to a realm must happen inside of a write block
            backgroundThreadRealm.executeTransaction( transactionRealm -> {
                Task innerOtherTask = transactionRealm.where(Task.class).equalTo("_id", otherTask.getName()).findFirst();
                innerOtherTask.setStatus(TaskStatus.Complete);
            });

            Task yetAnotherTask = Tasks.get(0);
            String yetAnotherTaskName = yetAnotherTask.getName();
            // all modifications to a realm must happen inside of a write block
            backgroundThreadRealm.executeTransaction( transactionRealm -> {
                Task innerYetAnotherTask = transactionRealm.where(Task.class).equalTo("_id", yetAnotherTaskName).findFirst();
                innerYetAnotherTask.deleteFromRealm();
            });

            // because this background thread uses synchronous realm transactions, at this point all
            // transactions have completed and we can safely close the realm
            backgroundThreadRealm.close();
        }
    }
}

```

#### Kotlin

```kotlin

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import android.util.Log
import io.realm.*
import io.realm.annotations.PrimaryKey

import io.realm.annotations.Required
import io.realm.kotlin.where
import java.util.concurrent.ExecutorService
import java.util.concurrent.Executors
import java.util.concurrent.FutureTask

class MainActivity : AppCompatActivity() {
    lateinit var uiThreadRealm: Realm

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Realm.init(this) // context, usually an Activity or Application

        val realmName: String = "My Project"
        val config = RealmConfiguration.Builder()
            .name(realmName)
            .build()

        uiThreadRealm = Realm.getInstance(config)

        addChangeListenerToRealm(uiThreadRealm)

        val task : FutureTask<String> = FutureTask(BackgroundQuickStart(), "test")
        val executorService: ExecutorService = Executors.newFixedThreadPool(2)
        executorService.execute(task)

    }

    fun addChangeListenerToRealm(realm : Realm) {
        // all tasks in the realm
        val tasks : RealmResults<Task> = realm.where<Task>().findAllAsync()

        tasks.addChangeListener(OrderedRealmCollectionChangeListener<RealmResults<Task>> { collection, changeSet ->
            // process deletions in reverse order if maintaining parallel data structures so indices don't change as you iterate
            val deletions = changeSet.deletionRanges
            for (i in deletions.indices.reversed()) {
                val range = deletions[i]
                Log.v("QUICKSTART", "Deleted range: ${range.startIndex} to ${range.startIndex + range.length - 1}")
            }

            val insertions = changeSet.insertionRanges
            for (range in insertions) {
                Log.v("QUICKSTART", "Inserted range: ${range.startIndex} to ${range.startIndex + range.length - 1}")
            }

            val modifications = changeSet.changeRanges
            for (range in modifications) {
                Log.v("QUICKSTART", "Updated range: ${range.startIndex} to ${range.startIndex + range.length - 1}")
            }
        })
    }

    override fun onDestroy() {
        super.onDestroy()
        // the ui thread realm uses asynchronous transactions, so we can only safely close the realm
        // when the activity ends and we can safely assume that those transactions have completed
        uiThreadRealm.close()
    }

    class BackgroundQuickStart : Runnable {

        override fun run() {
            val realmName: String = "My Project"
            val config = RealmConfiguration.Builder().name(realmName).build()

            val backgroundThreadRealm : Realm = Realm.getInstance(config)

            val task : Task = Task()
            task.name = "New Task"
            backgroundThreadRealm.executeTransaction { transactionRealm ->
                transactionRealm.insert(task)
            }

            // all tasks in the realm
            val tasks : RealmResults<Task> = backgroundThreadRealm.where<Task>().findAll()

            // you can also filter a collection
            val tasksThatBeginWithN : List<Task> = tasks.where().beginsWith("name", "N").findAll()
            val openTasks : List<Task> = tasks.where().equalTo("status", TaskStatus.Open.name).findAll()

            val otherTask: Task = tasks[0]!!

            // all modifications to a realm must happen inside of a write block
            backgroundThreadRealm.executeTransaction { transactionRealm ->
                val innerOtherTask : Task = transactionRealm.where<Task>().equalTo("name", otherTask.name).findFirst()!!
                innerOtherTask.status = TaskStatus.Complete.name
            }

            val yetAnotherTask: Task = tasks.get(0)!!
            val yetAnotherTaskName: String = yetAnotherTask.name
            // all modifications to a realm must happen inside of a write block
            backgroundThreadRealm.executeTransaction { transactionRealm ->
                val innerYetAnotherTask : Task = transactionRealm.where<Task>().equalTo("name", yetAnotherTaskName).findFirst()!!
                innerYetAnotherTask.deleteFromRealm()
            }

            // because this background thread uses synchronous realm transactions, at this point all
            // transactions have completed and we can safely close the realm
            backgroundThreadRealm.close()
        }

    }
}

enum class TaskStatus(val displayName: String) {
    Open("Open"),
    InProgress("In Progress"),
    Complete("Complete"),
}

open class Task() : RealmObject() {
    @PrimaryKey
    var name: String = "task"

    @Required
    var status: String = TaskStatus.Open.name
    var statusEnum: TaskStatus
        get() {
            // because status is actually a String and another client could assign an invalid value,
            // default the status to "Open" if the status is unreadable
            return try {
                TaskStatus.valueOf(status)
            } catch (e: IllegalArgumentException) {
                TaskStatus.Open
            }
        }
        set(value) { status = value.name }
}

```

## Output
Running the above code should produce output resembling the following:

```shell
Successfully authenticated anonymously.

Updated range: 0 to 1

Deleted range: 0 to 1

Successfully logged out.
```
