---
layout: post
title:  "Faster builds - part 2"
categories: builds buck
---

In [part 1][1] we talked about how we made our builds faster by breaking them down into small chunks so we could
 distribute them amongst multiple cores. In part 2 we're going to talk about incremental builds and caching.
 
### Only build what you've changed

Historically build tools have given a tentative nod in the direction of incremental builds. Most will only recompile
 a source file if it's corresponding .class file has a less recent timestamp. It's never really worked out too well
 though and is full of problems. What happens if a file is deleted or moved? Which timestamp do you use if you're
 comparing multiple sources and classes? I may have been unlucky but everywhere I've worked we have mostly used
 incremental compiles in the IDE but when it comes to a full compile, test and package, you first clean your workspace
 and then you build _everything_.
 
What does this cost a development team? 

Let's say your team commits on average 100 times per day. That's our average at LMAX, ymmv. Before committing a developer
 should ensure their code compiles, some basic static analysis passes and the tests pass. Perhaps that takes 3 or 4 builds
 per commit. If the build takes 5 minutes:

$$
\begin{align}

& 5\ \text{minutes} \\ 
\times 
& 100\ \text{commits} \\ 
\times 
& 3\ \text{builds} \\ 
\approx 
& 25\ \text{hours spent building per day}

\end{align}
$$


Those commits come in bursts, a developer or pair work on something for an hour or two and then they're ready to commit.
 They've been running their unit tests in the IDE and now they're ready to run the static analysis and all the other 
 tests.
 
They run their 1st build, 5 minutes later it tells them they've got an extra import in one of their .java files. Fix
 that and build again, that's right, 5 minutes for a one line change.



[1]: {{ "/builds/buck/2015/10/07/faster-builds-1.html" | prepend: site.baseurl }}
