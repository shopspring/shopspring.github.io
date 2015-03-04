---
layout: post
title:  "Automated iOS Integration Testing in a real world consumer app - at Spring"
date:   2015-03-02 19:29:22
author: "Octavian Costache"
---

Ever since we started [Spring](http://shopspring.com) we’ve set up a [continuous build](http://jenkins-ci.org/) and integration tests. We decided to start testing when our team was two engineers and we knew what we were building.
 
Testing an app and its backends manually is very time consuming, and when we did find critical issues we wanted to guard ourselves against breaking them a second time. We believe that having tests allowed us to move more predictably and faster over time.

<!--more-->

Writing and maintaining tests can sometimes feel like a distraction, but if you integrate them into your development process we believe that the benefits are worth it.

Before going further, wanted to share a video of how our tests look like when they are running. Our whole suite of tests takes about 10 minutes, so I’ve cut the video down to only a minute.

<iframe width="768" height="505" src="https://www.youtube.com/embed/-UVkVXbX8zw" frameborder="0" allowfullscreen></iframe>

Early on we decided to focus on **end-to-end tests** as opposed to unit tests. This choice was a natural one for us:

+ as a pragmatic engineering team we wanted to get as much code coverage with as little work as possible.
+ we knew what we wanted to build product wise, but our backends were under heavy development and prone to frequent refactorings.

#### Broad level overview of our testing infrastructure

In order to run end-to-end tests, we’ve created from early on a setup that allows us to run our entire stack locally.

We run each binary in “test” mode, on top of a local test database (separate from the dev database), with binaries exposing paths like `/tests/setup` and `/tests/teardown` to be called by the app.

#### Setting up the app

We picked [KIF](https://github.com/kif-framework/KIF) as a framework for a few reasons. It’s [easy enough to integrate](https://github.com/kif-framework/KIF#installation-from-github), supports the core actions we needed (waiting for views, tapping on views) and it’s easy to extend to other actions. KIF gives us access to resetting the app in between tests easily, and because of its excellent integration with Xcode it allows the developer to debug code as part of a test run.

KIF tests run as part of the app, on the main run thread, and with access to the entire app as it is running. Because of that we can control the `/setup` and `/teardown` of tests - we can log the user out, clean up app state, clean up the singletons we have and the data they store (like our event tracking).

Because the tests are ObjectiveC we can also extend the test primitives with custom expectations - for example we’ve written a waitForGetHttpRequest method, and a waitForNotification method.

It integrates nicely with Xcode and allows you to run tests individually, or test suites individually.

<img src="/images/xcode.png" border=1 width="768">

KIF works by using the iOS accessibility framework to run a set of gestures and expect a set of outcomes for each test. 

After giving your views accessibility labels, you use the KIF APIs to describe a set of gestures that the test should perform, and then an expected outcome. Contrary to popular claims, this does not make your app accessible.

The most common actions that we use are tapping on views and entering text in text fields, and the two most common outcomes are verifying that a view is on the screen, or verifying that a view is no longer on the screen.

For advanced test scenarios KIF also allows you to scroll views programmatically, tap the status bar, enter text.

There might be other frameworks that are good choices, comparing them is not the goal of this blog post - it’s possible that others have some of these properties too.

Here’s how a test at Spring looks like:

{% highlight objc %}
// Same name we will use for the Setup function on the server.
- (void)testBrandPageFollowing {
  [tester setUpServer:NSStringFromSelector(_cmd)];

  [tester tapViewWithAccessibilityLabel:@"username button"];
  [tester tapViewWithAccessibilityLabel:@"FOLLOW"];

  [tester tapViewWithAccessibilityLabel:kMeSectionName];
  [tester tapViewWithAccessibilityLabel:@"BRANDS (1)"];
  [tester waitForViewWithAccessibilityLabel:@"unfollow button"];
  
  [tester tearDownServer:NSStringFromSelector(_cmd)];
}
{% endhighlight %}

Here’s how the SetUp method looks like:

{% highlight objc %}
// Knows how to call /reset and /setup on our server running in test mode
// waits for the responses to get back and fails the test on /reset or
// setup failures.
- (void)setUpServer:(NSString *)testName {
  [self waitForGetHttpRequest:@"/tests/reset" params:nil];
  [[TMCache sharedCache] removeAllObjects];
  [JLEventTracker dropEvents];

  // At this point we have a clean app in a logged in user state.  
  [self createUserAndLogThemIn];
  [self waitForGetHttpRequest:@"/tests/setup" params:@{@"test": testName}];

  [self initViewControllersToState:BCAppStateApp];
}
{% endhighlight %}

Notice how at the beginning of every test we take the following steps:

+ **make a request to `localhost:3000/tests/reset`** which truncates the entire database. The server is set up in such a way that it only allows this to happen in test mode, and against a DB who’s name ends in _test.
+ **the app then does some client side cleanup** - for us that means deleting our cache, clearing up our custom event tracking, creating a user and logging them in from the app.
+ **make a request to `localhost:3000/tests/setup?test=testBrandPageFollowing`** which knows how to create the right data that this test expects (see next section)
+ **initialize the app to the home state** - for us a feed of products with the user being logged in.

#### What the server side looks like

Server side we’re running the entire stack that the app depends on in “test” mode. What this means in practice is that we have a `“-run_mode=test”` flag on our binaries that, when set, will expose the three paths that we need - `/tests/reset`, `/tests/setup` and /tests/teardown.

+ **The reset is simple**, it truncates our DB. This code is well guarded and there are multiple checks in every function call - we really don’t want this code to make it to production.

+ **The `/tests/setup` path takes a parameter `?name=testNameHere`** and, through some magic, calls a function with the same name on our backends that’s part of the SetUps singleton. That method inserts the right expected setup data in the database, and once that’s done the KIF test can initialize the app and expect that the data that we’ve just created shows up in the right places.

+ **The `/tests/teardown?name=testNameHere` is used to make assertions about server side effects** that we expect from the app - most importantly our data tracking. <p>When run in Test Mode our backends use a mock SpringEventLog that records all the data that we send down from the app - and the teardown method can have expectations for the number of events that we expect to be sent, and some of the contents of those events (if that content is important to the test). <p>We consider this an amazing benefit of this piece of infrastructure - having to make sure manually that the app you’re about to release tracks the right data is something very time consuming and difficult to get right.

#### How our setup ties everything together

Tests become a burden on a team unless they are tied and embedded into your team’s development process. 

Having the tests run as part of your code review process, and having a continuous build that the team is always willing to fix is important, otherwise tests go easily out of date and easily become failing tests that nobody wants to fix.

We use [Phabricator](http://phabricator.org) for code reviews - so we tied it all together to run whenever a [diff is being sent for review](http://phabricator.org/applications/differential/), so that the reviewer sees the results of all the tests that have run.

Phabricator is not difficult to extend into doing this, though it does take a little bit of tinkering around to get it right. Let us know if you are interested in that [over email](mailto:hey@shopspring.com), and we can share what we’ve learned - but this is outside the scope of this blog post.

#### Always room for improvement

Right now our tests take about 10 minutes to run. This is somewhat slow and is causing some unhappiness on our engineering team - and will only increase as we add more tests. 

The tests are not easily parallelizable, both due to the stateful way we Set Up and Tear Down the DB - but also due to the way KIF is set up. We might consider in the future sending separate test files to separate build machines - but we’re not there just yet. 

But even so, after two years it seems like we're not feeling huge pains and this is holding up pretty well.

#### Closing thoughts

These tests have been incredibly useful for us in allowing us to move fast and with confidence, and we're excited to share what we've done with the world in the hope that it will inspire other people to do the same.

We’re looking forward to evolving  this is going to look a year from now. Until then, I hope this blog post was useful. 

If you want to learn more we'd love to [hear from you](mailto:hey@shopspring.com) and if you'd want to work on one of Apple's Best Apps of 2014 as a full-time job - [we're hiring](https://spring.recruiterbox.com/).

*Thanks to Bryan Irace, Brian Bonczek, Anoop Ranganath and Adam Ernst for reading drafts of this.*
