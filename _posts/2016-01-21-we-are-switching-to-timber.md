---
layout: post
title:  "We are switching to Timber for Android Logging"
date:   2016-01-21 01:51:59
author: abhishek
categories: Tech, Android
---
At [moldedbits](http:///www.moldedbits.com), we have decided to switch to [Timber](https://github.com/JakeWharton/timber) library over `android.util.Log`. `Timber` has following benefits over standard android logging

### No Tags
Timber automagically assigns TAG to a log statement. By default, it is filename but a custom tag can be provided if needed
{% highlight java %}
// default TAG
Timber.d("This is a debug log”);
// Custom TAG
Timber.tag("CustomTag").d("Debug log with custom tag”);
{% endhighlight %}

### Custom Behaviours
Developer can add custom logging engines(called Trees) for custom logging. Timber ships with a default logging behaviour class DebugTree. These Trees can be installed conditionally
like

{% highlight java %}
// This makes sure that logs are only generated in debug mode
// even when proguard is not in action
if(BuildConfig.DEBUG) {
    // DebugTree has all usual logging functionality
    Timber.plant(new Timber.DebugTree());
}
{% endhighlight %}

Developer can also write custom behaviours e.g. printing line no along with log statement.

{% highlight java %}
/**
 * Sample tree which prints line no along with tag
 */
public class LineNoDebugTree extends Timber.DebugTree {
    @Override
    protected String createStackElementTag(StackTraceElement element) {
        return super.createStackElementTag(element) + ":" + element.getLineNumber();
    }
}
{% endhighlight %}

### Logging to multiple sinks
Single log statement can print log on logcat and report to some other source like Crashlytics. All you have to do is install two Trees on Timber instance in your Application class.
like this

{% highlight java %}
// This makes sure that logs are only generated in debug mode
// even when proguard is not in action
if(BuildConfig.DEBUG) {
    // DebugTree has all usual logging functionality
    Timber.plant(new Timber.DebugTree());
}

// planting another tree in parallel to existing Debug Tree
// this can send crash/exception reports to Crashlytics alongside existing logging/crash
// reporting mechanism. This can also be done conditionally for release builds only.
Timber.plant(new CrashlyticsTree());
{% endhighlight %}

`CrashlyticsTree` class looks like this

{% highlight java %}
public class CrashlyticsTree extends Timber.Tree {
    private static final String CRASHLYTICS_KEY_PRIORITY = "priority";
    private static final String CRASHLYTICS_KEY_TAG = "tag";
    private static final String CRASHLYTICS_KEY_MESSAGE = "message";

    @Override
    protected void log(int priority, @Nullable String tag, @Nullable String message, @Nullable Throwable t) {
        if (priority == Log.VERBOSE || priority == Log.DEBUG || priority == Log.INFO) {
            return;
        }

        Crashlytics.setInt(CRASHLYTICS_KEY_PRIORITY, priority);
        Crashlytics.setString(CRASHLYTICS_KEY_TAG, tag);
        Crashlytics.setString(CRASHLYTICS_KEY_MESSAGE, message);

        if (t == null) {
            Crashlytics.logException(new Exception(message));
        } else {
            Crashlytics.logException(t);
        }
    }
}
{% endhighlight %}

Clearly its very easy to use and makes debugging more efficient.
Happy Coding !!
