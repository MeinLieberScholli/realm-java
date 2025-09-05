# Display Collections - Java SDK
Android apps often populate the UI using
[RecyclerView](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.html)
or [ListView](https://developer.android.com/reference/android/widget/ListView) components.
Realm offers **adapters** to display realm object
collections. These collections implement
the `OrderedRealmCollections` interface. RealmResults
and RealmList are examples of these adaptors.
With these adapters, UI components update when your app changes
Realm objects.

## Install Adapters
Add these dependencies to your application level `build.gradle` file:

```gradle
dependencies {
   implementation 'io.realm:android-adapters:4.0.0'
   implementation 'androidx.recyclerview:recyclerview:1.1.0'
}
```

Realm hosts these adapters on the
[JCenter](https://mvnrepository.com/repos/jcenter)
artifact repository. To use `jcenter` in your Android app, add it to your
project-level `build.gradle` file:

```gradle
buildscript {
    repositories {
        jcenter()
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

> Seealso:
> Source code: [realm/realm-android-adapters](https://github.com/realm/realm-android-adapters) on GitHub.
>

## Example Models
The examples on this page use a Realm object named `Item`.
This class contains a string named "name" and an identifier number named
"id":

#### Java

```java

import io.realm.RealmObject;

public class Item extends RealmObject {
    int id;
    String name;

    public Item() {}

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

```

#### Kotlin

```kotlin

import io.realm.RealmObject

open class Item(var id: Int = 0,
                var name: String? = null): RealmObject()

```

## Display Collections in a ListView
Display Realm objects in a
[ListView](https://developer.android.com/reference/android/widget/ListView) by extending
[RealmBaseAdapter](https://github.com/realm/realm-android-adapters/blob/master/adapters/src/main/java/io/realm/RealmBaseAdapter.java).
The adapter uses the `ListAdapter` interface. Implementation works
like any `ListAdapter`. This provides support for automatically-updating
Realm objects.

Subclass `RealmBaseAdapter` to display
Item objects in a `ListView`:

#### Java

```java
import android.util.Log;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.ListAdapter;
import android.widget.TextView;
import com.mongodb.realm.examples.model.java.Item;
import io.realm.OrderedRealmCollection;
import io.realm.RealmBaseAdapter;

class ExampleListAdapter extends RealmBaseAdapter<Item> implements ListAdapter {
    String TAG = "REALM_LIST_ADAPTER";

    ExampleListAdapter(OrderedRealmCollection<Item> realmResults) {
        super(realmResults);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if (convertView == null) {
            Log.i(TAG, "Creating view holder");
            // create a top-level layout for our item views
            LinearLayout layout = new LinearLayout(parent.getContext());
            layout.setLayoutParams(
                    new ViewGroup.LayoutParams(
                            ViewGroup.LayoutParams.MATCH_PARENT,
                            ViewGroup.LayoutParams.MATCH_PARENT));

            // create a text view to display item names
            TextView titleView = new TextView(parent.getContext());
            titleView.setLayoutParams(
                    new ViewGroup.LayoutParams(
                            ViewGroup.LayoutParams.MATCH_PARENT,
                            ViewGroup.LayoutParams.MATCH_PARENT));

            // attach the text view to the item view layout
            layout.addView(titleView);
            convertView = layout;
            viewHolder = new ViewHolder(titleView);
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }

        // as long as we
        if (adapterData != null) {
            final Item item = adapterData.get(position);
            viewHolder.title.setText(item.getName());
            Log.i(TAG, "Populated view holder with data: " + item.getName());
        } else {
            Log.e(TAG, "No data in adapter! Failed to populate view holder.");
        }
        return convertView;
    }

    private static class ViewHolder {
        TextView title;

        public ViewHolder(TextView textView) {
            title = textView;
        }
    }
}

```

To display list data in an activity, instantiate a `ListView`. Then,
attach an `ExampleListAdapter`:

```java
// instantiate a ListView programmatically
ListView listView = new ListView(activity.getApplicationContext());
listView.setLayoutParams(
        new ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT));

// create an adapter with a RealmResults collection
// and attach it to the ListView
ExampleListAdapter adapter =
        new ExampleListAdapter(
                realm.where(Item.class).findAll());
listView.setAdapter(adapter);
ViewGroup.LayoutParams layoutParams =
        new ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT);
activity.addContentView(listView, layoutParams);

```

#### Kotlin

```kotlin
import android.util.Log
import android.view.View
import android.view.ViewGroup
import android.widget.LinearLayout
import android.widget.ListAdapter
import android.widget.TextView
import com.mongodb.realm.examples.model.kotlin.Item
import io.realm.OrderedRealmCollection
import io.realm.RealmBaseAdapter

internal class ExampleListAdapter(realmResults: OrderedRealmCollection<Item?>?) :
    RealmBaseAdapter<Item?>(realmResults), ListAdapter {
    var TAG = "REALM_LIST_ADAPTER"

    override fun getView(position: Int,
                         convertView: View?,
                         parent: ViewGroup): View {
        var convertView = convertView
        val viewHolder: ViewHolder
        if (convertView == null) {
            Log.i(TAG, "Creating view holder")
            // create a top-level layout for our item views
            val layout = LinearLayout(parent.context)
            layout.layoutParams = ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT
            )

            // create a text view to display item names
            val titleView = TextView(parent.context)
            titleView.layoutParams = ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT
            )

            // attach the text view to the item view layout
            layout.addView(titleView)
            convertView = layout
            viewHolder = ViewHolder(titleView)
            convertView.tag = viewHolder
        } else {
            viewHolder = convertView.tag as ViewHolder
        }

        // as long as we
        if (adapterData != null) {
            val item = adapterData!![position]!!
            viewHolder.title.text = item.name
            Log.i(TAG, "Populated view holder with data: ${item.name}")
        } else {
            Log.e(TAG, "No data in adapter! Failed to populate view holder.")
        }
        return convertView
    }

    private class ViewHolder(var title: TextView)
}

