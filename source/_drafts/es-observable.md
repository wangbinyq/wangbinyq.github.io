---
title: Observable
tags:
---

## Motivation
The Observable type represents one of the fundamental protocols for processing asynchronous streams of data. It is particularly effective at modeling streams of data which originate from the environment and are pushed into the application, such as user interface events. By offering Observable as a component of the ECMAScript standard library, we allow platforms and applications to share a common push-based stream protocol.

## Observale

```js
interface Observable {

    constructor(subscriber : SubscriberFunction);

    // Subscribes to the sequence with an observer
    subscribe(observer : Observer) : Subscription;

    // Subscribes to the sequence with callbacks
    subscribe(onNext : Function,
              onError? : Function,
              onComplete? : Function) : Subscription;

    // Returns itself
    [Symbol.observable]() : Observable;

    
    // Observable.of creates an Observable of the values provided as arguments.
    // The values are delivered synchronously when subscribe is called.
    // Observable.of("red", "green", "blue").subscribe({
    //     next(color) {
    //         console.log(color);
    //     }
    // });
    // > red
    // > green
    // > blue
    static of(...items) : Observable;

    // If the argument has a Symbol.observable method,
    //   then it returns the result of invoking that method.
    //   If the resulting object is not an instance of Observable,
    //   then it is wrapped in an Observable which will delegate subscription.
    // Otherwise, the argument is assumed to be an iterable and
    //   the iteration values are delivered synchronously
    //   when subscribe is called.
    static from(observable) : Observable;

}

interface Subscription {

    // Cancels the subscription
    unsubscribe() : void;

    // A boolean value indicating whether the subscription is closed
    get closed() : Boolean;
}

function SubscriberFunction(observer: SubscriptionObserver) : (void => void)|Subscription;
```

## An Implemtation

```js
class Observable {

  constructor (subscriber) {
    this._subscriber = subscriber
  }

  subscribe (observer, ...args) {
    // convert arguments to observer
    if (typeof observer === 'function') {
      observer = {
        next: observer,
        error: args[0],
        complete: args[1]
      }
    } else if (typeof observer !== 'object') {
      observer = {}
    }

    // return the subscription that observe
    // this observable
    return new Subscription(observer, this._subscriber)
  }

  [Symbol.observable]() { return this }

  // Converts items to an Observable
  static of (...items) {
    return new Observable((observer) => {
      for (let item of items) {
        observer.next(item)
        if (observer._closed) {
          return
        }
      }
      observer.complete()
    })
  }

  // Converts an observable or iterable to an Observable
  static from (x) {
    // if x is observable
    if (x[Symbol.observable]) {
      const observable = x[Symbol.observable]()
      // if it is Observable instance
      // just return
      if (observable.constructor === Observable) {
        return observable
      }

      // conver to Observable
      return new Observable((observer) => observable.subscribe(observer))
    }

    // if x is iterable
    if (x[Symbol.iterator]) {
      return Observable.of(...Array.from(x))
    }
  }
}


class Subscription {
  constructor (observer, subscriber) {
    this._observer = observer

    try {
      subscriber(observer)
    } catch (e) {
      observer.error(e)
    }
  }

  unsubscribe () {
    this._observer._closed = true
  }

  get closed () {
    return this._observer._closed
  }
}
```

#

statement or expressuin is sync, eager and pull
function is sync, lazy and pull
iterator or generator is sync, lazy, pull and multiple values
promise is async, eager and push
observable is async/sync, lazy and push

subscribe an observable is just like a function calling.
It starts executing some expression at that time.

## FRP (functional reactive programming)

- rxjs
  - Observable
  - Observer: an object with three callbacks: onNext, onComplete, onError
  - Subject: is equivalent to an EventEmmiter, the only way to multicasting a value or event to multiple observer. Subject also is an Observer, it can subscribe to an Observable.
  - Operators: some operater on the Observable and return an new Observable. This is like a higher order function is functional programming language.
  - Schedulers: execution context. subscribeOn: subscribe context (default: synchronously and immediately), observeOn: notification context
  - cold observable: every subscribe start an new executing context.
  - hot observable: subscribe doesn't start new executing context if it is already running.
- bacon.js
  - the `_` of Events
  - stream: is an Observable.
  - EventStream: a stream of events.
  - Property: a reactive property
- most.js
  - integrate with language features, promise, iterator, generator, asynchronous generator.
  - stream: sequence of events that occur over time. are asynchronous and may be infinite.
  - source: represents a view of events over time.
  - sink: observer
- xstream
  - stream: event emmiter
  - listener: next, error, complete
  - producer: like cold observer
