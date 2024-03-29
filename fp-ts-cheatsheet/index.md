
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [fp-ts cheatsheet](#fp-ts-cheatsheet)
  - [pipe & flow](#pipe--flow)
      - [When to use flow over pipe](#when-to-use-flow-over-pipe)
  - [Options](#options)
    - [O.fromNullable](#ofromnullable)
    - [O.fromPredicate](#ofrompredicate)
    - [O.fromEither](#ofromeither)
    - [O.map](#omap)
    - [O.flatten](#oflatten)
    - [O.chain](#ochain)
    - [O.fold / O.match](#ofold--omatch)
    - [O.alt](#oalt)
    - [O.toNullable / O.toUndefined](#otonullable--otoundefined)
    - [O.getOrElse](#ogetorelse)
    - [O.getOrElseW](#ogetorelsew)
    - [O.isNone / O.isSome](#oisnone--oissome)
  - [Task](#task)
  - [Either](#either)
    - [E.fromNullable](#efromnullable)
    - [E.fromOption](#efromoption)
    - [E.fromPredicate](#efrompredicate)
    - [E.map / E.mapLeft / E.bimap](#emap--emapleft--ebimap)
    - [E.tryCatch](#etrycatch)
    - [E.fold / E.match](#efold--ematch)
    - [E.chain / E.chainFirst](#echain--echainfirst)
    - [E.getOrElse](#egetorelse)
  - [TaskEither](#taskeither)
    - [TE.tryCatch](#tetrycatch)
    - [TE.map](#temap)
    - [TE.fold](#tefold)
  - [Eq](#eq)
    - [Primitive equality](#primitive-equality)
    - [Compare structures](#compare-structures)
    - [Compare arrays](#compare-arrays)
    - [Custom definitions](#custom-definitions)
    - [More Eq instances](#more-eq-instances)
      - [Option](#option)
      - [Either](#either-1)
  - [Ord](#ord)
    - [Primitive comparison](#primitive-comparison)
    - [Custom comparison](#custom-comparison)
    - [Min, max, clamp, lt, geq, between](#min-max-clamp-lt-geq-between)
    - [Sort an array](#sort-an-array)
    - [More Ord instances](#more-ord-instances)
      - [Option](#option-1)
      - [Tuple](#tuple)
  - [IO](#io)
  - [IOEither](#ioeither)
  - [Misc](#misc)
    - [MonoidAny & MonoidAll](#monoidany--monoidall)
  - [SequenceT](#sequencet)
      - [Option](#option-2)
      - [Task](#task-1)
      - [TaskEither](#taskeither-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# fp-ts cheatsheet

## pipe & flow
The `pipe` operator can be used to chain a sequence of functions, from left to right. 

The first argument can be any arbitrary value and subsequent arguments must be functions of arity one. The return type of a preceding function in the pipeline must match the input type of the subsequent function.

In short, you can use the `pipe` operator to transform any value using a sequence of functions.

The flow of control can be modelled as follows:
```
A -> (A->B) -> (B->C) -> (C->D)
```

The `flow` operator is similar to `pipe` but the first argument must be a function rather then any arbitrary value. The first function is also allowed to have an arity of more than one.

The flow of control can be modelled as follows:
```
(A->B) -> (B->C) -> (C->D)
```

Example:
```typescript
import { pipe, flow } from 'fp-ts/lib/function'

const add1 = (x: number) => x + 1
const multiply2 = (x: number) => x * 2

// Without pipe or flow
multiply2(add1(1)) // 4

// The following are equivalent
pipe(1, add1, multiply2)        // 4
pipe(1, flow(add1, multiply2))  // 4    
flow(add1, multiply2)(1)        // 4 

```
#### When to use flow over pipe
A general rule of thumb is when you want to avoid using an anonymous function. In Typescript, a good example of an anonymous function are callbacks.

Example:
```typescript
import { pipe, flow } from 'fp-ts/lib/function'

function concat(
  a: number,
  transformer: (a: number) => string,
): [number, string] {
  return [a, transformer(a)]
}

// With pipe
concat(1, (n) => pipe(n, add1, multiply2, toString)) // [1, '4']

// With flow
concat(1, flow(add1, multiply2, toString)) // [1, '4']

```

## Options
**Options** are containers that wrap values that could be either `undefined` or `null`. If the value exists, we say the Option is of the `Some` type. If the value is undefined or null, we say it has the `None` type.

The Option type is a discriminated union of `None` and `Some`.

```typescript
type Option<A> = None | Some<A>
```

In most cases, you won't need to use Option; optional chaining is less verbose.  
But the Option type is more than just checking for null. Option can be used to represent failing operations. And just like how you can lift undefined into an Option, you can also lift an Option into another fp-ts container, like Either.

From now on, the Option container will be referred to using the "O" shorthand.
Some and None types can be accessed as `O.Some` and `O.None`

The easiest way to build an `Option` is to use the some or none constructor that returns a value encapsulated in an `Option`.

Example:
```typescript
O.some(42)    // { _tag: 'Some', value: 42 } 
O.none        // { _tag: 'None' }
```
Option provide a wide set of tools.

### O.fromNullable
`O.fromNullable` creates a container object tagged as None or Some, depending on the value of its argument being null/undefined or not.
If you have a value to check it you can use `O.fromNullable`.

```typescript
O.fromNullable('Existing')    // { _tag: 'Some', value: 'Existing' } 
O.fromNullable(null)          // { _tag: 'None' }
```

### O.fromPredicate
With `O.fromPredicate` you can pass your own validation function to build an Option (similar to [E.fromPredicate](#efrompredicate) but it builds an Option object instead).

```typescript
const isUnderage = (x: { age: number }) =>
  pipe(
    x,
    O.fromPredicate(({ age }) => age < 18)
  )

isUnderage({ age: 30 }) // { _tag: "None" }
isUnderage({ age: 17 }) // { _tag: "Some", value: { age: 17 } }
```

### O.fromEither
You can build an `Option` from an `Either`. If the Either is in its left state (~ error) you get an `Option.None`,
otherwise you get an `Option.Some` of the value in the Either

```typescript
const leftEither = Either.left('whatever')
const rightEither = Either.right('value')

const noneValue = Option.fromEither(leftEither)       // Option.None
const optionValue = Option.fromNullable(rightEither)  // Option.Some("value")
```

### O.map
The map operator allows to access and transform one value to another:

```typescript
pipe(
  { bar: 'hello' },
  O.fromNullable,
  O.map(({ bar }) => bar)
); // { _tag: 'Some', value: 'hello' }
```

```typescript
pipe(
  undefined,
  O.fromNullable,
  O.map(({ bar }) => bar)
); // { _tag: 'None' }
```

It works by performing a comparison over the `_tag` property.
If the `_tag` is `Some`, it transforms the value using the function passed into `map`.
However, if the `_tag` is `None`, no operation is performed. The container remains in the `None` state.

### O.flatten
The O.flatten operator allows to traverse nested pipes to access an inner property of an object:
```typescript
const foo = { bar: undefined }

// optional chaining:
pipe(foo, (f) => f?.bar?.buzz) // undefined

// using Option:
pipe(
  foo,
  O.fromNullable,
  O.map(({ bar }) =>
    pipe(
      bar,
      O.fromNullable,
      O.map(({ buzz }) => buzz),
    ),
  ),
  O.flatten,
) // { _tag: 'None' }
```
The result is `{ _tag: 'None' }` which basically means that foo.bar.buzz is None.

Without the O.flatten, the result would be:

`{ _tag: 'Some', value: { _tag: 'None' } }`

### O.chain
The `O.chain` operator implements a flatmap, merging the concepts behind O.map and O.flatten:
```typescript
pipe(
  foo,
  O.fromNullable,
  O.map(({ bar }) => bar),
  O.chain(
    flow(
      O.fromNullable,
      O.map(({ buzz }) => buzz),
    ),
  ),
) // { _tag: 'None' }
```
Which is very similar to the example provided for O.flatten, but more concise.
It can be used as a logical 'AND'.  

### O.fold / O.match
It takes two functions and an Option value. The first function is the (lazy) default value and is executed if the Option is `None`,
the second function is applied to the `Some` value if the Option is `Some` and the result is returned.

```typescript
const isUnderage = (x: { age: number }) =>
  pipe(
    x,
    O.fromPredicate(({ age }) => age < 18),
    O.fold(
      () => 'Adult',
      () => 'Underage'
    )
  );

isUnderage({ age: 30 }) // "Adult"
isUnderage({ age: 15 }) // "Underage"
```

### O.alt
Returns the left-most non-`None` value.  
It can be used as a logical 'OR'.  

```typescript
const isUppercase = (x: string) => x.toUpperCase() === x ? O.of(x) : O.none
const isLowercase = (x: string) => x.toLowerCase() === x ? O.of(x) : O.none

const checkCase = (x: string) => {
  return pipe(
    isUppercase(x),
    O.alt(() => isLowercase(x)),
    O.fold(
      () => "It's camelcase",
      () => "It's lowercase or uppercase"
    ) 
  )
}
 
checkCase("thisislower")   // It's lowercase or uppercase
checkCase("THISISUPPER")   // It's lowercase or uppercase
checkCase("CamelCase")     // It's camelcase
```

### O.toNullable / O.toUndefined
If Your `Option` contains a value, you get it, otherwise you get `null` / `undefined`.

```typescript
const noneValue = O.none
const someValue = O.of('value')

O.toUndefined(noneValue) // undefined
O.toUndefined(someValue) // "value"
O.toNullable(noneValue) // null
O.toNullable(someValue) // "value"
```

### O.getOrElse
Extracts the value out of the structure, if it exists. Otherwise it returns the given default value.

`O.getOrElse` takes a function as parameter that returns the default value for `none`.

The default value must have the same type as your initial `Option`. Eg: `getOrElse` on an `Option<number>` must return a number.
If you want to return another type, you can use `O.getOrElseW`.

Example:
```typescript
const sillyFunction = (x?: string) => pipe(
  O.fromNullable(x),
  O.getOrElse(() => "error")
)

sillyFunction(undefined)   // error
sillyFunction("ok")       // ok
```

### O.getOrElseW
Similar to `O.getOrElse`, but it is less strict, in the sense that the default value can have a different type of the initial `Option`.

Example:
```typescript
const sillyFunction = (x?: string) => pipe(
  O.fromNullable(x),
  O.getOrElseW(() => 42) // returns number instead of string
)
```

### O.isNone / O.isSome
Returns `true` if the option is `None` / `Some`, `false` otherwise.

```typescript
const noneValue = O.none 
const someValue = O.some("value") 

O.isNone(noneValue)  // true
O.isNone(someValue)  // false

O.isSome(noneValue)  // false
O.isSome(someValue)  // true
```

## Task
A task is a function that returns a promise which is expected to never reject().

Tasks are expected to always succeed but can fail when an error occurs outside our expectations. In this case, the error is thrown and breaks the functional pipeline. An analogy to this is awaiting a Promise that throws an error without putting a try-catch-finally block in front.    
**Remember to test your assumptions before using Task.**

A Task is more than a glorified promise; it is also an expression of intent.
A Promise provides no indication about whether the function can fail. As such, in the imperative model, you are forced to handle these errors using a try-catch-finally block.

```typescript
type Task<A> = () => Promise<A>
```

Example:
```typescript
// The following already implements the Task interface...
async function boolTask(): Promise<boolean> {
  try {
    await asyncFunction()
    return true
  } catch (err) {
    return false
  }
}

// ... but we can remove the “Promise” ambiguity by adjusting the syntax. 
import { Task } from 'fp-ts/lib/Task'
const boolTask: Task<boolean> = async () => {
  try {
    await asyncFunction()
    return true
  } catch (err) {
    return false
  }
}
```

## Either
`Either` is another fp-ts container (just like Option) that's basically a *synchronous* operation that can succeed or fail.
Much like Option, where it is `Some` or `None`, the Either type is either `Right` or `Left`. Right represents success and Left represents failure.

```typescript
type Either<E, A> = Left<E> | Right<A>
```

Eithers are very useful for catching error scenarios in FP, and have a clue as to why our program has derailed. With an analogy, they can be considered as the try-catch construct equivalent of functional programming, but with type safe checking. So when we manipulate an Either, if we enter the left branch, we most of the time won't carry out further manipulations, and just return the Error context that is encapsulated in the left branch (it is still accessible tho, through different helpers). If we are in the right branch, we'll keep on manipulating the value and passing it through the right branch.
We need the Eithers because we cannot break pipelines by throwing errors.
Either is great for casual errors like validation as well as more serious, errors like missing files or broken sockets.
It also captures logical disjunction (a.k.a ||) in a type.

Example:  
```typescript
const passwordMinLenght = (pass: string, minLen: number): E.Either<Error, string> => (
  (pass.length < minLen)
    ? E.left(new Error('Password too short'))
    : E.right(pass)
)

passwordMinLenght('test', 1)    // {_tag: "Right", right: "test"}
passwordMinLenght('test', 8)    // {_tag: "Left", left: Error}
```

The easiest way to build an `Either` is to use the right or left constructor that returns a value encapsulated as a right or left `Either`.
From now on, the Either container will be referred to using the "E" shorthand.

```typescript
const leftValue = E.left("value"); // -> you are on the left branch
const rightValue = E.right("value"); // -> you are on the right branch
```

But you have also a `fromNullable`, `fromOption` and `fromPredicate` helpers.
### E.fromNullable
Construct a new Either from a nullable object. If the value is `null` or `undefined` you need to provide your Either what to put in the left track,
otherwise it will put your value in the right track.

```typescript
pipe(
  null,
  E.fromNullable(new Error('The object is null')),
)   // {_tag: "Left", left: Error}

pipe(
  42,
  E.fromNullable(new Error('The object is null')),
)   // {_tag: "Right", right: 42}
```

### E.fromOption
Construct a new Either from an Option.
It works exactly as the fromNullable as we've seen an Option could represent a nullable value.

```typescript
pipe(
  O.some(42),
  E.fromOption(() => 'error')
)   // {_tag: "Right", right: 42}

pipe(
  O.none,
  E.fromOption(() => 'error')
)   // {_tag: "Left", left: "error"}
```

### E.fromPredicate
Construct a new Either based on the given predicate.
You have to first pass a function checking your value is correct (should I go right or left track).
This function can be a simple function returning a boolean, or a type guard. And then pass a value for the left track.

```typescript
const check = (
  E.fromPredicate(
    (n: number) => n > 0,
    () => 'error'
  )
)

check(42) // {_tag: "Right", right: 42}
check(-1) // {_tag: "Left", left: "error"}
```

```typescript
type EvenNumber = number;
const isEven = (num: number) => num % 2 === 0;
const isEvenTypeGuard = (num: number): num is EvenNumber => num % 2 === 0;
const eitherBuilder = E.fromPredicate(
  isEven, // here we could use isEvenTypeGuard to infer the number type and have an E.Either<E, EvenNumber>
  (number) => `${number} is an odd number`
);

// Here when you use eitherBuilder you get something with the type E.Either<string, number>
const leftValue = eitherBuilder(3);   // {_tag: "Left", left: '3 is an odd number')
const rightValue = eitherBuilder(4);  // {_tag: "Right", right: 4}
```

### E.map / E.mapLeft / E.bimap
Similar to `O.map` but applied to an Either container.
`map` will only transform your data if you are in the right branch.

If you want to map on the left branch, you can use `mapLeft`.
``` typescript
const doubleIfEven = (n: number) => n % 2 === 0 ? 2 * n : n

const rightValue = E.right(2);
const leftValue = E.left(2);

pipe(
  rightValue,
  E.map(doubleIfEven)
) // {_tag: "Right", right: 4}

pipe(
  rightValue,
  E.map(doubleIfEven)
) // {_tag: "Right", right: 4}

pipe(
  leftValue,
  E.map(doubleIfEven)
) // {_tag: "Left", left: 2}

pipe(
  leftValue,
  E.mapLeft(doubleIfEven)
) // {_tag: "Left", left: 4}
```

Eventually, there is also a `bimap` helper mapping the first function to the left branch,
and the second function to the right branch
```typescript
const evenErrorMessage = (n: number) => {
  if (n % 2 === 0) return `${n} is even but in Error State`;

  return `${n} is odd and in Error State`;
};

const doubleOrError = E.bimap(evenErrorMessage, doubleIfEven);

const rightValue = E.right(2);
const leftValue = E.left(3);

pipe(
  rightValue,
  E.bimap(evenErrorMessage, doubleIfEven)
); // {_tag: "Right", right: 4}

pipe(
  leftValue,
  E.bimap(evenErrorMessage, doubleIfEven)
); // {_tag: "Left", left: "3 is odd and in Error State"
```

### E.tryCatch
Constructs a new Either from a function that might throw.

Example:
```typescript
const myUnsafeFunction = (x: number) => {
  if (x > 10) return x
  throw new Error()
}

const safeFunction = (x: number) =>
  E.tryCatch(
    () => myUnsafeFunction(x),
    () => new Error('Error!!!')
  )

safeFunction(20)  // {_tag: "Right", right: 20}
safeFunction(2)   // {_tag: "Left", left: Error}
```

### E.fold / E.match
Takes two functions and an Either value, if the value is a `Left` the inner value is applied to the first function, 
if the value is a `Right` the inner value is applied to the second function.

Example:
```typescript
const onLeft = (s: string) => `Error ${s}`
const onRight = (s: string) => `Success ${s}`

pipe(
  E.right("42"),
  E.fold(onLeft, onRight)
) // Success 42

pipe(
  E.left("-1"),
  E.fold(onLeft, onRight)
) // Error -1 
```

### E.chain / E.chainFirst
`chain` will only transform your data if you are in the right branch.
Otherwise, it will return your Either as it was. To "flatten" your Either, there are two cases:

1. you chain a function with the same left type (returning `Either<E, *>`). 
You can use the `chain` method and it will return an `Either<E, *>` (what is returned by your chained function)

2. you chain a function with a different left type (returning `Either<D, *>`).
To flatten your Either, you will have to combine both error types.
You have to use the `chainW` method, where `W` means widen, as you extend the error type. It will return an `Either<E | D, *>`
```typescript
  const doubleIfEven = (n: number): E.Either<string, number> =>
    n % 2 === 0 ? E.right(2 * n) : E.left(`${n} is not Even`);

  pipe(
    E.right(2),
    E.chain(doubleIfEven)
  );  // {_tag: "Right", right: 4}

  pipe(
    E.right(3),
    E.chain(doubleIfEven)
  );  // {_tag: "Left", left: "3 is not Even"}

  pipe(
    E.left("Initial Error"),
    E.chain(doubleIfEven)
  );  // {_tag: "Left", left: "Initial Error"}
  
  pipe(
    E.left(false),
    E.chainW(doubleIfEven)
  );  // {_tag: "Left", left: false}
```

If you want to use your data but return it unaltered to the rest of your pipeline (like some kind of side effect),
but still fail if something wrong happened, you can use the `chainFirst` (or `chainFirstW`) combinator:
```typescript
const doubleAndTellIfEven = (n: number): E.Either<NotEvenError, string> =>
  n % 2 === 0
    ? E.right(`${2 * n} is an even number!`)
    : E.left(notEvenError(n));

pipe(
  E.right(2),
  E.chainFirstW(doubleAndTellIfEven), // this goes right branch, but we drop the string returned and pass the initial value
  E.getOrElse(() => 0)                // here we get the original 2
);

pipe(
  E.right(3),
  E.chainFirstW(doubleAndTellIfEven), // this goes left branch due to failed validation
  E.getOrElse(() => 0)                // here we get default 0
);
```

### E.getOrElse
The `getOrElse` destructor and its `getOrElseW` version work exactly as the one from `Option`.
You have to pass it what to return if you are in the left branch.

```typescript
const leftValue = E.left("Division by Zero!");
const rightValue = E.right(10);

E.getOrElse(() => 0)(leftValue);  // 0
E.getOrElse(() => 0)(rightValue); // 10
```


## TaskEither
`TaskEither` merges together the concepts of Taks and Either, applied to the asynchronous scenarios, e.g. network calls of an application.

A TaskEither can be basically seen as the wrapper for the construct "then / catch" in FP (and everything in the middle).
Indeed, in fp-ts there is TE.tryCatch.

### TE.tryCatch

```typescript
// ES6:
fetch('https://example.com/api/v1/users')
  .then(res => ...)
  .catch(e => console.error(e))

// fp-ts:
const ok = await pipe(
  TE.tryCatch(
    () => axios.get('https://httpstat.us/200'),
    (reason) => new Error(`${reason}`),
  ),
  TE.map((resp) => resp.data),
)()
```
```typescript
const curried = (url: string) => async () => {
  const result = await pipe(
    TE.tryCatch(
      () => fetch(url),
      (reason) => new Error(`${reason}`)
    ),
    TE.map((resp) => resp.status),
    invokeTask => invokeTask()
  )
  console.log(result)
} 

setTimeout(curried("http://not-existing"), 1000)     // {_tag: "Left", left: Error}
setTimeout(curried("https://httpstat.us/200"), 2000) // {_tag: "Right", right: 200}
```

### TE.map
Same concept of O.map and E.map, only applied to TaskEithers

### TE.fold
Placed at the end of a TaskEither pipe, in a condition of success, returns a Task.
```typescript
import {
  absurd,
  constVoid,
  pipe,
  unsafeCoerce,
} from 'fp-ts/lib/function'

const result = pipe(
  TE.tryCatch(
    () => axios.get('https://httpstat.us/200'),
    () => constVoid() as never,
  ),
  TE.map((resp) => unsafeCoerce<unknown, Resp>(resp.data)),
  TE.fold(absurd, T.of),
) // Not executing the promise

// Result is of type:
// T.Task<Resp>
```

## Eq
The Eq type class represents types which support decidable equality.

Instances must satisfy the following laws:
- Reflexivity: `E.equals(a, a) === true`
- Symmetry: `E.equals(a, b) === E.equals(b, a)`
- Transitivity: `if E.equals(a, b) === true and E.equals(b, c) === true, then E.equals(a, c) === true`


### Primitive equality
```typescript
import { eq } from "fp-ts"

eq.eqBoolean.equals(true, true)                                    // true
eq.eqDate.equals(new Date("1984-01-27"), new Date("1984-01-27"))   // true
eq.eqNumber.equals(3, 3)                                           // true
eq.eqString.equals("Cyndi", "Cyndi")                               // true
```

### Compare structures
```typescript
type Point = {
  x: number
  y: number
}

const eqPoint: eq.Eq<Point> = eq.getStructEq({
  x: eq.eqNumber,
  y: eq.eqNumber,
})

eqPoint.equals({ x: 0, y: 0 }, { x: 0, y: 0 })  // true
```
```typescript
type Point = {
  x: number
  y: number
}

const eqPoint: eq.Eq<Point> = eq.getStructEq({
  x: eq.eqNumber,
  y: eq.eqNumber,
})

type Vector = {
  from: Point
  to: Point
}

const eqVector: eq.Eq<Vector> = eq.getStructEq({
  from: eqPoint,
  to: eqPoint,
})

eqVector.equals(
  { from: { x: 0, y: 0 }, to: { x: 0, y: 0 } },
  { from: { x: 0, y: 0 }, to: { x: 0, y: 0 } }
) // true
```

### Compare arrays
```typescript
import { array, eq } from "fp-ts"

const eqArrayOfStrings = array.getEq(eq.eqString)

eqArrayOfStrings.equals(["Time", "After", "Time"], ["Time", "After", "Time"]) // true
```
```typescript
import { array, eq } from "fp-ts";

type Point = {
  x: number
  y: number
}

const eqPoint: eq.Eq<Point> = eq.getStructEq({
  x: eq.eqNumber,
  y: eq.eqNumber,
})

const eqArrayOfPoints: eq.Eq<Array<Point>> = array.getEq(eqPoint)

eqArrayOfPoints.equals(
  [
    { x: 0, y: 0 },
    { x: 4, y: 0 },
  ],
  [
    { x: 0, y: 0 },
    { x: 4, y: 0 },
  ]
) // true
```

### Custom definitions
```typescript
import { eq } from "fp-ts"

type User = {
  userId: number
  name: string
}

const eqUserId = eq.contramap((user: User) => user.userId)(eq.eqNumber)

eqUserId.equals({ userId: 1, name: "Giulio" }, { userId: 1, name: "Giulio Canti" }) // true
eqUserId.equals({ userId: 1, name: "Giulio" }, { userId: 2, name: "Giulio" })       // false
```

### More Eq instances
Many data types provide Eq instances.

#### Option
```typescript
import { eq, option } from "fp-ts"

const E = option.getEq(eq.eqNumber)

E.equals(option.some(3), option.some(3))  // true
E.equals(option.none, option.some(4))     // false
E.equals(option.none, option.none)        // true
```

#### Either
```typescript
import { either, eq } from "fp-ts"

const E = either.getEq(eq.eqString, eq.eqNumber)

E.equals(either.right(3), either.right(3))    // true
E.equals(either.left("3"), either.right(3))   // false
E.equals(either.left("3"), either.left("3"))  // true
```

## Ord
If you need to decide on the order of two values, you can make use of the compare method provided by Ord instances. Ordering builds on equality.   
Note that compare returns an Ordering, which is one of these values -1 | 0 | 1. We say that:
- x < y if and only if compare(x, y) is equal to -1
- x is equal to y if and only if compare(x, y) is equal to 0
- x > y if and only if compare(x, y) is equal to 1

Note that all `Ord` instances also define the equals method, because it is a prerequisite to be able to compare data.


### Primitive comparison
```typescript
import { ord } from "fp-ts"

ord.ordNumber.compare(4, 5)   // -1
ord.ordNumber.compare(5, 5)   // 0
ord.ordNumber.compare(6, 5)   // 1

ord.ordBoolean.compare(true, false)                                   // 1
ord.ordDate.compare(new Date("1984-01-27"), new Date("1978-09-23"))   // 1
ord.ordString.compare("Cyndi", "Debbie")                              // -1

ord.ordBoolean.equals(false, false)  // true
```

### Custom comparison
```typescript
const strlenOrd = ord.fromCompare((a: string, b: string) =>
  a.length < b.length ? -1 : a.length > b.length ? 1 : 0
)
strlenOrd.compare("Hi", "there")          // -1
strlenOrd.compare("Goodbye", "friend")    // 1
```
But most of the time, you can achieve the same result in a simpler way with contramap:
```typescript
const strlenOrd = ord.contramap((s: string) => s.length)(ord.ordNumber)
strlenOrd.compare("Hi", "there")           // -1
strlenOrd.compare("Goodbye", "friend")     // 1
```

### Min, max, clamp, lt, geq, between
Take the smaller (min) or larger (max) element of two, or take the one closest to the given boundaries (clamp).
```typescript
ord.min(ord.ordNumber)(5, 2)                      // 2
ord.max(ord.ordNumber)(5, 2)                      // 5

ord.clamp(ord.ordNumber)(3, 7)(2)                 // 3
ord.clamp(ord.ordString)("Bar", "Boat")("Ball")   // Bar

ord.lt(ord.ordNumber)(4, 7)                       // true
ord.geq(ord.ordNumber)(6, 6)                      // true

ord.between(ord.ordNumber)(6, 9)(7)               // true
ord.between(ord.ordNumber)(6, 9)(6)               // true
ord.between(ord.ordNumber)(6, 9)(9)               // true
ord.between(ord.ordNumber)(6, 9)(12)              // false
```

### Sort an array
```typescript
import { array, ord } from "fp-ts"

const sortByNumber = array.sort(ord.ordNumber)
sortByNumber([3, 1, 2])                           // [1, 2, 3]
```
```typescript
import { array, ord } from "fp-ts"

type Planet = {
  name: string
  diameter: number // km
  distance: number // AU from Sun
}

const planets: Array<Planet> = [
  { name: "Earth", diameter: 12756, distance: 1 },
  { name: "Jupiter", diameter: 142800, distance: 5.203 },
  { name: "Mars", diameter: 6779, distance: 1.524 },
  { name: "Mercury", diameter: 4879.4, distance: 0.39 },
  { name: "Neptune", diameter: 49528, distance: 30.06 },
  { name: "Saturn", diameter: 120660, distance: 9.539 },
  { name: "Uranus", diameter: 51118, distance: 19.18 },
  { name: "Venus", diameter: 12104, distance: 0.723 },
]

const diameterOrd = ord.contramap((x: Planet) => x.diameter)(ord.ordNumber)
const distanceOrd = ord.contramap((x: Planet) => x.distance)(ord.ordNumber)

console.log(array.sort(distanceOrd)(planets))   // Mercury, Venus, Earth, Mars, ...
console.log(array.sort(diameterOrd)(planets))   // Mercury, Mars, Venus, Earth, ...
```

### More Ord instances
Many data types provide Ord instances.

#### Option
```typescript
import { option, ord } from "fp-ts"

const O = option.getOrd(ord.ordNumber) 
O.compare(option.none, option.none)         // 0
O.compare(option.none, option.some(1))      // -1
O.compare(option.some(1), option.none)      // 1
O.compare(option.some(1), option.some(2))   // -1
O.compare(option.some(1), option.some(1))   // 0
```

#### Tuple
```typescript
import { ord } from "fp-ts"

const O = ord.getTupleOrd(ord.ordString, ord.ordNumber)
O.compare(["A", 10], ["A", 12])    // -1
O.compare(["A", 10], ["A", 4])     // 1
O.compare(["A", 10], ["B", 4])     // -1
```


## IO
IO<A> represents a non-deterministic synchronous computation that can cause side effects, yields a value of type A and never fails.

Example:
```typescript
import { io } from 'fp-ts'

const random: io.IO<number> = () => Math.random()
random() // random number
```
```typescript
const getItem = (key: string): io.IO<option.Option<string>> => {
  return () => option.fromNullable(localStorage.getItem(key))  // Synchronous side effect
}
```


## IOEither
IOEither<E, A> represents a synchronous computation that either yields a value of type A or fails yielding an error of type E.

Example:
```typescript
import * as fs from 'fs'
import { ioEither } from 'fp-ts'

const readFileSync = (path: string): ioEither.IOEither<Error, string> => {
  return ioEither.tryCatch(
    () => fs.readFileSync(path, "utf8"),
    (reason) => new Error(String(reason))
  ) // Synchronous side effect
}
```

## Misc
```typescript
import { constVoid } from "fp-ts/function" // A thunk that returns always void.
```

### MonoidAny & MonoidAll
```typescript
import { MonoidAny, MonoidAll } from 'fp-ts/lib/boolean'

const heightAndWidthOk = (image): boolean => MonoidAll.concat(
  image.height >= 42,
  image.width >= 42,
)
heightAndWidthOk({height: 10, width: 100}) // false
heightAndWidthOk({height: 100, width: 100}) // true

const heightOrWidthOk = (image): boolean => MonoidAny.concat(
  image.height >= 42,
  image.width >= 42,
)
heightOrWidthOk({height: 10, width: 10}) // false
heightOrWidthOk({height: 10, width: 100}) // true
```

## SequenceT
Tuple sequencing, i.e., take a tuple of monadic actions and does them from left-to-right, returning the resulting tuple.

Example:
#### Option
```typescript
import { sequenceT } from 'fp-ts/lib/Apply'
import * as O from 'fp-ts/lib/Option'
import { constVoid } from 'fp-ts/lib/function'

const printInfo = (title?: string, description?: string) => 
  pipe(
    sequenceT(O.option)(O.fromNullable(title), O.fromNullable(description)),
    O.fold(
      constVoid,
      ([passedTitle, passedDescription]) => console.log(`Title: ${passedTitle}. Description: ${passedDescription}`),
    ),
  )()

printInfo()
```
#### Task
```typescript
import * as T from 'fp-ts/lib/Task'
import { sequenceT } from 'fp-ts/lib/Apply'
import { pipe } from 'fp-ts/lib/pipeable'

pipe(
  sequenceT(T.task)(T.of(42), T.of("tim")),
  T.map(([answer, name]) => console.log(`Hello ${name}! The answer you're looking for is ${answer}`))
)()
```

#### TaskEither
```typescript
import { sequenceT } from 'fp-ts/lib/Apply'
import * as TE from 'fp-ts/lib/TaskEither'

pipe(
  sequenceT(TE.taskEither)(TE.left("no bad"), TE.right("tim")),
  TE.map(([answer, name]) => console.log(`Hello ${name}! The answer you're looking for is ${answer}`)),
  TE.mapLeft(console.error)
)()

```
