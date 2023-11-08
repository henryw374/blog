Title: Clojurescript using JS libraries via importmap
Date: 2023-11-08
Tags: clojure

In this post I am going to look at using the [importmap](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap) feature (supported by all modern browsers), as an alternative way for Clojurescript apps to access npm dependencies.


# The Problem

When a Clojurescript app depends on a regular JS library, such as React for example, then it is typical to:

* have the code from the npm library 'processed' in some fashion (e.g. to target a specific JS version)
* bundle the 3rd party code with the application code and have it delivered from the application server

There are different ways this can happen, for example:

* use `:target :bundle` with [a bundler such as webpack](https://clojurescript.org/guides/webpack)
* use shadow-cljs with its default [shadow provider](https://shadow-cljs.github.io/docs/UsersGuide.html#js-provider)

I am a fan of shadow-cljs and so would typically use the second option. What this actually does is `:simple` optimizations on 3rd-party code, which means Google Closure code is going to read 3rd-party libs when the app is being built. Sometimes though, Closure cannot understand the 3rd party code, for example because [it doesnt have support for Class fields](https://github.com/google/closure-compiler/issues/2731). According to this really interesting [talk from Alex Davis](https://youtu.be/fT28NeZtaAg?si=1Tbxw3NMk3Cmy_aO&t=1194), he is seeing more and more popular JS libraries that Closure can't handle. I've only had the issue once myself and thankfully was able to configure shadow to use a different file than the problem one.

So, what to do?

# Using importmap

Sticking with shadow but using it with a different provider (e.g. webpack) is an option, but for browser apps there is another interesting option to consider: get pre-processed 3rd party libraries directly in the browser via a `script` tag (e.g. from a CDN such as unpkg).

I used a version of this approach for my [experiments that called the deja-fu library's rationale into question](https://widdindustries.com/blog/clojurescript-datetime-lib-comparison.html). There, I just had a couple of script tags for 3rd party libs, followed by a script tag getting the application code. This meant script tags were order-sensitive and would not scale well because transitive dependencies would not be fetched automatically. Still, for a simple app it worked fine.

Recently though, all browsers have got support for [importmap](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap), which is best explained by example:

```html

<script type="importmap">
{
  "imports": {
    "react": "https://esm.sh/react@18.2.0",
    "react-dom/client": "https://esm.sh/react-dom@18.2.0",
     "@tanstack/react-router": "https://esm.sh/@tanstack/react-router",
     "my-demo-app": "/cljs-importmap-demo/cljs-out/main.js"
  }
}
</script>
<script  type="module">
    import start from "my-demo-app";
    start();
</script>

```

The `imports` map is a bit like package.json dependencies - it says what libraries are needed and details of how to get them - all must arrive as ES6 modules. Transitive dependencies are also retrieved. After the importmap, the script tag with `type=module` says to interpret the code within as an ES6 module. Here, it just imports the application code and starts it.

The module `my-demo-app` just contains the application code, not any 3rd party libraries. To generate the module from clojuresript code is just a matter of using Shadow-cljs documented options for that, for example:

```clojure
{:target     :esm
 :js-options {:js-provider :import}
 :modules    {:main {:exports {'default 'com.widdindustries.demo-app.app/init}}}}
```

Here is [an example app demonstrating this technique](https://widdindustries.com/cljs-importmap-demo) and
here is [the source code](https://github.com/henryw374/cljs-importmap-demo) for that.

# Pros and cons

This is the first time I've tried using importmap. I failed to google any experience reports of anyone using it from Clojurescript, hence this post. Here are some pros and cons I am aware of so far:

Using importmap, 3rd party libraries can be cached. The application code will also be cached, but is likely to change at a faster rate than library versions get changed, so will be downloaded more frequently, but will be smaller than the tradition bundled version.

The importmap is specified in the html file, but will also need to be specified again for a page that loads tests for example. Also, it may be required to use a dev-time version of a library locally, but deploy with the optimized one. For example, React performance profiling tools only work with the dev-time React version. It is possible to conditionally create the importmap, for example, if on localhost, create one map, if deployed a different one.

The 3rd party libraries are being retrieved wholesale - ie no dead code elimination could happen here. Is significant dead code elimination a thing in JS-land these days though? I've heard of Rollup, but I haven't tried out how it would help trim down React and the like.

importmap is a relatively recent addition to browsers - so might not be suitable for some potential users.

Loading speed vs bundled apps, aka time-to-interactive (TTI)? I haven't measured anything yet. Please comment if you have experience of this.

Any more you'd add? Please use the link below to discuss. 
