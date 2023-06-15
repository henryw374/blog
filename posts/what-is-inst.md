Title: What is #inst ? 
Date: 2023-06-14
Tags: clojure,date-time

This post looks at the meaning of the `#inst` reader literal from [Extensible Data Notation](https://github.com/edn-format/edn) (hereafter referred to as 'edn'), how it behaves by default in Clojure(script) and when it might not be sufficient for representing date/time information.

The majority of the content of this post comes from the Rationale section of [time-literals](https://github.com/henryw374/time-literals), a Clojure(Script) library which provides tagged literals for java.time objects.

## What is #inst ?

Support for edn and its Reader Literals were a headline addition in [Clojure 1.4](https://github.com/clojure/clojure/blob/master/changes.md#21-reader-literals) and with that came built-in support for the [#inst tag](https://github.com/clojure/clojure/blob/master/changes.md#211-instant-literals). The `#inst` tag is a part of the edn spec, where it is defined as representing [an instant in time](https://github.com/edn-format/edn#inst-rfc-3339-format), which means a point in time relative to UTC that is given to (at least) millisecond precision. The format of `#inst` is RFC3339, which is [like ISO8601 but slightly wider](https://medium.com/easyread/understanding-about-rfc-3339-for-datetime-formatting-in-software-engineering-940aa5d5f68a).

In Clojure(script), `#inst` is read as a legacy platform `Date` object by default, but as is made clear by the edn spec and by [this talk from Rich Hickey](https://github.com/matthiasn/talk-transcripts/blob/master/Hickey_Rich/AreasOfInterestForClojuresCore.md#extensible-reader) the default implementation is just that: `#inst` may be read to whatever internal representation is useful to a consuming program. For example a program running on the jvm could read `#inst` tags to java.time.Instant (or java.time.OffsetDateTime if wanting to preserve the UTC offset information). It seems to me unfortunate that Clojure(script) provided defaults for `#inst` because users may not realise it is 'just a default', but that's just my opinion. My guess is that Clojure is trying to be both simple and easy in this case.

Although edn readme doesn't say this explicitly, to avoid 'reinventing the wheel', when conveying data using edn format, built-in elements seem to me to be preferable to user defined elements. For example, if one wants to convey a map, `{:a 1 :b 2}` is preferred to `#foo/map "[[:a 1] [:b 2]]"`  - unless of course one wanted to convey something additional about the map, ordering perhaps. Similarly, if conveying an `instant in time` use `#inst`.

## When the default is not enough

There are two situations where reader literals are useful:

1. Conveying `edn` data between processes
2. REPL I/O (iow "working at the REPL")

Although they have many similarities and overlap, Clojure allows these cases to be considered separately and for good reason, as explained below: 

### The need for more Tagged Elements representing Dates in edn

There are many kinds of things relating to date and time that are not an `instant in time`, so `#inst` would not be an appropriate way to tag them. For example the month of a particular year such as 'January 1990' or a calendar date such as 'the first of June, 3030'. There are no built-in edn tags for these but tags can be provided in the user space, as they are by the [the time-literals](https://github.com/henryw374/time-literals) library.

Note that the default Clojure reader behaviour is to accept partially specified instants, such as `#inst "2020"` (and read that to a Date with millisecond precision) - but this is specific to the Clojure implementation and not valid edn (ie not RFC3339).

### Round-tripping at the REPL

Clojure provides two mechanisms for printing objects - abstract and concrete as this code printing the same object shows:

```clojure
(let [h (java.util.HashMap.)]
  {:abstract (pr-str h)
   :concrete (binding [*print-dup* true]
               (pr-str h))})
=> {:abstract "{}", :concrete "#=(java.util.HashMap. {})"}
```

The concrete representation is sometimes useful to know and also the string output can be passed back to the reader to recreate the same internal representation again, which is known as `round-tripping`.

The default readers and printers of platform date objects don't allow round-tripping, [the reason for which is unknown](https://ask.clojure.org/index.php/11898/printing-and-reading-date-types).

This is relevant to the two java.time types which logically correspond to `#inst` (java.time.Instant and java.time.OffsetDateTime). The [the time-literals](https://github.com/henryw374/time-literals) library contains specific readers and printers for those objects so that they do round-trip. 

When conveying these objects in edn format, they should be tagged as `#inst` (as per above argument about preferring built-in elements). To do that with `time-literals`, simply provide your own implementation of `clojure.core/print-method` for Instant and/or OffsetDateTime. With `*print-dup*` true, the concrete type will still be printed.

## When reader literals are NOT useful 

Consider this code from a Clojure namespace:

```clojure
(ns foo.bar)

(def one-day #time/period "P1D")

(defn one-more-day [period]
  (-> period (.plusDays 1)))

```

Now answer:
1. Will it compile?
2. If it can be made to compile, will `(one-more-day one-day)` work?

Go back and have a look if required, I will reveal the answer in the next sentence

The answer to 1. is maybe, ie only if `*data-readers*` contains a mapping for `time/period` AND the reader function is already loaded in the process. Just having a mapping in data_readers.cljc is not enough. Add a side-effecting require for that reader function you say? No thanks.

The answer to 2. is again maybe. If the mapping for `time/period` is set up AND the reader function returned a java.time.Period then it will work. 

So tl;dr reader literals in code can be made to work but is not good practice IMHO. That goes for user-defined literals but also `#inst` and `#uuid`. Typing a few extra characters to call the actual constructor function directly is not so hard. 

I don't mean I never have files with literals in them, but not in code I expect anybody else (incl. myself at a later date) to just be able to 'pick up and run'. If I'm flowing around in my own space then it's fine. If I get to 'crystalising stuff out' - e.g. to tests for CI, then I replace any literals.

Btw if you want to do a find/replace for `#inst` in your source files then `clojure.instant/read-instant-date` or `cljs.reader/parse-timestamp` are probably the functions you need ;-)



