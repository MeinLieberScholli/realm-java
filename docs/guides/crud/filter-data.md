# Filter Data - Java SDK
## Query Engine
To filter data in your realm, use the Realm query engine.

There are two ways to access the query engine with the Java SDK:

- Fluent interface
- Realm Query Language

## Fluent Interface
The Java SDK uses a [Fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)
to construct multi-clause queries that are passed to the query engine.

See RealmQuery API
for a complete list of available methods.

There are several types of operators available to filter a
Realm collection.
Filters work by **evaluating** an operator expression for
every object in the collection being
filtered. If the expression resolves to `true`, Realm
Database includes the object in the results collection.

An **expression** consists of one of the following:

- The name of a property of the object currently being evaluated.
- An operator and up to two argument expression(s).
- A literal string, number, or date.

### About the Examples In This Section
The examples in this section use a simple data set for a
task list app. The two Realm object types are `Project`
and `Task`. A `Task` has a name, assignee's name, and
completed flag. There is also an arbitrary number for
priority (higher is more important) and a count of
minutes spent working on it. A `Project` has zero or more
`Tasks`.

See the schema for these two classes, `Project` and
`Task`, below:

#### Java

```java

import org.bson.types.ObjectId;

import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.RealmClass;
import io.realm.annotations.Required;

public class ProjectTask extends RealmObject {
    @PrimaryKey
    public ObjectId _id;
    @Required
    public String name;
    public String assignee;
    public int progressMinutes;
    public boolean isComplete;
    public int priority;
    @Required
    public String _partition;
}

```

```java

import org.bson.types.ObjectId;

import io.realm.RealmList;
import io.realm.RealmObject;
import io.realm.annotations.PrimaryKey;
import io.realm.annotations.RealmClass;
import io.realm.annotations.Required;

public class Project extends RealmObject {
    @PrimaryKey
    public ObjectId _id;
    @Required
    public String name;
    public RealmList<ProjectTask> tasks = new RealmList<>();
}

```

#### Kotlin

```kotlin
import io.realm.RealmObject
import io.realm.annotations.PrimaryKey
import io.realm.annotations.Required
import org.bson.types.ObjectId

open class ProjectTask(
    @PrimaryKey
    var _id: ObjectId = ObjectId(),
    @Required
    var name: String = "",
    var assignee: String? = null,
    var progressMinutes: Int = 0,
    var isComplete: Boolean = false,
    var priority: Int = 0,
    var _partition: String = ""
): RealmObject()

```

```kotlin
import io.realm.RealmList
import io.realm.RealmObject
import io.realm.annotations.PrimaryKey
import io.realm.annotations.Required
import org.bson.types.ObjectId

open class Project(
    @PrimaryKey
    var _id: ObjectId = ObjectId(),
    @Required
    var name: String = "",
    var tasks: RealmList<ProjectTask> = RealmList(),
): RealmObject()

```

### Comparison Operators
The most straightforward operation in a search is to compare
values.

|Operator|Description|
| --- | --- |
|`between`|Evaluates to `true` if the left-hand numerical or date expression is between or equal to the right-hand range. For dates, this evaluates to `true` if the left-hand date is within the right-hand date range.|
|equalTo|Evaluates to `true` if the left-hand expression is equal to the right-hand expression.|
|greaterThan|Evaluates to `true` if the left-hand numerical or date expression is greater than the right-hand numerical or date expression. For dates, this evaluates to `true` if the left-hand date is later than the right-hand date.|
|greaterThanOrEqualTo|Evaluates to `true` if the left-hand numerical or date expression is greater than or equal to the right-hand numerical or date expression. For dates, this evaluates to `true` if the left-hand date is later than or the same as the right-hand date.|
|`in`|Evaluates to `true` if the left-hand expression is in the right-hand list.|
|lessThan|Evaluates to `true` if the left-hand numerical or date expression is less than the right-hand numerical or date expression. For dates, this evaluates to `true` if the left-hand date is earlier than the right-hand date.|
|lessThanOrEqualTo|Evaluates to `true` if the left-hand numeric expression is less than or equal to the right-hand numeric expression. For dates, this evaluates to `true` if the left-hand date is earlier than or the same as the right-hand date.|
|notEqualTo|Evaluates to `true` if the left-hand expression is not equal to the right-hand expression.|

