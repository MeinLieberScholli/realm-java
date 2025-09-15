# Testing - Java SDK
You can test your application using unit tests or integration tests.
**Unit tests** only assess the logic written in your application's code.
**Integration tests** assess your application logic, database queries and
writes, and calls to your application's backend, if you have one. Unit tests
run on your development machine using the JVM, while integration tests
run on a physical or emulated Android device. You can run integration
tests by communicating with actual instances of Realm
or an App backend using Android's built-in instrumented tests.

Android uses specific file paths and folder names in Android projects
for unit tests and instrumented tests:

|Test Type|Path|
| --- | --- |
|Unit Tests|/app/src/test|
|Instrumented Tests|/app/src/androidTest|

Because the SDK uses C++ code via Android Native for data
storage, unit testing requires you to entirely mock interactions with
Realm. Prefer integration tests for logic that requires
extensive interaction with the database.

## Integration Tests
This section shows how to integration test an application that uses
the Realm SDK. It covers the following concepts in the test
environment:

- acquiring an application context
- executing logic on a `Looper` thread
- how to delay test execution while asynchronous method calls complete

### Application Context
To initialize the SDK, you'll need to provide an application or activity
[context](https://developer.android.com/reference/android/content/Context).
This isn't available by default in Android integration tests. However,
you can use Android's built-in testing [ActivityScenario](https://developer.android.com/reference/androidx/test/core/app/ActivityScenario)
class to start an activity in your tests. You can use any activity from
your application, or you can create an empty activity just for testing.
Call `ActivityScenario.launch()` with your activity class as a
parameter to start the simulated activity.

Next, use the `ActivityScenario.onActivity()` method to run a lambda
on the simulated activity's main thread. In this lambda, you should call
the `Realm.init()` function to initialize the SDK with your activity
as a parameter. Additionally, you should save the parameter passed to
your lambda (the newly created instance of your activity) for future
use.

Because the `onActivity()` method runs on a different thread, you
should block your test from executing further until this initial setup completes.

The following example uses an `ActivityScenario`, an empty testing
activity, and a `CountDownLatch` to demonstrate how to set up an
environment where you can test your Realm application:

#### Java

```java
AtomicReference<Activity> testActivity = new AtomicReference<Activity>();
ActivityScenario<BasicActivity> scenario = ActivityScenario.launch(BasicActivity.class);

// create a latch to force blocking for an async call to initialize realm
CountDownLatch setupLatch = new CountDownLatch(1);

scenario.onActivity(activity -> {
    Realm.init(activity);
    testActivity.set(activity);
    setupLatch.countDown(); // unblock the latch await
});

// block until we have an activity to run tests on
try {
    Assert.assertTrue(setupLatch.await(1, TimeUnit.SECONDS));
} catch (InterruptedException e) {
    Log.e("EXAMPLE", e.getMessage());
}

```

#### Kotlin

```kotlin
var testActivity: Activity? = null
val scenario: ActivityScenario<BasicActivity>? =
    ActivityScenario.launch(BasicActivity::class.java)

// create a latch to force blocking for an async call to initialize realm
val setupLatch = CountDownLatch(1)

scenario?.onActivity{ activity: BasicActivity ->
    Realm.init(activity)
    testActivity = activity
    setupLatch.countDown() // unblock the latch await
}

```

### Looper Thread
Realm functionality such as
Live objects and change notifications only
work on [Looper](https://developer.android.com/reference/android/os/Looper) threads.
Threads configured with a `Looper` object pass events over a message
loop coordinated by the `Looper`. Test functions normally don't have
a `Looper` object, and configuring one to work in your tests can be
very error-prone.

Instead, you can use the [Activity.runOnUiThread()](https://developer.android.com/reference/android/app/Activity#runOnUiThread(java.lang.Runnable))
method of your test activity to execute logic on a thread that already
has a `Looper` configured. Combine `Activity.runOnUiThread()` with
a `CountDownLatch` as described in the delay section to prevent your test from completing
and exiting before your logic has executed. Within the `runOnUiThread()`
call, you can interact with the SDK just like you normally would in your
application code:

#### Java

```java
testActivity.get().runOnUiThread(() -> {
    // instantiate an app connection
    String appID = YOUR_APP_ID; // replace this with your test application App ID
    App app = new App(new AppConfiguration.Builder(appID).build());

    // authenticate a user
    Credentials credentials = Credentials.anonymous();
    app.loginAsync(credentials, it -> {
        if (it.isSuccess()) {
            Log.v("EXAMPLE", "Successfully authenticated.");

            Realm.getInstanceAsync(config, new Realm.Callback() {
                @Override
                public void onSuccess(@NonNull Realm realm) {
                    Log.v("EXAMPLE", "Successfully opened a realm.");
                    // read and write to realm here via transactions
                    testLatch.countDown();
                    realm.executeTransaction(new Realm.Transaction() {
                        @Override
                        public void execute(@NonNull Realm realm) {
                            realm.createObjectFromJson(Frog.class,
                                    "{ name: \"Doctor Cucumber\", age: 1, species: \"bullfrog\", owner: \"Wirt\", _id: 0 }");
                        }
                    });
                    realm.close();
                }
                @Override
                public void onError(@NonNull Throwable exception) {
                    Log.e("EXAMPLE", "Failed to open the realm: " + exception.getLocalizedMessage());
                }
            });
        } else {
            Log.e("EXAMPLE", "Failed login: " + it.getError().getErrorMessage());
        }
    });
});

```

#### Kotlin

```kotlin
testActivity?.runOnUiThread {
    // instantiate an app connection
    val appID: String = YOUR_APP_ID // replace this with your App ID
    val app = App(AppConfiguration.Builder(appID).build())

    // authenticate a user
    val credentials = Credentials.anonymous()
    app.loginAsync(credentials) {
        if (it.isSuccess) {
            Log.v("EXAMPLE", "Successfully authenticated.")

            Realm.getInstanceAsync(config, object : Realm.Callback() {
                override fun onSuccess(realm: Realm) {
                    Log.v("EXAMPLE", "Successfully opened a realm.")
                    // read and write to realm here via transactions
                    realm.executeTransaction {
                        realm.createObjectFromJson(
                            Frog::class.java,
                            "{ name: \"Doctor Cucumber\", age: 1, species: \"bullfrog\", owner: \"Wirt\", _id:0 }"
                        )
                    }
                    testLatch.countDown()
                    realm.close()
                }
                override fun onError(exception: Throwable) {
                    Log.e("EXAMPLE",
                        "Failed to open the realm: " + exception.localizedMessage)
                }
            })
        } else {
            Log.e("EXAMPLE", "Failed login: " + it.error.errorMessage)
        }
    }
}

```

### Delay Test Execution While Async Calls Complete
Because the SDK uses asynchronous calls for common operations, tests need a way
to wait for those async calls to complete. Otherwise, your tests will
exit before your asynchronous (or multi-threaded) calls run. This example
uses Java's built-in [CountDownLatch](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CountDownLatch.html). Follow these steps to use a `CountDownLatch` in your own tests:

1. Instantiate a `CountDownLatch` with a count of 1.
2. After running the async logic your test needs to wait for, call that
`CountDownLatch` instance's `countDown()` method.
3. When you need to wait for async logic, add a `try`/`catch` block
that handles an `InterruptedException`. In that block,
call that `CountDownLatch` instance's `await()` method.
4. Pass a timeout interval and unit to `await()`, and wrap
the call in a `Assert.assertTrue()` assertion. If the logic takes
too long, the `await()` call times out, returning false and failing
the test.

### Testing Backend
Applications that use an App backend should not connect to the
production backend for testing purposes for the following reasons:

- you should always keep test users and production users separate
for security and privacy reasons
- tests often require a clean initial state, so there's a good chance
your tests will include a setup or teardown method that deletes all
users or large chunks of data

You can use environments to manage separate
apps for testing and production.

## Unit Tests
To unit test Realm applications that use Realm,
you must [mock](https://en.wikipedia.org/wiki/Mock_object) Realm (and your
application backend, if you use one). Use the following libraries to
mock SDK functionality:

- [Robolectric](http://robolectric.org/)
- [PowerMock](https://powermock.github.io/)
- [Mockito](https://site.mockito.org/)

To make these libraries available for unit testing in your Android project,
add the following to the `dependencies` block of your application
`build.gradle` file:

```
 testImplementation "org.robolectric:robolectric:4.1"
 testImplementation "org.mockito:mockito-core:3.3.3"
 testImplementation "org.powermock:powermock-module-junit4:2.0.9"
 testImplementation "org.powermock:powermock-module-junit4-rule:2.0.9"
 testImplementation "org.powermock:powermock-api-mockito2:2.0.9"
 testImplementation "org.powermock:powermock-classloading-xstream:2.0.9"
```

> Note:
> Mocking the SDK in unit tests requires Robolectric, Mockito, and
Powermock because the SDK uses Android Native C++ method calls to
interact with Realm. Because the frameworks required to
override these method calls can be delicate, you should use the
versions listed above to ensure that your mocking is successful. Some
recent version updates (particularly Robolectric version 4.2+) can
break compiliation of unit tests using the SDK.
>

To configure your unit tests to use Robolectric, PowerMock, and Mockito
with the SDK, add the following annotations to each unit test class that
mocks the SDK:

#### Java

```java
@RunWith(RobolectricTestRunner.class)
@Config(sdk = 28)
@PowerMockIgnore({"org.mockito.*", "org.robolectric.*", "android.*", "jdk.internal.reflect.*", "androidx.*"})
@SuppressStaticInitializationFor("io.realm.internal.Util")
@PrepareForTest({Realm.class, RealmConfiguration.class, RealmQuery.class, RealmResults.class, RealmCore.class, RealmLog.class})

```

#### Kotlin

```kotlin
@RunWith(RobolectricTestRunner::class)
@Config(sdk = [28])
@PowerMockIgnore(
    "org.mockito.*",
    "org.robolectric.*",
    "android.*",
    "jdk.internal.reflect.*",
    "androidx.*"
)
@SuppressStaticInitializationFor("io.realm.internal.Util")
@PrepareForTest(
    Realm::class,
    RealmConfiguration::class,
    RealmQuery::class,
    RealmResults::class,
    RealmCore::class,
    RealmLog::class
)

```

Then, bootstrap Powermock globally in the test class:

#### Java

```java
// bootstrap powermock
@Rule
public PowerMockRule rule = new PowerMockRule();

```

#### Kotlin

```kotlin
// bootstrap powermock
@Rule
var rule = PowerMockRule()

```

Next, mock the components of the SDK that might query native C++ code
so we don't hit the limitations of the test environment:

#### Java

```java
// set up realm SDK components to be mocked. The order of these matters
mockStatic(RealmCore.class);
mockStatic(RealmLog.class);
mockStatic(Realm.class);
mockStatic(RealmConfiguration.class);
Realm.init(RuntimeEnvironment.application);
// boilerplate to mock realm components -- this prevents us from hitting any
// native code
doNothing().when(RealmCore.class);
RealmCore.loadLibrary(any(Context.class));

```

#### Kotlin

```kotlin
// set up realm SDK components to be mocked. The order of these matters
PowerMockito.mockStatic(RealmCore::class.java)
PowerMockito.mockStatic(RealmLog::class.java)
PowerMockito.mockStatic(Realm::class.java)
PowerMockito.mockStatic(RealmConfiguration::class.java)
Realm.init(RuntimeEnvironment.application)
PowerMockito.doNothing().`when`(RealmCore::class.java)
RealmCore.loadLibrary(ArgumentMatchers.any(Context::class.java))

```

Once you've completed the setup required for mocking, you can start
mocking components and wiring up behavior for your tests. You can also
configure PowerMockito to return specific objects when new objects of
a type are instantiated, so even code that references the default
realm in your application won't break your tests:

#### Java

```java
// create the mocked realm
final Realm mockRealm = mock(Realm.class);
final RealmConfiguration mockRealmConfig = mock(RealmConfiguration.class);
// use this mock realm config for all new realm configurations
whenNew(RealmConfiguration.class).withAnyArguments().thenReturn(mockRealmConfig);
// use this mock realm for all new default realms
when(Realm.getDefaultInstance()).thenReturn(mockRealm);

```

#### Kotlin

```kotlin
// create the mocked realm
val mockRealm = PowerMockito.mock(Realm::class.java)
val mockRealmConfig = PowerMockito.mock(
    RealmConfiguration::class.java
)
// use this mock realm config for all new realm configurations
PowerMockito.whenNew(RealmConfiguration::class.java).withAnyArguments()
    .thenReturn(mockRealmConfig)
// use this mock realm for all new default realms
PowerMockito.`when`(Realm.getDefaultInstance()).thenReturn(mockRealm)

```

After mocking a realm, you'll have to configure data for your test
cases. See the full example below for some examples of how you can
provide testing data in unit tests.

### Full Example
The following example shows a full JUnit `test`
example mocking Realm in unit tests. This example tests
an activity that performs some basic Realm operations.
The tests use mocking to simulate those operations when that activity is
started during a unit test:

#### Java

```java
package com.mongodb.realm.examples.java;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;

import android.os.AsyncTask;
import android.util.Log;
import android.widget.LinearLayout;
import android.widget.TextView;

import com.mongodb.realm.examples.R;
import com.mongodb.realm.examples.model.java.Cat;

import io.realm.Realm;
import io.realm.RealmResults;

public class UnitTestActivity extends AppCompatActivity {

    public static final String TAG = UnitTestActivity.class.getName();
    private LinearLayout rootLayout = null;

    private Realm realm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Realm.init(getApplicationContext());
        setContentView(R.layout.activity_unit_test);
        rootLayout = findViewById(R.id.container);
        rootLayout.removeAllViews();

        // open the default Realm for the UI thread.
        realm = Realm.getDefaultInstance();

        // clean up from previous run
        cleanUp();

        // small operation that is ok to run on the main thread
        basicCRUD(realm);

        // more complex operations can be executed on another thread.
        AsyncTask<Void, Void, String> foo = new AsyncTask<Void, Void, String>() {
            @Override
            protected String doInBackground(Void... voids) {
                String info = "";
                info += complexQuery();
                return info;
            }

            @Override
            protected void onPostExecute(String result) {
                showStatus(result);
            }
        };

        foo.execute();

        findViewById(R.id.clean_up).setOnClickListener(view -> {
            view.setEnabled(false);
            Log.d("TAG", "clean up");
            cleanUp();
            view.setEnabled(true);
        });
    }

    private void cleanUp() {
        // delete all cats
        realm.executeTransaction(r -> r.delete(Cat.class));
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        realm.close(); // remember to close realm when done.
    }

    private void showStatus(String txt) {
        Log.i(TAG, txt);
        TextView tv = new TextView(this);
        tv.setText(txt);
        rootLayout.addView(tv);
    }

    private void basicCRUD(Realm realm) {
        showStatus("Perform basic Create/Read/Update/Delete (CRUD) operations...");

        // all writes must be wrapped in a transaction to facilitate safe multi threading
        realm.executeTransaction(r -> {
            // add a cat
            Cat cat = r.createObject(Cat.class);
            cat.setName("John Young");
        });

        // find the first cat (no query conditions) and read a field
        final Cat cat = realm.where(Cat.class).findFirst();
        showStatus(cat.getName());

        // update cat in a transaction
        realm.executeTransaction(r -> {
            cat.setName("John Senior");
        });

        showStatus(cat.getName());

        // add two more cats
        realm.executeTransaction(r -> {
            Cat jane = r.createObject(Cat.class);
            jane.setName("Jane");

            Cat doug = r.createObject(Cat.class);
            doug.setName("Robert");
        });

        RealmResults<Cat> cats = realm.where(Cat.class).findAll();
        showStatus(String.format("Found %s cats", cats.size()));
        for (Cat p : cats) {
            showStatus("Found " + p.getName());
        }
    }

    private String complexQuery() {
        String status = "\n\nPerforming complex Query operation...";

        Realm realm = Realm.getDefaultInstance();
        status += "\nNumber of cats in the DB: " + realm.where(Cat.class).count();

        // find all cats where name begins with "J".
        RealmResults<Cat> results = realm.where(Cat.class)
                .beginsWith("name", "J")
                .findAll();
        status += "\nNumber of cats whose name begins with 'J': " + results.size();

        realm.close();
        return status;
    }
}

```

```java
import android.content.Context;

import com.mongodb.realm.examples.java.UnitTestActivity;
import com.mongodb.realm.examples.model.java.Cat;

import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.core.classloader.annotations.PowerMockIgnore;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.core.classloader.annotations.SuppressStaticInitializationFor;
import org.powermock.modules.junit4.rule.PowerMockRule;
import org.robolectric.Robolectric;
import org.robolectric.RobolectricTestRunner;
import org.robolectric.RuntimeEnvironment;
import org.robolectric.annotation.Config;

import java.util.Arrays;
import java.util.List;

import io.realm.Realm;
import io.realm.RealmConfiguration;
import io.realm.RealmObject;
import io.realm.RealmQuery;
import io.realm.RealmResults;
import io.realm.internal.RealmCore;
import io.realm.log.RealmLog;

import com.mongodb.realm.examples.R;

import static org.mockito.Matchers.any;
import static org.mockito.Matchers.anyInt;
import static org.mockito.Matchers.anyString;
import static org.mockito.Mockito.doCallRealMethod;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.powermock.api.mockito.PowerMockito.doNothing;
import static org.powermock.api.mockito.PowerMockito.mock;
import static org.powermock.api.mockito.PowerMockito.mockStatic;
import static org.powermock.api.mockito.PowerMockito.when;
import static org.powermock.api.mockito.PowerMockito.whenNew;

@RunWith(RobolectricTestRunner.class)
@Config(sdk = 28)
@PowerMockIgnore({"org.mockito.*", "org.robolectric.*", "android.*", "jdk.internal.reflect.*", "androidx.*"})
@SuppressStaticInitializationFor("io.realm.internal.Util")
@PrepareForTest({Realm.class, RealmConfiguration.class, RealmQuery.class, RealmResults.class, RealmCore.class, RealmLog.class})
public class TestTest {
    // bootstrap powermock
    @Rule
    public PowerMockRule rule = new PowerMockRule();

    // mocked realm SDK components for tests
    private Realm mockRealm;
    private RealmResults<Cat> cats;

    @Before
    public void setup() throws Exception {
        // set up realm SDK components to be mocked. The order of these matters
        mockStatic(RealmCore.class);
        mockStatic(RealmLog.class);
        mockStatic(Realm.class);
        mockStatic(RealmConfiguration.class);
        Realm.init(RuntimeEnvironment.application);
        // boilerplate to mock realm components -- this prevents us from hitting any
        // native code
        doNothing().when(RealmCore.class);
        RealmCore.loadLibrary(any(Context.class));

        // create the mocked realm
        final Realm mockRealm = mock(Realm.class);
        final RealmConfiguration mockRealmConfig = mock(RealmConfiguration.class);
        // use this mock realm config for all new realm configurations
        whenNew(RealmConfiguration.class).withAnyArguments().thenReturn(mockRealmConfig);
        // use this mock realm for all new default realms
        when(Realm.getDefaultInstance()).thenReturn(mockRealm);

        // any time we ask Realm to create a Cat, return a new instance.
        when(mockRealm.createObject(Cat.class)).thenReturn(new Cat());

        // set up test data
        Cat p1 = new Cat();
        p1.setName("Enoch");
        Cat p2 = new Cat();
        p2.setName("Quincy Endicott");
        Cat p3 = new Cat();
        p3.setName("Sara");
        Cat p4 = new Cat();
        p4.setName("Jimmy Brown");
        List<Cat> catList = Arrays.asList(p1, p2, p3, p4);

        // create a mocked RealmQuery
        RealmQuery<Cat> catQuery = mockRealmQuery();
        // when the RealmQuery performs findFirst, return the first record in the list.
        when(catQuery.findFirst()).thenReturn(catList.get(0));
        // when the where clause is called on the Realm, return the mock query.
        when(mockRealm.where(Cat.class)).thenReturn(catQuery);
        // when the RealmQuery is filtered on any string and any integer, return the query
        when(catQuery.equalTo(anyString(), anyInt())).thenReturn(catQuery);
        // when a between query is performed with any string as the field and any int as the
        // value, then return the catQuery itself
        when(catQuery.between(anyString(), anyInt(), anyInt())).thenReturn(catQuery);
        // When a beginsWith clause is performed with any string field and any string value
        // return the same cat query
        when(catQuery.beginsWith(anyString(), anyString())).thenReturn(catQuery);

        // RealmResults is final, must mock static and also place this in the PrepareForTest
        // annotation array.
        mockStatic(RealmResults.class);
        // create a mock RealmResults
        RealmResults<Cat> cats = mockRealmResults();
        // the for(...) loop in Java needs an iterator, so we're giving it one that has items,
        // since the mock RealmResults does not provide an implementation. Therefore, any time
        // anyone asks for the RealmResults Iterator, give them a functioning iterator from the
        // ArrayList of Cats we created above. This will allow the loop to execute.
        when(cats.iterator()).thenReturn(catList.iterator());
        // Return the size of the mock list.
        when(cats.size()).thenReturn(catList.size());

        // when we ask Realm for all of the Cat instances, return the mock RealmResults
        when(mockRealm.where(Cat.class).findAll()).thenReturn(cats);
        // when we ask the RealmQuery for all of the Cat objects, return the mock RealmResults
        when(catQuery.findAll()).thenReturn(cats);

        this.mockRealm = mockRealm;
        this.cats = cats;
    }

    @Test
    public void shouldBeAbleToAccessActivityAndVerifyRealmInteractions() {
        doCallRealMethod().when(mockRealm)
                .executeTransaction(any(Realm.Transaction.class));

        // create test activity --  onCreate method calls methods that
        // query/write to realm
        UnitTestActivity activity = Robolectric
                .buildActivity(UnitTestActivity.class)
                .create()
                .start()
                .resume()
                .visible()
                .get();

        // click the clean up button
        activity.findViewById(R.id.clean_up).performClick();

        // verify that we queried for Cat instances five times in this run
        // (2 in basicCrud(), 2 in complexQuery() and 1 in the button click)
        verify(mockRealm, times(5)).where(Cat.class);

        // verify that the delete method was called. We also call delete at
        // the start of the activity to ensure we start with a clean db.
        verify(mockRealm, times(2)).delete(Cat.class);

        // call the destroy method so we can verify that the .close() method
        // was called (below)
        activity.onDestroy();

        // verify that the realm got closed 2 separate times. Once in the
        // AsyncTask, once in onDestroy
        verify(mockRealm, times(2)).close();
    }

    @SuppressWarnings("unchecked")
    private <T extends RealmObject> RealmQuery<T> mockRealmQuery() {
        return mock(RealmQuery.class);
    }

    @SuppressWarnings("unchecked")
    private <T extends RealmObject> RealmResults<T> mockRealmResults() {
        return mock(RealmResults.class);
    }
}

```

#### Kotlin

```kotlin
package com.mongodb.realm.examples.kotlin

import android.os.AsyncTask
import android.os.Bundle
import android.util.Log
import android.view.View
import android.widget.LinearLayout
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import com.mongodb.realm.examples.R
import com.mongodb.realm.examples.model.java.Cat
import io.realm.Realm

class UnitTestActivity : AppCompatActivity() {
    private var rootLayout: LinearLayout? = null
    private var realm: Realm? = null
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Realm.init(applicationContext)
        setContentView(R.layout.activity_unit_test)
        rootLayout = findViewById(R.id.container)
        rootLayout!!.removeAllViews()

        // open the default Realm for the UI thread.
        realm = Realm.getDefaultInstance()

        // clean up from previous run
        cleanUp()

        // small operation that is ok to run on the main thread
        basicCRUD(realm)

        // more complex operations can be executed on another thread.
        val foo: AsyncTask<Void?, Void?, String> = object : AsyncTask<Void?, Void?, String>() {
            protected override fun doInBackground(vararg params: Void?): String? {
                var info = ""
                info += complexQuery()
                return info
            }

            override fun onPostExecute(result: String) {
                showStatus(result)
            }
        }
        foo.execute()
        findViewById<View>(R.id.clean_up).setOnClickListener { view: View ->
            view.isEnabled = false
            Log.d("TAG", "clean up")
            cleanUp()
            view.isEnabled = true
        }
    }

    private fun cleanUp() {
        // delete all cats
        realm!!.executeTransaction { r: Realm -> r.delete(Cat::class.java) }
    }

    public override fun onDestroy() {
        super.onDestroy()
        realm!!.close() // remember to close realm when done.
    }

    private fun showStatus(txt: String) {
        Log.i(TAG, txt)
        val tv = TextView(this)
        tv.text = txt
        rootLayout!!.addView(tv)
    }

    private fun basicCRUD(realm: Realm?) {
        showStatus("Perform basic Create/Read/Update/Delete (CRUD) operations...")

        // all writes must be wrapped in a transaction to facilitate safe multi threading
        realm!!.executeTransaction { r: Realm ->
            // add a cat
            val cat = r.createObject(Cat::class.java)
            cat.name = "John Young"
        }

        // find the first cat (no query conditions) and read a field
        val cat = realm.where(Cat::class.java).findFirst()
        showStatus(cat!!.name)

        // update cat in a transaction
        realm.executeTransaction { r: Realm? ->
            cat.name = "John Senior"
        }
        showStatus(cat.name)

        // add two more cats
        realm.executeTransaction { r: Realm ->
            val jane = r.createObject(Cat::class.java)
            jane.name = "Jane"
            val doug = r.createObject(Cat::class.java)
            doug.name = "Robert"
        }
        val cats = realm.where(Cat::class.java).findAll()
        showStatus(String.format("Found %s cats", cats.size))
        for (p in cats) {
            showStatus("Found " + p.name)
        }
    }

    private fun complexQuery(): String {
        var status = "\n\nPerforming complex Query operation..."
        val realm = Realm.getDefaultInstance()
        status += """

            Number of cats in the DB: ${realm.where(Cat::class.java).count()}
            """.trimIndent()

        // find all cats where name begins with "J".
        val results = realm.where(Cat::class.java)
            .beginsWith("name", "J")
            .findAll()
        status += """

            Number of cats whose name begins with 'J': ${results.size}
            """.trimIndent()
        realm.close()
        return status
    }

    companion object {
        val TAG = UnitTestActivity::class.java.name
    }
}

```

```kotlin
import android.content.Context
import android.view.View
import com.mongodb.realm.examples.R
import com.mongodb.realm.examples.kotlin.UnitTestActivity
import com.mongodb.realm.examples.model.java.Cat
import io.realm.Realm
import io.realm.RealmConfiguration
import io.realm.RealmObject
import io.realm.RealmQuery
import io.realm.RealmResults
import io.realm.internal.RealmCore
import io.realm.log.RealmLog
import java.lang.Exception
import java.util.*
import org.junit.Before
import org.junit.Rule
import org.junit.Test
import org.junit.runner.RunWith
import org.mockito.ArgumentMatchers
import org.mockito.Mockito
import org.powermock.api.mockito.PowerMockito
import org.powermock.core.classloader.annotations.PowerMockIgnore
import org.powermock.core.classloader.annotations.PrepareForTest
import org.powermock.core.classloader.annotations.SuppressStaticInitializationFor
import org.powermock.modules.junit4.rule.PowerMockRule
import org.robolectric.Robolectric
import org.robolectric.RobolectricTestRunner
import org.robolectric.RuntimeEnvironment
import org.robolectric.annotation.Config

@RunWith(RobolectricTestRunner::class)
@Config(sdk = [28])
@PowerMockIgnore(
    "org.mockito.*",
    "org.robolectric.*",
    "android.*",
    "jdk.internal.reflect.*",
    "androidx.*"
)
@SuppressStaticInitializationFor("io.realm.internal.Util")
@PrepareForTest(
    Realm::class,
    RealmConfiguration::class,
    RealmQuery::class,
    RealmResults::class,
    RealmCore::class,
    RealmLog::class
)
class TestTest {
    // bootstrap powermock
    @Rule
    var rule = PowerMockRule()

    // mocked realm SDK components for tests
    private var mockRealm: Realm? = null
    private var cats: RealmResults<Cat>? = null
    @Before
    @Throws(Exception::class)
    fun setup() {
        // set up realm SDK components to be mocked. The order of these matters
        PowerMockito.mockStatic(RealmCore::class.java)
        PowerMockito.mockStatic(RealmLog::class.java)
        PowerMockito.mockStatic(Realm::class.java)
        PowerMockito.mockStatic(RealmConfiguration::class.java)
        Realm.init(RuntimeEnvironment.application)
        PowerMockito.doNothing().`when`(RealmCore::class.java)
        RealmCore.loadLibrary(ArgumentMatchers.any(Context::class.java))

        // create the mocked realm
        val mockRealm = PowerMockito.mock(Realm::class.java)
        val mockRealmConfig = PowerMockito.mock(
            RealmConfiguration::class.java
        )
        // use this mock realm config for all new realm configurations
        PowerMockito.whenNew(RealmConfiguration::class.java).withAnyArguments()
            .thenReturn(mockRealmConfig)
        // use this mock realm for all new default realms
        PowerMockito.`when`(Realm.getDefaultInstance()).thenReturn(mockRealm)

        // any time we ask Realm to create a Cat, return a new instance.
        PowerMockito.`when`(mockRealm.createObject(Cat::class.java)).thenReturn(Cat())

        // set up test data
        val p1 = Cat()
        p1.name = "Enoch"
        val p2 = Cat()
        p2.name = "Quincy Endicott"
        val p3 = Cat()
        p3.name = "Sara"
        val p4 = Cat()
        p4.name = "Jimmy Brown"
        val catList = Arrays.asList(p1, p2, p3, p4)

        // create a mocked RealmQuery
        val catQuery = mockRealmQuery<Cat>()
        // when the RealmQuery performs findFirst, return the first record in the list.
        PowerMockito.`when`(catQuery!!.findFirst()).thenReturn(catList[0])
        // when the where clause is called on the Realm, return the mock query.
        PowerMockito.`when`(mockRealm.where(Cat::class.java)).thenReturn(catQuery)
        // when the RealmQuery is filtered on any string and any integer, return the query
        PowerMockito.`when`(
            catQuery.equalTo(
                ArgumentMatchers.anyString(),
                ArgumentMatchers.anyInt()
            )
        ).thenReturn(catQuery)
        // when a between query is performed with any string as the field and any int as the
        // value, then return the catQuery itself
        PowerMockito.`when`(
            catQuery.between(
                ArgumentMatchers.anyString(),
                ArgumentMatchers.anyInt(),
                ArgumentMatchers.anyInt()
            )
        ).thenReturn(catQuery)
        // When a beginsWith clause is performed with any string field and any string value
        // return the same cat query
        PowerMockito.`when`(
            catQuery.beginsWith(
                ArgumentMatchers.anyString(),
                ArgumentMatchers.anyString()
            )
        ).thenReturn(catQuery)

        // RealmResults is final, must mock static and also place this in the PrepareForTest
        // annotation array.
        PowerMockito.mockStatic(RealmResults::class.java)
        // create a mock RealmResults
        val cats = mockRealmResults<Cat>()
        // the for(...) loop in Java needs an iterator, so we're giving it one that has items,
        // since the mock RealmResults does not provide an implementation. Therefore, any time
        // anyone asks for the RealmResults Iterator, give them a functioning iterator from the
        // ArrayList of Cats we created above. This will allow the loop to execute.
        PowerMockito.`when`<Iterator<Cat>>(cats!!.iterator()).thenReturn(catList.iterator())
        // Return the size of the mock list.
        PowerMockito.`when`(cats.size).thenReturn(catList.size)

        // when we ask Realm for all of the Cat instances, return the mock RealmResults
        PowerMockito.`when`(mockRealm.where(Cat::class.java).findAll()).thenReturn(cats)
        // when we ask the RealmQuery for all of the Cat objects, return the mock RealmResults
        PowerMockito.`when`(catQuery.findAll()).thenReturn(cats)
        this.mockRealm = mockRealm
        this.cats = cats
    }

    @Test
    fun shouldBeAbleToAccessActivityAndVerifyRealmInteractions() {
        Mockito.doCallRealMethod().`when`(mockRealm)!!
            .executeTransaction(ArgumentMatchers.any(Realm.Transaction::class.java))

        // create test activity --  onCreate method calls methods that
        // query/write to realm
        val activity = Robolectric
            .buildActivity(UnitTestActivity::class.java)
            .create()
            .start()
            .resume()
            .visible()
            .get()

        // click the clean up button
        activity.findViewById<View>(R.id.clean_up).performClick()

        // verify that we queried for Cat instances five times in this run
        // (2 in basicCrud(), 2 in complexQuery() and 1 in the button click)
        Mockito.verify(mockRealm, Mockito.times(5))!!.where(Cat::class.java)

        // verify that the delete method was called. We also call delete at
        // the start of the activity to ensure we start with a clean db.
        Mockito.verify(mockRealm, Mockito.times(2))!!.delete(Cat::class.java)

        // call the destroy method so we can verify that the .close() method
        // was called (below)
        activity.onDestroy()

        // verify that the realm got closed 2 separate times. Once in the
        // AsyncTask, once in onDestroy
        Mockito.verify(mockRealm, Mockito.times(2))!!.close()
    }

    private fun <T : RealmObject?> mockRealmQuery(): RealmQuery<T>? {
        @Suppress("UNCHECKED_CAST")
        return PowerMockito.mock(RealmQuery::class.java) as RealmQuery<T>
    }

    private fun <T : RealmObject?> mockRealmResults(): RealmResults<T>? {
        @Suppress("UNCHECKED_CAST")
        return PowerMockito.mock(RealmResults::class.java) as RealmResults<T>
    }
}

```

> Seealso:
> See the [Unit Testing Example App](https://github.com/realm/realm-java/tree/master/examples/unitTestExample)
for an example of unit testing an application that uses
Realm.
>
