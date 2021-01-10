---
layout: "post"
title: "ReactiveX | Part II: Observable & Observer from scratch"
permalink: /:categories/:title
categories: reactive_programming observable
description: Build the core concept from scratch
---

The best way to have a solid understanding about a problem is to build things from bottom up. In this post, we'll implement the core concept of Rx from scratch, step by step, but without optimization & non-essential parts.

The observables/publishers are the things that can be observed. They "push" data to their consumers. The `WeatherStation` from previous part is an example for observable.

The observers/subscribers is the listeners. They consume the data and react to these. In previous part, the `PC`, `TV` and `SmartPhone` classes are observers: they receive data from `WeatherStation` and display it.

Observable is no more than a function. Let's create a function named `myObservable`:

```js
function myObservable(observer) {}
```

Observer is no more than an object has 3 methods: `next` to get the value, `error` to get the error and `complete` takes no args

```js
const observer = {
    next(value) { console.log('next -> ', value) }
    error(err) {}
    complete() { console.log('complete') }
}
```

Now let's put some logics into `myObservable`

```js
function myObservable(observer) {
    for (let i = 0; i < 10; i ++) {
        observer.next(i)
    }
    observer.complete();
}
myObservable(observer)
```

The output should look like this

```console
next ->  0
next ->  1
next ->  2
next ->  3
next ->  4
next ->  5
next ->  6
next ->  7
next ->  8
next ->  9
complete
```

This is basic observable.

What if some async tasks need to be executed inside?

Let's update `myObservable`:

```js
function myObservable(observer) {
    const id = setTimeout(() => {
        observer.next('hello')
        observer.complete()
    }, 1000)
    return () => clearTimeout(id)
}
myObservable(observer)
```

"hello" and then "complete" will be shown after a second. Also notice that `myObservable` now returns an unsubscription function to cancel the async task.

```js
const unsub = myObservable(observer);
unsub();
```

With this update, the async task is cancelled before the "hello" appears, pretty cool 😎

But there are still have some problems. Let's change the code:

```js
function myObservable(observer) {
    for (let i = 0; i < 10; i++ ){
        observer.next(i)
    }
    observer.complete()
    observer.next('haha')
}
const observer = {
    next(value) { console.log('next -> ', value) }
    error(err) {}
    complete() { console.log('complete') }
}
myObservable(observer)
```

The problem is when observer is completed, the `next` method still could be called, same problem with `error` method. Let's have a class named `SafeObserver`

```js
class SafeObserver {
  constructor(observer) {
    this.observer = observer;
    this.isUnsubscribed = false;
  }

  next(value) {
    if (this.isUnsubscribed) {
      return;
    }
    if (this.observer && this.observer.next) {
      this.observer.next(value);
    }
  }

  error(err) {
    if (this.isUnsubscribed) {
      return;
    }
    this.isUnsubscribed = true;
    this.observer.error(err);
  }

  complete() {
    if (this.isUnsubscribed) {
      return;
    }
    this.isUnsubscribed = true;
    this.observer.complete();
  }
}

const safeObserver = new SafeObserver(observer);
myObservable(safeObserver);
```

Now there is no more "next" when the observer is completed.

We just built observable & observer from scratch. In real world, RxJS provides us with creation operations to easily generate Observables for a lot of sync & async case.