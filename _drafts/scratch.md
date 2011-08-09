---
layout: post
title: What DDD isn't
---

Well it's certainly not creating a couple of HashMaps in your code that contain properties that vaguely resemble something that one of your business sponsors mentioned in passing, and calling it DDD. No. It's not that although I suppose that's a start.

It's not even creating a couple of classes that represent business concepts and putting a `public long getId();` method on them that returns a database generated sequence number. I mean that's a start, it really is, but people have been doing that for years - at best it's a relational model of the business domain.

For a start, get rid of the database generated ids - one test you can do that helps you to fathom this as follows. Imagine a conversation with a domain expert about a domain concept, let's say the guy behind the counter in the local kebab shop. (You've decided there's just not enough IT in kebabs these days and so you're going to write him some software to help him out). You ask him, "if someone walks in and asks for 3 chicken shish, one chilli sauce, two garlic, how do you keep track of which is which?" Which of these does he say?

a) "I phone up my friend with the computer and ask him to spark up Oracle, generate me the next number in kebab_sequence and then I write it on the wrapper."
b) "I take a pseudo-random source of data, normally the temperature at the middle of my donner skewer and use it to seed a random number generator from which I can generate a UUID, of course I write this on the wrapper, it takes a while to write it my friend."
c) "C for chilli, G for garlic?"

You've guessed it, at the point of kebab creation he does not, I repeat, *does not* generate a unique identifier for the finely crafted and indeed utterly unique kebab. In some ways the two garlic-sauced kebabs are indistinguishable and could even get confused!

Now as a software developer with years of experience it's very easy to smile sweetly at kebab man and go off and generate that primary key anyway. You're *sure* that all systems need ids right? That's how you've always done it. How else would you uniquely identify that kebab when you want to give it to the customer, or work out how much it costs?

Well... no. This is an issue of dependency management, temporal coupling and of context.

Dependency management because in order to generate the id you normally have to persist the data to the database - *it does not have identity until it has been persisted*. That means you can't do anything with it that might use the id until it has been persisted. You can't give it a customer, you can't work out its cost, nothing. You do have a database right? You are ready to persist it? You can afford the I/O cost of persistence right now?

Temporal coupling because in most cases you begin a resource and lock-hungry database transaction now and keep it running until you've finished doing things with the kebab, even if those things might be slow... like eating it? You could persist it, commit, and then do things with it, but what if the chef drops it on the floor as he's passing it to you and have to invoke `replaceKebabForFree()`? What about the kebab identifier you've used up - does it exist or not? Did you pay for the dropped kebab or the replacement kebab?

And context most importantly of all because in all the scenarios we've discussed there is absolutely no reason to uniquely distinguish your kebabs from all the other kebabs that have ever been served up. You could perhaps imagine up a context where unique identity would be useful - a kebab archivist, travelling the world collecting data on every kebab ever produced in order to publish his great work "kebabs of the world".

Are you building software for the kebab archivist?
