
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
    - [O.getOrElse](#ogetorelse)
  - [Task](#task)
  - [Either](#either)
    - [E.map](#emap)
    - [E.tryCatch](#etrycatch)
    - [E.fromOption](#efromoption)
    - [E.fromPredicate](#efrompredicate)
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


### O.getOrElse
Extracts the value out of the structure, if it exists. Otherwise returns the given default value.

Example:
```
const sillyFunction = (x?: string) => pipe(
  O.fromNullable(x),
  O.getOrElse(() => "error")
)

sillyFunction(undefined)  // error
sillyFunction("ok")       // ok
```

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

## Eq
The Eq type class represents types which support decidable equality.

Instances must satisfy the following laws:
- Reflexivity: `E.equals(a, a) === true`
- Symmetry: `E.equals(a, b) === E.equals(b, a)`
- Transitivity: `if E.equals(a, b) === true and E.equals(b, c) === true, then E.equals(a, c) === true`


### Primitive equality
```
import { eq } from "fp-ts"

eq.eqBoolean.equals(true, true)                                    // true
eq.eqDate.equals(new Date("1984-01-27"), new Date("1984-01-27"))   // true
eq.eqNumber.equals(3, 3)                                           // true
eq.eqString.equals("Cyndi", "Cyndi")                               // true
```

### Compare structures
```
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
```
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
```
import { array, eq } from "fp-ts"

const eqArrayOfStrings = array.getEq(eq.eqString)

eqArrayOfStrings.equals(["Time", "After", "Time"], ["Time", "After", "Time"]) // true
```
```
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
```
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
```
import { eq, option } from "fp-ts"

const E = option.getEq(eq.eqNumber)

E.equals(option.some(3), option.some(3))  // true
E.equals(option.none, option.some(4))     // false
E.equals(option.none, option.none)        // true
```

#### Either
```
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
```
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
```
const strlenOrd = ord.fromCompare((a: string, b: string) =>
  a.length < b.length ? -1 : a.length > b.length ? 1 : 0
)
strlenOrd.compare("Hi", "there")          // -1
strlenOrd.compare("Goodbye", "friend")    // 1
```
But most of the time, you can achieve the same result in a simpler way with contramap:
```
const strlenOrd = ord.contramap((s: string) => s.length)(ord.ordNumber)
strlenOrd.compare("Hi", "there")           // -1
strlenOrd.compare("Goodbye", "friend")     // 1
```

### Min, max, clamp, lt, geq, between
Take the smaller (min) or larger (max) element of two, or take the one closest to the given boundaries (clamp).
```
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
```
import { array, ord } from "fp-ts"

const sortByNumber = array.sort(ord.ordNumber)
sortByNumber([3, 1, 2])                           // [1, 2, 3]
```
```
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
```
import { option, ord } from "fp-ts"

const O = option.getOrd(ord.ordNumber) 
O.compare(option.none, option.none)         // 0
O.compare(option.none, option.some(1))      // -1
O.compare(option.some(1), option.none)      // 1
O.compare(option.some(1), option.some(2))   // -1
O.compare(option.some(1), option.some(1))   // 0
```

#### Tuple
```
import { ord } from "fp-ts"

const O = ord.getTupleOrd(ord.ordString, ord.ordNumber)
O.compare(["A", 10], ["A", 12])    // -1
O.compare(["A", 10], ["A", 4])     // 1
O.compare(["A", 10], ["B", 4])     // -1
```

