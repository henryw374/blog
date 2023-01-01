Title: Tick is out of alpha!
Date: 2021-09-28
Tags: clojure

[Tick](https://github.com/juxt/tick) provides a powerful, cross-platform date-time API way beyond what 
java.time offers. It is implemented on top of [cljc.java-time](https://github.com/henryw374/cljc.java-time) which again is
cross-platform as has exactly the same API as java.time.

For years now, the API has been `alpha`, by which we mean "Ready to use with the caveat that the API might still 
undergo minor changes". With the current release, the API of tick has been split into 

* a `tick.core` namespace, which will have no breaking changes in future releases
* a `tick.alpha.interval` namespace, which contains just the functions pertaining to Allen's interval
calculus. As its name suggests, this is still alpha. Apologies if the title of this post is misleading, I didn't 
want to stuff it with caveats.

There are plans to revisit the interval functions and documentation and some changes to the API may
arise from that work.

If you are upgrading from an earlier (0.4* version), here is what needs to change:

* replace all `tick.alpha.api` requires with `tick.core`
* If you used the `+` or `-` functions, note that these now only work where the arguments are all
Periods or all Durations. To move a time by an amount, use `>>` or `<<` instead, keeping the arguments the same. 
This change was made to make +/- analagous to clojure.core's +/-, where the arguments are commutative and
associative. It also means there is now just one way to 'move' a time in Tick.
* The `parse` function has now been removed from the API. Instead, choose the appropriate function for the 
format of the string you need to parse, e.g. `(t/date "2020-02-02")`. The parse function was slow by 
definition, as it tried to find an appropriate entity matching the string it was given. It also meant
 that if
for some reason the string you passed it was not in the format you expected, it might be parsed into
a different date-time entity than you expected, which is never going to be good.
* The compiler will report about any other functions not found in tick.core. Those will be the interval ones,
so adding a require of `tick.alpha.interval` will address those.

That's it. 

Making breaking changes may be frowned on in the Clojure community, but they seem entirely reasonable
to me in this case because:

* The api always had an `alpha` warning.
* There is no obligation to upgrade. Tick is an `end-user` kind of library: I doubt any other libs
depend on it, so dependency hell is not an issue.

One last thing, the current version is `RC`, release candidate. IOW please kick the tyres and let us
know of any problems. We have already been using it in production for a while, with no issues. After a 
period of a couple of months, we'll remove the RC label.  

 




