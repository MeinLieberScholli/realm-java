# Log Realm Events - Java SDK
The SDK logs events to the Android system log automatically. You can
view these events using [Logcat](https://developer.android.com//studio/debug/am-logcat).

## Set the Client Log Level
Realm uses the log levels defined by [Log4J](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/Level.html).
To configure the log level for Realm logs in your application, pass a
`LogLevel` to
`RealmLog.setLevel()`:

#### Java

```java
RealmLog.setLevel(LogLevel.ALL);

```

#### Kotlin

```kotlin
RealmLog.setLevel(LogLevel.ALL)

```

> Tip:
> To diagnose and troubleshoot errors while developing your application, set the
log level to `debug` or `trace`. For production deployments, decrease the
log level for improved performance.
>
