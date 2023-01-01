Title: Wrapper-free Firebase with Clojurescript's Re-Frame
Date: 2021-02-19
Tags: clojure

I recently made my first crud-style hobby app with Firebase and 
I wanted to write it in Clojurescript.... of course ;-)

Looking around the internet for pointers, I found some Clojurescript-Firebase wrapper libraries I am 
not too keen on, and no great demo apps either... so I created [my own demo 'todo-list' app](https://github.com/henryw374/firerebase-clojurescript-todo-list)

The README there contains a small list of instructions that should get you up and running quickly and deploying the 
app to the internet in no time. Then you can spend many happy hours building from there.

# Firebase Wrappers?

[Googling for 'Clojurescript Firebase'](https://lmgtfy.app/?q=clojurescript+firebase) I found a couple 
of 'wrapper libraries', [re-frame-firebase](https://github.com/deg/re-frame-firebase) 
and [cljs-firebase-client](https://github.com/fbielejec/cljs-firebase-client). 

I've [written before](https://widdindustries.com/why-not-interop/) about
wrappers - tl;dr approach with caution. 

There are obvious issues with the wrappers mentioned above: 

* They're made to work for now-old versions of Firebase.
* They tie in to specifics like shadow or cljsjs
* They introduce some new stuff I have to understand 
* ... and ofc they don't have the glitzy docs and tutorials that the Firebase site does. 

The [cljs-firebase-client](https://github.com/fbielejec/cljs-firebase-client) lib
is billed as a library, but it looks very unfinished. It might have some snippets you 
could borrow, especially if you want to use Shadow.

The [re-frame-firebase](https://github.com/deg/re-frame-firebase) lib has an api to cover pretty 
much all of Firebase I think, so there's a lot of code. It also has some dependencies I'm not sure 
I want:

* Re-frame wrapper lib (iron) 
* clojure.spec

and of most concern is way state from Firebase (auth-state, database contents) is stored in 
the Re-frame 'app-db'. The authors note this is [a potential bug](https://github.com/deg/re-frame-firebase/blob/ed5eabb6caa404af3395e3c97cb59a7c4d302c6b/src/com/degel/re_frame_firebase/database.cljs#L56)
and indeed it is. It's also not necessary, as I'll demonstrate below.   

# Wrapper-free Firebase

The Firebase docs are great and got me a non-cljs 'hello, world' very easily. Could I bring in 
Clojurescript without introducing much new complexity? Well, I've attempted that in the todo app - see what
you think.

One thing you'll notice with Firebase is that you pick and choose the APIs you want to include. For example 
if you want to use a database,
you get a choice of two different ones. This is nice and in my todo-list app, I use just 'auth' and 'realtime database',
so that's all you'll see any code for. 

## Auth with Re-Frame

The todo app requires users to authenticate with a Google login. 

Firebase has a simple api to trigger this. The interesting bit from a Clojurescript point of view
is listening for the user information once they have successfully authenticated:

```clojure
(defn user-info []
  (let [auth-state (r/atom nil) ]
    (.onAuthStateChanged (auth)
      (fn [user]
           (reset! auth-state (user->data user))))
    auth-state))
``` 

This function that returns a Reagent atom that will contain user information when it is available.
We can call this function any time and use the result in a reactive context, such as in a Reagent component -
It doesn't matter if the user has already authenticated or not at the time the function is invoked. 

There's no need to store the user data in the app-db. Doing so would leave us with more state to 
manage, clean up and so on. There's no Re-frame involvement required, but we could tie this 
function into a re-frame subscription as I'll demonstrate next.

## 'Realtime database' with Re-Frame

Pushing data to the database is pretty straightforward and well documented and you can imagine how
those calls might be wrapped up as [Re-Frame 'effects'](https://day8.github.io/re-frame/api-builtin-effects/#what-are-effects),
as they have been in the 'todo' demo app.

Listening for data with Re-frame is a bit more interesting though. Similar to the atom that contained
the auth data above, we can have a function to return a Reaction (same thing as reactive atom effectively) 
containing the current value at some path in the Firebase database:

```clojure
(defn on-value [{:keys [path]}]
  (let [ref ^js (db/fb-ref path)
        val (r/atom nil)
        callback (fn [x] (reset! val (->clj x)))]
    (.on ref "value" callback)
    (reagent.atom/make-reaction
      (fn [] @val)
      :on-dispose #(do (.off ref "value" callback)))))
```

The 'Reaction' object returned from this function will be updated with the current value whenever 
it changes - nice! Now, if we want to use that as part of a Re-frame subscription, we can call it 
from a [Signal function](https://github.com/day8/re-frame/blob/2965ffeda9b8f3b687e2c3e0ba9a62e7fe64c0bb/src/re_frame/core.cljc#L201)

```clojure
(rf/reg-sub ::foo 
  (fn this-is-the-signal-function [[_ args]]
    {:bar (on-value args)})
  (fn this-is-compute-function [{:keys [bar]}]
     ;... do some further transform with 'bar' value from db
  ))
```  

If you haven't used signal functions before, it's well worth a read of the [hefty docstring](https://github.com/day8/re-frame/blob/2965ffeda9b8f3b687e2c3e0ba9a62e7fe64c0bb/src/re_frame/core.cljc#L201) 
to understand them. The todo-list app demonstrates this in action.

So... we can read and write data, and the Re-frame 'app-db' is nowhere in sight. I haven't got 
anything against the app-db - but I don't want to stuff in there unnecessarily - because for any 
data in there, you have to understand what effects put it there, how it's lifecycle is managed and so on. I might write more about this in a later post.

Firebase stores the data in the cloud and keeps a local copy of that sync'ed in the [browser's store](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Client-side_storage) in 
case the connection drops. Re-frame handles doing minimal computation work, de-duping subscriptions etc. 
So... let's just lean on all that awesome machinery!

What about data the user is editing? In the todo app, that is component-local state only when 
the user is actually changing it, and is sent off to Firebase via `re-frame/dispatch` at
the appropriate time.

In fact, in this little example you might see Re-Frame as overkill and just stick with Reagent.

One last point about the Firebase database - it's json data of course, so it's not going to work to 
create Clojure maps with the usual edn goodies like 
non-string keys, namespaced keywords & etc. A nice approach to stay in JS/JSON
land is to use [cljs-bean](https://github.com/mfikes/cljs-bean) for your data, rather than native 
Clojure datastructures.

## Compile and Deploy
 
The todo-list README demonstrates doing a build with advanced compilation.
  
The build includes the `:infer-externs` option, which uses the `^js` [tag metadata](https://code.thheller.com/blog/shadow-cljs/2017/11/06/improved-externs-inference.html)
to let the compiler know to leave the calls to the Firebase APIs as they are.

Keep these compiler debug opts handy if you do get any problems with the minified build (ie you see
error messages in the browser console like 'x.y is not a function'):

```clojure
(def debug-opts
  {:pseudo-names true
   :pretty-print true
   :source-map true
   })
``` 
 
## Conclusion

Firebase seems pretty nice for a hobby project - and maybe for more serious apps too. Using it with
Clojurescript and Re-Frame is straightforward and a natural fit. For example, Firebase `onValue` lets
you listen for the latest value in some part of the database and that is easily hooked into a Re-frame
subscription so the view magically updates whenever the database does. Simples!

Some points of note:

* Firebase-wrapping libraries aren't necessary because you'll learn just the bits you need from firebase docs.
* Avoid putting Firebase state in the Re-Frame app-db. [Signal functions](https://github.com/day8/re-frame/blob/2965ffeda9b8f3b687e2c3e0ba9a62e7fe64c0bb/src/re_frame/core.cljc#L201) will hold state for just as long as necessary.
* No need for externs, `^js` type hints are all that's required.
* Bundling Firebase libraries via npm or cljsjs is strictly optional.
* Develop the app locally with the Firebase emulator
* Stay JSON-friendly with [cljs-bean](https://github.com/mfikes/cljs-bean)