---
layout: post
title: The Operational Cost Savings Of Paying Off Technical Debt
---

### *The technical debt in our code increases the cost of maintaining our code.

### *Is the cost of removing this technical debt less than the cost of leaving it in?*

Inspired by blogs from [Tim Ottinger][1] and [Mike Feathers][2] I have been trying to come up with a way of analysing code to justify and explore the impact of the statement above in economic terms - i.e. how much will the business save by removing technical debt and how to decide which technical debt to remove first?

What is _the cost of maintaining our code_?
-------------------------------------------

This is the cost to the business of maintaining a "bug fix", "maintenance" or "operational support" team together with the testing and organisational support processes that these teams normally require. This team generally fill their backlog with small to medium-sized change requests and defects, these items often come through a user-facing helpdesk and series of approval and prioritisation meetings. I'm making the assumption that there would at some stage in the past have been a capital-funded project to deliver the feature to which these CRs and Defects apply. That project is now complete and has handed over to the maintenance team. 

The items on the backlog often fall into the following categories:

* Genuine change requests that a user realises would make the system more valuable now that they've had some time to think about it and use it in anger.
* Not-so-genuine change requests that should really have been part of the scope of the original project but budget ran out before the team got to that bit.
* Defects discovered after hand-over to the maintenance team (or at least not fixed before hand-over).

What is _the cost of removing this technical debt_?
---------------------------------------------------

There's no single answer to this one. Sometimes we might have to rewrite, sometimes we can refactor, in most cases we see poor test-coverage as a technical symptom that matches up with the diagnosis of technical debt and at minimum we might want to improve test-coverage. How much will this cost to do? There's nothing new here - we can only apply our existing scoping and estimating techniques to come up with a proposal to put a team together to resolve the issue. Of course we run the risk that the solution being proposed is no better than what we're replacing... please don't do that! After all we've now learnt an awful lot more about the business domain and about what the user really wants. Use the issues raised against this area to inform the new solution. Perhaps this is information that the original project team didn't have, or maybe they had it but other constraints stopped them from acting on it.

How do we do it?
------------------

Pre-amble over, we're down to the crux of this blog. If cost-to-fix(CTF) < cost-to-maintain(CTM) then you have business case for addressing the technical debt and if not, you don't. Or you could treat this as a way of measuring the time taken to achieve ROI instead - it depends on how these things are treated in your organisation.

How do you measure CTF and CTM? Before we start, let's work out which area of our system we're going to focus on - which area has the highest CTM (and perhaps additionally - over what time period)?

In summary:
1. Combine change management with version control.
2. Measure churn on defect commits grouped by versions and packages.
3. Count packages with the highest ratio of change to defects.
 
The first thing we have to do is combine our change management system (e.g. Jira) with our version control system (e.g. Git). This allows us to work out, for any particular commit, why it was committed. Was it a change or a defect, when was it released, how long did it take to do, etc.

Next we group the commit data by release and package, counting the number of change issues and defect issues. So if a change is committed to matt.a.MyClass against issue number ABC-123 which is a Change Request with a fix version of v1.0 then that scores 1 point in the change requests column. If someone commits again to the same file using the same issue then I ignore that - I don't want frequent commit habits to skew the results. This gives us something like this:

Release | Package    | Number of change requests |  Number of defects
-------:|:-----------|--------------------------:|------------------:
v1.0    |matt.a      | 3                         | 2
v1.0    |matt.b      | 4                         | 10
v2.0    |matt.a      | 3                         | 3
v2.0    |matt.b      | 2                         | 4
v2.0    |matt.c      | 3                         | 2
v3.0    |matt.b      | 3                         | 5
v3.0    |matt.c      | 1                         | 1

So we could summarise this as follows. In most cases I take only the top 5 worst offenders (by defect count) per release in order to weight the results towards packages that are consistently causing us problems.

