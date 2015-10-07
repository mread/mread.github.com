---
layout: post
title:  "Faster builds - part 1"
categories: builds buck
---

At LMAX we've been using Buck and a monolithic source tree for about 18 months. Prompted by the recent article from Google 
 on how they do this kind of thing at massive scale and the article from Netflix on engineering efficiency I thought I
 should post on how we do these things at LMAX where we call our engineering efficiency team "Technical Engineering".

## Life before Buck

In 2013 most development was taking place against a single large source tree with a handful of other source trees that
 got occasional updates. The main tree contained all the code, configuration and tests for our core exchange. The 
 satellite modules (about 15) either depended on code in the exchange or were depended on by the this code.

![Before Buck]({{ "/assets/before-buck-1.png" | prepend: site.baseurl }})

This is fairly standard coarse-grained modularisation of the code, mostly driven by either a conceptual separation of modules, 
 a deployment lifecycle separation or in a few isolated cases, business concept separation. Each module was a separate 
 Subversion project, i.e. each had the /trunk, /branches, /tags structure. So when you commited a change to "monitoring"
 that relied on a change to "exchange" then you would be making two separate commits, each with its own Subversion 
 revision number. For a brief period there would be code in the repository that didn't work with other code in the 
 repository.
 
Bringing these into sync wasn't pleasant and often took a developer a whole day if the dependency being updated was 
 deep within a tree.
 
This kind of friction discourges devs from using modules at all and breeds a resentment for the tools and processes.
 We've all been here and many teams accept this as an acceptable price of modularisation - after-all, there's so
 many tools in Java-land that support this approach as a defacto standard - Maven, Ivy, Artifactory, etc.
 
## Why do we accept this?

Looking at our build process objectively, a few things seem counter-intuitive.

  1. Why do we go to such effort to compile separately then integration-test things that have binary dependencies on
    each other? Why not just compile them together?
  2. Why can't I throw more cores at this build process to make it go faster?
  3. Why do most of my builds start from a clean-slate and rebuild the same code over and over again?
  
## Enter Buck

As an experiment, I once tried to compile all our Java code from all the modules in one go, i.e. a single call to `javac`.
 It took about 70 seconds to compile. This seemed really very significant.
 
Why? At that time our build process would compile, test and package the code for deployment in about 5 minutes but only if 
 you'd already compiled and packaged all the other modules already (a couple of minutes each) and setup the versions 
 correctly (yuk). Where was all that additional time going?

We started dropping BUCK files into our main module and watched the build times drop. Once the main module had decent
 coverage we started bringing the other modules under the same source tree and watched the integration process disappear.
 The team were happier, modularisation stopped being a dirty word, dependency management for both 3rd party tools and
 internal modules became quick and easy. A whole class of expensive activities just vanished from our process.
   
## Why is it faster?

### Parallelism

Take the basic anatomy of building a Java application.

![lifecycle 1]({{ "/assets/java-build-lifecycle-1.png" | prepend: site.baseurl }})

Let's add some testing.

![lifecycle 2]({{ "/assets/java-build-lifecycle-2.png" | prepend: site.baseurl }})

And a little bit of static analysis.

![lifecycle 3]({{ "/assets/java-build-lifecycle-3.png" | prepend: site.baseurl }})

#### What does Buck do with this?

There are 5 verbs in this diagram - analyse, compile (x2), test and package. Buck calls these rules. Each rule can have
 inputs and an output. From these rules we can construct a directed acyclic graph of dependent rules such that the 
 rule depends on another if it needs the outputs as its inputs.

![lifecycle 4]({{ "/assets/java-build-lifecycle-4.png" | prepend: site.baseurl }})

By laying out the rules in this way we can see that a degree of parallelism is possible during execution. i.e. you can
 analysis the app java and compile both the app java and the test java in parallel. Now multiply this by the number of
 java modules in your app, building a single DAG containing all the rules. Execute the DAG across multiple cores and you
 can expect to have all your cores at 100% utilisation. For the majority of the build. 

Note that this is a single (simple) Java module, resulting in a single .jar file. I draw attention to this point
 because fans of Maven often write-off this feature of Buck because Maven also has a parallel mode but last time I 
 looked this only applied to modules as a whole, not the individual activities within a module.
 
Note also that Buck doesn't quite do exactly this as currently it runs all the tests at the end. Hoping this will
change one day.

## TBC

In the next installment we'll cover some of the other reasons why building in this way is faster.