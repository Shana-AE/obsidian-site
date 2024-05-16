  
[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2Fplans%3Fdimension%3Dpost_audio_button%26postId%3D4c2739326f57&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Ffunctional-dependency-injection-in-typescript-4c2739326f57&source=-----4c2739326f57---------------------post_audio_button-----------)

![](https://miro.medium.com/v2/resize:fit:700/1*qZpCgNmezGUY446592L4pw.jpeg)

Photo by [Robert Katzki](https://unsplash.com/@ro_ka?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/wallpapers/colors?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

> For discussion, check this [Reddit thread](https://www.reddit.com/r/programming/comments/ufv8s0/functional_dependency_injection_in_typescript/?utm_source=share&utm_medium=web2x&context=3).

## What is Dependency Injection?

Dependency Injection (DI) is the separation of the logic of a unit of code from its dependencies. **Functional dependency injection (FDI)** is a way of modeling and implementing DI using an FP approach. It simply denotes modeling the dependencies of a function as parameters and passing(injecting) those dependencies as arguments.

## But What Is A Dependency Really?

“Dependency” is a somewhat vague term, which can be the most confusing part about DI. I mean, what the hell is a dependency? Is a logger instance in the body of a function a dependency? Maybe the database query used to fetch the user’s data that is crucial to the function’s output? And what about the utility function imported from some library and used in the body of a function?.

Loggers and DB-queries can always be considered as dependencies for any function. Utility functions? well, is the utility function pure (given the same input it always returns the same output on every call AND has no side effects)? if yes then it’s not a dependency, otherwise it is.

Here are a couple of tips to spot a dependency:

- ==If you need to use something like Jest’s “spy” method to fake a function call inside the function you’re testing, you’re looking at a dependency.==
- If something your function depends on behaves differently or has different configurations between different environments, sign it up for the dependency party.
- If something that your function depends on has side effects; it’s a dependency.

## Why Does Dependency Injection Matter?

A function that is structured in a DI-manner is _dependency transparent_ meaning that it explicitly defines the components that are dependencies without having you––The caller––dig into the implementation of that function. This leads to a high-level understanding of what side effects a function might be performing, and more importantly, it makes unit testing a straightforward task.

To understand these benefits, we’ll start off with a code that doesn’t use DI, and then we’ll counter that with one that does use it.

Let’s imagine that we have a voting module for voting on issues. Our module exposes a function called `voteOnIssue` . The function takes as input:

1. a `userId`
2. An `issueId`
3. A `vote` which has two possible values upvote and downvote.

The function needs to do some validation before it accepts the vote, it checks for the following:

1. The issue is still open for voting.
2. The user has not already voted on the issue.
3. The user is eligible to vote on the issue. For example, voters might be required to be at least 18.

Here’s an initial implementation of the function:

![](https://miro.medium.com/v2/resize:fit:700/1*WsXepFDjoe5y1DPcRQdICQ.png)

This function has the following dependencies:

1. `queryGetIssueData`: gets the issue’s data from our database and returns it.
2. `queryHaveUserVoted`: checks if a user has voted on a particular issue, it’s a promise that when resolved returns a boolean value.
3. `checkUserIsEligibleToVote`: checks if the user is eligible to vote on the issue.
4. `queryInsertVote`: inserts the vote into the database.
5. `logger`: a logger.

Now, how do we go about unit-testing this function? obviously `voteOnIssue` is an impure function with side effects, so we need to create test doubles for its dependencies in our tests.

Let’s create a test file to test `voteOnIssue`:

![](https://miro.medium.com/v2/resize:fit:700/1*J_mnEwRKqYkFBhrUSQmVjw.png)

Here are a couple of downsides to the way `voteOnIssue` is structured and this approach to unit-testing:

1. `voteOnIssue` is **not dependency transparent,** and anyone writing tests for this function has to dig into its implementation to figure out its dependencies and create the necessary test doubles, which is time-consuming.
2. The test is tightly coupled with the implementation of the function. For example, let’s say that we decided to switch out a _deprecated_ dependency `queryGetIssueData` for a new one called `queryGetIssueById`, next time you run the test it will fail since it’s not spying on the correct dependency anymore. Jest won’t be able to rescue you here because it doesn’t know about the implementation of your function.
3. Testing `voteOnIssue` requires us to use **spies** which entails a cognitive overhead for the junior developer that has just joined your team.
4. Things become a bit more complex when the function you’re testing and any of its dependencies reside in the same module. Take a look at [this](https://stackoverflow.com/questions/51269431/jest-mock-inner-function#:~:text=If%20an%20ES6%20module%20directly%20exports%20two%20functions%20(not%20within%20a%20class%2C%20object%2C%20etc.%2C%20just%20directly%20exports%20the%20functions%20like%20in%20the%20question)%20and%20one%20directly%20calls%20the%20other%2C%20then%20that%20call%20cannot%20be%20mocked.) on Stackoverflow. Again, think of the implication of this on junior developers.

## Implementing DI Using High Order Functions

Now we’re going to reimplement `voteOnIssue` so that it uses FDI (Functional Dependency Injection):

![](https://miro.medium.com/v2/resize:fit:700/1*PPx_55DayaLR4SxA0S637g.png)

![](https://miro.medium.com/v2/resize:fit:700/1*nD-ZSicr2OSxOSRM_OFDLQ.png)

`voteOnIssue` is now a HOF ([High-order function](https://en.wikipedia.org/wiki/Higher-order_function)) that first takes its dependencies and returns a new function that takes the actual input from the domain perspective: `userId`, `issueId`, and `vote`.

## Unit testing Our DI-code

Now it’s time to reap the benefits. Let’s create a test file(I’m using Jest here) and test our `voteOnIssue` function:

![](https://miro.medium.com/v2/resize:fit:700/1*QQx5BDWDrQV7yAeeUQtTEQ.png)

![](https://miro.medium.com/v2/resize:fit:700/1*mbTHpoFkUA1918t7QxQsow.png)

Here are the benefits of our new implementation of `voteOnIssue` and the implications of that on our unit-tests:

1. We easily hand-crafted our stubs without having to use spies, just simple functions. Who doesn’t know how to create them?
2. Since `voteOnIssue` is now dependency transparent and we’re passing these dependencies as arguments, we won’t run into the issues that we would run into when we use spies to test functions used as a dependency (like in the case of swapping out a dependency with another one).
3. This model is quite simple. No complex scenarios like how do we deal with spying on a function that resides in the same module as the function we’re testing.

> For discussion, check this [Reddit thread](https://www.reddit.com/r/programming/comments/ufv8s0/functional_dependency_injection_in_typescript/?utm_source=share&utm_medium=web2x&context=3).

## A Two-Layer Module Structure

I’m quite certain that at some point while demonstrating how we can use high order functions to implement DI, you were thinking something along the lines of “that’s neat, but I don’t want to keep passing around my dependencies every single time I use this function.”. And I wholeheartedly agree with you; we’re introducing noise in the codebase as the consumers of those functions are not interested in knowing about those dependencies. They just want to give it the input and get the output.

So how do we get around this?

The solution is to introduce a two-layer structure to your modules. One layer contains the uninjected code, and the other contains the code injected with dependencies. You use the uninjected layer (I usually call it logic) in your tests, and you use the injected one in your application (I typically call it index.ts).

![](https://miro.medium.com/v2/resize:fit:700/1*GjJlhq8RTrgPgb1lg7qIRA.png)

Here’s an implementation that follows the structure for the queries module:

![](https://miro.medium.com/v2/resize:fit:2156/1*nD1Lvktf9wGxnhHdr0n3hA.png)

![](https://miro.medium.com/v2/resize:fit:2300/1*LcZxoxr63cV1k5rEFb18kg.png)

As you can see, in `queries/index.ts` we’re importing the functions that we want to use in the rest of our application from the uninjected layer `queries/logic.ts` injecting them with their dependencies and then re-exporting them.

## Wrap up

- Dependency Injection is not exclusive to OOP and classes; it can be easily implemented in FP using HOFs.
- DI-structured functions are dependency transparent. A function is dependency-transparent when it explicitly defines its dependencies.
- Adding a two-layer module structure can smooth out the verbosity of passing around dependencies.
- Structuring functions in the way the article suggests makes unit-testing a straightforward task.
- There are better ways than using plain promises and throwing errors to model async operations, good candidates for this would be `Task/AsyncEither`, but I’ll leave that to another article.
- You could use the `ReaderMonad` to handle dependency injection, but I find HOFs to be doing a good job without introducing the concept of Monads.

**_Last updated: 2022–05–15_**

[

Typescript

](https://medium.com/tag/typescript?source=post_page-----4c2739326f57---------------typescript-----------------)

[

Dependency Injection

](https://medium.com/tag/dependency-injection?source=post_page-----4c2739326f57---------------dependency_injection-----------------)

[

Functional Programming

](https://medium.com/tag/functional-programming?source=post_page-----4c2739326f57---------------functional_programming-----------------)

[

Test Driven Development

](https://medium.com/tag/test-driven-development?source=post_page-----4c2739326f57---------------test_driven_development-----------------)

[

Unit Testing

](https://medium.com/tag/unit-testing?source=post_page-----4c2739326f57---------------unit_testing-----------------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F4c2739326f57&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Ffunctional-dependency-injection-in-typescript-4c2739326f57&user=Hugo+Nteifeh&userId=93bfcf01399c&source=-----4c2739326f57---------------------clap_footer-----------)

445

5

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F4c2739326f57&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Ffunctional-dependency-injection-in-typescript-4c2739326f57&source=--------------------------bookmark_footer-----------)

[

![Hugo Nteifeh](https://miro.medium.com/v2/resize:fill:72:72/1*FjXKZ-ABx7tKR_Vz9BXy5Q.jpeg)





](https://hugonteifeh.medium.com/?source=post_page-----4c2739326f57--------------------------------)

[

## Written by Hugo Nteifeh

](https://hugonteifeh.medium.com/?source=post_page-----4c2739326f57--------------------------------)

[89 Followers](https://hugonteifeh.medium.com/followers?source=post_page-----4c2739326f57--------------------------------)

Software engineer with an interest in linguistics, psychology, and politics.

Follow

[](https://medium.com/m/signin?actionUrl=%2F_%2Fapi%2Fsubscriptions%2Fnewsletters%2F976e03ed4e27&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Ffunctional-dependency-injection-in-typescript-4c2739326f57&newsletterV3=93bfcf01399c&newsletterV3Id=976e03ed4e27&user=Hugo+Nteifeh&userId=93bfcf01399c&source=-----4c2739326f57---------------------subscribe_user-----------)

## More from Hugo Nteifeh

[

![Polymorphism in Typescript](https://miro.medium.com/v2/resize:fit:679/1*RNl3M3aPcCk0xaroZuRbXw.jpeg)

](https://hugonteifeh.medium.com/polymorphism-in-typescript-59fad77f8cd7?source=author_recirc-----4c2739326f57----0---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

![Hugo Nteifeh](https://miro.medium.com/v2/resize:fill:20:20/1*FjXKZ-ABx7tKR_Vz9BXy5Q.jpeg)



](https://hugonteifeh.medium.com/?source=author_recirc-----4c2739326f57----0---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

Hugo Nteifeh

](https://hugonteifeh.medium.com/?source=author_recirc-----4c2739326f57----0---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

## Polymorphism in Typescript

### FP and OOP teaming up to fight code redundancy



](https://hugonteifeh.medium.com/polymorphism-in-typescript-59fad77f8cd7?source=author_recirc-----4c2739326f57----0---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

5 min read·Sep 17, 2020

](https://hugonteifeh.medium.com/polymorphism-in-typescript-59fad77f8cd7?source=author_recirc-----4c2739326f57----0---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F59fad77f8cd7&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Fpolymorphism-in-typescript-59fad77f8cd7&user=Hugo+Nteifeh&userId=93bfcf01399c&source=-----59fad77f8cd7----0-----------------clap_footer----86cae378_cb69_4315_a614_4d13f71ea46f-------)

218

[

1

](https://hugonteifeh.medium.com/polymorphism-in-typescript-59fad77f8cd7?responsesOpen=true&sortBy=REVERSE_CHRON&source=author_recirc-----4c2739326f57----0---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F59fad77f8cd7&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Fpolymorphism-in-typescript-59fad77f8cd7&source=-----4c2739326f57----0-----------------bookmark_preview----86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

![Nominal typing in Typescript](https://miro.medium.com/v2/resize:fit:679/1*BwwFQ0CF8Xo1A1XXdtPH8Q.jpeg)

](https://hugonteifeh.medium.com/nominal-typing-in-typescript-c712e7116006?source=author_recirc-----4c2739326f57----1---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

![Hugo Nteifeh](https://miro.medium.com/v2/resize:fill:20:20/1*FjXKZ-ABx7tKR_Vz9BXy5Q.jpeg)



](https://hugonteifeh.medium.com/?source=author_recirc-----4c2739326f57----1---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

Hugo Nteifeh

](https://hugonteifeh.medium.com/?source=author_recirc-----4c2739326f57----1---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

## Nominal typing in Typescript

### Define nominal types within a structural type system



](https://hugonteifeh.medium.com/nominal-typing-in-typescript-c712e7116006?source=author_recirc-----4c2739326f57----1---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

5 min read·Aug 17, 2020

](https://hugonteifeh.medium.com/nominal-typing-in-typescript-c712e7116006?source=author_recirc-----4c2739326f57----1---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2Fc712e7116006&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Fnominal-typing-in-typescript-c712e7116006&user=Hugo+Nteifeh&userId=93bfcf01399c&source=-----c712e7116006----1-----------------clap_footer----86cae378_cb69_4315_a614_4d13f71ea46f-------)

171

[](https://hugonteifeh.medium.com/nominal-typing-in-typescript-c712e7116006?responsesOpen=true&sortBy=REVERSE_CHRON&source=author_recirc-----4c2739326f57----1---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fc712e7116006&operation=register&redirect=https%3A%2F%2Fhugonteifeh.medium.com%2Fnominal-typing-in-typescript-c712e7116006&source=-----4c2739326f57----1-----------------bookmark_preview----86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

![Rosebox: CSS in Typescript](https://miro.medium.com/v2/resize:fit:679/1*XAStjYzuBPNyMbiXiKZhFQ.png)

](https://hugonteifeh.medium.com/rosebox-css-in-typescript-8534afaa7583?source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

![Hugo Nteifeh](https://miro.medium.com/v2/resize:fill:20:20/1*FjXKZ-ABx7tKR_Vz9BXy5Q.jpeg)



](https://hugonteifeh.medium.com/?source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

Hugo Nteifeh

](https://hugonteifeh.medium.com/?source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

in

[

Level Up Coding

](https://levelup.gitconnected.com/?source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

## Rosebox: CSS in Typescript

### Style with functions, objects and IntelliSense



](https://hugonteifeh.medium.com/rosebox-css-in-typescript-8534afaa7583?source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

4 min read·Dec 23, 2020

](https://hugonteifeh.medium.com/rosebox-css-in-typescript-8534afaa7583?source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fgitconnected%2F8534afaa7583&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Frosebox-css-in-typescript-8534afaa7583&user=Hugo+Nteifeh&userId=93bfcf01399c&source=-----8534afaa7583----2-----------------clap_footer----86cae378_cb69_4315_a614_4d13f71ea46f-------)

61

[](https://hugonteifeh.medium.com/rosebox-css-in-typescript-8534afaa7583?responsesOpen=true&sortBy=REVERSE_CHRON&source=author_recirc-----4c2739326f57----2---------------------86cae378_cb69_4315_a614_4d13f71ea46f-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F8534afaa7583&operation=register&redirect=https%3A%2F%2Flevelup.gitconnected.com%2Frosebox-css-in-typescript-8534afaa7583&source=-----4c2739326f57----2-----------------bookmark_preview----86cae378_cb69_4315_a614_4d13f71ea46f-------)

[

See all from Hugo Nteifeh

](https://hugonteifeh.medium.com/?source=post_page-----4c2739326f57--------------------------------)

## Recommended from Medium

[

![Why I Prefer Regular Merge Commits Over Squash Commits](https://miro.medium.com/v2/resize:fit:679/0*Ei5Qo9EduVteZ-AF)

](https://doctorderek.medium.com/why-i-prefer-regular-merge-commits-over-squash-commits-cadd22cff02c?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Dr. Derek Austin 🥳](https://miro.medium.com/v2/resize:fill:20:20/1*Ycih7sit-0tCvHiE2qpACg.jpeg)



](https://doctorderek.medium.com/?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

Dr. Derek Austin 🥳

](https://doctorderek.medium.com/?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

in

[

Better Programming

](https://betterprogramming.pub/?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

## Why I Prefer Regular Merge Commits Over Squash Commits

### I used to think squash commits were so cool, and then I had to use them all day, every day. Here’s why you should avoid squash



](https://doctorderek.medium.com/why-i-prefer-regular-merge-commits-over-squash-commits-cadd22cff02c?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

·5 min read·Oct 1, 2022

](https://doctorderek.medium.com/why-i-prefer-regular-merge-commits-over-squash-commits-cadd22cff02c?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fbetter-programming%2Fcadd22cff02c&operation=register&redirect=https%3A%2F%2Fbetterprogramming.pub%2Fwhy-i-prefer-regular-merge-commits-over-squash-commits-cadd22cff02c&user=Dr.+Derek+Austin+%F0%9F%A5%B3&userId=e5294c417caf&source=-----cadd22cff02c----0-----------------clap_footer----457ba668_d282_47d9_951c_7c42fe8d7324-------)

1.3K

[

59

](https://doctorderek.medium.com/why-i-prefer-regular-merge-commits-over-squash-commits-cadd22cff02c?responsesOpen=true&sortBy=REVERSE_CHRON&source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fcadd22cff02c&operation=register&redirect=https%3A%2F%2Fbetterprogramming.pub%2Fwhy-i-prefer-regular-merge-commits-over-squash-commits-cadd22cff02c&source=-----4c2739326f57----0-----------------bookmark_preview----457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Node.js image](https://miro.medium.com/v2/resize:fit:679/0*gE-9O3BNZKbB6Y4d.png)

](https://vishnucprasad.medium.com/clean-architecture-with-inversify-in-node-js-with-typescript-a-code-driven-guide-8a65e4bd1a48?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Vishnu C Prasad](https://miro.medium.com/v2/resize:fill:20:20/1*Gi6-a8HmXVp6Cbu0E644xw.jpeg)



](https://vishnucprasad.medium.com/?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

Vishnu C Prasad

](https://vishnucprasad.medium.com/?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

in

[

Stackademic

](https://blog.stackademic.com/?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

## Clean Architecture with Inversify in Node.js with TypeScript: A Code-Driven Guide

### Introduction



](https://vishnucprasad.medium.com/clean-architecture-with-inversify-in-node-js-with-typescript-a-code-driven-guide-8a65e4bd1a48?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

3 min read·Aug 8, 2023

](https://vishnucprasad.medium.com/clean-architecture-with-inversify-in-node-js-with-typescript-a-code-driven-guide-8a65e4bd1a48?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fstackademic%2F8a65e4bd1a48&operation=register&redirect=https%3A%2F%2Fblog.stackademic.com%2Fclean-architecture-with-inversify-in-node-js-with-typescript-a-code-driven-guide-8a65e4bd1a48&user=Vishnu+C+Prasad&userId=ca0477e1f4ac&source=-----8a65e4bd1a48----1-----------------clap_footer----457ba668_d282_47d9_951c_7c42fe8d7324-------)

7

[](https://vishnucprasad.medium.com/clean-architecture-with-inversify-in-node-js-with-typescript-a-code-driven-guide-8a65e4bd1a48?responsesOpen=true&sortBy=REVERSE_CHRON&source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F8a65e4bd1a48&operation=register&redirect=https%3A%2F%2Fblog.stackademic.com%2Fclean-architecture-with-inversify-in-node-js-with-typescript-a-code-driven-guide-8a65e4bd1a48&source=-----4c2739326f57----1-----------------bookmark_preview----457ba668_d282_47d9_951c_7c42fe8d7324-------)

## Lists

[

![](https://miro.medium.com/v2/resize:fill:48:48/1*zPtGTCNOwu1p3kzn_sZFVQ.png)

![](https://miro.medium.com/v2/da:true/resize:fill:48:48/0*kQIvhDkl0ixPpv4z)

![](https://miro.medium.com/v2/resize:fill:48:48/1*ERYx0IB1pN-5ZX98cKAoUw.png)

## General Coding Knowledge

20 stories·782 saves



](https://eddiebarth.medium.com/list/general-coding-knowledge-f2d429d4f0cd?source=read_next_recirc-----4c2739326f57--------------------------------)

[

![](https://miro.medium.com/v2/resize:fill:48:48/1*YVn9yVLEQWnGq9wQkd0NuA.png)

![](https://miro.medium.com/v2/resize:fill:48:48/1*h6xHqH2IxgfQ6ugptLOrFA.png)

![](https://miro.medium.com/v2/resize:fill:48:48/1*d7OxX8vQ5XvfToMohqQNJw.png)

## Natural Language Processing

1081 stories·556 saves



](https://medium.com/@AMGAS14/list/natural-language-processing-0a856388a93a?source=read_next_recirc-----4c2739326f57--------------------------------)

[

![What are “decorators” in typescript and how to use “decorators”?](https://miro.medium.com/v2/resize:fit:679/1*qkEn_I6hzFel_nvlrE3Y3g.png)

](https://medium.com/@InspireTech/what-are-decorators-in-typescript-and-how-to-use-decorators-d82d15c5851f?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Hemant](https://miro.medium.com/v2/resize:fill:20:20/1*E_4tjRjksnC5N-6tyImMBA.png)



](https://medium.com/@InspireTech?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

Hemant

](https://medium.com/@InspireTech?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

## What are “decorators” in typescript and how to use “decorators”?

### A Decorator is a special kind of declaration that can be attached to a class declaration, method, accessor, property, or parameter…



](https://medium.com/@InspireTech/what-are-decorators-in-typescript-and-how-to-use-decorators-d82d15c5851f?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

11 min read·Sep 3, 2023

](https://medium.com/@InspireTech/what-are-decorators-in-typescript-and-how-to-use-decorators-d82d15c5851f?source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2Fd82d15c5851f&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40InspireTech%2Fwhat-are-decorators-in-typescript-and-how-to-use-decorators-d82d15c5851f&user=Hemant&userId=eedcb2e9ccc1&source=-----d82d15c5851f----0-----------------clap_footer----457ba668_d282_47d9_951c_7c42fe8d7324-------)

56

[](https://medium.com/@InspireTech/what-are-decorators-in-typescript-and-how-to-use-decorators-d82d15c5851f?responsesOpen=true&sortBy=REVERSE_CHRON&source=read_next_recirc-----4c2739326f57----0---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fd82d15c5851f&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40InspireTech%2Fwhat-are-decorators-in-typescript-and-how-to-use-decorators-d82d15c5851f&source=-----4c2739326f57----0-----------------bookmark_preview----457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Stop Using localStorage](https://miro.medium.com/v2/resize:fit:679/0*ORJjEuRQB57bd_SX.jpg)

](https://medium.com/@julienetienne/stop-using-localstorage-64a6d6805da8?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Julien Etienne](https://miro.medium.com/v2/resize:fill:20:20/1*QADoVQ9X8JvC9DL_2W_LIw.jpeg)



](https://medium.com/@julienetienne?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

Julien Etienne

](https://medium.com/@julienetienne?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

## Stop Using localStorage

### Bid farewell to localStorage! Embrace IndexedDB for speed, type-safe storage, and non-blocking data transactions. #IndexedDB #WebDev



](https://medium.com/@julienetienne/stop-using-localstorage-64a6d6805da8?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

5 min read·Dec 22, 2023

](https://medium.com/@julienetienne/stop-using-localstorage-64a6d6805da8?source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F64a6d6805da8&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40julienetienne%2Fstop-using-localstorage-64a6d6805da8&user=Julien+Etienne&userId=272fb8983fbd&source=-----64a6d6805da8----1-----------------clap_footer----457ba668_d282_47d9_951c_7c42fe8d7324-------)

2.2K

[

34

](https://medium.com/@julienetienne/stop-using-localstorage-64a6d6805da8?responsesOpen=true&sortBy=REVERSE_CHRON&source=read_next_recirc-----4c2739326f57----1---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F64a6d6805da8&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40julienetienne%2Fstop-using-localstorage-64a6d6805da8&source=-----4c2739326f57----1-----------------bookmark_preview----457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![‘import’ vs ‘import type’ in TypeScript](https://miro.medium.com/v2/resize:fit:679/1*c_otGxMhQQYeAODaylsf5A.png)

](https://medium.com/@quizzesforyou/import-vs-import-type-in-typescript-8e5177b62bea?source=read_next_recirc-----4c2739326f57----2---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![quizzesforyou.com](https://miro.medium.com/v2/resize:fill:20:20/1*jJiNcwBfdg97p2JszUo-vA.png)



](https://medium.com/@quizzesforyou?source=read_next_recirc-----4c2739326f57----2---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

quizzesforyou.com

](https://medium.com/@quizzesforyou?source=read_next_recirc-----4c2739326f57----2---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

## ‘import’ vs ‘import type’ in TypeScript

### When working with TypeScript modules, it’s essential to understand the difference between `import` and `import type`. These two import…



](https://medium.com/@quizzesforyou/import-vs-import-type-in-typescript-8e5177b62bea?source=read_next_recirc-----4c2739326f57----2---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

·2 min read·Sep 2, 2023

](https://medium.com/@quizzesforyou/import-vs-import-type-in-typescript-8e5177b62bea?source=read_next_recirc-----4c2739326f57----2---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2F8e5177b62bea&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40quizzesforyou%2Fimport-vs-import-type-in-typescript-8e5177b62bea&user=quizzesforyou.com&userId=ea8b34d6dc8b&source=-----8e5177b62bea----2-----------------clap_footer----457ba668_d282_47d9_951c_7c42fe8d7324-------)

12

[

1

](https://medium.com/@quizzesforyou/import-vs-import-type-in-typescript-8e5177b62bea?responsesOpen=true&sortBy=REVERSE_CHRON&source=read_next_recirc-----4c2739326f57----2---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2F8e5177b62bea&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40quizzesforyou%2Fimport-vs-import-type-in-typescript-8e5177b62bea&source=-----4c2739326f57----2-----------------bookmark_preview----457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Best Practices for Developing with TypeScript in 2023](https://miro.medium.com/v2/resize:fit:679/0*ORa6_7pu-fBaWHm0.jpg)

](https://medium.com/@worachote/best-practices-for-developing-with-typescript-in-2023-de88fc6f96a0?source=read_next_recirc-----4c2739326f57----3---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

![Mark Worachote](https://miro.medium.com/v2/resize:fill:20:20/0*1qm6A8AoOQ_nsiT-)



](https://medium.com/@worachote?source=read_next_recirc-----4c2739326f57----3---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

Mark Worachote

](https://medium.com/@worachote?source=read_next_recirc-----4c2739326f57----3---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

## Best Practices for Developing with TypeScript in 2023

### TypeScript has quickly become one of the most popular languages for front-end development. With its optional static typing, rich feature…



](https://medium.com/@worachote/best-practices-for-developing-with-typescript-in-2023-de88fc6f96a0?source=read_next_recirc-----4c2739326f57----3---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

3 min read·Nov 10, 2023

](https://medium.com/@worachote/best-practices-for-developing-with-typescript-in-2023-de88fc6f96a0?source=read_next_recirc-----4c2739326f57----3---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2Fde88fc6f96a0&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40worachote%2Fbest-practices-for-developing-with-typescript-in-2023-de88fc6f96a0&user=Mark+Worachote&userId=8ba387cabeb3&source=-----de88fc6f96a0----3-----------------clap_footer----457ba668_d282_47d9_951c_7c42fe8d7324-------)

33

[](https://medium.com/@worachote/best-practices-for-developing-with-typescript-in-2023-de88fc6f96a0?responsesOpen=true&sortBy=REVERSE_CHRON&source=read_next_recirc-----4c2739326f57----3---------------------457ba668_d282_47d9_951c_7c42fe8d7324-------)

[](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fde88fc6f96a0&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40worachote%2Fbest-practices-for-developing-with-typescript-in-2023-de88fc6f96a0&source=-----4c2739326f57----3-----------------bookmark_preview----457ba668_d282_47d9_951c_7c42fe8d7324-------)

[

See more recommendations

](https://medium.com/?source=post_page-----4c2739326f57--------------------------------)

[

Help

](https://help.medium.com/hc/en-us?source=post_page-----4c2739326f57--------------------------------)

[

Status

](https://medium.statuspage.io/?source=post_page-----4c2739326f57--------------------------------)

[

About

](https://medium.com/about?autoplay=1&source=post_page-----4c2739326f57--------------------------------)

[

Careers

](https://medium.com/jobs-at-medium/work-at-medium-959d1a85284e?source=post_page-----4c2739326f57--------------------------------)

[

Blog

](https://blog.medium.com/?source=post_page-----4c2739326f57--------------------------------)

[

Privacy

](https://policy.medium.com/medium-privacy-policy-f03bf92035c9?source=post_page-----4c2739326f57--------------------------------)

[

Terms

](https://policy.medium.com/medium-terms-of-service-9db0094a1e0f?source=post_page-----4c2739326f57--------------------------------)

[

Text to speech

](https://speechify.com/medium?source=post_page-----4c2739326f57--------------------------------)

[

Teams

](https://medium.com/business?source=post_page-----4c2739326f57--------------------------------)