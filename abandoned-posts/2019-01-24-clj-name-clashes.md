---
layout: post
title: Avoid Name Clashes
description: How to avoid name-clash and masking bugs with Clojure 
category: clojure 
---

Let's start with a simple Clojure map
```
{:name "Henry"
 :occupation :devlpr}
```

See anything wrong there?

Hmm, looks innocent enough. Here's some a Clojure function that uses that data.

```
(defn describe [person]
 (let [{:keys [name occupation]} person]
  (str  name " is a " (name occupation))))
```

The bug here is pretty obvious
