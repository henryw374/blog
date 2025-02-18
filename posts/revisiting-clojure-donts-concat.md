Title: Revisiting 'Clojure Don'ts : concat
Date: 2025-02-14
Tags: clojure

# Nostalgia City

I've recently started maintaining a Clojure codebase that hasn't been touched for over a decade - all Clojure devs that built and maintained it are long gone. It's using java8, Clojure 1.6 and libs like `korma` and `noir` - remember those? Contrary to the prevailing Clojure lore, upgrading Clojure will not be just a matter of changing version numbers in the lein project.clj.

I find one of the most dated aspects of the project is the laziness. I only use laziness as an explicit choice and have done so for many years. Laziness is a feature I find I rarely need, but is sometimes just the right fit. 

A lot of the original Clojure collection functions are lazy and it is still common to see new code written with them - I think because they are still seen as an idiomatic default, rather than a conscious choice. Non-lazy versions like `mapv` and `filterv` came later and transducers later still, but of course the old functions must continue to work as before.

Investigating a bug in the codebase led me back to this great blog post, [Clojure Dont's: Concat](https://stuartsierra.com/2015/04/26/clojure-donts-concat) also written around a decade ago. The rest of this post will discuss that post, so if you haven't please read that (and ofc the rest of the 'Dont's series is also good').

# Revisiting the post

I had first read the post many years ago and had forgotten the details - I guess, the main thing I remembered was 'don't use concat' - which is maybe a good heuristic but actually missed the main point which could be phrased as `build lazy sequences starting from the outside` - I'll explain the outside thing further on. 

Reading it again, I had go over it couple of times to fully understand it - if it was crystal clear to you then you've no need to read on. To check your understanding - answer this: what difference would it make (wrt overflow) to change the order of the args to `concat`  in the `build-result` function?

Following is my attempt to make the post's message even clearer.

The post mentions that `seq` realises the collection and causes the overflow. Just in case it is not clear, `seq` does not in general realise lazy collections in entirety, it just realises the first element.

To demonstrate that, have a look at the following, which is like `range` but the numbers in the sequence descend to one :

```clojure

(defn range-descending [x]
  (when (pos? x)
    (lazy-seq
      (cons x (range-descending (dec x))))))

(let [_ (seq (range-descending 4000))]
  nil) ; => ok, no overflow

```

This is what one might call an `outside-in lazy sequence`. As the sequence is generated, one might picture it like this:

```clojure
(4000, LazySeq-obj)
(4000, 3999, LazySeq-obj)
(4000, 3999, 3998, LazySeq-obj)
...
```

Calling seq on the collection, only the first element is realized, so no overflow.

The equivalent to the way `concat` was used in the original post would be more like this:

```clojure
  (defn range-descending-ohno [x]
    (when (pos? x)
      (lazy-seq
        (conj (range-descending (dec x)) x))))
```

Now visualising the sequence generation, it would look more like this:

```clojure 
(conj LazySeq-obj 4000)
(conj (conj LazySeq-obj 3999) 4000)
...
(conj `...` (conj nil 1) `...` 4000)    
```

Now when calling `seq` (as in `(seq (range-descending-ohno 4000))`), the whole sequence needs to be realised for `seq` to get to the first element (4000 in the example). As the post says: `seq has to recurse through them until it finds an actual value`. One might call this an `inside-out lazy sequence`. 

# Conclusion

The original post concludes `Don’t use lazy sequence operations in a non-lazy loop` - which I would update to add `don't use laziness at all unless required`.

If deciding to use laziness, avoid building sequences inside-out - this might be in your direct usage of e.g. `lazy-seq` or hiding in plain sight in your usage of e.g. clojure.core functions such as concat.


# Further Reading

* The inside-out lazy seq topic is also covered in [Clojure Brain Teasers](https://pragprog.com/titles/mmclobrain/clojure-brain-teasers/) if you want more pictures and explanation (in Boom Goes the Dynamite chapter).
* [Clojure's Deadly sin](https://clojure-goes-fast.com/blog/clojures-deadly-sin/) is a very well considered and comprehensive look into the problems of laziness in clojure.