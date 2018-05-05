---
title: "Tips for refactoring to RxJava 2 with Kotlin"
layout: post
date: 2018-05-05 14:00
image: /assets/images/coffee.jpg
headerImage: true
tag:
- android
category: blog
author: miquel
description: Refactoring from RxJava 1 to version 2 can be tricky, but we can improve the experience with the help of Kotlin
---

We recently finished our refactoring from RxJava 1 to version 2. It was an
interesting trip, and thankfully, less painful that we expected. On our way
we learned few things that I will share here.

### Observables or Single/Maybe/Completable/Flowable?

The first big question you will have is if you want to keep using `Observable`
or you will want to switch to the newly introduced `Single`, `Maybe`, `Completable`
or `Flowable`. Some of them were available on RxJava 1 already but with limitations:
For example, `Completable` on RxJava 1 didn't work with Retrofit.

Our decision was to switch to those when possible.

Most of those Retrofit calls that used Observable now use `Single` and `Completable`.

Our caching data layer now uses `Maybe` instead of `Observable` that return null
(which was a bad idea to start with).

Other data streams were switched to `Flowable` from `Observable` for better back
pressure handling.

However, this came with the added cost of also modifying our implementation.
Before you start your upgrade process, decide if you want to also do this or
not. It may not be the best idea depending on your situation.

### Is your codebase well tested?

The lack of unit tests will definitely affect your refactoring.

Switching our tests to RxJava 2 was easier than expected, here's an example:

**RxJava 1**
```kotlin
val subscriber = TestSubscriber.create<Void>()
usecase.request(data).subscribe(subscriber)
```

**RxJava 2**
```kotlin
val subscriber = usecase.request(data).test()
```

I would be very careful about refactoring a large codebase that is not well
covered by tests. If that's your case, I'd consider to at least cover the most
important business critical use cases before attempting the refactor.

### Do your Observables emit null values?

RxJava 2 no longer accepts nulls, and those will cause `NullPointerException`
errors.

If you are using nulls as part of your logic, you will have to switch
to a different solution, for example:

- `Completable`: When you only care about the completion.
- `Single` or `Flowable` with Optional: emit a value that can be there or not.
- Maybe: emit only when you have a value, otherwise empty.

It will depend on your use cases on what you want to do.

### Are you using Kotlin?

Refactoring to RxJava 2 with Kotlin has some clear advantages, specially
if you are not using Retrolambda on Java. Instead, if you are using
anonymous classes, you will have to migrate all of them.

For example, all `Action1` anonymous classes will have to be migrated to `Consumer`
classes.

**RxJava 1**

```java
.doOnNext(new Action1<Boolean>() {
    @Override
    public void call(Boolean isValid) {
        aMethod(isValid);
    }
})
```

**RxJava 2**

```java
.doOnNext(new Consumer<Boolean>() {
    @Override
    public void accept(Boolean isValid) {
        aMethod(isValid);
    }
})
```

But on the other hand, with Kotlin (or with Retrolambda) the change is transparent.

**RxJava 1**

```kotlin
.doOnNext {
     aMethod(it)
}
```

**RxJava 2**

```kotlin
.doOnNext {
    aMethod(it)
}
```

My recommendation is that if you are also migrating your codebase to Kotlin,
do that first, and then after migrate to RxJava 2, because the task will be
easier once your codebase is in Kotlin.

However, be ready to deal with some surprises too. An example of that is
`BiFunction` in `combineLatest`. Type inference won't work in those cases
anymore and you will have to specify that you are using a `BiFunction`
explicitly.

**RxJava 1***

```kotlin
Observable.combineLatest(email, password, { email, password ->
    email.isValid() && password.isValid()
})
```

**RxJava 1***

```kotlin
Observable.combineLatest(email, password,
    BiFunction<Boolean, Boolean, Boolean> { email, password ->
        email.isValid() && password.isValid()
    }
)
```

Another dangerous one is `andThen`.

Using `andThen` like this:

```kotlin
.andThen { aSingle() }
```

Is not the same as doing it like this:

```kotlin
.andThen(aSingle())
```

The first example is creating a SingleSource that does nothing!

Be careful, just because it compiles it does not mean it will work how you thought.

### Do I have to migrate all my codebase at once?

No. Don't do that! Instead, use the Interop library to connect the parts of
your codebase that are still in version 1 to the parts migrated to version 2.

```kotlin
return RxJavaInterop.toV1Single(myV2Single)
```

However, if you use Kotlin, I recommend you to create extension
functions to make the task easier:

```kotlin
fun <T> Single<T>.toV1(): rx.Single<T> = RxJavaInterop.toV1Single(this)
```

Then, you can do the above operation like this:

```kotlin
return myV2Single.toV1()
```

Here's the full snippet with some of the extensions that we found useful:

<script src="https://gist.github.com/miquelbeltran/aec73df3f2511a98c0ec51e4cc9a44b4.js"></script>

This way, you can do the following:

```kotlin
fun myOldObservableRx1(): rx.Observable {
    return myV2Single
        .toV1()
        .toObservable()
}
```

In this example a method that was returning an RxJava 1 `Observable` can use
inside a `Single` that is already in version 2. There's no need to switch
the whole codebase at once.

### One final tip: Imports

I'd advice you to disable automatic imports during the migration, to avoid
collisions with the `rx` or the `java.util.Observable` package. As well, avoid using
wildcard imports (star) and always explicitly declare them.

## Summary

- Decide beforehand if you want to switch to the new Observable types from RxJava 2.
- Invest time implementing tests around your version 1 codebase before the migration.
- Check whenever you are using nulls in your Rx streams.
- Consider migrating to Kotlin before performing the refactor.
- Make good use of the interop methods to migrate method by method safely.

