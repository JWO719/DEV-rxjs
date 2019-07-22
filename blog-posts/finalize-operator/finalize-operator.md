# The Finalize Operator

Retrieving data from a database was a feature of an application that I was building recently, and the `GET` request was wrapped in an observable.

Whenever the `GET` request was initiated, I wanted an animated ellipses, indicating that the process was ongoing, to be

On subscription to an observable, the subscriber is provided with three handlers to respond to the data that the observable emits, and in order they are:

- The `next` handler - executed when the Observable source emits a new value from its data sequence,
- The `error` handler - executed when an error occurs in the emission of values from the Observable's data sequence, and
- The `complete` handler - executed when there is no more value available to be emitted from the observable sequence

Assuming the `getResults` method below returns an observable, the `next`, `error` and `complete` handlers are exemplified in its subscribe method

```typescript
getResults().subscribe(
  results => console.log('Next handler executed with results: ', results),
  error => console.log('Error handler executed with error: ', error),
  () => console.log(`Complete handler executed. All values have been emitted`),
);
```

Being a newbie to observables, I placed the method that hid the animated ellipses in the `complete` method like so

```typescript
getResults().subscribe(
  results => displayResults(results),
  error => notifyOnError(error.message),
  () => hideAnimatedEllipses())
);
```

and the animated ellipses was hidden (as long as the request returned no errors). Whenever there was an error, the animated ellipses still danced around the user interface alongside the error message displayed.

In order to solve this, the first thing I did was to execute the `hideAnimatedEllipses()` method in the `next` and `error` handlers. Sure thing! Until I found her.. Who?
_The finalizer!_ She not only solved my problem, she also exposed the fault in my understanding of the three subscription handlers.

I got to realise that after the `error` handler is executed, further calls to the `next` handler will have no effect, and that after the `complete` handler is executed, further calls to `next` handler will have no effect too. That was why the animated ellipses continued to confidently dance on the user interface in the first instance even after the error message was displayed.

> **finalize**: The `finalize` operator is executed when an observable's source terminates on complete or on error.

I realised that in the execution of the `finalize` operator function is where the `hideAnimatedEllipses()` function should properly reside, and so the code is now like so

```typescript
getResults()
  .pipe(finalize(() => hideAnimatedEllipses()))
  .subscribe(results => displayResults(results), error => notifyOnError(error.message));
```

In essense

> The **`next`** handler is executed when the observable source emits a new value from its data sequence,

> The **`error`** handler is executed when an error occurs in the emission of values from the Observable's data sequence. After it is executed, further calls to `next` will have no effect, and

> The **`complete`** handler is executed when there is no more value available to be emitted from the observable sequence. After it is executed, further calls to `next` will have no effect

> The **`finalize`** operator is executed when an observable's source terminates on complete or on error. It acts like the `finally` method of a promise or a `try-catch-finally`. Before RxJS v5.5, it was denoted as `finally`.

Cheers!😃
