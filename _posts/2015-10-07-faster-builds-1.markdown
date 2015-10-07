---
layout: post
title:  "Faster builds - part 1"
categories: builds buck
---

At [LMAX][1] we've been using [Buck][2] and a monolithic source tree for about 18 months. Prompted by the [recent talk from Google][3] 
 on how they do this kind of thing at massive scale and [some similar][4] [articles from Twitter][5] I thought I
 should post on how we do these things at LMAX where we call our engineering effectiveness team "Technical Engineering".

## Life before Buck

In 2013 most development was taking place against a single large source tree with a handful of other source trees that
 got occasional updates. The main tree contained all the code, configuration and tests for our core exchange. The 
 satellite modules (about 15) either depended on code in the exchange or were depended on by this code.

![Before Buck]({{ "/assets/before-buck-1.png" | prepend: site.baseurl }})

This is fairly standard coarse-grained modularisation of the code, mostly driven by either a conceptual separation of modules, 
 a deployment lifecycle separation or in a few isolated cases, business concept separation. Each module was a separate 
 Subversion project, i.e. each had the /trunk, /branches, /tags structure. So when you commited a change to "monitoring"
 that relied on a change to "exchange" then you would be making two separate commits, each with its own Subversion 
 revision number. For a brief (and sometimes not so brief) period there would be code in one module that didn't 
 work with code in the other modules.
 
Keeping these modules in-sync wasn't pleasant and often took a developer a whole day if the dependency being updated was 
 deep within a tree. We had to develop tooling just to assist this task.
 
This kind of friction discourges devs from using modules at all and breeds a resentment for the tools and processes.
 We've all been here and many teams accept this as an acceptable price of modularisation - after-all, there's so
 many tools in Java-land that support this approach as a defacto standard - Maven, Ivy, Artifactory, etc.
 
Looking at our build process objectively, a few things seem counter-intuitive.

  1. Modularisation is good but why should introducing modules have such a negative impact on our build process? 
  2. Why can't I throw more cores at this build process to make it go faster?
  3. Why do most of my builds start from a clean-slate and rebuild the same code over and over again?
  
## Enter Buck

As an experiment, I once tried to compile all our Java code from all the modules in one go, i.e. a single call to `javac`.
 It took about 70 seconds to compile. This seemed really very significant.
 
Why? At that time our build process would compile, test and package the code for deployment in about 5 minutes but only if 
 you'd already compiled and packaged all the other modules (a couple of minutes each) and setup the versions 
 correctly (yuck, hours). Where was all that additional time going?

We started dropping BUCK files into our main module and watched the build times drop. Once the main module was largely 
 building with Buck we started moving the other modules under the same source tree and watched as manual module integration
 just didn't need to happen anymore. The team were happier, modularisation stopped being a dirty word, dependency 
 management for both 3rd party tools and internal modules became quick and easy. A whole class of expensive activities 
 just vanished from our process.
 
Taking a leaf out of the LMAX continuous performance tuning book we started measuring build times a few months before we
 began migrating to Buck (we now have data for half a million builds). I had always hoped that we would see the total 
 time spent building drop - if you're not waiting for builds then you're writing awesome code right? Instead we saw 
 the number of builds per day increase, in fact it's now more than 5 times higher than it was 18 months ago. I guess we
 developers crave feedback, if we can get feedback quicker then we just grab more.
   
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
 inputs and an output. From these rules we can construct a directed acyclic graph of dependent rules such that a
 rule depends on another if it needs the outputs as its inputs.

![lifecycle 4]({{ "/assets/java-build-lifecycle-4.png" | prepend: site.baseurl }})

By laying out the rules in this way we can see that a degree of parallelism is possible during execution. i.e. you can
 analysis the app java and compile both the app java and the test java in parallel. 

Note that this is an example of a single (simple) Java module, resulting in a single .jar file. I draw attention to this point
 because fans of Maven often write-off this feature of Buck because Maven also has a parallel mode but last time I 
 looked this only applied to modules as a whole, not the individual activities within a module.

Now multiply this by the number of Java modules in your app, building a single DAG containing all the rules. Execute 
 the DAG across multiple cores and you can expect to have all your cores at 100% utilisation for the majority of the 
 build. Our build has several thousand parallelisable jobs so effective multi-core execution really makes a 
 difference.
 
Actually Buck doesn't quite parallelise exactly this way as tests are run in parallel with each other after all the 
build steps are complete. We don't find this makes much difference to the optimal build times. I'm hoping to have time 
to submit a pull request for this one day.

## To be continued...

In the next installment we'll cover some of the other reasons why building in this way is faster.

[1]: http://www.lmax.com
[2]: https://buckbuild.com
[3]: https://www.youtube.com/watch?v=W71BTkUbdqE
[4]: http://spectrum.ieee.org/view-from-the-valley/computing/software/twitters-tips-for-making-software-engineers-more-efficient
[5]: http://www.gigamonkeys.com/flowers/
