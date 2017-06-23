---
layout: post
title:  "Getting started with Reactive on Android"
date:   2017-06-22 11:08:00
author: anuj
categories: Technical RxJava
---

At moldedbits, we love reactive programming. The power of reactive lies in the operators, so we have created a [github repo][rx_operators] with examples for all the different operators. We are starting a series of blog posts to accompany the samples. This is first in that series.

The first section we will cover is operators for [creating Observables][creating-obserables].

##### Basic operators

Code is [here][simple] and [here][empty].

Some simple operators to create an observable are

```Java
// SimpleCreationJava.java
Observable.just(1, 2); // emits the given parameters.
Observable.range(1, 5); // emits a range of sequential integers
Observable.just(1, 2).repeat(2); // repeats the sequence of items
```

Sometimes we need edge case observables for writing tests. These can be created as

```Java
// EmptyJava.java
Observable.empty() // emits nothing and then completes
Observable.never() // emits nothing at all
Observable.error(new Exception("Sample exception")) // emits nothing and signals an error
```

##### Observable.create

[code][create-sample]

_Note_: This is for advanced use only, and should be used sparingly.

Things get more interesting when we get to [Observable.create][create]. The official documentation says that `create` "Provides an API (via a cold Observable) that bridges the reactive world with the callback-style world." and its type signature is

`public static <T> Observable<T> create(ObservableOnSubscribe<T> source)`

We took the official sample code, and completed it. The following sample creates an Observable that emits the click events on a view as a stream.

We will need an [AutoClosable][auto-closable], which can clean up once the observable completes. For this we created a ClickProvider, which when subscribed to starts listening for click events on a view, and when closed unsets the listener.

```Java
static class ClickProvider implements AutoCloseable {
    private View view;
    private View.OnClickListener onClickListener;
    ClickProvider(View view) {
        this.view = view;
    }
    ClickProvider listen(Callback callback) {
        onClickListener = callback::onEvent;
        view.setOnClickListener(onClickListener);
        return this;
    }
    @Override
    public void close() throws Exception {
        view.setOnClickListener(null);
    }
}
```

An interface to receive those events,

```Java
interface Callback {
    void onEvent(View view);
    void onFailure(Exception e);
}
```

And finally we can create the observable as,

```Java
Observable<View> observable = Observable.create(emitter -> {
    Callback callback = new Callback() {
        @Override
        public void onEvent(View view) {
            emitter.onNext(view);
        }
        @Override
        public void onFailure(Exception e) {
            emitter.onError(e);
        }
    };
    AutoCloseable c = clickProvider.listen(callback);
    emitter.setCancellable(c::close);
});
```

##### Observable.generate

Another interesting operator is [generate][generate-javadoc]. Its many signatures can get confusing, so we created [samples][generate-sample].

Generate takes as parameters an `initialState` as the starting point, an `emitter` that creates new events, and a `disposableState` that handles the stream completion.

A simple example is

```Java
// GenerateJava.java
// Returns initial state as 3
Callable<Integer> initialState = () -> 3;

// Takes an integer and emits a polygon with that many sides
BiFunction<Integer, Emitter<Polygon>, Integer> emitter =
    (integer, polygonEmitter) -> {
        if (integer < 6) {
            polygonEmitter.onNext(new Polygon(integer++));
        } else {
            polygonEmitter.onComplete();
        }
        return integer;
    };

// Takes the final state and does any required cleanup
Consumer<Integer> disposableState = integer -> ... );

Observable.generate(initialState, emitter, disposableState)
        .subscribeOn(Schedulers.io())
        .subscribe(...);
```

##### Observable.defer

_See also:_ [Deferring Observable code until subscription in RxJava][dan-defer] by Dan Lew

Defer makes sure that your observable code does not run until it is subscribed, and creates a fresh observable for each subscription.

An interesting use case is getting the latest value of an object's field.

```Kotlin
// Polygon.kt (Kotlin)
class Polygon(var sides: Int) {

    fun createObservable(): Observable<Int> {
        return Observable.just(sides)
    }

    fun deferObservable(): Observable<Int> {
        return Observable.defer { Observable.just(sides) }
    }
}

fun test() {
    val polygon: Polygon = Polygon(3)
    val createObservable = polygon.createObservable()
    val deferObservable = polygon.deferObservable()
    polygon.sides = 4

    createObservable.subscribe { Log.d(TAG, "$it") } // Prints 3
    deferObservable.subscribe { Log.d(TAG, " $it") } // Prints 4
}
```

##### Observable.from

Some of the most useful operators to create Observables are the ones that convert from Iterable, Callable and Future to Observables. All the samples are [here][from-samples]

RxKotlin adds convenience methods to convert from kotlin.collections.MutableList and kotlin.collections.List to Observables.

```java
// FromJava.java
String[] arr = new String[] {"this", "is", "an", "array"};
Observable.fromArray(arr).subscribe(...);

Publisher<String> publisher = s -> {
    for (int i=0; i<4; i++) {
        s.onNext("String " + i);
    }
    s.onComplete();
};
Observable.fromPublisher(publisher).subscribe(...);
```

```Kotlin
// FromKotlin.kt
listOf("this", "is", "a", "string").toObservable()
                .subscribe { ... }
```

##### Others

There are a few other operators to create observables. They are self explanatory and the sample code can be found in the rx_operators repo.

##### Conclusion

In this post we covered the different operators for creating observables. In the next posts, we will look at how we can transform the observables.

Happy coding!

The moldedbits Team

[reactive-website]: http://reactivex.io/
[rx_operators]: https://github.com/moldedbits/rx_operators
[creating-obserables]: http://reactivex.io/documentation/operators.html#creating
[create]: http://reactivex.io/documentation/operators/create.html
[auto-closable]: https://docs.oracle.com/javase/8/docs/api/java/lang/AutoCloseable.html
[create-sample]: https://github.com/moldedbits/rx_operators/blob/master/app/src/main/java/com/moldedbits/reactiveoperators/creating/create/CreateJava.java
[generate-sample]: https://github.com/moldedbits/rx_operators/blob/master/app/src/main/java/com/moldedbits/reactiveoperators/creating/create/GenerateJava.java
[generate-javadoc]: http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Observable.html
[simple]: https://github.com/moldedbits/rx_operators/blob/master/app/src/main/java/com/moldedbits/reactiveoperators/creating/simple
[empty]: https://github.com/moldedbits/rx_operators/blob/master/app/src/main/java/com/moldedbits/reactiveoperators/creating/empty
[dan-defer]: http://blog.danlew.net/2015/07/23/deferring-observable-code-until-subscription-in-rxjava/
[from-samples]: https://github.com/moldedbits/rx_operators/tree/master/app/src/main/java/com/moldedbits/reactiveoperators/creating/from