```

To display list data in an activity, instantiate a `ListView`. Then,
attach an `ExampleListAdapter`:

```kotlin
// instantiate a ListView programmatically
val listView = ListView(activity!!.applicationContext)
listView.layoutParams = ViewGroup.LayoutParams(
    ViewGroup.LayoutParams.MATCH_PARENT,
    ViewGroup.LayoutParams.MATCH_PARENT
)

// create an adapter with a RealmResults collection
// and attach it to the ListView
val adapter = ExampleListAdapter(realm.where(Item::class.java).findAll())
listView.adapter = adapter
val layoutParams = ViewGroup.LayoutParams(
    ViewGroup.LayoutParams.MATCH_PARENT,
    ViewGroup.LayoutParams.MATCH_PARENT
)
activity!!.addContentView(listView, layoutParams)

```

## Display Collections in a RecyclerView
Display Realm objects in a
[RecyclerView](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.html)
by extending [RealmRecyclerViewAdapter](https://github.com/realm/realm-android-adapters/blob/master/adapters/src/main/java/io/realm/RealmRecyclerViewAdapter.java).
The adapter extends `RecyclerView.Adapter`. Implementation works like any
`RecyclerView` adapter. This provides support
for automatically-updating Realm objects.

Subclass `RealmRecyclerViewAdapter` to display
Item objects in a `RecyclerView`:

#### Java

```java
import android.util.Log;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.recyclerview.widget.RecyclerView;
import com.mongodb.realm.examples.model.java.Item;
import io.realm.OrderedRealmCollection;
import io.realm.RealmRecyclerViewAdapter;

/*
 * ExampleRecyclerViewAdapter: extends the Realm-provided
 * RealmRecyclerViewAdapter to provide data
 * for a RecyclerView to display
 * Realm objects on screen to a user.
 */
class ExampleRecyclerViewAdapter
        extends RealmRecyclerViewAdapter<Item,
        ExampleRecyclerViewAdapter.ExampleViewHolder> {
    String TAG = "REALM_RECYCLER_ADAPTER";

    ExampleRecyclerViewAdapter(OrderedRealmCollection<Item> data) {
        super(data, true);
        Log.i(TAG, "Created RealmRecyclerViewAdapter for "
                + getData().size() + " items.");
    }

    @Override
    public ExampleViewHolder onCreateViewHolder(ViewGroup parent,
                                                int viewType) {
        Log.i(TAG, "Creating view holder");
        TextView textView = new TextView(parent.getContext());
        textView.setLayoutParams(
                new ViewGroup.LayoutParams(
                        ViewGroup.LayoutParams.MATCH_PARENT,
                        ViewGroup.LayoutParams.WRAP_CONTENT));
        return new ExampleViewHolder(textView);
    }

    @Override
    public void onBindViewHolder(ExampleViewHolder holder,
                                 int position) {
        final Item obj = getItem(position);
        Log.i(TAG, "Binding view holder: " + obj.getName());
        holder.data = obj;
        holder.title.setText(obj.getName());
    }

    @Override
    public long getItemId(int index) {
        return getItem(index).getId();
    }

    class ExampleViewHolder extends RecyclerView.ViewHolder {
        TextView title;
        public Item data;

        ExampleViewHolder(TextView view) {
            super(view);
            title = view;
        }
    }
}

