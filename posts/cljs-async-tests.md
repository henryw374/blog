Title: Less Boilerplate in Cljs Async Tests
Date: 2019-09-04
Tags: clojure

I can mostly avoid async tests for my re-frame cljs apps. Even integration-style tests that 
use [enzyme](https://github.com/airbnb/enzyme) or [react-testing-library](https://github.com/testing-library/react-testing-library) 
can be made synchronous (if using Re-frame), by using `day8.re-frame.test/run-test-sync` and faking server responses 
with synchronous promises. This is nice because they're easier to understand and debug. 
Sometimes though, async is necessary. 

The Clojurescript site describes [how to do async tests](https://clojurescript.org/tools/testing) using the
`cljs.test/async` macro and Re-frame has a nice `run-test-async` wrapper around that. 

If you are using `cljs.test/async` you have to make sure your test code calls the `done` function in every case, including 
on errors, timeout etc. So to remove some boilerplate, here's a new [utility](https://github.com/henryw374/Cljs-Async-Timeout-Tests) which provides a macro like `cljs.test/async`, but which on timeout or uncaught failure (exception or failed promise), will fail the test, 
calling the `done` function for you.

Here's how you'd use it:

```
(ns myns.ns
  (:require [clojure.test :refer [is deftest]]
            [widdindustries.timeout-test :refer [async-timeout async-timeout-at]]))

(deftest my-test
  (async-timeout done 
     ;; do some stuff that will call `done` when it succeeds.
     ;; lib expects any async body will result in a promise
   ))
```  

If you have any feedback please comment or raise an issue.

Henry
