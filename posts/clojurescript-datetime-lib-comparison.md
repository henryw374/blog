Title: Performance comparison of Clojurescript date/time libraries
Date: 2021-08-16
Tags: clojure

In this post I am going to compare the performance of two Clojurescript date-time libraries, in the context of 
a typical single-page web application.

# The Libraries

## Cljc.java-time

[cljc.java-time](https://github.com/henryw374/cljc.java-time) (disclaimer: authored by myself) has the same API as java.time, 
but targets both Clojure and Clojurescript. 
It is implemented on top of a pure Javascript implementation of `java.time`
called JS-Joda.

It is also the underlying library for Juxt's [Tick](https://github.com/juxt/tick) 
 which provides a powerful date-time API beyond what java.time offers. In this blog post I am considering cljc.java-time
 instead of Tick, because I expect a larger proportion of readers will already have some familiarity 
 with java.time's API. 

## Deja-fu

[Deja-fu](https://github.com/lambdaisland/deja-fu) is a new Clojurescript date/time library 
positioning itself as being for
"applications where dealing with time is not enough of their core business to justify these large dependencies".
The 'large dependendencies' being referred to there are those required by `cljc.java-time`.

Deja-Fu's API offers a pure-cljs `Time` entity and otherwise wraps the platform js/Date objects, 
via the goog.date API.

The long established [Cljs-time](https://github.com/andrewmcveigh/cljs-time) is similar to Deja-fu in that it also wraps the goog.date API. I'm not measuring that here, but I would expect to see similar results.

# Motivation

Deja-fu appears to offer a trade off between a "good-enough date/time library, that's very lightweight" 
vs a "complete date/time library that's heavier". My feeling is that cljc.java-time is not meaningfully heavier and 
that `light` date/time libraries are often heavy on developer time, bugs or both. 

In my own experience of developing Clojurescript web 
applications with cljc.java-time, I haven't see any performance problems - I'm already using Clojurescript and React so there's
already a significant build size, but given my users' devices and 
network connections (reasonably up to date, but nothing special) the applications seem to perform
very well.

I once did [a talk](https://www.youtube.com/watch?v=UFuL-ZDoB2U)
 introducing cljc.java-time and related libraries, but only briefly talked about build size - should I have said it 
 was only suitable
if date/time was so core to the app that "large dependencies" could be justified ? FYI Build size [is already discussed](https://github.com/juxt/tick/blob/master/docs/cljs.adoc)
in the documentation. 

Over the years since I released cljc.java-time I've come across (and generally ignored) the `too large/heavy` PoV a couple of times, 
but Deja-fu's recent appearance has prompted me to put it head to head with cljc.java-time in an experiment.
 
# The Experiment

For my experiment I have written two versions of a basic Clojurescript web application. 

People are using Clojurescript
in various places, including highly constrained environments like microcontrollers, but based on what 
I see the React webapp is what the majority are targeting and the use-case for which I would like people to have
some more help when choosing a date/time API. 

These apps have been deployed on the web so that tools such as [PageSpeed](https://developers.google.com/speed/pagespeed/insights/)
can be used to test them. They are hosted on Firebase, but just because I already had a dummy project set up there. They don't
use any Firebase APIs.

<table>
  <thead>
    <tr>
      <th>Version</th>
      <th>TTI  (mobile)</th>
      <th>TTI (desktop)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://friendly-eats-demo-e71b7.web.app/js-date.html">Deja-fu version</a></td>
      <td>2.1s</td>
      <td>0.6s</td>
    </tr>
    <tr>
      <td><a href="https://friendly-eats-demo-e71b7.web.app/java-time.html">cljc.java-time version</a></td>
      <td>2.2s</td>
      <td>0.7s</td>
    </tr>
  </tbody>
</table>

[//]: # | Version             |  TTI  (mobile)| TTI (desktop) | 
[//]: # |---------------------|---------------|---------------|
[//]: # | [Deja-fu version](https://friendly-eats-demo-e71b7.web.app/js-date.html) | 2.1s | 0.6s |
[//]: # | [cljc.java-time version](https://friendly-eats-demo-e71b7.web.app/java-time.html) | 2.2s | 0.7s |

 TTI (time-to-interactive) shown in the table above was taken from PageSpeed analysis. The cljc.java-time version is slower
  according to that analysis, but not in any meaningful way.
  
Consider that (the Javascript behind) cljc.java-time and React are fixed-size costs though. Being a small demo app, they 
 are disproportionately big. If application code grows over time with features their 
relative size will reduce ofc.

The memory usage for both apps was roughly the same, as observed in a recent version of Chrome. Deja-fu
is using js/Date objects, which have a single number field (representing an offset from the unix epoch). The cljc.java-time version is using 
LocalDate objects which 
have 3 numeric fields: year, month and day. Having the additional two fields could become significant if a large 
amount of date objects need to live in memory.
 
What about download size? Well, let's imagine that every time a user visits these apps, the Clojurescript code has 
been changed and released, so cannot be retrieved from cache and must be re-downloaded.
It will only take 2-3 visits before the total amount of data downloaded across those visits is greater 
in the Deja-fu version. This is because in the cljc.java-time version, the underlying data/time lib is downloaded separately, 
and so is cacheable. This is very simple to set up and I would put it in the 'no brainer' category
if data allowance is a significant issue for an app. 

Maybe we could modularize the Deja-fu version so the library code can be cached over visits, 
but my main point here is YMMV.

Are there more metrics that we should look at here? Please suggest anything you think is significant.

# The code 

Shown below is the code that is different between the two versions.

Two functions are required

* `tomorrow` returns tomorrow's date. This is needed so the date picker will only let users 
pick future dates
* `interval-calc` works out the number of days between 2 dates.

The [source code for these can be found here](https://github.com/henryw374/cljs-date-lib-comparison).

## Deja-fu 

```clojure 

(ns time-lib-comparison.js-date
  (:require [lambdaisland.deja-fu :as deja-fu]
            [time-lib-comparison.app-main :as app]))

(def millis-per-day (* 1000 60 60 24))

(defn interval-calc [event-date]
  (let [now (deja-fu/local-date)
        event-date (deja-fu/parse-local-date event-date)
        interval-millis (- (deja-fu/epoch-ms event-date)
                          (deja-fu/epoch-ms now))]
    (/ interval-millis millis-per-day)))

(defn tomorrow []
  (-> (deja-fu/local-date)
      (update :days inc)))

```

### cljc.java-time 

```clojure 

(ns time-lib-comparison.java-time
  (:require [cljc.java-time.local-date :as date]
            [cljc.java-time.temporal.chrono-unit :as cu]
            [time-lib-comparison.app-main :as app]))

(defn interval-calc [event-date]
  (-> cu/days (cu/between (date/now) (date/parse event-date))))

(defn tomorrow []
  (-> (date/now)
      (date/plus-days 1)))

```

# And now, a twist

The libraries have been weighed up against each other performance-wise, but
of course that is only one part of the story. 

Did you notice any bugs in the Deja-fu version? Go back and look if you want, I will reveal the issues in the next sentence.

Firstly, the Deja-fu version is expecting that there are 24 hours in a day - and generally that's right, except when crossing 
a DST boundary. The other issue is in the `tomorrow` function of the Deja-fu version. It returns a date, but not tomorrow's.  

If you already know java.time, then it's not just different method
signatures you'd have to deal with in using Deja-fu, but actual semantics. For example, if you 'add' a month to the 
31st January, what happens?
A decision had to be made by the API authors, and that decision was made differently.

I did think about putting bugs in the cljc.java-time version too but they looked like obvious typos, like `(plusDays 2)` for
`tomorrow`.

## Is this a fair test?

I have chosen some requirements for the app where in the Deja-fu version I need to use the much 
maligned js/Date API. If you knew you'd be going down that road from the start, you might think twice about choosing a 
`lightweight` date/time library, but generally we can't be sure what requirements will come our way. 

Also, if the app needed to do custom parsing and formatting from/to strings, for the cljc.java-time version I'd need to bring in a JSJoda addon, 
which takes TTI up to 2.5 seconds on mobile, whereas with the Deja-fu version TTI would be the same.

# Looking to the future

[Tempo](https://github.com/henryw374/tempo) is my work-in-progress attempt to make a date-time API 
with the common parts 
of java.time
and the new platform API for Javascript called `Temporal` (to be available in browsers sometime soon, 
possibly this year).

The fact that Temporal is a platform API is the big reason of course.

My feeling is there is sufficiently large overlap between Java and Javascript's platform date-time APIs to make a useful 
library that will suit cross-platform 
[library authors needing some basic date/time functionality such as Malli](https://github.com/metosin/malli/issues/49) and 
perhaps also as a basis for a 'lite' version of the Tick library. 

Will cljc.java-time become irrelevant in a Tempo future? I don't think so because I think for many it will continue to be a solid, 
familiar choice that comes with minimal overhead and maximum stackoverflow-ability. [Not everyone loves java.time](http://www.menodata.de/blog/2013/12/02/die-neue-zeitbibliothek-von-java8-eine-rezension/#more-174)
but it's usually good enough.

# Conclusion

The main take-away I would hope readers
get from this is that that cljc.java-time should not be dismissed out of hand for some common use cases, just 
as we don't generally pick C over Clojure because an equivalent program might be less resource intensive.

There may well be Clojurescript applications where cljc.java-time would be inappropriate, but my feeling is 
for typical Clojurescript web applications it's not an issue.
 
I'd definitely be interested to hear about your opinions on this
and Clojurescript build sizes in general. What are you shipping? How did you make decisions 
about what build size was acceptable (including
the use of Clojurescript itself)?

Feel free to use [this thread on Clojureverse](https://clojureverse.org/t/performance-comparison-of-clojurescript-date-time-libraries/8046) to comment. 