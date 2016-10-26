---
layout: post
title:  "On young programing languages"
date:   2016-10-24 22:48:22 +0100
categories: jekyll update
---

I've come in contact with a programming language in what I would describe as an early development stage, but was already being used. From this experience I would like to discuss one aspect of language design. Namely how large should an ecosystem coupled to the language be?

I can think of several examples of what one might want to include in a language ecosystem. The abstract specification? Include a simple compiler? Perhaps a high performant, reliable, industry standard compiler? Maybe add a (cloud) execution environment? Provide a tailored IDE? Or even throw in version control system and package management?

I'm tempted to argue that designing a language should only involve the abstract specification. It is in the end the one thing that truly matters. It determines how easily programming concepts can be expressed and therefore drive it's adoption. A language designer should not engage in such trivialities as building a compiler or any of the other after-mentioned travesties. However the inner pragmatist in me realizes the world doesn't work like that.

A language will never see wide adoption if it is not a pleasure to develop in. That is why we see a rapid rise of JavaScript, Go and the other bunch. The very basis of any development environment is a compiler. (Or an interpreter but they are equivalent for this discussion.) No one but the language designer is incentivized to build one. Sometimes even to it's [detriment](http://tratt.net/laurie/blog/entries/the_bootstrapped_compiler_and_the_damage_done.html) But why stop there? Surely a debugger is also needed. People need to debug code to fix their issues faster and thus enhance their experience. We could also wrap everything up in a nice IDE for maximum efficiently. But why stop there? We could also add a cloud execution environment for super ease of access and zero starting cost. Maybe even add a version control system so developers don't lose their work.

Ok, wait a second. I think we can all agree this has gone too far. Why provide an execution environment when we have Amazon EC2? Why built an IDE to compete with Atom, Vim or InteliJ? And why try to outshine the brilliance of Git? The lessons of composability are increasingly being [lost](https://medium.com/@mkozlows/why-atom-cant-replace-vim-433852f4b4d1#.kckq1wpkh). We should only design and build what we know, what we are good at and leave space for other experts to their own best work.

But taking a step back it seems we've drawn an arbitrary distinction. A line in the sand. There is no fundamental reason as to why building a compiler "inconveniences" the user more than setting up Vim to work in the new ecosystem. Why not put it all in the cloud? Compiling, executing code and running anything outside the browser is surely heresy in modern day and age. "Language as a service" is surely the best way to get your new and awesome language out there. The idea of typing code, clicking a button and having a running and deployed service is surely appealing.

Well I would say no. Sure "Language as a service" is a convenient idea and perhaps it has a place somewhere. But by going with that approach we are depriving user of composability and choice. We are forcing them to use software that wasn't built by the experts in the field. We are also forcing them to use new and unfamiliar tools thus losing their expertise and time.
