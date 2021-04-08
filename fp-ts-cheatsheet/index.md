<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of contents**

- [fp-ts cheatsheet](#fp-ts-cheatsheet)
  - [pipe](#pipe)
  - [flow](#flow)

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
