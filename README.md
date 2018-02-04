# RxJS-If-Else

`if/else` statements are a staple for handling conditional actions. It's natural for most developers to reach for the `if/else` statement at the point when a decision needs to be made in code. In the reactive programming paradigm (eg with RxJS) this conditional statements is mysteriously unavailable. How can you code without it? 

The trick, more streams. But more on that later, first let's walk through an example.  Let's say we're writing a transit app. Today we're writing in a new feature that checks if the next street cars is pet friendly. I'll start with an example in RxJS that walks a thin line between reactive functional programming and imperative programming (the kind full of `if/else` statements). Then we'll clean it up with the "more streams" trick and find out how to do away with if/else statements and why RxJS doesn't offer a dedicated if/else operator (as of RxJS 5) in the first place.

### New feature: Is this train pet friendly?
As a user of the app, I need to know if the next train is pet friendly.  When I click the "Get Next Train" button, a message with details include pet info should be displayed.



#### TrainApiService
We'll write the feature against this TrainApiService class that has 2 methods. The first is `getNextTrain()` which returns train details (name, id and remaining minuets until arrival).  And the second being `isPetFriendly()`, which takes a train id and return true or false.

```typescript
interface iTrainDetails {
  readonly id: number;
  readonly name: string;
  readonly minuets: number;
}

class TransitApiService {
  getNextTrain(): iTrainDetails { }
  isPetFriendly(trainId): boolean { }
}

```
Leveraging this train API service, let's write out a stream that subscribes to button click events and returns details about the next train. The details need to include whether or not the next trian is pet friendly.

```javascript
const nextTrainButtonClicks$ = Rx.Observable
	.fromEvent(button, 'click')
	.share();
```

```javascript
// show train details at each click
nextTrainButtonClicks$
  .map(trainApiServie.getNextTrian)
  .map((train) => {
  	let messageDetials;
  	if (trainApiService.isPetFriendly(train.id)) {
  	  messageDetails = `${train.name} is coming in ${train.minuets} minuet(s). This train is pet friendly.
  	} else {
  	  messageDetails = `${train.name} is coming in ${train.minuets} minuet(s). This train is not pet friendly.
  	}
  	return messageDetails
  })
  .do(ui.showTrainDetails)
  .subscribe()

```
It's not terribly difficult to see what's going on here:

- each click is emitted in a stream
- each click is mapped to train details of the nexts train (`.map(trainApiServie.getNextTrian)`)
- the next `.map()` goes into a bit of conditional logic to check if the next train is pet friendly with `if (trainApiService.isPetFriendly(train.id))`.  Depending on the condition, a message is returned with the necessary pet information.
- the `.do(ui.showTrainDetails)` takes the previouse message and updates the ui in the app. 

What's interesting here is that this stream has a branching condition in the middle: 

`if (trainApiService.isPetFriendly(train.id))`

But, regardless of the conditional outcome, it ultimately ends up displaying a message : 

`.do(ui.showTrainDetails)`

What if, instead of only displaying a final message.  The outcome of the condition would _also_ have to determin if a "pet freindly icon" was displayed in the ui:
`ui.showPetIcon()`. This means the stream of clicks would have to end with two different outcomes. That might look something like this:

```javascript
// show train details at each click
nextTrainButtonClicks$
  .map(trainApiServie.getNextTrian);
  .map((train) => {
  	let messageDetials;
  	const isPetFriendly = trainApiService.isPetFriendly(train.id)
  	if (isPetFriendly) {
  	  messageDetails = `${train.name} is coming in ${train.minuets} minuet(s). This train is pet friendly.
  	} else {
  	  messageDetails = `${train.name} is coming in ${train.minuets} minuet(s). This train is not pet friendly.
  	}
  	return {
  	  petFriendly: isPetFriendly,
  	  trainDetails: messageDetails,
  	};
  })
  .do((trainMessage) => {
    ui.showTrainDetails(trainMessage);
    if (trainMessage.petFriendly) {
      ui.showPetIcon()
    }
  })
  .subscribe()

```
At this point it's getting tough to follow what's going on. Where using a conditional statement to build up the train details message. And _another_ conditional statement inside the final `do()` operator to update the ui with the pet icon.  There's also a few new variable we have to keep track of in order to achivie the branching logic:

- `let messageDetials;`
- `const isPetFriendly = trainApiService.isPetFriendly(train.id);`

There's a lot to keep in mind while writing out a stream like this. The big take-away here is that you shouldn't have to.  Rx streams can be very readable if you approach them differently. 

Let's dig into what makes up a condition for a moment and then come back to our app. 


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

How can we branch to the else portion of this condition with the filter operator?  You cannot, but it's okay, instead you can break the statement into multiple streams. One for each branch of the condition. Then compose them together with a `merge` operator.

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
   

`if` statements, however, don't end there, there's also the `else if` statement. 

```javascript
if (isSomething)  {
  something()
} else if (isBetterThings) {
  betterThings()
} else {
  defaultThing()
}
```

This essential translates to more branches. By following that same approach as before in the RxJS way, we can break each branch into it's own stream and merge them all together at the end. 

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
  .do(correctThings);
```

The beauty here is that the final stream is just a composition of the things the developer is after. Compared to the imperative version (`if/else`), this one reads like a natural conversation. I don't need to bother considering the other streams. I can just zero in on the what the final steam is composed of and assume the goal based on the verbiage.

"I want `somethings$`, `betterThings$` and `defaultThings$`" 

The imperative version, on the other hand, will cost more mental overhead to read through as we saw before in our transit app.  At each branch of the `if` statements you'll have to mentally process the condition line:

`if(something !== somethingElse) {`

Before you can read into the action:

`doSomething()` 

### Merge

Now that we're armed with this pattern, refactoring out that conditional statements from our transit app should be a snap. Let's convert each branch of our condition and then compose them together. 

We'll create a source stream of clicks that map to "next-trains". Then we'll build a branch for pet friendly trains and one for non pet friendly trains.  Lastly we'll merge them together and call it a day. 


```javascript
// source stream of clicks-mapped-to-trains
const nextTrains$ = nextTrainButtonClicks$
  .map(trainApiServie.getNextTrian)
  .share();
  
// branch stream of pet friendly trains
const petFriendlyTrains$ = nextTrains$
  .filter((train) => trainApiService.isPetFriendly(train.id))
  .map((train => `${train.name} is coming in ${train.minuets} minuet(s). This train is pet friendly.`)
  .do(ui.showTrainDetails)
  .do(ui.showPetIcon);
  
// branch stream of non pet friendly trains
const nonPetFriendlyTrains$ = nextTrains$
  .filter((train) => !trainApiService.isPetFriendly(train.id))
  .map((train => `${train.name} is coming in ${train.minuets} minuet(s). This train is not pet friendly.`)
  .do(ui.showTrainDetails);

// merge then all together  
Rx.Observable.of('')
  .merge(
    petFriendlyTrains$
    nonPetFriendlyTrains$
  )
  .subscribe();

```


