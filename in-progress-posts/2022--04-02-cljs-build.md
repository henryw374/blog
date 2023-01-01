

## Specific problems this aim to address

I have used many Cljs build+test configurations over time and here are some of the issues I have had:

* Hard to test `:advanced` compiled code. tl;dr if you release code having compiled it with
  :advanced opt, it should be tested under `:advanced` as well. This applies to library development as well of course.
  This is especially important if your code
  is doing interop, but also because [not all valid Clojurescript will survive `:advanced`](https://clojure.atlassian.net/browse/CLJS-3315).

* Not REPL friendly. Having a CLI is good if you want flexibility to do adhoc stuff at the command line, but
generally I want that flexibility at the REPL. For command-line-only things like Continuous Integration (CI) I'm not
looking for adhoc flexibility and invoking a clojure function via Clojure CLI will be fine.

* Lots of boilerplate/config-files for Cljs or tests. This continues the previous point. The contents
  of config files like  `shadow-cljs.edn` or `tests.edn` will just become arguments to Clojure functions
  invoked by the CLI. Since I'm calling the functions directly, not via CLI, I don't need the config files.
  Config files often contain needless repetition or [provide special workarounds](https://shadow-cljs.github.io/docs/UsersGuide.html#config-merge)
  in order to make them dynamic. This is not necessary if doing everything from the REPL.

* Multiple processes/jvms. Running Shadow-CLJS and Kaocha-cljs2
  (with Funnel), and my own app server, via CLI, I might end up running 3 jvms, but via REPL, only one jvm is needed to do all this.
  I can split things into separate processes if I want to, but generally having more than one just means more to think about and
  manage.

* Having to individually understand how to watch+repl+test every single open source Clojurescript project I come across.
  There are many ways of setting up
  a Clojurescript project, but essentially when I open up someone else's project, as long as it has a `deps.edn` file, I want to be
  able to use my own watch+repl+test config with it, without changing the project at all. At the end of the day every Clojurescript project
  is just some source code and maybe deps - I want to be able to treat them all the same.

* Github actions CI setup. I have failed to find a single Clojurescript open source lib on Github that uses Kaocha-cljs2 and
  github actions. The Cljs projects I maintain will be the first then possibly - and they should all require close to zero
  individual configs to do so.

* Try to avoid any confusion arising from 'global install' of shadow or other npm things. Working out what version of things you
  have is hard enough already. 