> Example:
> The following example uses the query engine's
comparison operators to:
>
> - Find high priority tasks by comparing the value of the `priority` property value with a threshold number, above which priority can be considered high.
> - Find just-started or short-running tasks by seeing if the `progressMinutes` property falls within a certain range.
> - Find unassigned tasks by finding tasks where the `assignee` property is equal to `null`.
> - Find tasks assigned to specific teammates Ali or Jamie by seeing if the `assignee` property is in a list of names.
>
> #### Java
>
> ```java
> RealmQuery<ProjectTask> tasksQuery = realm.where(ProjectTask.class);
> Log.i("EXAMPLE", "High priority tasks: " + tasksQuery.greaterThan("priority", 5).count());
> Log.i("EXAMPLE", "Just-started or short tasks: " + tasksQuery.between("progressMinutes", 1, 10).count());
> Log.i("EXAMPLE", "Unassigned tasks: " + tasksQuery.isNull("assignee").count());
> Log.i("EXAMPLE", "Ali or Jamie's tasks: " + tasksQuery.in("assignee", new String[]{"Ali", "Jamie"}).count());
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> val tasksQuery = realm.where(ProjectTask::class.java)
> Log.i("EXAMPLE", "High priority tasks: " + tasksQuery.greaterThan("priority", 5).count())
> Log.i("EXAMPLE", "Just-started or short tasks: " + tasksQuery.between("progressMinutes", 1, 10).count())
> Log.i("EXAMPLE", "Unassigned tasks: " + tasksQuery.isNull("assignee").count())
> Log.i("EXAMPLE", "Ali or Jamie's tasks: " + tasksQuery.`in`("assignee", arrayOf("Ali", "Jamie")).count())
>
> ```
>
>

### Logical Operators
You can make compound predicates using logical operators.

|Operator|Description|
| --- | --- |
|and|Evaluates to `true` if both left-hand and right-hand expressions are `true`.|
|not|Negates the result of the given expression.|
|or|Evaluates to `true` if either expression returns `true`.|

> Example:
> We can use the query language's logical operators to find
all of Ali's completed tasks. That is, we find all tasks
where the `assignee` property value is equal to 'Ali' AND
the `isComplete` property value is `true`:
>
> #### Java
>
> ```java
> RealmQuery<ProjectTask> tasksQuery = realm.where(ProjectTask.class);
> Log.i("EXAMPLE", "Ali has completed " +
>         tasksQuery.equalTo("assignee", "Ali").and().equalTo("isComplete", true).findAll().size() +
>         " tasks.");
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> val tasksQuery = realm.where(ProjectTask::class.java)
> Log.i("EXAMPLE", "Ali has completed " +
>             tasksQuery.equalTo("assignee", "Ali").and()
>                 .equalTo("isComplete", true).findAll().size + " tasks.")
>
> ```
>
>

### String Operators
You can compare string values using these string operators.
Regex-like wildcards allow more flexibility in search.

|Operator|Description|
| --- | --- |
|beginsWith|Evaluates to `true` if the left-hand string expression begins with the right-hand string expression. This is similar to `contains`, but only matches if the left-hand string expression is found at the beginning of the right-hand string expression.|
|`contains`|Evaluates to `true` if the left-hand string expression is found anywhere in the right-hand string expression.|
|endsWith|Evaluates to `true` if the left-hand string expression ends with the right-hand string expression. This is similar to `contains`, but only matches if the left-hand string expression is found at the very end of the right-hand string expression.|
|like|Evaluates to `true` if the left-hand string expression matches the right-hand string wildcard string expression. A wildcard string expression is a string that uses normal characters with two special wildcard characters: The `*` wildcard matches zero or more of any character The `?` wildcard matches any character. For example, the wildcard string "d?g" matches "dog", "dig", and "dug", but not "ding", "dg", or "a dog".|
|equalTo|Evaluates to `true` if the left-hand string is lexicographically equal to the right-hand string.|

