---
layout: post
title:  "Automated Testing for Android"
date:   2016-11-10 23:03:00
author: anuj
categories: Technical Android
comments: true
---

Target Audience: Android developers

Android has historically been a tricky beast for writing tests. Things have changed, however, and testing Android apps has never been easier.

[developers.android.com](https://developer.android.com/training/testing/start/index.html) has some great documentation for getting started. With this post, we want to demonstrate testing in real world scenarios.

In most of our applications, we use [Retrofit](http://square.github.io/retrofit/) for network calls and the [MVP pattern](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter). From our experience, MVP helps us write code which is better, and inherently easier to test. For this demo, we have written a simple app that shows a login form and makes an API call.

Typically, there's four categories of tests,

- Local Unit tests
- Instrumented Unit tests
- User interface tests
- Integration tests

#### Local Unit Testing

Use this for parts of your code that have no dependency on the Android framework, or depend on something small that can be mocked.

Methods as below are ideal candidates for unit testing

```Java
// Checks for valid email address
boolean isValidUsername(String username) {
    Pattern pattern = Pattern.compile("\\A[^@]+@([^@\\.]+\\.)+[^@\\.]+\\z");
    Matcher matcher = pattern.matcher(username);
    return matcher.matches();
}
```

To setup, add the following to your gradle file

```xml
dependencies {
    // Required -- JUnit 4 framework
    testCompile 'junit:junit:4.12'
    // Optional -- Mockito framework
    testCompile 'org.mockito:mockito-core:1.10.19'
}
```

and then create a test file in `src/test/your/package`

```Java
package com.moldedbits.android.login;

// Imports...

@RunWith(JUnit4.class)
public class LoginPresenterUnitTests {

    private LoginPresenter mPresenter;

    @Before
    public void setup() {
        mPresenter = new LoginPresenter(null, null);
    }

    @Test
    public void validationTest() {
        assertThat(mPresenter.isValidUsername(""), is(false));
        assertThat(mPresenter.isValidUsername("abc"), is(false));
        assertThat(mPresenter.isValidUsername("a@b.com"), is(true));
    }
}
```
To run an individual test, simply right click on the test and select `run validationTest()`, or you can run all the tests in the class in a similar way.

That was simple, but what if we wanted to test how our business logic interacts with the UI. This is where the MVP pattern really shines. Since the only Android dependencies are in View, we can mock it out and unit test the Presenter.

Here's how it looks

```Java
package com.moldedbits.android.login;

// Imports...

@RunWith(MockitoJUnitRunner.class)
public class LoginPresenterTest {

    @Mock
    LoginView mLoginView;

    LoginPresenter mPresenter;

    @Before
    public void setup() {
        mPresenter = new LoginPresenter(mLoginView, null);
    }

    @Test
    public void loginInvalidUsername() {
        mPresenter.login("abc", "abc");
        verify(mLoginView, times(1)).setUsernameError("Invalid input");
    }

    @Test
    public void loginEmptyUsername() {
        mPresenter.login("", "abc");
        verify(mLoginView, times(1)).setUsernameError("Required");
    }

    @Test
    public void loginEmptyPassword() {
        mPresenter.login("a@b.c", "");
        verify(mLoginView, times(1)).setPasswordError("Required");
    }
}
```
JUnit4 and MockitoJUnitRunner run the tests on your local JVM, and are very fast. Hence we prefer them wherever possible.

We can even take this further and mock retrofit

```Java
package com.moldedbits.android.login;

// Imports

@RunWith(MockitoJUnitRunner.class)
public class LoginPresenterMockApiTest {

    @Mock
    LoginView mLoginView;

    @Mock
    APIService mAPIService;

    @Mock
    Call<LoginResponse> mCall;

    @Captor
    ArgumentCaptor<Callback<LoginResponse>> mCallback;

    private LoginPresenter mPresenter;

    @Before
    public void setup() {
        MockitoAnnotations.initMocks(this);
        mPresenter = new LoginPresenter(mLoginView, mAPIService);

        when(mAPIService.login(any(LoginRequest.class)))
                .thenReturn(mCall);
    }

    @Test
    public void loginSuccess() {
        mPresenter.login("a@b.c", "abc");

        verify(mCall).enqueue(mCallback.capture());
        mCallback.getValue().onResponse(mCall, getMockSuccessResponse());

        verify(mLoginView, times(1)).onLoginSuccess(any(User.class));
    }

    @Test
    public void loginFailure() {
        mPresenter.login("a@b.c", "abc");

        verify(mCall).enqueue(mCallback.capture());
        mCallback.getValue().onResponse(mCall, getMockFailureResponse());

        verify(mLoginView, times(1)).onLoginFailure("Invalid credentials");
    }

    private Response<LoginResponse> getMockSuccessResponse() {
        LoginResponse successResponse = new LoginResponse();
        successResponse.setUser(getMockUser());
        return Response.success(successResponse);
    }

    private Response<LoginResponse> getMockFailureResponse() {
        String errorMessage = "{" +
                "  \"status\": \"failure\"," +
                "  \"error\": \"Invalid credentials\"" +
                "}";
        ResponseBody responseBody = ResponseBody.create(MediaType.parse("string/json"),
                errorMessage.getBytes());
        return Response.error(401, responseBody);
    }

    private User getMockUser() {
        return new User(1, "Mock User");
    }
}
```
Important thing to note here is the use of [ArgumentCaptor](http://site.mockito.org/mockito/docs/current/org/mockito/ArgumentCaptor.html). You can capture the arguments passed and call any method, pretty cool stuff!

#### Instrumented Unit tests
There are, however, times when you need the Android jars in your tests. Fortunately, it is pretty darn simple as well. All you need to do is update your test runner, and place your test in the folder `src/androidTest/your/package`

```java
...
@RunWith(AndroidJUnit4.class)
public class LoginIntegrationTest {
  ...
}
```
You will also need to change the following in your build.gradle file

```xml
defaultConfig {
  ...
  testInstrumentationRunner 'android.support.test.runner.AndroidJUnitRunner'
}

dependencies {
  ...
  androidTestCompile 'com.android.support:support-annotations:25.0.0'
  androidTestCompile 'com.android.support.test:runner:0.5'
  androidTestCompile 'com.android.support.test:rules:0.5'
  // Optional -- UI testing with Espresso
  androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
}
```

#### Integration tests

Espresso makes it very easy to create an run integration tests for your code. Its just like writing a test script, you identify the view, perform an action, and optionally verify the result. We can create complex interactions, across multiple activities and fragments. Here is a simple example,

```Java
package com.moldedbits.android;

// Imports...

@RunWith(AndroidJUnit4.class)
@SmallTest
public class LoginIntegrationTest {

    private String mUsername, mPassword;

    @Rule
    public ActivityTestRule<LoginActivity> mActivityRule = new ActivityTestRule<>(LoginActivity.class);

    @Test
    public void loginSuccess() {
        initValidCredentials();

        onView(withId(R.id.input_username))
                .perform(typeText(mUsername), closeSoftKeyboard());

        onView(withId(R.id.input_password))
                .perform(typeText(mPassword), closeSoftKeyboard());

        onView(withId(R.id.button_submit))
                .perform(click());


        IdlingResource resource = OkHttp3IdlingResource.create("OkHttp",
                APIProvider.getClient());
        Espresso.registerIdlingResources(resource);

        onView(allOf(withId(android.support.design.R.id.snackbar_text),
                withText("Logged in as test@moldedbits.com")))
                .check(matches(withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE)));

        Espresso.unregisterIdlingResources(resource);
    }

    @Test
    public void loginFailure() {
        initInvalidCredentials();

        onView(withId(R.id.input_username))
                .perform(typeText(mUsername), closeSoftKeyboard());

        onView(withId(R.id.input_password))
                .perform(typeText(mPassword), closeSoftKeyboard());

        onView(withId(R.id.button_submit))
                .perform(click());


        IdlingResource resource = OkHttp3IdlingResource.create("OkHttp", APIProvider.getClient());
        Espresso.registerIdlingResources(resource);

        onView(allOf(withId(android.support.design.R.id.snackbar_text),
                withText("Invalid credentials")))
                .check(matches(withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE)));

        Espresso.unregisterIdlingResources(resource);
    }

    private void initValidCredentials() {
        mUsername = "test@moldedbits.com";
        mPassword = "foobarfoo";
    }

    private void initInvalidCredentials() {
        mUsername = "test@moldedbits.com";
        mPassword = "wrong";
    }
}
```

You will need to add the following dependency for the above example,

```xml
dependencies {
  ...
  androidTestCompile 'com.jakewharton.espresso:okhttp3-idling-resource:1.0.0'
}
```

#### Conclusion

With JUnit4 and Espresso, writing and running automated tests for Android is now simple and efficient. In a future post, we will demonstrate how we integrate testing in our continuous integration system.

For this post, you can find the sample code on [GitHub](https://github.com/moldedbits/android-test-demo).


Happy coding!

The moldedbits Team

{% if page.comments %}
{% include disqus.html %}
{% endif %}
