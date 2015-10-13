---
layout: post
title:  "Faster builds - part 2"
categories: builds buck
---

In [part 1][1] we talked about how we made our builds faster by breaking them down into small chunks so we could
 distribute them amongst multiple cores. In part 2 we're going to talk about incremental builds and caching.
 
### Only build what you've changed

Historically build tools have given a tentative nod in the direction of incremental builds.



[1]: {{ "/builds/buck/2015/10/07/faster-builds-1.html" | prepend: site.baseurl }}
