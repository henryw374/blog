Title: Simplify Your Business Logic
Date: 2018-09-17
Tags: clojure

A brief explanation of how to leverage your business logic code as much as possible by [Simplifying](https://www.infoq.com/presentations/Simple-Made-Easy) it, with some specific tips for Clojure developers,
including using the cross-platform [tick](https://github.com/juxt/tick) date/time library.

## Separate Decisions from Dependencies

The idea to 'Separate Decisions from Dependencies' is to only use pure functions for writing code that 'makes decisions'
and wire those functions together as necessary with all the plumbing bits of your system (that'll get your data to and from databases/services/users etc).
 Rich Hickey would call such a separation a [Simplification](https://www.infoq.com/presentations/Simple-Made-Easy).

Maybe you think you are doing this already, but ask yourself:

__Is all your business logic available cross-platform?__ 

For Clojurists that means your business logic is (or could be) in .cljc files. For 
JS/Node developers that generally means all your business logic modules are free of IO.

Doing this should be zero-cost, but I rarely see it in the wild. 

### Why Cross-Platform ?

Many authors and speakers have expounded the virtues of _Separating Decisions from Dependencies_  or _Isolating Computation From State_ [1], typically
citing reliability/testability/visibility as benefits. I am totally in agreement with them, but why go 
further and say Decisions code must be cross platform?

Thinking here of a typical modern web/mobile application, we have situations like:

* A user of the front end wants to explore taking an action of some sort and the app needs to respond as fast as
possible to show consequences, ideally without any server round-trip. Then later, on completion, some back end code must validate
the input, use it to carry out actions, schedule jobs & etc. 

* A screen of the app shows the user some derived data. The app also needs to periodically email users
with the same derived data (think bank account summary).

* A screen in the app shows a data summary of some sort. A user can drill-down into some detail and the server
sends more specific data covering the required slice of the overall dataset. Some of the calculations on the detail screen are
 the same as those that were used to make the output for the summary screen.

In all these situations we need to have the same logic wherever the app runs. 

Cross-Platform business logic is not typically available in stacks where the various app components (web ui, mobile app, server)
are on disparate platforms. Clojure and Node are two stacks where I've personally used this idea to great effect, including on Mobile applications written on 
React Native.

Briefly stated, if you've not got cross-platform logic the problems you might have are:

* Needless duplication - I don't need to say any more
* Storing up trouble - "User: Ooh, nice feature, can we have that on the _insert other platform here_ as well?"
* Decisions relying on Dependencies - usually cases that somehow slipped in when you weren't looking

### Known Issues?

The JVM and JS runtimes differ significantly, for the purposes of business logic that usually means:

* Dates & Times: Until now, we've been making do with clj/s-time [2] but now there is a better 
option __[tick](https://github.com/juxt/tick)__, which is an intuitive Clojure(Script) api over the *java.time* api. 
* Numbers: There's just the one number type in JS, but many on the JVM. This may be a blocker for some, I'll aim to revisit in a later post.
* String Formatting: No clojure.core/format equivalent exists in Cljs. Instead try [Sprintf](https://github.com/alexei/sprintf.js)


[1] A couple of my favourites are Gary Bernhardt's [Boundaries](https://eventil.com/talks/jYSOmz-gary-bernhardt-boundaries)
 and Stuart Sierra's [Thinking in Data](https://www.infoq.com/presentations/Thinking-in-Data).
 
[2] [cljs-time](https://github.com/andrewmcveigh/cljs-time) only partially implements clj-time. Working in finance as I mostly do, dates are everywhere and the 
differences too often [leak out](https://github.com/andrewmcveigh/cljs-time/issues/92)
