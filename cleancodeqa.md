**CASEY**: Thanks [@unclebobmartin](https://twitter.com/unclebobmartin) for taking the time to answer these questions! Maybe we can start with just some clarification.

Most explanations on Clean Code I have seen from you include all the things I mentioned in the video - preferring inheritance hierarchies to if/switch statements, not exposing internals (the "Law of Demeter"), etc. But it sounds like you were surprised to hear me say that. Could you take a minute before we get started to explain more fully your ideas on type design, so I can get a sense for where the disconnect is here?

**BOB**: The disconnect.  Hmmm.  I’m not sure there is one.  

I watched the first half of your video.  After that I figured that I had caught the drift.  I responded in one thread that I thought your analysis was essentially correct. I also thought that your rhetoric was a bit inaccurate in representing “clean code”.  I don’t remember exactly what that inaccuracy was; and it doesn’t really matter. 

So…. Yes, absolutely, the structures you were presenting are not the best way to squeeze every nanosecond of performance out of a system. Indeed, using those structures can cost you a lot of nanoseconds.  They are not efficient at the nanosecond level.  Long ago this would have been generally important.  We worried about the cost of function call overhead and indirection.  We even unwound loops if we could.  This was especially true in embedded real time environments.

But the kinds of environments where that kind of parsimony is important are nowadays few and far between.  The vast majority of software systems require less than 1% of a modern processor’s power. What’s more, processors are so cheap and available that it is a trivial matter to add more of them to a system. These facts change the trade-off away from program performance to the performance of the development team, and their ability to create systems, and keep those systems running. And it is those needs where the ideas of Clean Code come into play.

It is economically better for most organizations to conserve programmer cycles than computer cycles. So if there is a disconnect between us, I think it is only in the kinds of contexts that we prioritize.  If you are trying to squeeze every nanosecond from a battery of GPUs, then Clean Code may not be for you; at least in the most taxing of your deepest inner loops.  On the other hand, if you are trying to squeeze every man-hour of productivity from a software development team, then Clean Code can be an effective strategy towards that end.

**CASEY**: Just to get a little bit more concrete, so I understand what you're saying here, can we pick some specific software examples? For instance, I would assume we are both familiar with using Visual Studio and CLANG/LLVM. Would those both be reasonable examples of what you are calling the vast majority of software that requires less than 1% of a modern processor?

**BOB**: No, it seems to me that an IDE is a very specialized software system.  There are only a few in existence, and only a very few that have become popular.  

IDE's are interesting systems in that they span a huge domain of contexts.  There are portions in which nanoseconds are extremely important, and other parts where they matter very little.  A modern IDE has to be able to parse large extents of code on a keystroke by kestroke basis.  Making sure that the parsing code preserves nanoseconds can have a big effect.  On the other hand, the code that sets up a configuration dialog does not need even a tiny fractioin of that kind of efficiecy.  

And, as an aside, the kind of efficiency the compile engine of an IDE needs is more algorithmic than cycle-lean.  Cycle-lean code can increase efficiency by an order of magnitude; but the right choice of algorithm can increase efficiency by many orders of magnitude.

No, the kind of software I was referring to that requires <1% of a modern processor are the regular run-of-the-mill systems that most programmers engage in.  A website, a calendar app, a process control dashboard (for a simple process).  In fact, just about any Rails app, or any Python or Ruby app.  Even most Java apps.  You simply would not choose languages like that if nanoseconds were your concern.  

A language like Clojure (my "go to" language at the moment) is likely 30X slower than an equivalent Java app, and probably 60X slower than an equivalent C app.  But I don't care that much.  First, I can drop into Java if I have to (and I have for compute bound tasks).  Second, for many apps, adding processors is simple and cheap.  And so I usually find the programmer time saving to be cost effective.

Don't get me wrong.  I'm an old assembler and C hacker from the 70s and 80s.  I assiduously counted microseconds when it mattered (nanoseconds were way beyond anything we could imagine).  So I know how important cycle-lean code can be.  But today's processors are 10,000 times faster than the machines we used in those days (literally).  So for most software systems nowadays we have the ability to trade some of those extra cycles per second for programmer efficiency.

**CASEY**: If I understand you correctly, you are saying there are two broad categories of software, so we perhaps have to discuss each one separately. It sounds like most software I actually use falls into the category where "nanoseconds can matter", in your terminology - in other words, Visual Studio, LLVM, GCC, Microsoft Word, PowerPoint, Excel, Firefox, Chrome, fmmpeg, TensorFlow, Linux, Windows, MacOS, OpenSSL, etc. I assume based on your answer that you would agree all of those do have to be concerned about performance?
