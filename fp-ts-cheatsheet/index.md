
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [fp-ts cheatsheet](#fp-ts-cheatsheet)
  - [pipe](#pipe)
  - [flow](#flow)
  - [Options](#options)
    - [O.map](#omap)
    - [O.fromNullable](#ofromnullable)
    - [O.flatten](#oflatten)
    - [O.chain](#ochain)
  - [Task](#task)
  - [Either](#either)
    - [E.Either](#eeither)
    - [E.left and E.right](#eleft-and-eright)
    - [E.map](#emap)
  - [TaskEither](#taskeither)
    - [TE.tryCatch](#tetrycatch)
    - [TE.map](#temap)
    - [TE.fold](#tefold)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# fp-ts cheatsheet

## pipe
The `pipe` operator can be used to chain a sequence of functions, from left to right.

The return type of a preceding function in the pipeline must match the input type of the subsequent function.

Example:
```
import { pipe } from 'fp-ts/lib/function'

function add1(num: number): number {
  return num + 1
}

function multiply2(num: number): number {
  return num * 2
}

pipe(1, add1, multiply2) // 4
```

## flow
The `flow` operator is similar to `pipe` but the first argument is a function and not the starting value of the chain.

Example:
```
import { flow } from 'fp-ts/lib/function'

pipe(1, flow(add1, multiply2, toString))
flow(add1, multiply2, toString)(1) // this is equivalent
```

## Options
**Options** are containers that wrap values that could be either `undefined` or `null`. If the value exists, we say the Option is of the `Some` type. If the value is undefined or null, we say it has the `None` type.

The Option type is a discriminated union of `None` and `Some`.

```
type Option<A> = None | Some<A>
```

From now on, the Option container will be referred to using the "O" shorthand.
Some and None types can be accessed as `O.Some` and `O.None`

Option provide a wide set of tools.

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
A task is a function that returns a promise which is expected to never reject():

```
type Task<A> = () => Promise<A>
```

## Either
 `Either`  is another fp-ts container (just like Option) that's basically  a *synchronous* operation that can succeed or fail.

```
type Either<E, A> = Left<E> | Right<A>
```
Eithers are very useful for catching error scenarios in FP. With an analogy, they can be considered as the try-catch construct equivalent of functional programming, but with type safe checking.

### E.Either
The E.Either generic type expresses the Either concept and it's used like this:

```
const passwordMinLenght: Either<ShortPasswordError, Password> = ...
```

### E.left and E.right
It's the error branch of an Either. Implementing a check in FP and returning E.left basically means "Error!":
```
const passwordMinLenght = (pass, minLen) => (
  (pass.length < minLen)
    ? E.left(new Error('Password too short')
    : E.right(pass)
)
```
### E.map
Similar to O.map but applied to an Either container.

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