---
layout: post
title:  "Jumpstarting Android Projects"
date:   2017-08-30 08:00:00
author: anuj
categories: Technical Android
---

Starting a new Android project means writing a lot of boilerplate code, setting up your dependencies, setting up networking, dependency injection and a myriad of other small things before you can even get started on the actual app. In the spirit of DRY, we have created an automated process at moldedbits to jumpstart Android projects and get started quickly. And in the spirit of open source, we are putting this out in the public domain.

In this post, we will demo setting up an application with the dependencies, networking and dependency injection setup and the login implemented with [Argus](https://github.com/moldedbits/argus-android).

#### Setting up

We have a python script to help with the initial setup. You can clone this [repo](https://github.com/moldedbits/JumpstartScript) on your machine and navigate to the folder on a terminal.

Next, to setup the project, all you need to do is run

```
$ python android-jumpstart.py
```
This will clone the repository [android-jumpstart](https://github.com/moldedbits/android-jumpstart) locally and then ask you a few questions to customize it.

```
...
Enter new app name: JumpDemo
...
Enter new app package: com.moldedbits.jump
...
```
This will create a project for you in the folder _JumpDemo_ that is all set to be imported into Android Studio.

```
$ cd JumpDemo
$ ls
README.md	  	buildSrc		gradle.properties	jumpstart.keystore
app			      config			gradlew			      settings.gradle
build.gradle	gradle			gradlew.bat		    version.properties
```
With this we can import the project into Android Studio

<img src="/assets/images/android-studio-import.png" alt="Studio Import" style="width: 350px; margin:auto;"/>

This is our base project all setup. Some of the features already implemented by this are,

* Dependency injection with Dagger 2
* Networking with Retrofit
* Gradle build types, signing keystore, dependencies, version code auto increment
* Mocking framework for unit tests

A detailed and updated list of the features is available on the [project page](https://github.com/moldedbits/android-jumpstart).

#### Adding login

Most apps require a login flow with email, phone and social logins. To further automate this, we have created another library to make our lives easier, [Argus](https://github.com/moldedbits/argus-android). Including this is really simple. The first thing we have to do is to include Argus as a dependency in our app level build.gradle.

```
compile 'com.moldedbits.argus:argus:0.2.0'
```
Next, we modify the class BaseApplication to configure Argus.

```java
// initialize Argus
ArrayList<BaseProvider> loginProviders = new ArrayList<>();

loginProviders.add(new EmailLoginProvider() {
    @Override
    protected void doServerLogin(String username, String password) {

    }
});

ArgusTheme.Builder themeBuilder = new ArgusTheme.Builder();
themeBuilder.logo(R.mipmap.ic_launcher);

new Argus.Builder()
        .argusStorage(new DefaultArgusStorage(getApplicationContext()))
        .nextScreenProvider(new SimpleNextScreenProvider(ExampleAppActivity.class))
        .loginProviders(loginProviders)
        .theme(themeBuilder.build())
        .build();
```
And finally, we add the Argus activity to AndroidManifest.xml

```xml
<activity
    android:name="com.moldedbits.argus.ArgusActivity"
    android:label="@string/app_name"
    android:screenOrientation="portrait"
    android:theme="@style/AppTheme">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```
And that is all. With this, we have setup an app with all the dependencies and scaffolding setup to get the developer started straight on the business logic of the app, and a basic login framework.

<img src="/assets/images/jump-demo-login.png" alt="Demo Login" style="width: 300px; margin:auto;"/>

Happy coding!

The moldedbits Team
