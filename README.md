
GENERATOR FUNCTION
==============

## 1. Iterator and Iterable?

- Iterators and Iterable are the concepts that allow to iterate over data structures in a standardized way. (`for...of`)
- Iterable is any object has a method named `Symbol.iterator` that returns an iterator.
- How to check object is Interable:

```
  function isIterable(obj) {
    // Check if the object is not null and has a Symbol.iterator property
    return obj != null && typeof obj[Symbol.iterator] === 'function';
  }

  // Examples
  console.log(isIterable([1, 2, 3])); // true (Array is iterable)
  console.log(isIterable('hello'));   // true (String is iterable)
  console.log(isIterable(new Map())); // true (Map is iterable)
  console.log(isIterable({}));        // false (Plain object is not iterable)
  console.log(isIterable(null));      // false (null is not iterable)
  console.log(isIterable(undefined)); // false (undefined is not iterable)
```

- Iterating over interable:

```
  const iterable = [10, 20, 30];

  for (const value of iterable) {
    console.log(value); // Outputs: 10, 20, 30
  }
```

```
  var a = [1,3,5,7,9];
  var it = a[Symbol.iterator]();

  it.next().value; // 1
  it.next().value; // 3
  it.next().value; // 5

```

- Iterator is an object that provides a sequence of values, one at a time. Iterable has `next()` method.
  - next(): returns an object with two properties:
      value: The next value in the sequence.
      done: A boolean indicating whether the iterator has completed iterating over the sequence.

- Relationship between Iterables and Iterators:
  - Iterables Produce Iterators: When you call the Symbol.iterator method on an iterable, you get an iterator.
  - Iterators Consume Iterables: The iterator's next() method is used to access each element in the iterable.

- We can build a custom iterable:

```
  const myIterable = {
    data: [1, 2, 3],
    [Symbol.iterator]: function() {
      let index = 0;
      const data = this.data;
      return {
        next: function() {
          if (index < data.length) {
            return { value: data[index++], done: false };
          } else {
            return { done: true };
          }
        }
      };
    }
  };

  for (const value of myIterable) {
    console.log(value); // Outputs: 1, 2, 3
  }

  const it = myIterable[Symbol.iterator]()

  console.log(it.next().value)

  console.log(it.next().value)

  console.log(it.next().value)
```

## 2. What is Generator Function?

- Generator function is function that can be paused and resumed to produce a sequence of values over time.
- Generator function is available in ES6. It is defined by using `function*` syntax and `yield` a value.
- Generator function return a `iterator`.

Normal Function

```
  var x = 1;

  function foo() {
    x++;
  }

  function bar() {
    x++;
  }

  foo();
  console.log("x:", x);

  bar();
  console.log("x:", x);
```

Generator Function

```
  var x = 1;

  function* foo() {
    yield;
    console.log('Go down');
    x++;
  }

  function bar() {
    x++;
  }

  const it = foo();

  const res1 = it.next()
  console.log("x:", x);
  console.log(res1);

  bar();
  console.log("x:", x);

  const res2 = it.next();
  console.log("x:", x);
  console.log(res2);
```

## 3. Generator function Input and Outpout

- Generator function is still a function, which means it still has some basic tenets that haven’t changed—namely, that it still accepts arguments (aka input), and that it can still return a value (aka output)

```
  function *foo(x,y) {
    return x * y;
  }

  var it = foo( 6, 7 );
  var res = it.next();

  res.value; // 42
```

- Generator function has more powerful to pass massaging via `yield` and `next(..)`

```
  function *foo(x) {
    var y = x * (yield);
    return y;
  }
  
  var it = foo(6);
  it.next();
  
  var res = it.next(7);
  res.value; // 42
```

- In generator function messages can go in both directions:
  - yield .. as an expression can send out messages in response to next(..) calls
  - next(..) can send values to a paused yield expression.

```
  function *foo(x) {
    var y = x * (yield "Hello"); // <-- yield a value!
    return y;
  }

  var it = foo(6);
  var res = it.next(); // first `next()`, don't pass anything
  res.value; // "Hello"
  res = it.next(7);
  res.value; // 42
```

Note: We don’t pass a value to the first next() call. Because only a paused yield could accept such a value passed by a next(..), and at the beginning of the generator when we call the first next(), there is no paused yield to accept such a value.

## 4. Multiple Iterators

- Each time you construct an iterator, you are implicitly constructing an instance of the generator which that iterator will control.
- You can have multiple instances of the same generator running at the same time, and they can even interact:

```
  function *foo() {
    var x = yield 2;
    z++;
    var y = yield (x * z);
    console.log( x, y, z );
  }

  var z = 1;
  var it1 = foo();
  var it2 = foo();

  var val1 = it1.next().value; 
  var val2 = it2.next().value;

  val1 = it1.next( val2 * 10 ).value;
  val2 = it2.next( val1 * 5 ).value;

  it1.next( val2 / 2 );
  it2.next( val1 / 4 ); 
```

## 5. Stop the generator

- We have two ways to stop the generator function:

```
  function *something() {
    try {
      var nextVal;
      while (true) {
        if (nextVal === undefined) {
          nextVal = 1;
        } else {
          nextVal = (3 * nextVal) + 6;
        }
      
        yield nextVal;
      }
    }
    // cleanup clause
    finally {
      console.log( "cleaning up!" );
    }
  }
```

  Using `break`

```
var it = something();

for (var v of it) {
  console.log( v );
  
  if (v > 500) {
    break;
  }
}

console.log(it.next())

```

  Using `return(..)`

```
var it = something();

for (var v of it) {
  console.log( v );
  
  if (v > 500) {
    console.log(it.return( "Hello World" ).value);
  }
}

console.log(it.next())
```