```

To display list data in an activity, instantiate a `RecyclerView`. Then,
attach an `ExampleRecyclerViewAdapter`:

```java
// instantiate a RecyclerView programmatically
RecyclerView recyclerView =
        new RecyclerView(activity.getApplicationContext());
recyclerView.setLayoutManager(
        new LinearLayoutManager(activity.getApplicationContext()));
recyclerView.setHasFixedSize(true);
recyclerView.addItemDecoration(new DividerItemDecoration(
        activity.getApplicationContext(),
        DividerItemDecoration.VERTICAL));

// create an adapter with a RealmResults collection
// and attach it to the RecyclerView
ExampleRecyclerViewAdapter adapter =
        new ExampleRecyclerViewAdapter(
                realm.where(Item.class).findAll());
recyclerView.setAdapter(adapter);
ViewGroup.LayoutParams layoutParams =
        new ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT);
activity.addContentView(recyclerView, layoutParams);

```

#### Kotlin

```kotlin
import android.util.Log
import android.view.ViewGroup
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView
import com.mongodb.realm.examples.model.kotlin.Item
import io.realm.OrderedRealmCollection
import io.realm.RealmRecyclerViewAdapter

/*
 * ExampleRecyclerViewAdapter: extends the Realm-provided
 * RealmRecyclerViewAdapter to provide data
 * for a RecyclerView to display
 * Realm objects on screen to a user.
 */
internal class ExampleRecyclerViewAdapter(data: OrderedRealmCollection<Item?>?) :
    RealmRecyclerViewAdapter<Item?,
            ExampleRecyclerViewAdapter.ExampleViewHolder?>(data, true) {
    var TAG = "REALM_RECYCLER_ADAPTER"

    override fun onCreateViewHolder(parent: ViewGroup,
                                    viewType: Int): ExampleViewHolder {
        Log.i(TAG, "Creating view holder")
        val textView = TextView(parent.context)
        textView.layoutParams = ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        return ExampleViewHolder(textView)
    }

    override fun onBindViewHolder(holder: ExampleViewHolder, position: Int) {
        val obj = getItem(position)
        Log.i(TAG, "Binding view holder: ${obj!!.name}")
        holder.data = obj
        holder.title.text = obj.name
    }

    override fun getItemId(index: Int): Long {
        return getItem(index)!!.id.toLong()
    }

    internal inner class ExampleViewHolder(var title: TextView)
        : RecyclerView.ViewHolder(title) {
        var data: Item? = null
    }

    init {
        Log.i(TAG,
            "Created RealmRecyclerViewAdapter for ${getData()!!.size} items.")
    }
}

```

To display list data in an activity, instantiate a `RecyclerView`. Then,
attach an `ExampleRecyclerViewAdapter`:

```kotlin
// instantiate a RecyclerView programmatically
val recyclerView = RecyclerView(activity!!.applicationContext)
recyclerView.layoutManager =
    LinearLayoutManager(activity!!.applicationContext)
recyclerView.setHasFixedSize(true)
recyclerView.addItemDecoration(
    DividerItemDecoration(activity!!.applicationContext,
        DividerItemDecoration.VERTICAL))

// create an adapter with a RealmResults collection
// and attach it to the RecyclerView
val adapter = ExampleRecyclerViewAdapter(realm.where(Item::class.java).findAll())
recyclerView.adapter = adapter
val layoutParams = ViewGroup.LayoutParams(
    ViewGroup.LayoutParams.MATCH_PARENT,
    ViewGroup.LayoutParams.MATCH_PARENT
)
activity!!.addContentView(recyclerView, layoutParams)

```

