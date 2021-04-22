
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [fp-ts cheatsheet](#fp-ts-cheatsheet)
  - [pipe & flow](#pipe--flow)
  - [Options](#options)
    - [O.map](#omap)
    - [O.fromNullable](#ofromnullable)
    - [O.flatten](#oflatten)
    - [O.chain](#ochain)
  - [Task](#task)
  - [Either](#either)
    - [E.map](#emap)
    - [E.tryCatch](#etrycatch)
  - [TaskEither](#taskeither)
    - [TE.tryCatch](#tetrycatch)
    - [TE.map](#temap)
    - [TE.fold](#tefold)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# fp-ts cheatsheet

## pipe & flow
The `pipe` operator can be used to chain a sequence of functions, from left to right.

The return type of a preceding function in the pipeline must match the input type of the subsequent function.

The `flow` operator is similar to `pipe` but the first argument is a function and not the starting value of the chain.

Example:
```
import { pipe, flow } from 'fp-ts/lib/function'

const add1 = (x: number) => x + 1
const multiply2 = (x: number) => x * 2

// The following are equivalent
pipe(1, add1, multiply2)        // 4
pipe(1, flow(add1, multiply2))  // 4    
flow(add1, multiply2)(1)        // 4 
```

## Options
**Options** are containers that wrap values that could be either `undefined` or `null`. If the value exists, we say the Option is of the `Some` type. If the value is undefined or null, we say it has the `None` type.

The Option type is a discriminated union of `None` and `Some`.

```
type Option<A> = None | Some<A>
```

In most cases, you won't need to use Option; optional chaining is less verbose.  
But the Option type is more than just checking for null. Option can be used to represent failing operations. And just like how you can lift undefined into an Option, you can also lift an Option into another fp-ts container, like Either.

From now on, the Option container will be referred to using the "O" shorthand.
Some and None types can be accessed as `O.Some` and `O.None`

Option provide a wide set of tools.

Example:
```
O.some(42)    // { _tag: 'Some', value: 42 } 
O.none        // { _tag: 'None' }
```

### O.map
The map operator allows to access and transform one value from another:
```
pipe(
  { bar: 'hello' },
  O.fromNullable,
  O.map(({ bar }) => bar),
) // { _tag: 'Some', value: 'hello' }
```
```
pipe(
  undefined,
  O.fromNullable,
  O.map(({ bar }) => bar),
) // { _tag: 'None' }
```

### O.fromNullable
`O.fromNullable` creates a container object tagged as None or Some, depending on the value of its argument being null/undefined or not.

```
O.fromNullable('Existing')    // { _tag: 'Some', value: 'Existing' } 
O.fromNullable(null)          // { _tag: 'None' }
```

### O.flatten
The O.flatten operator allows to traverse nested pipes to access an inner property of an object:
```
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
The O.chain operator implements a flatmap, merging the concepts behind  O.map and O.flatten:
```
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

## Task
A task is a function that returns a promise which is expected to never reject().

Tasks are expected to always succeed but can fail when an error occurs outside our expectations. In this case, the error is thrown and breaks the functional pipeline. An analogy to this is awaiting a Promise that throws an error without putting a try-catch-finally block in front.    
**Test your assumptions before using Task.**

A Task is more than a glorified promise; it is also an expression of intent.
A Promise provides no indication about whether the function can fail. As such, in the imperative model, you are forced to handle these errors using a try-catch-finally block.

```
type Task<A> = () => Promise<A>
```

Example:
```
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

```
type Either<E, A> = Left<E> | Right<A>
```

Eithers are very useful for catching error scenarios in FP. With an analogy, they can be considered as the try-catch construct equivalent of functional programming, but with type safe checking.  
We need the Eithers because we cannot break pipelines by throwing errors.

Example:  
```
const passwordMinLenght = (pass: string, minLen: number): E.Either<Error, string> => (
  (pass.length < minLen)
    ? E.left(new Error('Password too short'))
    : E.right(pass)
)

passwordMinLenght('test', 1)    // {_tag: "Right", right: "test"}
passwordMinLenght('test', 8)    // {_tag: "Left", left: Error}
```

### E.map
Similar to `O.map` but applied to an Either container.


### E.tryCatch
Constructs a new Either from a function that might throw.

Example:
```
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

### E.fromOption
Construct a new Either from an Option.

```
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

```
const check = (
  E.fromPredicate(
    (n: number) => n > 0,
    () => 'error'
  )
)

check(42) // {_tag: "Right", right: 42}
check(-1) // {_tag: "Left", left: "error"}
```


## TaskEither
`TaskEither` merges together the concepts of Taks and Either, applied to the asynchronous scenarios, e.g. network calls of an application.

A TaskEither can be basically seen as the wrapper for the construct "then / catch" in FP (and everything in the middle).
Indeed, in fp-ts there is TE.tryCatch.

### TE.tryCatch

```
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

### TE.map
Same concept of O.map and E.map, only applied to TaskEithers

### TE.fold
Placed at the end of a TaskEither pipe, in a condition of success, returns a Task.
```
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