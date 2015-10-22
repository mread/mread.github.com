---
layout: post
title:  "Faster builds - part 2"
categories: builds buck
---

In [part 1][1] we talked about how we made our builds faster by breaking them down into small chunks so we could
 distribute the work amongst multiple cores. In part 2 we're going to talk about incremental builds and caching.
 
### Only build what you've changed

Historically build tools have given a tentative nod in the direction of incremental builds. Most will only recompile
 a source file if it's corresponding .class file has a less recent timestamp. It's never really worked out too well
 though and is full of problems. What happens if a file is deleted or moved? Which timestamp do you use if you're
 comparing multiple sources and classes? I may have been unlucky but everywhere I've worked we have used
 incremental compiles in the IDE but when it comes to a full compile, test and package, you first clean your workspace
 and then you build _everything_.
 
#### What does this cost a development team? 

Let's say your team does 50 commits per day.
  
Those commits come in bursts, a developer or pair work on something for an hour or two and then they're ready to commit.
 They've been running their unit tests in the IDE and now they're ready to run the static analysis and the unit
 tests.
 
They run a first build, 5 minutes later checkstyle tells them they've got an extra import in one of their .java 
 files. Fix that and build again. That's right - run the whole 5 minute build again for a one line change.
   
So if the build takes 5 minutes and we average 3 goes at getting something commitable.

$$
\begin{align}

5\ &\text{minutes} \\ 
\times 
50\ &\text{commits} \\ 
\times 
3\ &\text{builds} \\ 
\approx 
12.5\ &\text{hours spent building per day}

\end{align}
$$

That's quite a lot of time, at least an extra team member's worth. Is that build time completely unproductive? Not
really, even _real_ developers need a break from time to time, and tea, don't forget tea.

The bigger problem here is how completely pointless it is to wait for a full 5 minutes for a result that you already
 know (or at least you think you do). Don't under-estimate the impact on developer morale of forcing them to do pointless
 repetitive activities. 


[1]: {{ "/builds/buck/2015/10/07/faster-builds-1.html" | prepend: site.baseurl }}
