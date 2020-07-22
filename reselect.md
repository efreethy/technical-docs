# Rival, Redux, and Reselect

## TLDR

If repeated/redundant work in component render cycles becomes a material concern, we should keep Reselect in mind as an option to speed things up.


## Motivation

At Rival, Redux was originally the arbiter of virtually all state management in our front end architecture. As our application grew in size and complexity, Redux introduced a performance bottleneck - due to the frequency it would trigger re-renders, and by extension repeat work, in our components. A lot of this repeat work was pointless, and contributed to our movement away from Redux as a state management tool. What I found fascinating about Reselect is that it was meant to solve this problem specifically. 

I think its worth understanding the benefits Reselect has to offer, and how it might be used to improve our React based applications and frameworks - even without Redux.


## Relevant context

[Reselect](https://github.com/reduxjs/reselect) was ideated in response to a performance complaint on the Redux issue tracker back in June 2015. The issue addressed a singular performance concern in Redux. In June '15 Robbert Binna submitted a pull request to address this. The solution was quickly promoted to be a member of the Redux ecosystem. There are many who consider the use of Reselect as idiomatic Redux. It currently lives under the reduxjs family of libraries. It is a battle-tested library. It is also popular, netting roughly 500,000 downloads a week.


## Problem Description

* As as redux based applications grows in size and complexity, the number of connected components grow with it.
* Redux state updates tend to occur more and more often, and eventually reach very high frequencies.
* This becomes a problem when every connected component is forced to re-render in response to an unrelated state update.
* We eventually reach the point that, for a given state update, a minority of components are actually concerned with the state difference.
* The majority of components that don’t care about the state change will still respond, repeat the work of their render cycles, and yield no difference in their rendered output - burning up CPU cycles.
* The work being done in these render cycles can be expensive, and can cause a perceivable degradation in the an applications overall performance.



## Enter Reselect

* Reselect was designed to address this performance issue.
* At its essence - the solution is to memoize the output of functions which contribute to the render cycle.
* The implementation hooks in very naturally to component based React applications.
*  If the inputs haven’t changed, then the memoized value is returned, otherwise the output is recomputed. This requires that the memoized functions are pure functions.
* Reselect does not attempt to reduce the number of render cycles. Connected components will continue to respond to all state updates, and will still re-render.
* Components which aren't concerned with the state updates will use memoized values in their render cycle, and all functions using the memo will finish in constant time. React then detects no differences in the component trees, and avoids expensive DOM updates.



## How to use it

* Reselect is no-frills and lightweight (1.7kB). Its API exports four functions.
* It is encouraged to be used within connected components, particularly in any calls made to the now infamous mapStateToProps function.
* The library introduces the idea of a selector, which is a useful alternative to the expensive functions which were originally triggered by mapStateToProps calls.
* A selector will still perform the same work as its expensive counterpart, however the selector will not re-compute when inputs haven't changed.



## Should we still care?

Maybe.

Even in the absence of Redux, Reselect can be a powerful companion in React based frameworks and applications. It wasn't designed specifically for Redux, and can be ported easily into React applications using any state model. One of Reacts most well known framework patterns is to pass state and props through component lifecycle functions. Reselect fits very nicely into this pattern, and can improve the render times of our components as they become more connected. It does this in a way that is transparent - separating the performance concern from our core business logic. 
