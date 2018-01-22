# RxJS-If-Else

`if/else` statements are a staple for handling conditional actions. It's natural for most developers to reach for the `if/else` statement at the point when a decision needs to be made in code. In the reactive programming paradigm (eg with RxJS) this conditional statement is mysteriously unavailable. How can you code without it? 

The trick is more streams. What's a stream? More on that later. First let's walk through an example of (...).  Let's say we're writing a transit app. We need to write a piece of code that determines which streetcar will arrive sooner. I'll start with an example in RxJS that walks a thin line between reactive functional programming and imperative programming (the kind full of `if/else` statements that you're probably used to seeing). Then we'll clean it up with the "more streams" trick and find out how to do away with if/else statements and why RxJS (as of RxJS 5) doesn't offer a dedicated if/else operator in the first place.

[Imperative example]

[Add notes, discuss what's going on]

### More streams
It's simple to cut out the `if` statement when you approach the problem in smaller pieces. Consider this `if` statement:

```javascript
if (isSomething)  {
  something()
}
```

The `something()` on line 2 wants to be called, but it's gated off by the `if (isSomething)` on line 1. In other words, if statements (line 1) act a lot like filters. The RxJS version is just that:

```javascript
source$
  .filter(isSomething)
  .do(something);
```

But the topic here is `if/else` not just `if`. 

```javascript
if (isSomething)  {
  something()
} else {
  aDifferentThing()
}
```

We can't branch to the else portion of this condition with the filter operator, but it's okay. Instead, you can break the statement into multiple streams, one for each branch of the condition then compose them together with a `merge` operator.

```javascript
const somethings$ = source$
  .filter(isSomthing)
  .do(something):

const differentThings$ = source$
  .filter(!isSomthing)
  .do(aDifferentThing):

// merge them together
const onlyTheRightThings$ = somethings$
  .merge(differentThings$)
  .do(correctThings)
```
   

What about the `else if` statement?

```javascript
if (isSomething)  {
  something()
} else if (isBetterThings) {
  betterThings()
} else {
  defaultThing()
}
```

This essentially translates to more branches. By following the RxJS approach, we can break each branch into its own stream and merge them all together at the end. 

```javascript
const somethings$ = source$
  .filter(isSomthing)
  .do(something);

const betterThings$ = source$
  .filter(isBetterThings)
  .do(betterThings);

const defaultThings$ = source$
  .filter((val) => !isSomthing(val) && !isBetterThings(val))
  .do(defaultThing);

// merge them together
const onlyTheRightThings$ = somethings$
  .merge(
    betterThings$,
    defaultThings$,
  )
  
  // I think you need a correctThings variable
  .do(correctThings);
```

The beauty here is that the final stream is just a composition of the things the developer is after. Compared to the imperative version (`if/else`), this one reads like a natural conversation. I don't need to bother considering the other streams. I can just zero in on the what the final steam is composed of and assume the goal based on the verbiage.

"I want `somethings$`, `betterThings$` and `defaultThings$`" 

The imperative version, on the other hand, will cost more mental overhead to read through.  At each branch of the `if` statements you'll have to mentally process the condition line:

`if(something !== somethingElse) {`

Before you can read into the action:

`doSomething()` 

[needs a closing statement for this portion]

Now that we're armed with this pattern, refactoring out that conditional statements from the transit app should be a snap. Let's convert each branch of our condition and then compose them together. The conditions are:

1. When the Dundas 505 is closer
2. When the College 506 is closer




