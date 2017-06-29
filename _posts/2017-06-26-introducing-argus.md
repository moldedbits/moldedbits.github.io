---
layout: post
title:  "Introducing Argus- OnBoarding in Android Apps Simplified"
date:   2017-06-26 05:51:59
author: abhishek
categories: Technical Android
---

#### What is Argus

Argus is a customizable and extensible open source library that allows developers to quickly integrate user on-boarding flow for Android apps, including login, signup and forgot password screens.

With in-built support for Google and Facebook login / signup, Argus makes it trivial to get up and running when starting with a new app.

A few lines of code can get you a screen looking like,

![screenshot]({{ site.url }}/assets/images/argus_sample.png){:style="width: 250px; margin:auto;"}

#### The motivation behind Argus

At [moldedbits](http:///www.moldedbits.com), we do a lot of client projects. Be it an enterprise app, a social networking app or an app for a small business, almost all apps need a user on-boarding flow, including login, registration forgot password. Having implemented almost similar flows across apps, we decided to abstract out the complexity into a separate library that could be reused.

#### Integrating in your apps

We built `Argus` android to simplify our lives and to speed up app development. Argus is a library that makes user on-boarding hassle free.

Argus works with providers for features like Login/Signup/ForgotPassword. These `Providers` take care of all flow and functionality of the feature they are responsible for. `Argus` comes with built-in providers for Google, Facebook and Email login/signup. These are sufficient for most of the use cases, however, if they aren't enough for you, you can create your own custom provider.

Argus is fully customizable and it adapts look and feel of your application.

A basic Argus configuration looks like

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

#### What is next

Argus is in a nascent stage right now and you are very much invited to contribute in its development. Feel free to open pull requests, report issues or suggest enhancements.


Happy coding !

The moldedbits Team