> Example:
> We use the query engine's string operators to find
projects with a name starting with the letter 'e' and
projects with names that contain 'ie':
>
> #### Java
>
> ```java
> RealmQuery<Project> projectsQuery = realm.where(Project.class);
> // Pass Case.INSENSITIVE as the third argument for case insensitivity.
> Log.i("EXAMPLE", "Projects that start with 'e': "
>         + projectsQuery.beginsWith("name", "e", Case.INSENSITIVE).count());
> Log.i("EXAMPLE", "Projects that contain 'ie': "
>         + projectsQuery.contains("name", "ie").count());
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> val projectsQuery = realm.where(Project::class.java)
> // Pass Case.INSENSITIVE as the third argument for case insensitivity.
> Log.i("EXAMPLE", "Projects that start with 'e': "
>             + projectsQuery.beginsWith("name", "e", Case.INSENSITIVE).count())
> Log.i("EXAMPLE", "Projects that contain 'ie': "
>         + projectsQuery.contains("name", "ie").count())
>
> ```
>
>

> Note:
> Case-insensitive string operators only support the
`Latin Basic`, `Latin Supplement`, `Latin Extended A`, and
`Latin Extended B (UTF-8 range 0-591)` character sets. Setting
the case insensitive flag in queries when using `equalTo`,
`notEqualTo`, `contains`, `endsWith`, `beginsWith`, or
`like` only works on English locale characters.
>

### Aggregate Operators
You can apply an aggregate operator to a collection property
of a Realm object. Aggregate operators traverse a
collection and reduce it
to a single value.

|Operator|Description|
| --- | --- |
|average|Evaluates to the average value of a given numerical property across a collection.|
|count|Evaluates to the number of objects in the given collection.|
|max|Evaluates to the highest value of a given numerical property across a collection.|
|min|Evaluates to the lowest value of a given numerical property across a collection.|
|sum|Evaluates to the sum of a given numerical property across a collection.|

> Example:
> We create a couple of filters to show different facets of
the data:
>
> - Projects with average tasks priority above 5.
> - Long running projects.
>
> #### Java
>
> ```java
> RealmQuery<ProjectTask> tasksQuery = realm.where(ProjectTask.class);
> /*
> Aggregate operators do not support dot-notation, so you
> cannot directly operate on a property of all of the objects
> in a collection property.
>
> You can operate on a numeric property of the top-level
> object, however:
> */
> Log.i("EXAMPLE", "Tasks average priority: " + tasksQuery.average("priority"));
>
> ```
>
>
> #### Kotlin
>
> ```kotlin
> val tasksQuery = realm.where(ProjectTask::class.java)
> /*
> Aggregate operators do not support dot-notation, so you
> cannot directly operate on a property of all of the objects
> in a collection property.
>
> You can operate on a numeric property of the top-level
> object, however:
> */Log.i("EXAMPLE", "Tasks average priority: " + tasksQuery.average("priority"))
>
> ```
>
>

## Filter, Sort, Limit, Unique, and Chain Queries
### About the Examples in This Section
The examples in this section use two Realm object types: `Teacher`
and `Student`.

See the schema for these two classes below:

#### Java

```java
import io.realm.RealmList;
import io.realm.RealmObject;

public class Teacher extends RealmObject {
    private String name;
    private Integer numYearsTeaching;
    private String subject;
    private RealmList<Student> students;
    public Teacher() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public Integer getNumYearsTeaching() { return numYearsTeaching; }
    public void setNumYearsTeaching(Integer numYearsTeaching) { this.numYearsTeaching = numYearsTeaching; }
    public String getSubject() { return subject; }
    public void setSubject(String subject) { this.subject = subject; }
    public RealmList<Student> getStudents() { return students; }
    public void setStudents(RealmList<Student> students) { this.students = students; }
}

```

```java
import io.realm.RealmObject;
import io.realm.RealmResults;
import io.realm.annotations.LinkingObjects;

public class Student extends RealmObject {
    private String name;
    private Integer year;
    @LinkingObjects("students")
    private final RealmResults<Teacher> teacher = null;
    public Student() {}

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public Integer getYear() { return year; }
    public void setYear(Integer year) { this.year = year; }
    public RealmResults<Teacher> getTeacher() { return teacher; }
}

```

