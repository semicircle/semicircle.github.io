---
layout: post
title: "Packaging Sencha Touch app with PhoneGap for Android"
date: 2013-04-10 10:49
comments: true
categories: html5 sencha
---

I found many posts about this topic, but most of them are too old. If you followed those instructions, possibly, you have already been trapped by some strange problems.

Here's my versions:

- Sencha Touch 2.2.0 beta
- PhoneGap 2.5.0
- Sencha Cmd 3.1.0.256
<!-- more -->
### My file structure is like:

- AppBase //Sencha app root.
	- app //app folder
	- touch //sencha sdk folder
	- app.js 
	- app.json
	- index.html
	- cordova.js //may be empty file? 
- Builds
	- android
		- project //the PhoneGap-Android proejct root
		- phonegapjs //the folder to save cordova.js

Hints:

- AppBase and Builds/android/project are mostly created by SenchaCmd and PhoneGap, if don't know about this, please google them elsewhere.
- because the cordova.js is different from platforms, and we saved the android's in 'phonegapjs'.
- Not all files are listed here, only the related ones.
- AppBase/cordova.js is the android one, you may wonder will this break the HttpServer/Chrome debugging environment? I can only say the android one didn't.  And if it does, it seems leaving it an empty file will be ok.

### Add cordova.js in app.json

``` json app.json
    "js": [
        {
            "path": "cordova.js"
        },
        {
            "path": "touch/sencha-touch-debug.js",
            "x-bootstrap": true
        },
        {
            "path": "app.js",
            "bundle": true,  /* Indicates that all class dependencies are concatenated into this file when build */
            "update": "delta"
        }
    ],
```

### Build the sencha application


```
$ cp ./phonegapjs/cordova-2.5.0.js ../../AppBase/cordova.js
$ sencha app build testing
$ sencha app build package
```
There as testing/native/production/package While testing, you may like to build it using 'testing', But, ALWAYS use 'package' instead of 'production' when generate a production version. because the production version highly depencds on the browsers' html5 app cache feature, which is not well supported on android 2.x.  And, will cause bugs like:

- App can only launch successful once, the second time it's will freeze at sencha's blue screen. Reinstallation didn't help, only remove it and install again will recover(but also can start once).
- App did not change when reinstall, only remove it and install again will help.

The sencha build process will also generate the compact version of the cordova.js, that's why we copied it at this step.

### Copy the everything into PhoneGap's project and build

```
$ cd android
$ rm -R ./project/assets/www/*
$ cp -R ../../AppBase/build/AppName/package/* ./project/assets/www/
$ ant -f ./project/build.xml debug
```

### Write your own scripts.

Here's mine:

```
cp ./phonegapjs/cordova-2.5.0.js ../../AppBase/cordova.js
cd ../../AppBase/
sencha app build testing
#sencha app build package
cd ../Builds/android/
rm -R ./project/assets/www/*
cp -R ../../AppBase/build/AppName/package/* ./project/assets/www/
cp -R ../../AppBase/locales ./project/assets/www/

ant -f ./project/build.xml debug
adb install -rs ./project/bin/AppName-debug.apk
adb shell am start -n com.example.namespace/.AppName
cp ./project/bin/AppName-debug.apk ~/Google\ Drive/
```

### More 

###### UX in Sencha Touch app?

I wrote like this, and SenchaCmd works fine while building:

``` js app.js
Ext.Loader.setPath({
    'Ext': 'touch/src',
    'AppName': 'app',
    'Ux': 'Ux'
});
```

###### A Bug in this Sencha Touch 2.2.0beta:

in touch/src/env/, if you get problem when sencha app build.

``` js touch/src/env/Feature.js
		Ext.onDocumentReady(function() {
            if (Ext.feature) {  // <-------this line.
                Ext.feature.registerTest({
                    ....
                });
            }
            return this;
        });
```

### Thanks to

[1. andidog.de](http://andidog.de/blog/2012/06/packaging-a-sencha-touch-2-application-with-phonegap-for-android/)

[2. another from andidog.de](http://andidog.de/blog/2012/07/dont-use-sencha-touch-production-mode-build-for-mobile/)

[3. cclerville](http://cclerville.blogspot.com/2013/01/sencha-touch-21-phonegap-220-android_3.html)

