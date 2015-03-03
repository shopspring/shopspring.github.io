---
layout: post
title:  "Why We Think GoLang Is Ready For Early Stage Startups"
date:   2013-10-28 10:00:00
author: "Octavian Costache"
---

We're building a mobile commerce app. Our backends do a lot of database reads, a reasonable amount of writes, cache objects (or talk with cache services) and serve a JSON api to an iOS app.

We needed high performance web frontends that talk to a range of other backend services and serve an API.

We picked Go as the language of choice from very early on.

<!--more-->

Such choices can have significant impact on a business so I wanted to write a blog post about how [Spring](http://shopspring.com) made this choice, touch upon a few of the small issues we've encountered in the process and show how in the end it proved to be the right choice.

### How we first became interested in GoLang

We wanted something easy to understand for new developers, fast and with a good toolset for debugging and performance tuning.

Like all tools, all languages ultimately are good at certain things and not great at others - and we wanted something great at short term productivity but also a good long term investment.

Go claims to address these exact needs - so we felt we needed to look into it.

### First step - we did our homework

The Go authors do a great job explaining language design choices, so we spent a few days [reading about Go](http://golang.org/doc/).

For some of our concerns the answers became obvious early on - significantly faster than interpreted languages, static typing catches a wide variety of bugs at compile time, easy to understand because it's so straightforward.

We made a list of Pros and Cons to make sure we're not missing anything.

#### PROS

+ fewer bugs due to stricter code checking upfront - type checked, compiled
+ easy to [profile for speed and memory leaks](http://blog.golang.org/profiling-go-programs)
+ [built in code formatting](http://blog.golang.org/go-fmt-your-code)
+ built for Google: monolithic huge code base, big compiled binaries, runs at scale
+ natively multithreaded
+ small memory footprint
+ thoughtfully designed language choices - no inheritance, elegant concurrency
+ actively developed and iterated upon by Google - better over time
+ could help recruiting engineers excited about good new tech
+ [active community](https://groups.google.com/forum/#!forum/golang-nuts)

#### CONS

+ not as many libraries - risk early on when dev speed important
+ cost for new devs to pick up Go
+ not proven at scale just yet - early signs seem to suggest that Go will scale well (both at large QPS but also at large LOC)
+ none of us is a seasoned Go developer
+ package versioning and dependency - not as powerful as others
+ intermediary cost to having two backend technologies - while migrating our early prototype to Go

There were still open questions.

We needed available libraries for common usecases, we wanted answers when we get stuck and we wanted to move fast. We wanted to understand the costs first hand, at least to some extent.

### Second step - we eased into it

We built a small binary that powers the search autocomplete functionality in our app. Small enough, self-contained, lots of touch points with our existing infrastructure and somewhat independent.

We wanted to see if our open questions would be addressed in this trial period - we aimed for about a month or two - while maintaining our small Go experiment.

### Third step - hands on experience, make a decision

[StackOverflow](http://stackoverflow.com/) is a very useful resource when you're stuck anything engineering related, but it hasn't reached critical mass for GoLang. Fortunately the [golang-nuts mailing list](https://groups.google.com/forum/#!forum/golang-nuts) is absolutely great. We asked and we were impressed with the quality and turn-around time of some very [good](https://groups.google.com/d/msg/golang-nuts/6hpUErAfMHI/X0bHeoZyfz0J) [answers](https://groups.google.com/d/msg/golang-nuts/6tyCz7Tc8Ow/1BCkkBWnTLEJ). Check!

Startups need to ship good products and not rewrite everything. Right now Go seems to be where node.js was a year or so ago. There are [high quality packages](https://code.google.com/p/go-wiki/wiki/Projects) written for the most common things, a few [postgres libraries](https://github.com/lib/pq), a few [ORMs](https://github.com/coopernurse/gorp), an [s3 library](https://wiki.ubuntu.com/goamz), and so on. They are [up to date](https://github.com/lib/pq/commits/master), bugs are [being fixed](https://github.com/lib/pq/issues), pull requests are being accepted. Most are very well written and well tested. Check!

Feature development speed is practically impossible to measure. To me it feels somewhere right under Ruby/Rails, but only by a little bit. You have to handle more errors, you have to write a little bit more explicit code but at the same time you have to read a little less Rails documentation to understand how things are glued together. More importantly though, Go feels fun to write.

We did end up spending some time where we expected - in libraries.

Because of its lack of magic we needed to extend Gorp with some subtle logic of our own. We had to dig deep into the [reflect](http://golang.org/pkg/reflect/) package, and that was more subtle than I wished it would have been. We ended up writing some not-very-fancy pre and post-request filters, but they are small and feels like we don't need more.

We spend more time handling errors, but honestly that feels like the right thing to do.

### Conclusion - GO!

Despite paying some small costs, we feel that overall Go is paying off really nicely.

Our backends rarely crash in production - and when they do the cause seems easy to find and fix. We haven't yet spent time optimizing and our frontends are faster than our equivalent prototype Rails server - at least for core functionality- plenty of people have done tests around [this](http://blog.iron.io/2013/03/how-we-went-from-30-servers-to-2-go.html). We haven't seen any memory bloat.

Lots of good libraries are already built, developers are excited about it.

The language feels ready as a first choice technology for an early stage startup building tech like we are - consumer facing products backed by a series of backends serving them data through some sort of APIs.

We love Go and have no regrets - and after talking with a [few other startups](http://poptip.com/) that also made this choice, it doesn't seem like they are look back either.


If you want to learn more we'd love to [hear from you](mailto:hey@shopspring.com) and if you'd want to build Go as a full-time job - [we're hiring](https://spring.recruiterbox.com/).