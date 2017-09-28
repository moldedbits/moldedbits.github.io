---
layout: post
title:  "Jumpstarting Android Projects"
date:   2017-09-28 05:00:00
author: anuj
categories: Technical Android
---

Do you remember the last time you setup an Android project? How long did it take you after creating a new android project from Android Studio to customize and scaffold it before you could write the first business logic? Did you remember to setup the

* dependencies correctly,
* networking,
* dependency injection,
* base classes,
* build types,
* version number auto increment and
* a million other small things?

And then to do it for every new project you start, that is hours and hours of wasted effort.

<img src="https://media.giphy.com/media/l46CbAuxFk2Cz0s2A/giphy.gif" />

Well, not any more.

In the spirit of DRY, we have created an automated process at moldedbits to jumpstart Android projects and get started quickly. And in the spirit of open source, we are putting this out in the public domain.

Enough talk, lets code. We will demonstrate setting up an application with the dependencies, networking and dependency injection setup and the login implemented with [Argus](https://github.com/moldedbits/argus-android), all within 5 minutes.

<img src="https://media.giphy.com/media/6kvVGhp7bp2WA/giphy.gif" />

#### Step 1 - Setting up the base project

*Time Required: 3 minutes*

We have a python script to help with the initial setup. You can clone this [repo](https://github.com/moldedbits/JumpstartScript) on your machine and navigate to the folder on a terminal.

```
$ git clone https://github.com/moldedbits/JumpstartScript
$ cd JumpstartScript
$ python android-jumpstart.py
```

This will clone the repository [android-jumpstart](https://github.com/moldedbits/android-jumpstart) locally and then ask you two to customize it.

```
...
Enter new app name: JumpDemo
...
Enter new app package: com.moldedbits.jump
...
```
This will create a project for you in the folder _JumpDemo_ that is all set to be imported into Android Studio.

__Voila, our base project all setup. Now fire up your Android Studio, import the project and start writing business logic.__

<img src="https://media.giphy.com/media/XreQmk7ETCak0/giphy.gif" />

Some of the features already implemented by this are,

* Dependency injection with Dagger 2
* Networking with Retrofit
* Gradle build types, signing keystore, dependencies, version code auto increment
* Mocking framework for unit tests

A detailed and updated list of the features is available on the [project page](https://github.com/moldedbits/android-jumpstart).

#### Step 2 - Adding login (Optional)

*Time Required: 2 minutes*

Most apps require a login flow with email, phone and social logins. To further automate this, we have created another library to make our lives easier, [Argus](https://github.com/moldedbits/argus-android). Including this is really simple. Follow the steps [here][argus-post] and you will have your login screen in less than two minutes.

And that is all. With this, we have setup an app with all the dependencies and scaffolding setup to get the developer started straight on the business logic of the app, and a basic login framework.

<img src="https://raw.githubusercontent.com/moldedbits/argus-android/master/images/ArgusLogin.png" alt="Demo Login" style="width: 300px; margin:auto;"/>

And all of it took less than 5 minutes!

<img src="https://media.giphy.com/media/5wWf7GW1AzV6pF3MaVW/giphy.gif" />

The jumpstart is, by design, very opinionated about how android apps should be written. You can pick the things you like and change / delete the things you do differently.

We will continue working on the jumpstart project to keep it updated, so you can expect to see some changes in the future. As always, if you find anything that can be improved, we welcome you to create a github issue or better to create a pull request.

Happy coding!

The moldedbits Team

[argus-post]: http://eng.moldedbits.com/technical/android/2017/06/26/introducing-argus.html