Package            | Average change requests /release | Average defects /release | Ratio of defects to changes
:------------------|---------------------------------:|-------------------------:|---------------------------:  
matt.a             | 2                                | 1.66                     | 0.83 
matt.b             | 3                                | 6.33                     | 2.11 
matt.c             | 1.33                             | 1                        | 0.75

I use a ratio in order to balance out areas that are changing a lot vs those that change less frequently. But does that run the risk of ignoring [sufficient design (Joshua Kerievsky)][3] or [prudent-deliberate design debt (Martin Fowler)][4]? Filtering for the top 5 worst offenders per release mitigates this to some extent - if I'm getting 6 defects for a package in a single release then I may well have been wrong to call this package's failings "prudent & deliberate". Statistically there's no doubt a simpler way of achieving the same result - I'm no statistician as you can probably see by now.

So from these results, I can see that each change request raised against package matt.b tends to result in 2 defects. Why could that be? There's a few possibilities here and the metrics should never replace a sound understanding of the code and development environment. Perhaps in release v1.0 that package was worked on by a junior programmer who was sadly run over by a bus; or perhaps it's not part of the core functionality of the system and shouldn't really be in the analysis at all. **Or perhaps it's really badly designed, has no test coverage at all and anyone brave enough to touch it instantly kicks off a cascade of defect inducing side-effects.**

What I see from my own analysis (5 years of commit history covering 12,000 .java files) is that the results pass the sanity check. I _know_ those packages with the high ratios, they've been causing us problems for years, we know they're full of WTFs. We have ratios as high as 8:1 for the really nasty areas. What this analysis gives us is unlikely to be a surprise to those working with the code daily - it's the fact that it's a repeatable and deterministic analysis that gives us the advantage. In my working environment at least, simply having the gut-feel that something isn't right isn't enough, for action to be taken there must be objective data to back it up.

So I've now achieved the prioritisation goal - I want to put forward a proposal to "fix" package matt.b cos that's costing us the most. But how much is that package costing us? This is simpler:

1. Add up the number of defects per year.
2. Multiply by an average cost per defect.

So according to the data above, in our 3 releases we had to fix 19 defects relating to matt.b. Let's say that we think our average defect costs us 3 days of effort (1 day of development, 1 day of test and 1 day to cover the various administrivia). 

19 x 3 = 57 days CTM - that's how much it's cost us in the last year simply to have this code in the system in its current condition.
 
We've already discussed CTF - just apply whatever approach you normally apply when estimating a piece of work. Let's say for argument that you think a team of 3 could refactor a particular area in 2 weeks giving a total cost of 3 x 5 x 2 = 30 days.

And that's it for this contrived example - 30 < 57 so we have a business case.

Further discussion
------------------

There are a few questions to ask at this early stage. In coding up an application that executes this analysis I discovered the need to massage the data. E.g.

1. Filter out merge commits - these were rarely committed against defect issues but best to be safe.
2. Deal with file moves, and in the case of Subversion, branch creation, which can look like a file move at times. This was mostly addressed by ignoring the part of the file path that related to the location within the repository. Git has potentially other issues relating to commit squashing - although depending on your local strategy for referencing the issue in the commit comment this may not be an issue. For genuine file moves as a result of refactoring - how should we detect these - should they be scored as "churn"?

And there are some pre-conditions for being able to do this analysis:

1. You must have a way of linking commits to issues. We follow a local convention of prefixing commit comment with the issue number from Jira - but if you didn't do that you're going to have to find another way. Perhaps in git the branch-name should also reference the issue but that data can be lost in merges.
2. It helps if your releases occur on a regular basis, are of similar duration and are serial in nature.

[1]: http://agileotter.blogspot.com/2010/10/heatmap-new-hotness.html
[2]: http://michaelfeathers.typepad.com/michael_feathers_blog/2011/01/measuring-the-closure-of-code.html
[3]:  https://elearning.industriallogic.com/gh/submit?Action=PageAction&album=blog2009&path=blog2009/2010/sufficientDesign&devLanguage=Java
[4]: http://martinfowler.com/bliki/TechnicalDebtQuadrant.html
