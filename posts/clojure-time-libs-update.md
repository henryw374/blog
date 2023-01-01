Title: Progress update on the Clojure(Script) date-time libraries
Date: 2020-01-05
Tags: clojure

In April 2019 I gave a [talk at Clojure/North](https://www.youtube.com/watch?v=UFuL-ZDoB2U) providing my motivation for working on new date-time libraries: `cljc.java-time`, `time-literals` and `tick`. Yes, you heard right, yet more date-time libraries! In the talk I argue that they provide novelty with respect to cross-platform development and improve the overall situation for Clojurescript date-time work. This post gives some updates on what has happened since and considers future developments for these libraries.

It's always hard to know how much usage open source projects are getting, but these young libraries are definitely seeing some healthy activity in terms of Github issues, contributions and stars. As expected, tick has had the majority of activity, but I'm aware of people using the ancillary libs in isolation too. 

For the rest of the post I'll talk about the todo list that I presented in the Clojure/North talk, how it has progressed and what's new on the list. I'd welcome PRs on any of these items but if you want any of them expedited by myself then please [get in touch](http://widdindustries.com/about/) 

# [Time Literals](https://github.com/henryw374/time-literals)

Useful to any Clojurist working with java.time, either with or without a wrapper library. java.time entities are printed as tagged literals that can be pasted back in to the REPL, written to files, sent over the wire and so on. 

Examples:

```
 #time/date"1969-07-20"
 #time/time"20:17"
``` 

## Todo

There has been some interest in having Transit encodings using the literals. Support for the `datafy/nav` is another possible feature, but although it has a similar theme of representing data, it isn't that closely related to `time-literals` so might live in a separate repo.

# [cljc.java-time](https://github.com/henryw374/cljc.java-time)

This is a  Clojure(Script) library that mirrors the java.time api through kebab-case-named vars. 

```
(ns my.cljc
  (:require  [cljc.java-time.local-date :as ld])
  
  ;create a date
  (def a-date (ld/parse "2019-01-01"))
  
  ;add some days
  (ld/plus-days a-date 99)
```

## Progress

As this library only aims to mirror java.time and is generated from it, the api was essentially 'done' on the first release. The type hinting has been improved since then, thanks to work which has now become [Tortilla](https://github.com/emlyn/tortilla))

Having (clojure.spec) specs and generators for java.time is something I [made a start](https://github.com/henryw374/time-literals) on, but while spec itself is changing then any lib using it is going to be subject to the same flux. There has been some interest in having [Malli](https://github.com/metosin/malli) use cljc.java-time for it's date and time logic, but IMO cljc.java-time brings too much baggage on the Clojurescript side - although read on to the next section for a possible way through for this and other libraries.

Whatever direction these specification efforts take, a set of predicate functions for the java.time entities is a generally useful thing and a recent contribution to cljc.java-time added instance predicates for all java.time entities, for example `(date? x)`. 

The other [open issues](https://github.com/henryw374/cljc.java-time/issues) are about listing discrepancies between the underlying time libs and wrapping the further flung corners of java.time.

# [cljs.java-time](https://github.com/henryw374/cljs.java-time)

This Clojurescript library is used by `cljc.java-time` (see above). It exposes the npm library [Js-Joda](https://github.com/js-joda/js-joda) (a faithful and mostly complete JS implementation of java.time) and extends the Clojurescript equivalence, comparison and hashing protocols to its entitites.

Because js-joda existed, it was a relatively small step to make a cross-platform time library, but although it is an enabler it is also something of a hindrance for two main reasons:

1) The myriad ways Clojurescript users consume npm libraries

2) It's size (minimum of 43k, gzipped)

On point 1), making a Clojurescript library that uses an npm lib sometimes feels a bit like being in the wild west, but at the end of the day, whatever setup you have it's not too hard to get this library working and if you can consume foreign-libs, it 'just works' out of the box. Having a single command that will drop you into a node repl with tick is pretty cool, for example:

```
clj -Sdeps '{:deps {org.clojure/clojurescript {:mvn/version "1.10.597" } tick {:mvn/version "0.4.23-alpha"} }}' -m cljs.main  -re node  --repl
```

Regarding point 2), I would say that in many contexts (including client projects I am currently engaged in) the size is simply not an issue when trading off 

* utility 
* overall build size 
* actual end user's experience 

In other contexts the size would be a problem though. I have briefly looked into reducing its size through minification & Dead Code Elimination and although I've made a little progress I don't think it's going to be possible to reduce it enough to bring it down hugely or to be on a par with `goog.date`.

These two issues together mean that `cljs.java-time` & related libs probably aren't going to be used by other Clojure(Script) libraries needing date/time functions.

I'd say the best looking solution on the horizon is the new platform date-time library being developed for Javascript aka [tc-39/proposal-temporal](https://github.com/tc39/proposal-temporal) (hereafter PT). If JS had a good date-time api built-in then it wouldn't be necessary to bring your own of course, so no need for Js-Joda and problems 1) and 2) go away, simples!

The PT api is currently being finalized and experimental work is underway to reimplement [Moment.js](https://github.com/moment/moment) with it. How far away it is from being included in the next version of Node, Chrome etc I really can't say. Even when it is there, the world's browsers won't get upgraded overnight so polyfills will be needed for some time in many cases. 

That aside, what does PT look like? Unfortunately from a cross-platform lib developer's PoV it is not that similar to java.time. Some obvious differences are naming and the entities where methods reside. Also there is no ZonedDateTime and the Duration and Period classes have been combined. However, although it looks pretty different, my feeling is that the differences are quite superficial. If you wanted a cross platform Clojure api that used PT and java.time, I think extending tick's protocols to the PT entities would likely be a good starting point, so you might end up with a `tick-lite` that doesn't use js-joda or any npm lib.

# [tick](https://github.com/juxt/tick)

This is a [Juxt](https://juxt.pro/index.html) library that I made cross-platform and am now helping to maintain. I'll try to briefly describe it as having the following features: 

1) A single-ns, intuitive api over java.time

2) Batteries-included - it depends on and configures `cljc.java-time`, `time-literals` & additional js-joda locale & timezone libs

3) Power tools - Interval Algebra and Scheduling

Unsurprisingly `tick` appears to have the most traction of all the libs (despite its alpha status) and it's been great to see a number of bug fixes and minor contributions from the community since April.

In the main api the things I'd most like to be addressed include:

* parsing (defer to DateTimeFormatter entirely)
* another look at `>>/<<` vs `+/-` and `range` 
* more [documentation of tick](https://juxt.pro/tick/docs/index.html). 

See the full list of [issues](https://github.com/juxt/tick/issues) for other open items.

# Conclusion 

It has been good to see a positive response to the talk and the libraries. In summary, there have been some minor bug fixes & tweaks since April, but I've outlined what I see as some potential features that could exist in future - if people want them! As I say, I'd welcome PRs on any of these items but if you want any of them expedited by myself then please [get in touch](http://widdindustries.com/about/).    
