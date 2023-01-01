---
layout: post
title: A cross-platform date-time library you can depend on
description: How Clojurists Together funded work to make cljc.java-time usable by other libraries
category: clojure 
---

Imagine you start playing with a new programming language, you write "Hello, World!" without too many semicolons and then you go to write a Fibonacci generator ... but find there's no in-built math operators in the language! Looking up the nascent docs you see the language authors say that if people need math, they can write thir own libraries!

Madness and absurdity on many levels! But let's keep going for a moment longer... multiple math libraries appear and gain usage. Then users of the language want a charting library, but that will need to do math so at least one charting library per math library is created. Someone proposes a math facade library... ok let's end the thought experiment there. 

So, is this a healthy ecosystem with great choice for users, or a mess?

Well, what about 

Clojure has many super powers but having the same code target the JVM and Javascript platforms is definitely a biggie. 

What about an implementation of java.time api in Clojure from scratch? That would be a huge effort to complete and definitely not something I'd spend my free time on.


# Work

[js-joda](https://github.com/js-joda/js-joda/issues/365#issuecomment-650001131)

js modules
https://github.com/henryw374/cljs-jsmodules-test

projs to benefit
https://github.com/lambdaisland/deep-diff2/pull/10
juxt regex
malli

Dealing with 2k+ lines of somebody else's


goog.require
https://google.github.io/closure-library/api/goog.html#require

I often seem to find myself in a situation with Clojurescript where there are apparently only a handful of people on earth who might know the answer to my question. So I asked

Hi all, I want to consume some es6 code from clojurescript, which doesn't come from a library - I mean the es6 code is just a local file. It is possible to require a .js file from cljs when it is written with goog.provide  I know, but this code is written as an es6 module.
The Google Closure compiler is fine reading this es6 code when I invoke it directly (not via cljs compiler) and I have a demo of that here https://github.com/henryw374/cljs-using-non-foreign-es6, but I can't work out how to write the clojurescript require so that it will work to read the es6 file.
I want this to work everywhere, including shadow so js modules https://clojurescript.org/reference/javascript-module-support is not going to be an option.
I'm starting to dig into the cljs compiler for answers but if anyone can give me some pointers it'd be appreciated :wink:


Hi folks,

Here is a demo project which compiles some end user code using js-joda with the Closure compiler. You can see the output file from the compilation has eliminated much of js-joda - the output file is 3.9k - about 10x smaller than the full minified js-joda.

FYI The Closure compiler is used by the Clojurescript compiler, hence why this is interesting to me since I maintain a cross-platform java+js time library, which uses js-joda: https://github.com/henryw374/cljc.java-time

I'm not that familiar with Closure compiler, but from my experiments so far, it is only able to eliminate js-joda in it's original source form. If I give it the output of the umd or esm builds from rollup, it gets confused and includes everything.

Even though it has eliminated most of js-joda in the output, it still includes a few functions that the end user is not calling, (these are adjuster related, 'get' and 'query') so I am still experimenting to see if any changes can be made to the source to eliminate those in the output as well.

https://clojureverse.org/t/cljc-java-time-will-drop-all-npm-foreign-lib-dependencies/6208

cljc.java-time will drop npm/foreign-lib dependencies

This is a heads up that I am planning to drop the npm/foreign-lib dependencies from cljc.java-time and would like to hear any feedback anyone has at this point.

Doing this should mean that other libraries could now depend on this library because:

1) There will be no transitive dependency headaches for users (ie exluding foreign-libs if you have a :bundle cljs build etc)
2) End users will benefit from dead code elimination: If they are not using part of a library that uses cljc.java-time, then nothing of cljc.java-time library will be included in an advanced optimization cljs build.

IOW libraries can depend on cljc.java-time and get a banana without a gorilla holding on to it.

... and being cross-platform and using java.time on the jvm, I hope this library could become the de-facto platform library for Clojure and Clojurescript.

How will this work?

I am creating a custom build of the js-joda source that results in a single js file that does a goog.provide().

In order for the jsjoda addon libs (timezone & locales) to work they will also need a custom build.

I am assuming here that no one is using any js libraries that are using jsjoda - but if you are please shout. If people are then I could provide an option to use cljc.java-time with npm dependency instead.

The downside for me is that each release of jsjoda that people want will need a new release of this custom build. However,
 since jsjoda just replicates java.time and that is stable, I don't expect that many releases.
 
 I am not sure how long until this will be ready, but weeks rather than months hopefully.
 
 FYI I have gratefully received some funding from Clojurists Together for this work.

