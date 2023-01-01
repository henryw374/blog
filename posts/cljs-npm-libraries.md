Title: How to package Clojurescript libraries
Date: 2020-04-17
Tags: clojure

# Update (2021-07-18)

The Reagent project, one of the better known
cljs projects that depends on a javascript lib, has abandoned Clojurescript's
dependency mechanisms (foreign-libs or deps.cljs) entirely, making users bring their own React, see [this issue](https://github.com/reagent-project/reagent/issues/537)
for details.


Original content from here on :

If you are authoring a Clojurescript library that doesn't depend on any regular Javascript (JS) code, transitively or otherwise then things are pretty straightforward: maven-package your library and put it in Clojars - job done!

Still reading? OK, well things are not as straightforward when libs do depend on JS code. As of this writing there is no guide I know of that explains everything you'd need to know, so think of this as a first draft of such a guide. Ideally the content here could go on to be included in the [Clojurescript Site](https://clojurescript.org/), as a solution for open tickets such as [this](https://github.com/clojure/clojurescript-site/issues/224). Also note that what I describe here all works since the [April 2020 release of Clojurescript](https://clojurescript.org/news/2020-04-24-release) - There were some changes in that release that significantly improve the situation regarding libraries.

The main consideration is how to package a library so that Clojurescript users can consume it, whatever their build setup. To that end, I have created [this companion repo](https://github.com/henryw374/clojurescript-library-consumers-test) to demonstrate different Cljs build setups (Shadow, target-bundle, cljsjs, npm-deps) all consuming the same (npm-depending) library and all targeting the same thing: a browser build with advanced optimizations. The library used as an example is one of mine, called [cljs.java-time](https://github.com/henryw374/cljs.java-time). This uses code from a single, standalone npm library 'js-joda' - so about as straightforward an example as you can get.

# Brief Background

In Clojure (JVM) land, consuming maven-packaged Java libraries is seamless and ubiquitous. If you find a Clojure library that looks useful and that library ultimately depends on one or more Java libraries, then installation will not an issue. The Java libraries will get pulled down along with all the Clojure libraries and the overall artifact size won't likely be a major concern, within reason.

With Clojurescript, it's a bit different. Having a low overall artifact sizes may be crucial to your users for one thing. For another, regular JS libs are stored in NPM, which Clojure dependency tools like tools.deps do not currently work with. 

Clearly having Clojurescript libraries depending on plain JS is possible, and has been since the early days of Clojurescript via `:foreign-libs`, but for good reasons the story doesn't end there. JS-using libraries (like [Reagent](https://github.com/reagent-project/reagent) for example) usually have something in the README to explain how to consume them - because it's not as straightforward as on the JVM.

## Why would Clojurescript libraries depend on NPM libraries?

A maven-packaged ecosystem exists that shadows a lot of stuff from npm, called [Cljsjs](https://github.com/cljsjs/packages), so why would anyone bother complicating their build with a second dependency/build tool?

* End users might have a set up where they want code from npm libraries A and B, which both depend on npm library C, but possibly different versions of C. People don't want to be in the business of sorting out these kinds of situations by hand. Thomas Heller goes into [more detail about why Cljsjs doesn't scale](https://code.thheller.com/blog/shadow-cljs/2017/09/15/js-dependencies-the-problem.html).
* Users are already happily using npm and don't want to have to package up foreign-libs for everything.

IOW - if people are using more than one or two JS libraries in their Clojurescript build, then using npm will likely be their preferred solution. Shadow-Cljs is one popular way to set that up and the newly released `:bundle` target in Clojurescript is an alternative. 

## Should your Library depend on Cljsjs?

We can't say what proportion of Cljs users are using npm in their build, but we can try to use some proxies to guess. The [Clojure survey](https://clojure.org/news/2020/02/20/state-of-clojure-2020) doesn't ask this specifically unfortunately, but what we can see is that there is still plenty of [activity in the Cljsjs repo](https://github.com/cljsjs/packages/graphs/commit-activity). It be good to see the graph over time of Clojars downloads of Cljsjs packages too... tbd. I would expect to see use of Cljsjs diminish over time, but my feeling is that it's not going to be abandoned any time soon. 

It is an option to have your library *not* depend on any Cljsjs libraries and have instructions in your readme for non-npm users that they'll need to add the Cljsjs dependencies themselves. This might be ok if the underlying npm libs are few and are not likely to change much - otherwise upgrading will be somewhat painful for users. 

Another consideration if your lib does depend on Cljsjs libraries will be users targeting Node. Unless they are using Shadow, `:foreign-libs` will be picked up but they work a bit strangely because [every 'require' results in the foreign lib being evaluated](https://github.com/clojure/clojurescript-site/issues/320), so prepare for some confused users or take steps to mitigate it, as [I did with js-joda](https://github.com/henryw374/packages/commit/13958337d9ba2e04563c03076bba24be6e1b0921).

In summary, you can decide if the (potential) users of your lib are likely to be the ones using npm already or not. If they might not use npm, and the Cljsjs dependency tree doesn't look too hairy, then perhaps you would decide to have Cljsjs dependencies for the time being.

# Authoring

## Cljsjs

Assuming you do want to package one or more Cljsjs libraries (whether your lib declares a dependencies on them or not), you need to look at packaging 'foreign-libs' that will contain all the code that would have come from npm (if they don't exist already in CLJSJS of course). What makes this code 'foreign' in Cljs parlance is that is not written in Google Closure style ([Google Closure](https://developers.google.com/closure) is a key tool the Clojurescript compiler uses under the hood). The npm code you want to consume may actually be amenable to Dead Code Elimination by Closure but let's ignore that for now.

Create a library that just packages one npm library and submit a PR to cljsjs. Cljsjs has a helpful [wiki](https://github.com/cljsjs/packages/wiki) with guides and explainers.

That Cljsjs library will contain a `deps.cljs` file that looks something like this:

```
{:foreign-libs
 [{:file "cljsjs/js-joda-core/js-joda.inc.js",
   :provides ["@js-joda/core"],
   :global-exports {"@js-joda/core" JSJoda}}]]
 :externs ["cljsjs/js-joda/common/js-joda.ext.js"]}
```

There are more opts you might need, see [the full list here](https://clojurescript.org/reference/compiler-options#foreign-libs) for more info. 

Note though that I am packaging `:externs` here. Users of your library should use compiler opt `:infer-externs true`, but for the Cljs.java-time library, that is not sufficient to survive advanced compilation. You can use the [library consumers test](https://github.com/henryw374/clojurescript-library-consumers-test) to see if you need hand-rolled externs for your library or not.

One important point for npm-compatibility is what you put in `:provides` and they keys of `:global-exports` - the name (in this example `@js-joda/core`) **must exactly match that of the npm package name**.

With this setup, your library code ns can require the JS lib like so:

```
(ns my-cool-lib
  (:require ["@js-joda/core" :as joda]))
  ```
  
Importantly, this require will work for both foreign-lib/Cljsjs users and npm users.  

Now you have a mvn-packaged foreign-lib, your library pom.xml can depend on it as it would any other non-npm lib.

## Npm

Package a `deps.cljs` file with your lib with contents like this:

```
{:npm-deps {"@js-joda/core" "1.12.0"}
 :externs ["cljsjs/js-joda/common/js-joda.ext.js"]}
```

This example is from `cljs.java-time` again and in this case just lists a single npm library and the required  npm version. The same externs file that was packaged with the Cljsjs library is included here as well because the npm-using users of your library will exclude the Cljsjs dependency to avoid getting the foreign-lib as well (Shadow users won't need to as it ignores foreign-libs) but they will still need the externs. 

Note that `:npm-deps` dependency in deps.cljs is **not** tied to Google Closure processing (it was in the past).

# How consumers will install your library

The best thing I can do to explain the possibilities here is to point you to the [library consumers test](https://github.com/henryw374/clojurescript-library-consumers-test). This actually demonstrates all of the possible ways your library could be consumed by users targeting browsers and examples of how to do so.

The test includes an example compiling with `:npm-deps true` option but if that doesn't work, don't fret, it is [not recommended](https://clojurescript.org/reference/compiler-options#npm-deps)

# Wrap-up

React wrappers aside, afaik there aren't that many Cljs libs depending on plain-JS libraries right now - in some cases that might be because it has been seen as complicated, but as this guide shows, it's not rocket science. 

I would advocate a change to Clojurescript that introduces a new compiler opt `:use-foreign-libs-from-deps?` that defaults to `false`. That would mean the non-Shadow npm users didn't have to track down and exclude Cljsjs dependencies from their build and hopefully act as a clear statement that Cljsjs is no longer the recommended path.

The situation for library authors would of course be more straightforward if all Cljs users were using npm, but I guess a signifcant proportion don't. It would be cool if the Clojure survey could track that more precisely. [Cuerdas](https://github.com/funcool/cuerdas) is one such Cljsjs-depending library, and has clearly had [some issues](https://github.com/funcool/cuerdas/commit/a8443724e22435e052af4149b27dd2169f3216ac) when they try to drop the Cljsjs dependency. 

As I say this guide is correct to the best of my knowledge and is something I would have found really helpful when first creating a Clojurescript library. If you have any feedback, corrections etc, I'd love to know!

& Thanks to David Nolen for explaining some of the finer points to me! 
