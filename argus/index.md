---
layout: project
title:  "Argus"
github: "https://github.com/moldedbits/argus-android"
---

#### What is Argus

Argus is a customizable and extensible open source library that allows developers to quickly integrate user on-boarding flow for Android apps, including login, signup and forgot password screens.

With in-built support for Google and Facebook login / signup, Argus makes it trivial to get up and running when starting with a new app.

A few lines of code can get you a screen looking like,

![screenshot]({{ site.url }}/assets/images/argus_sample.png){:style="width: 250px; margin:auto;"}

#### Getting started

Here is how you can quickly create a simple email-password based login screen with Argus.
First step is to start configuring Argus. Best place to do it is in your Application class.

Configuration Code looks like this

```java
ArrayList<BaseProvider> loginProviders = new ArrayList<>();
loginProviders.add(new SimpleEmailLoginProvider());

// argus is customizable, supplying my custom logo to argus
ArgusTheme argusTheme = new ArgusTheme.Builder()
                .logo(R.mipmap.ic_launcher)
                .build();

new Argus.Builder()
         // using default storage, don't want to do anything fancy here
         .argusStorage(new DefaultArgusStorage(getApplicationContext()))
         // this is the screen which should open after login is successfully completed
         .nextScreenProvider(new SimpleNextScreenProvider(HomeActivity.class))
         // we only want to implement login screen
         .loginProviders(loginProviders)
         // set custom theme
         .theme(argusTheme)
         .build();
```

`Argus` works with providers for features like Login/Signup/ForgotPassword. These `Providers` take care of all flow and functionality of the feature they are responsible for. Argus comes with builtin providers that are sufficient for most of the use cases, however, if those providers aren't enough for you, you can create your own custom provider. We will do a separate guide on how to create your own custom provider later.

For this example we will use `EmailLoginProvider` which comes by default with `Argus`. `EmailLoginProvider` is a abstract class which does all the work except final API call which is left for client code to handle. To do so you have to extend `EmailLoginProvider`. A simple implementation will look like this

```java
public class SimpleEmailLoginProvider extends EmailLoginProvider {

    /**
     * this function is called after user input was validated
     * this is where actual API call to your server will go
     */
    @Override
    protected void doServerLogin(String username, String password) {
        // need to set state signed-in in Argus here
        if(username.equals("valid@user.com") && password.equals("password")) {
            // do a real API call here and in on success do following
            if (resultListener != null) {
                resultListener.onSuccess(ArgusState.SIGNED_IN);
            }
        } else {
            if (context != null) {
                Toast.makeText(context, context.getString(R.string.invalid_email),
                        Toast.LENGTH_LONG).show();
            }
        }
    }
}
```

After that register `ArgusActivity` in your application's manifest file. Is it not mandatory to set `ArgusActivity` as launcher activity. You can also launch it via intent whenever your app requires login.
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

#### Download

Argus is hosted on jcenter. You can get it via gradle

`compile 'com.moldedbits.argus:argus:0.1.0'`

or via maven

```
<dependency>
    <groupId>com.moldedbits.argus</groupId>
    <artifactId>argus</artifactId>
    <version>0.1.0</version>
    <type>pom</type>
</dependency>
```

#### What is next

Argus is in a nascent stage right now and you are very much invited to contribute in its development. Feel free to open pull requests, report issues or suggest enhancements.
