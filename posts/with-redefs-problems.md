Title: When to avoid with-redefs
Date: 2019-01-07
Tags: clojure

[with-redefs](https://clojuredocs.org/clojure.core/with-redefs) is a handy clojure.core function to use when you want to redefine one or more vars temporarily (within a block) and you want these redefinitions to apply across thread boundaries

An alternative for doing something similar is [with-bindings](https://clojuredocs.org/clojure.core/with-bindings), and that is slightly different in that
the new bindings are only seen within the context of the current thread. If for example, you use with-bindings on a var and the value of the var is referenced when doing a `clojure.core/map` operation say, then it is possible you won't see the temp binding, since `map` is lazy and may be executed in a different thread.

Also, `with-bindings` requires the rebound vars are dynamic. 

So... with-redefs is generally a more powerful go-to tool than with-bindings?

No, `with-bindings` is more generally used on vars that are planned to get rebound, whereas `with-redefs` is intended when you want to change the way things normally work, for example when you're running a test and decide you don't really want to call an external system, and can use with-redefs to stub.

However, beware that `with-redefs` is not foolproof. Consider rebinding a function foo

```
(with-redefs [a.b.c/foo my-temp-fn]
  ... body that calls foo at some point, possibly from another thread)
```

Will the body here always see your temp binding?

**No**

Why not?

Well, interleaved threads is one situation. 

Let's say this code is called from two threads and this is the order of events:


Thread A executes the `with-redefs`

Thread B executes the `with-redefs`

Thread A executes the body and exits the `with-redefs` block (thereby restoring the root binding of foo)

Thread B now executes the body and does not see the temp binding!!

The docs for `with-redefs` say this is handy for tests, which implies that if you want to change a var in non-test code, then `alter-var-root` (assuming it is done once, on startup) is the way to go. 

However, there's nothing to stop you hitting an interleaving problem in tests unless you are running your tests serially and the with-redef wraps the test body. There isn't a `with-redefs-visible-only-via-this-block` or equivalent.

### Conclusion:

Use `with-redefs` cautiously. Remember this blog post when scratching your head about why some code is not seeing a temp binding.

