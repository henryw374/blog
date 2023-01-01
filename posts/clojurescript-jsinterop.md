Title: Accessing Javascript objects from Clojurescript
Date: 2021-03-01
Tags: clojure

There are choices as to how you do Clojurescript `interop` (
accessing a Javascript object's methods and properties)
and that's what I'm going to look into here.

IMHO Clojurescript is somewhat lacking when it comes to official documentation, hence this blog post, and the need to quote from twitter:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">In concrete terms, sounds like<br>✓ (.-length &quot;abc&quot;)<br>X (.-length <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3})<br>✓ (goog.object/get <a href="https://twitter.com/hashtag/js?src=hash&amp;ref_src=twsrc%5Etfw">#js</a> {:length 3} &quot;length&quot;)</p>&mdash; Mike Fikes (@mfikes) <a href="https://twitter.com/mfikes/status/882585745424338944?ref_src=twsrc%5Etfw">July 5, 2017</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

I'm going to look into the tradeoffs of what David and Mike say there.

Firstly, I'll try to make a clear distinction between JS data vs API: A data object is 
any object you could round-trip through JSON/stringify => JSON/parse. An object that may appear 
 to be a data object because it only contains properties (ie no methods), may not be because those
properties might be [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)s or setters
(e.g. the `length` property of String "foo" referred to in the twitter post is a `getter`).

If an object came to your program via a call made to a JS library or API, it's most likely not a data object
unless the library documentation explicitly says so. 

Secondly, be aware that when doing advanced compilation with Clojurescript, the compiler will change all the variable and function
names in your program to reduce overall build size. This works automatically for regular Clojurescript
code, but when doing `interop` extra configuration is sometimes required which takes
the form of type hints in the code or externs files.

Ok, with those two points in mind, let's proceed.

# Dot-access vs goog.object/get

I find the `(.-length "foo")` item in Mike's list interesting. According to the advice, we would also choose
`(.-length #js[])` when using the API of Javascript arrays.

Consider the alternative: 

```clojure
(goog.object/get #js[] "length")

; => returns 0
```

This demonstrates it is possible to use `goog.object` to access API properties. So, we have what appears to be just a stylistic choice between this and `(.-length #js[])`. 

Why choose either one? 

Firstly, note that `length` is a special-case property name: `length` never needs type hinting, because 
the names of properties that are part of the in-built JS or DOM API objects are always specifically
left alone by the Clojurescript compiler. That works whether we are referring to the `length` property of a
Javascript array, or the `length` property of some JS object you defined yourself: however it appears, 
if the property name appears in the core JS API, it is left untouched.

So to be more general, let's consider a property name that does not appear in the standard JS API, `xxx`:

So, IOW, working with the API of foo, we would have a choice of: `(.-xxx foo)` vs `(goog.object/get foo "xxx")` 

The goog.object one will survive advanced compilation (meaning it is compiled to `foo.xxx` or equivalent), 
whereas the dot-access version will need a type hint (as in `(.-xxx ^js foo)`) to avoid `xxx` 
being renamed under advanced compilation. 

So +1 for the goog.obj approach so far because less config is required.

However, in working with the API of some JS object, it's quite common to both access properties/getters and call methods:

```clojure
(let [foo ^js (some-fn)
      bar-prop (.-bar foo)]
    (.methodFoo foo (inc bar-prop)))
```  

We could have accessed `bar` with `goog.object/get`, but
this code is being consistent in using only dot-access for the API of `foo`. We could also have accessed `methodFoo` with 
goog.object/get (and then invoked it), but that would look pretty ugly.

So, in sticking to David Nolen's advice `dot access only for APIs` this code makes a clear statement 
that it is working with API of `foo`, not a data object called `foo` - regardless of whether it is methods, properties or both, that we need to 
use. This comes as the cost of having to remember to put type hints in. 

If we forgot to type hint `foo` in that example, the properties `bar` and `methodFoo` would be renamed and 
the code would fail at runtime. For this reason, if you use advanced compilation, you must test your code 
having advanced compiled it first. Doing advanced compilation is slow, do during development I generally avoid it, but continuous
integration tests and beyond should use the same compilation level as production.

If Google Closure did become able to optimise regular JS code (ie code that is now `foreign`), 
then code that is using strings to access JS APIs (`goog.object/get` & equivalent libraries ) would
then be broken.

I've come to think
type hints aren't so bad, because now [type hints are documented](https://code.thheller.com/blog/shadow-cljs/2017/11/06/improved-externs-inference.html)
I think it's easy enough to understand you just need to add `^js` when you first see the js object in scope.

# Conclusion 

I prefer dotted access for API access because it is stylistically distinct from data access, which makes my code easier to 
understand. It also future-proofs it against improvements in the compiler. This comes at a cost of having to add type hints,
but that cost is low and is mitigated by running tests under advanced compilation.

# More Choices
 
There are libraries that have been created for working with JS APIs such as [js-interop](https://github.com/applied-science/js-interop).
These aim to provide a trade-off between dotted access and goog.object/get in that type hints are not required, but aim to provide a
nice, straighforward syntax. Personally I don't use these libraries because I think regular dotted access is good enough.

If working with JS data, ie not API, then an alternative to `goog.object` is [Cljs Bean](https://github.com/mfikes/cljs-bean)
 
# Final Thoughts

So, now that's all cleared up, which [dot-access](https://cljs.github.io/api/syntax/dot) is preferred, `a.b.c` or `(.. a -b -c)` ...?

Dot access in the style of `a.b.c` does have [an issue](https://clojure.atlassian.net/browse/CLJS-3315),
which is a shame (until fixed) because to me that seems pretty tasteful.

Would you like to see official documentation on this topic? 
If so, raise an issue [here](https://github.com/clojure/clojurescript-site/issues)

Thanks to Thomas Heller for providing the point about future-proofing.
 
                                                          