#### Kotlin

```kotlin
import io.realm.RealmList
import io.realm.RealmObject

open class Teacher : RealmObject() {
    var name: String? = null
    var numYearsTeaching: Int? = null
    var subject: String? = null
    var students: RealmList<Student>? = null
}

```

```kotlin
import io.realm.RealmObject
import io.realm.RealmResults
import io.realm.annotations.LinkingObjects

open class Student : RealmObject() {
    var name: String? = null
    var year: Int? = null

    @LinkingObjects("students")
    val teacher: RealmResults<Teacher>? = null
}

```

### Filters
You can build filters using the operator methods of the
[fluent interface](https://en.wikipedia.org/wiki/Fluent_interface) exposed by the
`RealmQuery` class:

#### Java

```java
// Build the query looking at all teachers:
RealmQuery<Teacher> query = realm.where(Teacher.class);

// Add query conditions:
query.equalTo("name", "Ms. Langtree");
query.or().equalTo("name", "Mrs. Jacobs");

// Execute the query:
RealmResults<Teacher> result1 = query.findAll();

// Or alternatively do the same all at once (the "Fluent interface"):
RealmResults<Teacher> result2 = realm.where(Teacher.class)
        .equalTo("name", "Ms. Langtree")
        .or()
        .equalTo("name", "Mrs. Jacobs")
        .findAll();

```

#### Kotlin

```kotlin
// Build the query looking at all teachers:
val query = realm.where(Teacher::class.java)

// Add query conditions:
query.equalTo("name", "Ms. Langtree")
query.or().equalTo("name", "Mrs. Jacobs")

// Execute the query:
val result1 = query.findAll()

// Or alternatively do the same all at once (the "Fluent interface"):
val result2 = realm.where(Teacher::class.java)
    .equalTo("name", "Ms. Langtree")
    .or()
    .equalTo("name", "Mrs. Jacobs")
    .findAll()

```

This gives you a new instance of the class `RealmResults`,
containing teachers with the name "Ms. Langtree" or "Mrs. Jacobs".

`RealmQuery` includes several methods that can execute queries:

- `findAll()` blocks until
it finds all objects that meet the query conditions
- `findAllAsync()`
returns immediately and finds all objects that meet the query
conditions asynchronously on a background thread
- `findFirst()` blocks
until it finds the first object that meets the query conditions
- `findFirstAsync()`
returns immediately and finds the first object that meets the query
conditions asynchronously on a background thread

Queries return a list of references to the matching Realm
objects using the RealmResults type.

#### Link Queries
When referring to an object property, you can use **dot notation** to refer
to child properties of that object. You can refer to the properties of
embedded objects and relationships with dot notation.

For example, consider a query for all teachers with a student named
"Wirt" or "Greg":

#### Java

```java
// Find all teachers who have students with the names "Wirt" or "Greg"
RealmResults<Teacher> result = realm.where(Teacher.class)
        .equalTo("students.name", "Wirt")
        .or()
        .equalTo("students.name", "Greg")
        .findAll();

```

#### Kotlin

```kotlin
// Find all teachers who have students with the names "Wirt" or "Greg"
val result = realm.where(Teacher::class.java)
    .equalTo("students.name", "Wirt")
    .or()
    .equalTo("students.name", "Greg")
    .findAll()

```

You can even use dot notation to query inverse relationships:

#### Java

```java
// Find all students who have teachers with the names "Ms. Langtree" or "Mrs. Jacobs"
RealmResults<Student> result = realm.where(Student.class)
        .equalTo("teacher.name", "Ms. Langtree")
        .or()
        .equalTo("teacher.name", "Mrs. Jacobs")
        .findAll();

```

#### Kotlin

```kotlin
// Find all students who have teachers with the names "Ms. Langtree" or "Mrs. Jacobs"
val result = realm.where(Student::class.java)
    .equalTo("teacher.name", "Ms. Langtree")
    .or()
    .equalTo("teacher.name", "Mrs. Jacobs")
    .findAll()

```

### Sort Results
> Important:
> Realm applies the `distinct()`, `sort()` and
`limit()` methods in the order you specify. Depending on the
data set this can alter the query result. Generally, you should
apply `limit()` last to avoid unintended result sets.
>

You can define the order of query results using the
`sort()`
method:

#### Java

```java
// Find all students in year 7, and sort them by name
RealmResults<Student> result = realm.where(Student.class)
        .equalTo("year", 7)
        .sort("name")
        .findAll();

// Alternatively, find all students in year 7
RealmResults<Student> unsortedResult = realm.where(Student.class)
        .equalTo("year", 7)
        .findAll();
// then sort the results set by name
RealmResults<Student> sortedResult = unsortedResult.sort("name");

```

#### Kotlin

```kotlin
// Find all students in year 7, and sort them by name
val result: RealmResults<Student> = realm.where(Student::class.java)
    .equalTo("year", 7L)
    .sort("name")
    .findAll()

// Alternatively, find all students in year 7
val unsortedResult: RealmResults<Student> = realm.where(Student::class.java)
    .equalTo("year", 7L)
    .findAll()
// then sort the results set by name
val sortedResult = unsortedResult.sort("name")

```

Sorts organize results in ascending order by default. To organize results
in descending order, pass `Sort.DESCENDING` as a second argument.
You can resolve sort order ties between identical property values
by passing an array of properties instead of a single property: in the
event of a tie, Realm sorts the tied objects by subsequent
properties in order.

> Note:
> Realm uses non-standard sorting for upper and lowercase
letters, sorting them together rather than sorting uppercase first.
As a result, `'- !"#0&()*,./:;?_+<=>123aAbBcC...xXyYzZ` is the
actual sorting order in Realm. Additionally, sorting
strings only supports the `Latin Basic`, `Latin Supplement`,
`Latin Extended A`, and `Latin Extended B (UTF-8 range 0–591)`
character sets.
>

### Limit Results
You can cap the number of query results to a specific maximum number
using the `limit()`
method:

#### Java

```java
// Find all students in year 8, and limit the results collection to 10 items
RealmResults<Student> result = realm.where(Student.class)
        .equalTo("year", 8)
        .limit(10)
        .findAll();

```

#### Kotlin

```kotlin
// Find all students in year 8, and limit the results collection to 10 items
val result: RealmResults<Student> = realm.where(Student::class.java)
    .equalTo("year", 8L)
    .limit(10)
    .findAll()

```

Limited result collections automatically update like any other query
result. Consequently, objects might drop out of the collection as
underlying data changes.

> Tip:
> Some databases encourage paginating results with limits to avoid
reading unnecessary data from disk or using too much memory.
>
> Since Realm queries are lazy, there is no need to
take such measures. Realm only loads objects from query
results when they are explicitly accessed.
>

> Tip:
> Collection notifications
report objects as deleted when they drop out of the result set.
This does not necessarily mean that they have been deleted from the
underlying realm, just that they are no longer part of the
query result.
>

### Unique Results
You can reduce query results to unique values for a given field or fields
using the `distinct()` method:

#### Java

```java
// Find all students in year 9, and cap the result collection at 10 items
RealmResults<Student> result = realm.where(Student.class)
        .equalTo("year", 9)
        .distinct("name")
        .findAll();

```

#### Kotlin

```kotlin
// Find all students in year 9, and cap the result collection at 10 items
val result: RealmResults<Student> = realm.where<Student>(Student::class.java)
    .equalTo("year", 9L)
    .distinct("name")
    .findAll()

```

You can only call `distinct()` on integer, long, short, and `String`
fields; other field types will throw an exception. As with sorting,
you can specify multiple fields to resolve ties.

### Chain Queries
You can apply additional filters to a results collection by calling the
`where()` method:

#### Java

```java
// Find all students in year 9 and resolve the query into a results collection
RealmResults<Student> result = realm.where(Student.class)
        .equalTo("year", 9)
        .findAll();

// filter the students results again by teacher name
RealmResults<Student> filteredResults = result.where().equalTo("teacher.name", "Ms. Langtree").findAll();

```

#### Kotlin

```kotlin
// Find all students in year 9 and resolve the query into a results collection
val result: RealmResults<Student> = realm.where(Student::class.java)
    .equalTo("year", 9L)
    .findAll()

// filter the students results again by teacher name
val filteredResults =
    result.where().equalTo("teacher.name", "Ms. Langtree").findAll()

```

The `where()` method returns a `RealmQuery` that you can resolve into
a `RealmResults` using a `find` method. Filtered results can only
return objects of the same type as the original results set, but are
otherwise able to use any filters.

## Query with Realm Query Language
> Version added: 10.4.0

You can also query realms using Realm Query Language, a string-based
query language to constrain searches when retrieving objects from a realm.

You can use `RealmQuery.rawPredicate()`.
For more information about syntax, usage and limitations,
refer to the Realm Query Language reference.

Realm Query Language can use either the class and property names defined
in your Realm Model classes or the internal names defined with `@RealmField`.
You can combine raw predicates with other raw predicates or type-safe
predicates created with `RealmQuery`:

#### Java

```java
// Build a RealmQuery based on the Student type
RealmQuery<Student> query = realm.where(Student.class);

// Simple query
RealmResults<Student> studentsNamedJane =
        query.rawPredicate("name = 'Jane'").findAll();

// Multiple predicates
RealmResults<Student> studentsNamedJaneOrJohn =
        query.rawPredicate("name = 'Jane' OR name = 'John'").findAll();

// Collection queries
RealmResults<Student> studentsWithTeachers =
        query.rawPredicate("teacher.@count > 0").findAll();
RealmResults<Student> studentsWithSeniorTeachers =
        query.rawPredicate("ALL teacher.numYearsTeaching > 5").findAll();

// Sub queries
RealmResults<Student> studentsWithMathTeachersNamedSteven =
        query.rawPredicate("SUBQUERY(teacher, $teacher, $teacher.subject = 'Mathematics' AND $teacher.name = 'Mr. Stevens').@count > 0").findAll();

// Sort, Distinct, Limit
RealmResults<Student> students =
        query.rawPredicate("teacher.@count > 0 SORT(year ASCENDING) DISTINCT(name) LIMIT(5)").findAll();

// Combine two raw predicates
RealmResults<Student> studentsNamedJaneOrHenry =
        query.rawPredicate("name = 'Jane'")
                .rawPredicate("name = 'Henry'").findAll();

// Combine raw predicate with type-safe predicate
RealmResults<Student> studentsNamedJaneOrHenryAgain =
        query.rawPredicate("name = 'Jane'")
                .equalTo("name", "Henry").findAll();

```

#### Kotlin

```kotlin
// Build a RealmQuery based on the Student type
val query = realm.where(Student::class.java)

// Simple query
val studentsNamedJane = query.rawPredicate("name = 'Jane'").findAll()

// Multiple predicates
val studentsNamedJaneOrJohn =
    query.rawPredicate("name = 'Jane' OR name = 'John'").findAll()

// Collection queries
val studentsWithTeachers =
    query.rawPredicate("teacher.@count > 0").findAll()
val studentsWithSeniorTeachers =
    query.rawPredicate("ALL teacher.numYearsTeaching > 5").findAll()

// Sub queries
val studentsWithMathTeachersNamedSteven =
    query.rawPredicate("SUBQUERY(teacher, \$teacher, \$teacher.subject = 'Mathematics' AND \$teacher.name = 'Mr. Stevens').@count > 0")
        .findAll()

// Sort, Distinct, Limit
val students =
    query.rawPredicate("teacher.@count > 0 SORT(year ASCENDING) DISTINCT(name) LIMIT(5)")
        .findAll()

// Combine two raw predicates
val studentsNamedJaneOrHenry = query.rawPredicate("name = 'Jane'")
    .rawPredicate("name = 'Henry'").findAll()

// Combine raw predicate with type-safe predicate
val studentsNamedJaneOrHenryAgain =
    query.rawPredicate("name = 'Jane'")
        .equalTo("name", "Henry").findAll()

```
