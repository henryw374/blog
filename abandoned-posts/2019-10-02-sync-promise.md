---
layout: post
title: Cljs Tests with Synchronous Promises
description: Jimmy Page takes time out to help us write synchronous tests
category: clojure 
---

Led Zepellin,  
Iron Butterfly, 
Quiet Riot, 
Synchronous Promise,  

... just saying those names makes me want to start playing air guitar!

In Clojurescript apps where you have some Promise-producing thing, like `(js/fetch "www.example.com")`, it's not uncommon to want to stub that out in tests. Since it's being stubbed out, you could use that as an opportunity to have your test be synchronous, by having the stub return a [Synchronous Promise](https://github.com/henryw374/cljs-synchronous-promise). 

A synchronous promise has the same API as a regular js promise, except that it is initialised with its resolved value and ... it's synchronous! That means you won't have to use [async testing](http://widdindustries.com/cljs-async-tests/).

Once created, a synchronous promise will work with your promise-using code, for example:

```
(require '[widdindustries.synchronous-promise :as sp])

(-> (sp/resolved :val)
    (.then (fn [v] [v]))
    (.catch (fn [_] (throw (js/Error. "not called"))))
    (.then (fn [x] (js/console.log x) x)))

```

[Promesa](https://github.com/funcool/promesa) is a Clojure(Script) library for working with promises. In order to use that with Synchronous Promise, run the following:

```
(require '[widdindustries.synchronous-promise :as sp])

(promesa.impl/extend-promise! sp/SyncPromise) 

```

# How it works

This was fun to write and doing this I learned that `deftype` is the [Clojurescript equivalent of writing JS contructor functions](https://github.com/clojure/clojurescript/wiki/Working-with-Javascript-classes) 

```
(deftype SyncPromise [ok? v]
  Object
  (then [_ f]
    (if ok?
      (try (SyncPromise. true (f v))
           (catch js/Error e
             (SyncPromise. false e)))
      (SyncPromise. false v)))
  (catch [_ f]
    (if ok?
      (SyncPromise. true v)
      (SyncPromise. true (f v)))))

(defn resolved [v]
  (SyncPromise. true v))

(defn rejected [v]
  (SyncPromise. false v))
```


# Postscript

As of this writing, the library only works with the `then` and `catch` methods in the api. If you need more, please PR, or use the much [more comprehensive JS library](https://github.com/fluffynuts/synchronous-promise#readme)
