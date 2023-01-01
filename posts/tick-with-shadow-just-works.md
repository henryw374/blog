Title: Clojure(Script) date-time libs, Shadow-cljs & Babashka
Date: 2020-05-31
Tags: clojure

# Time libs on npm/Shadow

[tick](https://github.com/juxt/tick) is a Clojure(Script) library for working with time. [cljc.java-time](https://github.com/henryw374/cljc.java-time) is used by tick and provides a cross-platform version of the java.time api. 

In the latest releases, these 'just work' on Shadow-cljs, for example:

```
echo '{:deps { tick {:mvn/version "0.4.25-alpha"} thheller/shadow-cljs {:mvn/version "2.9.8"} }}' > deps.edn
echo '{:deps {}}' > shadow-cljs.edn
echo '{}' > package.json

npm install shadow-cljs --save-dev && npx shadow-cljs browser-repl
```

In the repl:

```
cljs.user=> (require '[tick.alpha.api :as t])
nil
cljs.user=> (t/today)
#time/date "2020-05-31"

```

Why is this news? Well, previously Shadow and other npm-using Clojurescript users of Tick/cljc.java-time had to include an extra shim library and manually include the js-joda dependencies in their package.json. A desire to move to the latest (and now scope-named) '@js-joda/core' packages, [a fix in the latest Clojurescript release](https://clojure.atlassian.net/browse/CLJS-3138) and an increase in my understanding of [how to package Clojurescript libraries](http://widdindustries.com/cljs-npm-libraries/) has resulted in these improvements. Note, there are still a few [need-to-knows for Clojurescript users](https://github.com/juxt/tick/blob/master/docs/cljs.adoc), but not too many!

# Babashka

In other news [cljc.java-time](https://github.com/henryw374/cljc.java-time) now works on a third platform, [babashka](https://github.com/borkdude/babashka/)!
