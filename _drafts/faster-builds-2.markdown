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
 comparing multiple sources and classes? My suspicion is most teams use incremental compiles in the IDE but when it 
 comes to a full compile, test and package, you first clean your workspace and then you build _everything_.
 
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

The problem isn't the first build that you run, it's all the subsequent builds that follow on from one line tweaks
 where you wait for a full 5 minutes for a result that you already know (or at least you think you do). Don't 
 under-estimate the impact on developer morale of being forced to do pointless repetitive activities. I've known 
 people change jobs to avoid a build process.
 
#### What's the solution?

In short, do this:

 * divide your codebase into small cohesive chunks
 * build a directed acyclic graph of the dependencies between those chunks
 * detect when anything changes in a chunk
 * invalidate the node for the invalid chunk and its downstream nodes
 * build only invalidated nodes
 
Luckily any of the normal ways of dividing your code into modules would do the job just fine. If anything err on the 
 side of [packaging things together that change together][2].
  
You may be wondering why we wouldn't go even more fine-grained than this and perhaps have each source file be a "chunk".
 The answer is found somewhere in the nature of tightly-coupled clusters of code, they tend to be interdependent such
 that a change to one would necessitate a change to others. As with all such optimisations, try it out, measure the
 effect, draw a conclusion.

#### How does Buck do this?

A _chunk_ in buck is normally called a module and is materialised by creating a file named BUCK in the root directory
 of the module. In that BUCK file you define some _rules_, for example if we wanted to compile some java:
 
~~~ python
java_library(
  name       = 'my-java',
  srcs       = glob( 'src/main/java/**/*.java' ),
  resources  = glob( 'src/main/resources/**/*' ),
  visibility = [ 'PUBLIC' ],
  deps       = [ ]
)
~~~

This says we have a module called _my-java_ that has code in the normal places. When you first run a build Buck will
 iterate all the source files and dependencies (the inputs) and create a single hash representing their contents. Then 
 it compiles the code, creates a .jar file and stores that on the filesystem together with the hash value.
 
Next time we run a build Buck rehashes the inputs and compares this hash value with the one we stored earlier. If they
 are the same then we don't need to recompile and re-jar. It turns out that passing a load of files through a hash
 function is really quite fast - at least compared to compiling. If you add a few tricks like using Linux inotify to
 detect really quickly if files have been changed, and storing the DAG in memory between builds, then the time it takes 
 to work out what you need to rebuild is really small.
  

#### How do we handle moves etc?

[1]: {{ "/builds/buck/2015/10/07/faster-builds-1.html" | prepend: site.baseurl }}
[2]: http://docs.google.com/a/cleancoder.com/viewer?a=v&pid=explorer&chrome=true&srcid=0BwhCYaYDn8EgOGM2ZGFhNmYtNmE4ZS00OGY5LWFkZTYtMjE0ZGNjODQ0MjEx&hl=en
