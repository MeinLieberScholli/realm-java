# Install Realm - Java SDK
> Note:
> The Java SDK is in best-effort maintenance mode and **no longer receives
new development or non-critical bug fixes. To develop your app with new
features, use the Kotlin SDK. You can use the Java SDK
with the Kotlin SDK in the same project.
>
> Learn more about how to Migrate from the Java SDK to the Kotlin SDK.
>

## Overview
This page details how to install Realm using the Java SDK in your project
and get started.

You can use multiple SDKs in your project. Because the Java SDK is no longer
receiving new development, this is useful if you want to
use new features in your app.

## Prerequisites
- [Android Studio](https://developer.android.com/studio/index.html) version 1.5.1 or higher.
- Java Development Kit (JDK) 11 or higher.
- An emulated or hardware Android device for testing.
- Android API Level 16 or higher (Android 4.1 and above).

## Installation
Realm only supports the Gradle build system. Follow these steps
to add the Realm Java SDK to your project.

> Note:
> Because Realm provides a ProGuard configuration as part
of the Realm library, you do not need to add any
Realm-specific rules to your ProGuard configuration.
>

### Project Gradle Configuration
To add local realm to your application, make
the following changes to your project-level Gradle build
file, typically found at <project>/build.gradle:

#### Gradle Plugin

> Tip:
> The Java SDK does not yet support the Gradle Plugin syntax. Fortunately,
you can still add the SDK to projects that use this syntax.
>

- Add a `buildscript` block that contains a `repositories` block and a `dependencies` block.
- Add the `mavenCentral()` repository to the `buildscript.repositories` block.
- Add the `io.realm:realm-gradle-plugin` dependency to the `buildscript.dependencies` block.

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "io.realm:realm-gradle-plugin:10.18.0"
    }
}

plugins {
    id 'com.android.application' version '7.1.2' apply false
    id 'com.android.library' version '7.1.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.10' apply false
    id "org.jetbrains.kotlin.kapt" version "1.6.20" apply false
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

#### Gradle Legacy

- Add the `io.realm:realm-gradle-plugin` dependency to the `buildscript.dependencies` block.
- Add the `mavenCentral()` repository to the `allprojects.repositories` block.

```groovy
buildscript {
    repositories {
        google()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:3.5.1"
        classpath "io.realm:realm-gradle-plugin:10.18.0"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

### Application Module Gradle Configuration
Then, make the following changes to your application-level
Gradle build file, typically found at <project>/app/build.gradle:

#### Gradle Plugin

- Apply the `kotlin-kapt` plugin if your application uses Kotlin
- Beneath the `plugins` block, apply the `realm-android` plugin.

```groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'org.jetbrains.kotlin.kapt'
}

apply plugin: "realm-android"

android {
    compileSdk 31
    defaultConfig {
        applicationId "com.mongodb.example-realm-application"
        minSdk 28
        targetSdk 31
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }
    kotlinOptions {
        jvmTarget = '11'
    }
}

dependencies {
    implementation 'io.realm:realm-gradle-plugin:10.10.1'
    implementation 'androidx.core:core-ktx:1.7.0'
    implementation 'androidx.appcompat:appcompat:1.4.1'
    implementation 'com.google.android.material:material:1.5.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.3'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```

#### Gradle Legacy

- Apply the `kotlin-kapt` plugin if your application uses Kotlin
- Apply the `realm-android` plugin

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-kapt'
apply plugin: 'realm-android'

android {
    compileSdkVersion 31
    buildToolsVersion "29.0.2"
    defaultConfig {
        applicationId "com.mongodb.example-realm-application"
        minSdkVersion 28
        targetSdkVersion 31
    }
    compileOptions {
        sourceCompatibility 1.11
        targetCompatibility 1.11
    }
    kotlinOptions {
        jvmTarget = '11'
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation "androidx.appcompat:appcompat:1.1.0"
    implementation "androidx.core:core-ktx:1.2.0"
}
```

After updating the `build.gradle` files, resolve the dependencies by
clicking File > Sync Project with Gradle Files.

## Supported Platforms
Realm's Java SDK enables you to build apps for the
following platforms:

- Android
- Wear OS
- Android Automotive OS
- Android TV
- Android Things
