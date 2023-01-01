---
layout: post
title: Tagged Literals & Readers Explained 
description: 
category: clojure
---

https://github.com/noprompt/meander/blob/8c0e9457befea5eee71a94a6d8726ed5916875dc/src/data_readers.cljc
https://github.com/clj-commons/ordered/blob/caec8d9f3ee690f4a483e14cd3c55acee97e7daa/src/data_readers.cljc
https://github.com/sicmutils/sicmutils/blob/348e4715b49c6057a9d4bd2629892d2bc7818464/src/data_readers.cljc
https://github.com/henryw374/js-literal/blob/00177bfbbe01749ae2e7ec4b69bb27cc201bcd3b/src/data_readers.cljc
https://github.com/clojure/data.xml/blob/12cc9934607de6cb4d75eddb1fcae30829fa4156/src/main/clojure/data_readers.cljc
https://github.com/juxt/reap/blob/29ffc8664df26041ebd93a53f009d2606d1a5b6c/src/data_readers.cljc

https://clojure.atlassian.net/browse/CLJ-2224 - print java.time.Instant as #inst
cljs.instant adds this. so if you have cljs in classpath then you have it

Alex Miller: data readers should read literals and one should not use reader functions for macro-like behavior.

Reader Literals were the [headline new feature in Clojure 1.4](https://github.com/clojure/clojure/blob/master/changes.md#21-reader-literals)
 in Clojure so that when data is conveyed in and out of processes
 
Tagged Literals are interpreted when reading:

* Source code: The Reader as part of compilation (Read + Eval)
* Data: from a file, wire etc.

Readers:

clojure.core/read
    implementation is clojure.lang.LispReader - reader used by clojure compiler for source code 
    basically deprecated for users.
cljs.core/read
    just uses clojure.edn under hood
clojure.edn
    what you'd use to read edn, if in clojure
clojure.tools.reader
    lib for reading clojure (not edn subset)
    clojurescript compiler big user of this
 
 
# What is `#inst` ?

Mention in Steve's talk about the meaning or API of a literal. Let's look at `#inst`.compilation

As described in the [edn specification](https://github.com/edn-format/edn#inst-rfc-3339-format), 
an inst tagged literal represents a point in time, represented in RFC-3339 format.

By default, Clojure readers will read this as a plaform Date object (java.util.Date, js/Date). 

A full RFC-3339 `date-time` has a date and a time (optionally with fractions of a second) and an unambiguous UTC offset.
The default Clojure readers go beyond the RFC/edn spec in interpreting partial versions of this though, so `#inst"2020"` will be 
read as `#inst"2020-01-01T00:00.0Z"`.     
 
By default, `Date` objects print as `inst` tags and so do instances of `java.util.Calendar` and `java.sql.Timestamp`. 

Those other objects also represent points in time, with 0 UTC offset.

RFC-3339 can retain offset, but the default reader loses that info on reading. 


asymmetric
print-dup 
round-tripping

printer, reader
    prn + print-dup
        printing for reading by reader
        *print-readably*
        *print-dup*
            same type after print and read
                may include eval strings
                    #=()
    print + print-method
        printing for reading by humans


https://medium.com/easyread/understanding-about-rfc-3339-for-datetime-formatting-in-software-engineering-940aa5d5f68a 

asking about print-dup and dates:
https://clojurians.slack.com/archives/C03S1KBA2/p1652964199374829
https://ask.clojure.org/index.php/11898/printing-and-reading-date-types

(let [d (java.sql.Timestamp. 1)]
(= d
(read-string
(binding [*print-dup* false]
(pr-str d)))))
 
Of course this is done in such a way as to not require

# Should a library provide data_readers.cljc 

In this [2012 Talk on Tagged Literals](https://www.infoq.com/presentations/Clojure-Data-Reader/) 
Steve Miner says that a library should not provide data_readers.cljc because that will bind tags to 
functions in a way that the user cannot control. 

but Clojure binds #inst. Why is Clojure allowed to do this, but not a library? 

Datomic provides `#db/id` tags. 

We can change how #inst is read in source code during compilation.

what does deja-fu do with goog.date ? cant use #inst?

why does clojure.edn allow forms to be returned by reader fns, but clojure.core/read does not?

 

