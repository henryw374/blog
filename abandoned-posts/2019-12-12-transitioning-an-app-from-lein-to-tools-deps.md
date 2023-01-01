---
layout: post
title: Experience Report - Transitioning apps from lein to tools.deps
description: Rationale, process and surprises
category: clojure 
---

# Motivation

My current client has a handful of applications built with Leiningen, and as lein builds go, they're not too bad. They're generally 200-300 lines long and contain config for running up repls, testing, compiling Clojurescript, linting, uberjarring & etc. 

As I say, nothing too out of the ordinary: there's the odd bit of metadata like `^:replace`, some prep-tasks, a bit of inline `~(lein.core/something)` snippets, and some big-ish strings containing bits of edn config. Just typing out that last sentence, I feel like I'm describing something pretty ugly. Well, it is ugly, but it's also quite familiar at this point and stable enough that I haven't felt motivated to try a new approach. I've used Boot on previous projects and although the immutable filesystem thing is cool, I'm not desperate to go back to it.

[Tools.deps](https://clojure.org/reference/deps_and_cli) if you haven't drunk the kool-aid, the most appealing part of it for me was that it provides just enough scaffold to set up your classpath and run some clojure code. That's it. Leiningen can also do that of course, via `lein run ...`, but this is basically all tools.deps does, and all I really want a Clojure build tool to do. Simples!

So, technically appealing as tools.deps is, I still didn't have the impetus to move a bunch of working builds off leiningen. Getting back to the client, there's currently a handful of builds and a fair amount of duplication between them. Among other things this is code/config relating to deployment, logging, auth, health checking and so on. As each new app came along I thought about sharing some of this code and each time so far decided it was best to leave it and see how things evolved. With only a handful of apps and a smallish team it's easy enough to keep an idea of what's going on and see how the code develops in the different projects. Sharing any of this code with leiningen would mean creating maven libraries, which would work but to date has felt too rigid for this situation. There's also now plan to make many more apps, like 10x more, so the bearable pain of syncing changes across the apps is about to get unbearable. The killer feature of tools.deps here then is the non-maven deps: Git deps, local roots and (less relevant here) local jars. Now we can have many small libs, that don't need all the packaging and deploying rigmarole! With little in-house libraries like this it's common to want to develop the libraries in tandem with the client code andy with local roots, tools.deps provides a solid mechanism for doing that, superior to [lein-checkouts]().

# Migrating the build

Step 1 was to use tools.deps with leiningen. This handy [lein middleware]() defers to tools.deps for the dependencies. A simple bit of scripting ported the deps out of leiningen project.clj files and nothing else needed changing. Well, almost nothing. Tools.deps has it's own dependency resolution mechanism, different from leiningen (maven), so there were a few dependency related wrinkles to iron out at this stage. 

Step 2 was to start moving off lein plugins for things like linting, to doing `lein run` and directly calling clojure functions. We already had functions to do linting and testing from the repl, so really it was a matter of adapting things a bit to output in the right format for continuous integration, exiting with an error code when appropriate etc. 

Uberjar turned out to be more difficult than expected. There are several uberjar tools out there, but what I wanted was a tool to exactly replace `lein uberjar` - ie aot compile and have the output be runnable via `java -jar ...`. [Uberdeps]() and [pack] can do the uberjar bit, the aot compile we do in a script but it's easy enough and the Uberdeps readme explains how.

The client apps are mostly all http servers which serve up resources for an SPA and provide some of the api for those. So as you can imagine, the uberjar process needs to compile Clojurescript and Scss among other things. In the new world of tools.deps I'd imagined this should simply be a matter of composing some functions that do these steps. However, it's interesting to see various tools are geared toward being called via `-m some.main` on the command line, rather than programmatically. 

# Development Environment

The things I needed to set up here were:

1) Adding my favourite debugging tools into `~/.clojure/deps.edn`, which had been in `~/.lein/profiles`

2) Using `.nrepl.edn` config in place of lein `:repl-options`

3) Re-importing Lein Cursive projects as deps.edn ones and changing Intellij run configurations. This presented no real problems. Cursive doesn't yet support running arbitrary mains via `-m` aliases yet, but it's probably not far off.

# Where's `lein clean`




