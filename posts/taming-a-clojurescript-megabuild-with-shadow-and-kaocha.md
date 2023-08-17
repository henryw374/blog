Title: Taming a Clojurescript mega-project with Shadow and Kaocha
Date: 2023-08-17
Tags: clojure

In this post I am going to look at applying my regular dev setup to a project with a lot of code and tests that take a very long time to run.

Firstly, here are my Clojurescript dev setup requirements:

* run a single test from the IDE - ie the one under the cursor
* run all tests in namespace similar to above
* run any subset of tests based on ns-pattern
* see nicely formatted test output - for example clearly highlighting the diff in expected/actual maps
* tests in CI output a standard format like junit xml, so results can be interpreted by CI tool
* tests in CI can run under advanced compilation (so as to be similar to the production build)
* fast incremental compile+reload

Anything you'd add? If so please see link at the bottom for how to discuss

I should also say that I'm generally developing in multi-person teams building SPA's with considerable business complexity. However, the list is still the same even for my own open source or hobby projects, but in some situations not everything above is a must-have. For example, if you only have a small number of fast-running tests then the items above about running subsets of tests will not be so important. 

I recently started to do some work on a project with around 800 Clojurescript (browser) tests - which in itself may not sound massive, but there are a number of slower DOM-clicking tests, so total test-run time above 20 minutes. It goes without saying that one would not want to run all the tests in one go - but this fact meant that builds and tests had been split apart and this had led to quite a bit of complexity in the build setup: Local dev was done with figwheel+webpack, configured with multiple [extra mains](https://figwheel.org/docs/extra_mains.html), and shadow+karma in the CI environment. 

With the existing build setup, on saving a file you could be waiting 10+ seconds to see the incremental build finish - ouch! To run a single test required knowing which figwheel 'extra-main' the test would be compiled into and loading the auto-test browser page for that, and then doing some other steps I'm not even going to get into... all in all, not ideal. 

So... what to do? My preferred Clojurescript build and test setup for the past couple of years has been shadow plus kaocha-cljs2. Everyone knows shadow of course, but [kaocha-cljs2](https://github.com/lambdaisland/kaocha-cljs2) seems weirdly unstarred (< 20) and un-discussed on the interwebs. The combination of these two gives me the above wishlist of course - that's why I chose it. But how well would it scale to the new megaproject? How easy would it be to set up?

## Setup

Possibly one reason kaocha-cljs2 seems under-appreciated is that by design there are more moving parts compared to other Clojurescript test setups I have used - for example one needs an extra server (Funnel) for 2-way communication between jvm and js environments. 

However, setup couldn't have been easier - and that's because I have a little ready-made shadow+kaocha_cljs2 template that I use in all my projects and libraries. I've called this template [tiado-cljs2](https://github.com/henryw374/tiado-cljs2) and if you want to try it on your project, you'll have it up and running in minutes - see the README for instructions.

## Does it scale?

In the way I set up Shadow on this project, there are 4 'watches' going at once, one for the main app, one for tests and a couple of others for some miscellaneous apps. Shadow incrementally compiles just what is needed so if changing a test file, just the test build kicks in. So compared to the old build, incremental compile is often around 5-10x faster. 

When it comes to the tests, I have used kaocha suites to split the tests based on ns-patterns - which in CI can be run concurrently. Kaocha-cljs2 doesn't support 'ns-patterns' out of the box yet (as kaocha does for clojure tests) - but luckily kaocha supports user-defined hooks so [adding it was not difficult](https://github.com/lambdaisland/kaocha-cljs2/issues/32).

With these mega test-suites, the default timeout was not enough. User-defined timeouts are not currently respected by kaocha-cljs2 - so [a little monkey patching was needed](https://github.com/lambdaisland/kaocha-cljs2/issues/31).

As well as running individual suites, the holy grail of clojurescript testing is surely having an IDE hotkey to run the test under the caret. This is achieved with a simple macro invoking `(kaocha.repl/run xxx)` - a macro was required (rather than a function) so that it can be invoked when either in a cljs or clj repl.

I could have had multiple test watches instead of one - each watching a subset of tests (this is a shadow feature). However the single one works well enough and makes life easier for developers as there is just a single place for tests to run.

So, whilst compilation and test-running times still seem a bit bigger compared to what I'm used to, the whole thing now feels far more manageable. There may be scope for modularization of the app  - I don't know yet, but I'm much happier to investigate that and experiment with a solid, speedier build+test setup. 

## Finally

The fact that all this works as well as it does is thanks to the shadow and kaocha maintainers of course. Clojurescript would not be in such a good place without them!