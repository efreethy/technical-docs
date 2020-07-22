# Understanding MobX

Last edited: 22 August 2018
Status: Draft

## **TLDR**

There appears to be a lot of intersection between the capabilities of MobX and the demands of Rivals front end application state model. As our state model becomes more complex, introducing more derived state (such as search indexes, venue map state trees, etc) - as well as more vectors for state updates (pub-sub, streams, and queues), we should keep this solution in mind as an option to alleviate some of the technical baggage that comes along for the ride.

## Goals 

Provide a description of MobX and expand on why it might be a useful companion in our React based frameworks.


## Useful Context

Some notable organizations using MobX in some capacity include AWS, Stack Overflow, Facebook open source , Mozilla, and Coinbase. A partial list exists [here](https://github.com/mobxjs/mobx/issues/681) which is a lot longer, but probably still incomplete. The library was created three years ago, and is still extremely popular - raking in 163,494 weekly downloads.


## What is Mobx - really?

**As described by me:**
MobX makes it easier to maintain apps with advanced application state, particularly when state changes happen often and require complex updates in other areas of the app. The library exposes a small set helpers intended to supplement the state model of a front end application. These helpers can be used collectively in order replace the maintenance logic that accumulates over time when maintaining large and complicated state models. These helpers can monitor your state model, and will automatically propagate updates to other places which are concerned with those changes. It does this in a way that is transparent - separating a single technical concern from the applications main business logic.

**As described by MobX:**
MobX is a battle tested library that makes state management simple and scalable by transparently applying functional reactive programming (TFRP). The philosophy behind MobX is very simple: Anything that can be derived from the application state, should be derived. Automatically.


## Problem Description

* As an application grows in size and complexity, its state model grows with it. Over time it evolves into a complex collection of objects, arrays, classes, and other javascript data structures.
* Eventually this web of data structures grows so large it becomes challenging to maintain and reason about.
* Many parts of the application end up depending on the state model. This includes:
    * React components depend on the application state (not to be confused with component state) as the input to their rendered output. When state changes these components need to re-render.
    * Derived regions of application state, like search indexes and list collections, care about the state changes of their derivatives.
* This results in what could be described as a state maintenance problem.
* Robust custom solutions are typically needed in order to address the problem.
* Custom solutions in this area can be delicate and prone to error. State maintenance is essentially one of the two hard programming problems - cache invalidation.



## Enter MobX

* At its essence - the purpose of MobX is to solve the application state maintenance problem. It offers a light-weight api (45.9kB minified / 3.5kb gzipped) that is intended to replace and simplify the areas of an app which are dedicated to state maintenance. It hooks in very naturally into React based applications, and can be used as a companion to any state model. More notably for Rival, it can work well with state models connected to asynchronous streams of events.



## How does it work?

* Kind of like the React component tree, MobX maintains an application state dependency tree. 
* In my opinion - MobX introduces two fundamental concepts:
    * Observables:
        * These are the areas of application state which have dependents.
        * The developer informs MobX about which parts application state need to be monitored - this is accomplished through installing `@observable` decorators on ordinary javascript primitives.
    * Observers:
        * These are the areas of the application which must undergo updates whenever application state changes.
        * This includes React components, any denormed regions of application state, such as a search indexes, list collections, etc.
        * The developer also informs MobX about which parts of the application should respond to the updates on the observables. This is achieved by installing `@observer` decorators.
        * In our applications, observers would generally fall into the following categories.
            * React components.
            * Denormed regions of application state, such as a search index.
            * Any other arbitrary location where a procedure would need to run in response to a state update.



## What are the performance concerns?

* MobX is fast, [by some benchmarks](https://twitter.com/mweststrate/status/718444275239882753) it is faster than Redux (granted these benchmarks were provided by the library author)
* MobX sees speed improvements when used with components that are broken out into smaller, more granular pieces. When components are more granular, more parts of the app can render independently, resulting in less overall processing on updates.
* Lists should be rendered as dedicated components. When the presentation of a list is mixed with other presentation, unrelated state updates can cause a lot of redundant processing over the list. It is recommended that Lists be defined in their own dedicated components, so only their state and props will govern how often their re-render.
* Counterintuitively - MobX recommends that values should be dereferenced as late as possible. For example `<DisplayName person={person} />` is recommended use instead of `<DisplayName name={person.name} />`. This is because with MobX observable components, MobX will automatically re-render a component if that component de-references an observable value (ie: `person.name`) from within its implementation.



## **Other goodies**

The [MobX documentation](https://mobx.js.org/index.html) is comprehensive and pretty accessible to newcomers.

* Support for describing [actions](https://mobx.js.org/refguide/action.html) - which represent a collection of state changes intended to succeed as a unit. These collections can be decorated with the `@action` decorator, and will guarantee state is restored if runtime errors are encountered. This can help reduce bugs which create hard to spot side effects.
* Tooling for running procedures based off of arbitrary state updates.  Helpers like [autorun](https://mobx.js.org/refguide/autorun.html), [when](https://mobx.js.org/refguide/when.html),  and [reaction](https://mobx.js.org/refguide/reaction.html)) can be used in different contexts in order to register callbacks with arbitrary state updates.
* Fits well into [pub-sub architectures](https://github.com/mobxjs/mobx-utils#fromresource). It exposes a special flavor of observable which can be kept in sync with some external datasources being subscribed to.
* Fits well into [stream architectures](https://github.com/mobxjs/mobx-utils#tostream)) as well. It exposes another flavor of observable designed for listening on streams, and propagating changes to subscribers
* MobX also exposes [queue processors](https://github.com/mobxjs/mobx-utils#queueprocessor) and [chunk processors](https://github.com/mobxjs/mobx-utils#chunkprocessor) as first class citizens.



## What place does this solution have in our frameworks?

* Pretty much anywhere we want to keep application state in sync across disparate regions of the application.
* The Venue map is a complex state tree that is subject to change as entity changes stream in through pub-sub. MobX could hook into entity change events in order to keep this state tree consistent.
* Our shared sdk doesn't have any global state management, but individuals services do. These services could be promoted to be observables. We could define React component wrappers in `rival-react` which wrap these services and propagate changes to children components. MobX will handle state change detection and update so we won't have to. The library might have a use in generalizing our current `StateContainer` pattern.
* For application state that is derived, such as search indexes used by esjs, or other arbitrary derivations such as lookup tables - these could be designated as observers of the application state they depend on.
* If we ever have a need to simplify areas like pub-sub, stream processing, or queue / chunk processing - MobX will be there to abstract away the complicated aspects of state invalidation.

## **What about browser support / dependencies?**

* MobX has zero dependencies.
* MobX 4 will run in any es5 environment, and will be actively maintained
* MobX >=5 runs on any browser with ES6 proxy support. It will throw an error on startup on older environments such as IE11, Node.js <6 or React Native Android on old JavaScriptCore how-to-upgrade.
* There are a [small set of differences](https://github.com/mobxjs/mobx#mobx-4-vs-mobx-5) between 4 and 5 to be aware of.

