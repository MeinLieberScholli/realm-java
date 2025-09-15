# Bundle a Realm File - Java SDK
Realm supports **bundling** realm files. When you bundle
a realm file, you include a database and all of its data in your
application download.

This allows users to start applications for the first time with a set of
initial data.

## Overview
To create and bundle a realm file with your application:

1. Create a realm file that
contains the data you'd like to bundle.
2. Bundle the realm file in the
/<app name>/src/main/assets folder of your production
application.
3. In your production application,
open the realm from the bundled asset file.

## Create a Realm File for Bundling
1. Build a temporary realm app that shares the data model of your
application.
2. Open a realm and add the data you wish to bundle.
3. Use the `writeCopyTo()`
method to copy the realm to a new file.

`writeCopyTo()` automatically compacts your realm to the smallest
possible size before copying.

## Bundle a Realm File in Your Production Application
Now that you have a copy of the realm that contains the initial data,
bundle it with your production application.

1. Search your application logs to find the location of the realm file
copy you just created.
2. Using the "Device File Explorer" widget in the bottom right of your
Android Studio window, navigate to the file.
3. Right click on the file and select "Save As". Navigate to the
/<app name>/src/main/assets folder of your production application.
Save a copy of the realm file there.

> Tip:
> If your application does not already contain an asset folder, you can
create one by right clicking on your top-level application
folder (<app name>) in Android Studio and selecting
New > Folder > Assets Folder in the menu.
>

## Open a Realm from a Bundled Realm File
Now that you have a copy of the realm included with your production
application, you need to add code to use it. Use the `assetFile()`
method when configuring your realm to open the realm
from the bundled file.
