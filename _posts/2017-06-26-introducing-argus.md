---
layout: post
title:  "Introducing Argus- OnBoarding in Android Apps Simplified"
date:   2017-06-26 05:51:59
author: abhishek
categories: Tech, Android
---
At [moldedbits](http:///www.moldedbits.com), we do a lot of client projects. Almost in every project user on-boarding(Login/Registration/Forgot Password etc.) is required at some point, be it a enterprise app, or a social networking app or some simple small business app. Every time we got a new project we had to implement Login Screen, Registration screen other related screens and then related functionalities like validations and other stuff. This was not DRY, we were doing it wrong and realized that we need [Argus](https://github.com/moldedbits/argus-android).

We built `Argus` android to simplify our lives and to speed up app development. Argus is a library that makes user on-boarding hassle free.

Argus works with providers for features like Login/Signup/ForgotPassword. These `Providers` take care of all flow and functionality of the feature they are responsible for. `Argus` comes with builtin providers like Google, Facebook signup and Email login/signup. These are sufficient for most of the use cases, however, if they aren't enough for you, you can create your own custom provider.

Argus is fully customizable and it adapts look and feel of your application. Its open source and free to use. Argus is in nascent stage right now and you are very much invited to contribute in its development. Feel free to open pull requests, report issues or suggest enhancements.

A basic Argus configuration will look like this in your app

{% highlight java %}
ArrayList<BaseProvider> loginProviders = new ArrayList<>();
loginProviders.add(new EmailLoginProvider());

// argus is customizable, supplying my custom logo to argus
ArgusTheme argusTheme = new ArgusTheme.Builder()
        .logo(R.mipmap.ic_launcher)
        .build();

new Argus.Builder()
        // using default storage, dont want to do anything fancy here
        .argusStorage(new DefaultArgusStorage(getApplicationContext()))
        // this is the screen which should open after login is successfully completed
        .nextScreenProvider(new SimpleNextScreenProvider(HomeActivity.class))
        // we only want to implement login screen
        .loginProviders(loginProviders)
        // set custom theme
        .theme(argusTheme)
        .build();
{% endhighlight %}

Getting started with Argus is easy, you can follow [this guide](https://github.com/moldedbits/argus-android/wiki/Getting-Started) to integrate it in your app now.

Enjoy!
Happy Coding !!
