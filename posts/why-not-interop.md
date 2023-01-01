Title: Why not Interop with java.time?
Date: 2021-01-04
Tags: clojure,date-time

Using interop syntax with the java.time API is not wrong, but there is an alternative that is superior in every respect - and it just got much better.

Some in the Clojure community would say that if a Java or JS API is good, then it needn't be 
'wrapped' in a library, because plain interop code is idiomatic and any wrapping
library might:

* Miss out something of the underlying API (so you likely need to know both anyway)
* Have fewer tests/documentation/community etc
* Introduce some subtle new semantics
* Not keep up with releases of the underlying

Those concerns all seem reasonable, but consider [this java.time wrapper](https://github.com/henryw374/cljc.java-time).
Every one of the above concerns is addressed: The Clojure API is identical to the underlying Java API (addresses first 3 points) and the
underlying API is stable (last point).

Okaaaay ... but if it's so similar, why use it at all?

The library's main reason to exist is because it works cross-platform, a huge win on my current projects - but let's leave that aside for the moment and consider the 
jvm-only POV.

## Camel->Kebab wrapping benefits

* All type hinting is done for you
* comp, apply, juxt and all other clojure.core higher-order fns can now be used rather than anon fn interop: `#(.bar %)`
* In fact, instead of seeing code like `#(.isAfter %)`, you'll use a properly namespaced clojure function `x.y/is-after` - much better!
* The Javadoc of the original API will be more readily available via Clojure docstrings (see [issue](https://github.com/henryw374/cljc.java-time/issues/16)) 

## Additional API

* Predicates, for example `(local-date? x)`

This could be used for spec definitions for example.

## Exceptions That Say Something Helpful

This is a very recent addition ([0.1.12](https://clojars.org/cljc.java-time) and takes a bit of explaining. In java.time, you have an `Instant` which is equivalent to a
`java.util.Date` in that it's instances represent `the start of a nanosecond on the timeline`. Now the tricky thing
about Instants is that the only field/data they contain is an offset from the UNIX Epoch (midnight, Jan 1st 1970, UTC). Instants
know nothing about years, months, days, hours etc etc. In the lingo, they are not 'calendar-aware', so you cannot add
a year to an Instant or ask for it's day of the week. BUT... the API kind of suggests that might be possible. 

Firstly, let's see the output of `Instant#toString`:

```java
2013-05-30T23:38:23.085Z
``` 

Hold on, how and why is that printing years, months & etc? Well, an Instant can be converted into a calendar representation and 
that's what the `toString()` implementation does. That auto-conversion is the exception rather than the rule though, if you try to format
an Instant as 'yyyy-mm-dd' you'll see what I mean.

On a related note, try adding a year to an Instant: 

```clojure
(-> (Instant/now)
    (.plus 1 ChronoUnit/YEARS))
```

It compiles ok, but at runtime you'll get a run-time exception: `Unsupported Field : YearOfEra`. 

So what's the problem here? People should just learn this basic fact about the java.time API before using it, right?

In practice, I see this issue about Instant and calendar-awareness come up a lot - via colleagues, github issues, stackoverflow & etc

My guess is that a lot of people only need to work with dates infrequently enough that they don't feel it's worth investing time
going through the tutorials, or if they do, the nuances quickly fade through lack of practice.

Having seen this problem in the wild again and again for many years I have decided to take action! 

If you now do some erroneous thing with Instant in cljc.java-time, like this:

``` 
(-> (cljc.java-time.instant/now)
    (cljc.java-time.instant/plus 1 cljc.java-time.temporal.chrono-unit/years))
```

You get an exception with message:

``` 
Hi there! - It looks like you might be trying to do something with a java.time.Instant that would require it to be 'calendar-aware', 
but since it isn't, it has no facility with working with years, months, days etc. 
To get around that, consider converting the Instant to a ZonedDateTime first or for formatting/parsing specifically, 
you might add a zone to your formatter. see https://stackoverflow.com/a/27483371/1700930. 
 
You can disable these custom exceptions by setting -Dcljc.java-time.disable-helpful-exception-messages=true
```

That message alone should at least prevent a lot of github issues and questions I get .... let's see!

I am adding that message because I think it's unlikely I could get java.time messages changed, I don't even 
know how I would do that. I do know how to make this suggestion for the new date-time API being made for the world's most popular
programming language though, raise [an issue!](https://github.com/tc39/proposal-temporal/issues/1233). That
improved error message may be my greatest contribution to humanity to date ;-)

# What about traditional wrappers like tick or clojure.java-time ?

These are traditional wrappers and definitely come with trade-offs. In my work we use date logic a lot, which I think
means it's an environment where it's worth learning to use a wrapper because of the extra+improved API. That being said, I use a wrapper for 80% of 
regular date-arithmetic - for things I consider more obscure or esoteric, or where performance might suffer, I drop
to cljc.java-time (which tick is using under the hood) - and in very rare cases - plain interop.

If you have any thoughts/questions, please let me know ;-)
