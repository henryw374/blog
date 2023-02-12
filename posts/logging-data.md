Title: Log data, not strings - with SLF4J
Date: 2023-02-09
Tags: clojure

A recent release of the popular Java logging abstraction SLF4J has a new API enabling [structured logging](https://stackify.com/what-is-structured-logging-and-why-developers-need-it/). New Clojure logging macros using this, e.g. `(log/info "request-for-help" {"priority" "high"})` are available as [slf4clj](https://github.com/henryw374/slf4clj).

# Why Log data?

A couple of years ago I made a survey of all the logging libraries one might use from Clojure. The majority of contenders ([clojure.tools.logging](https://github.com/clojure/tools.logging) foremost among them) I ruled out fairly early because they only log strings, not data.

To explain this limitation, in all logging frameworks you can format messages as
json or something and get `{level: "INFO", message "Commissioner Gordon called because Gotham city is under attack from the Joker"}`. Whilst useful, what is more queryable is to log arbitrary data (key value pairs) and have that data passed as-is to appenders, which might serialise to json, or write to a database. An equivalent message formatted as JSON might be `{level: "INFO", message-type: "request for help", urgency: "high", caller: "Commissioner Gordon", foe: "Joker", target:"Gotham" }`. Assuming you're not still shelling into boxes and grepping log files, IMO [structured log data](https://stackify.com/what-is-structured-logging-and-why-developers-need-it/) is something you can't go back from.

# Options for Logging data from Clojure

The requirement to log data left two main contenders, [MuLog](https://github.com/BrunoBonacci/mulog) and [Log4j2](https://logging.apache.org/log4j/2.x/). At the time I decided Log4j2 seemed like 
the boring, safe choice. Well, that didn't turn out to be quite right haha! 

As a result of opting for Log4j2, I put some [helper functions in a lib](https://github.com/henryw374/clojure.log4j2) for Clojure users writing log statements against Log4j2. Note: the [README](https://github.com/henryw374/clojure.log4j2) for those contains more detailed comparison of existing Clojure logging libraries.

Released since I made that review, and created as a result of log4shell is [Amperity's Dialog](https://github.com/amperity/dialog) which is a logging backend for [slf4j](https://www.slf4j.org/)(1.x, string based) - the de facto Java logging facade. Dialog also provides a
[Logging API based on logging strings](https://github.com/amperity/dialog/blob/main/src/clojure/dialog/logger.clj) or suggests you use clojure.tools.logging (strings again).

# Logging from Library Code

Log4j2 is a 'logging implementation' or 'backend'. Ideally one would write log statements against a logging abstraction, where log statements get channeled to whatever logging backend is in place. This is especially important when writing library code. Users of e.g. [Carmine](https://github.com/ptaoussanis/carmine) will find logs coming via Timbre whether they like it or not. 

Looking at the options here, assuming we want to log data ofc, MuLog might be a choice. It has be made to plug into an [slf4j 1.x backend](https://gitlab.com/nonseldiha/slf4j-mulog), so surely can be made to plug into other things.

The most obvious choice though is [SLF4J](https://www.slf4j.org/), apparently [the most popular Java library of any kind](https://en.wikipedia.org/wiki/SLF4J). The 1.x version of this has been around for a long time and as you'd expect from an API dating from the noughties it only logs strings. With the 2.0 release, that has changed.

# Logging data with Slf4j 

Enter slf4j 2.0 - which was released toward the end of 2022. The 2.0 version of this popular logging facade newly includes an [API for logging data](https://www.slf4j.org/manual.html#fluent), whilst remaining backwards compatible with the 1.x API.

The fluent API contains the `addKeyValue(String key, Object value)` method for structured logging. It's up to implementations as to what to do with the structured data. They may merge it to the [MDC](https://www.slf4j.org/api/org/slf4j/MDC.html) for that message for example as Log4j2-slf4j bridge does. The MDC is a map of String->String though, which is a problem if the value happens to be anything other than a string.

It is straightforward to use from Clojure as-is, but there are some convenience macros released as a new library [slf4clj](https://github.com/henryw374/slf4clj). The aim is not to re-create the whole API, just offer some shorthand for the majority of use cases.

As you'd expect there are macros for `debug`, `info`, `warn` etc and the args for each are deliberately the same as Clojure's [ex-info](https://clojuredocs.org/clojure.core/ex-info), namely 
`(<level> msg map)` or `(<level> msg map cause)`.

Here is an example:

```clojure
(require '[com.widdindustries.slf4clj.core :as log])

(log/info "request-for-help" {"urgency" "high", "caller" "Commissioner Gordon", "foe" "Joker", "target" "Gotham"})
```

## Migration path

If you're using clojure.tools.logging, you can keep your existing setup and just start writing slj4j 2.0 logging statements and that will likely 'just work' in that the data will get printed out in some string format according to your pattern config). Separately you can change your logging backend to 
something that does more with structured logs than just turning them into strings, like writing them as JSON for example.

# Logging APIs bundled with the JVM

If you're logging from a library it is possible to avoid having any logging dependency by using APIs included with the jvm.

java.util.logging (JUL) is the one you have most likely heard of. As you might guess though for something created in the early part of this century, it is a string-based logging API. [This question on Stackoverflow](https://stackoverflow.com/questions/11359187/why-not-use-java-util-logging) goes into some details about JUL but in the threads is mention of a newer platform Logging facade called [System.Logger](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/System.Logger.html) - which might be interesting but doesn't seem to have gained traction AFAICT.

# References

* [Better Java Logging](https://mccue.dev/pages/12-3-22-better-java-logging-2)

# Unrelated FYI

This is my first blog post since moving to [Quickblog](https://github.com/borkdude/quickblog) - a blogging tool powered by Clojure - thanks Borkdude!

