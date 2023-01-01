Title: Clojurescript jar embeds some core libraries
Date: 2022-02-09
Tags: clojure

*Update:* As of mid-2022, Clojurescript has fixed the problems described in this post.

Why having clojurescript in the classpath may lead to unexpected behaviour

The [clojurescript maven artifact](https://mvnrepository.com/artifact/org.clojure/clojurescript/1.11.4) 
lists compile dependencies which include: data.json, tools.reader and transit-clj and transit-java. 

However the clojurescript jar itself is something like an uberjar: It includes compiled data.json, tools.reader and transit-clj and transit-java
namespaces inside itself. That means that although it declares dependencies on those libraries, when you
use Clojurescript yourself, those libraries' artifacts are not used at all (despite being on your disk and in the classpath). 

The output of this command shows the dependencies I am referring to:

```
clj -Sdeps '{:deps {org.clojure/clojurescript {:mvn/version "1.11.4" } }}' -Stree
``` 

The effect is that if you want to use a different version of one of those libraries 
compared to the one Clojurescript was compiled with, you can't. Well, you can't with the regular clojurescript
artifact, you can use the non-aot version: `org.clojure/clojurescript$slim {:mvn/version “1.11.4”}`.

This was not an issue for a long time because those libraries didn't change. Now e.g. clojure.data.json has
changed, hence why I hit the problem. 

A handy technique I usually use to answer the question 'where on the classpath is namespace x.y.z getting loaded 
from?' is to call `io/resource` on it. Doing that with data.json and Clojurescript in the classpath 
gives result as follows: 

```clojure 
 (clojure.java.io/resource "clojure/data/json.clj")
 
 => ".../.m2/repository/org/clojure/data.json/2.4.0/data.json-2.4.0.jar!/clojure/data/json.clj"]
```

which is the wrong answer! I still don't understand why io/resource shows the file there, whereas calling `require`
on that ns returns the one embedded in Clojurescript.

One might ask why I would be using Clojurescript and clojure.data.json together in the same jvm. 
Well, in my case, in development I tend to have my server and client dependencies combined, so 
I run cljs compile and server side stuff in one vm. When deploying and testing, I separate them 
(meaning Clojurescript jar is only on the classpath when cljs source is being compiled). It is possible to run separate server and cljs jvm's 
locally of course, but that then means I can't have a single .nrepl.edn file for example. There could be 
other reasons for using these 2 together though, writing data-reader functions that use json possibly.  

I raised this on clojure slack and now Clojurescript's maintainers are aware, so hopefully
this gets fixed. 

A fix is likely to involve `shading`. This is where a library wants to use a fixed version 
of another library, so it copies the sources of that library into itself, but changes 
the namespaces/packages of the source library to be something different, and specific to itself.

My thanks go to Alex Miller for explaining this.  
 