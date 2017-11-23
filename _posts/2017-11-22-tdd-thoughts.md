---
published: false
layout: post
title: TDD Thoughts
---
## TDD Thoughts

Coming from a Computer Science background academically, and in a fervant effort to embrace the "E" in SRE (Site Reliability Engineering), I have been spending a fair amount of time "drinking the Kool-Aid" when it comes to modern development practices.

While controversial in some circles (a few minutes with your favorite search engine will find plenty who love it, and others that adamantly hate it), I think the industry at large has accepted that Test Driven Development (TDD), or perhaps more appropriately Behavior Driven Development (BDD), makes it easier for developers to have confidence in their code and iterate faster.

My own experience has certainly proved this, with "[red green refactor](http://http://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html)" being something I often wake up saying in my sleep (usually in a cold sweat, and a recurring black and white scene sitting in front of a large mission-critical codebase with no tests)...  However, while the general practice is well understood, there are common anti-patterns and nuance to be aware of.

In a recent round of "Google Engineering" (not working for Google, just typing random phrases in their search engine like a monkey trying to reproduce Shakespeare ;-) ), I came across an excellent video which describes some of the anti-patterns, and references an amazingly succinct blog post discussing the nuance of "mocking" (something I have personally struggled with).  While neither of these are "new", I found them rather enlightening so wanted to share!

- [The Little Mocker](https://8thlight.com/blog/uncle-bob/2014/05/14/TheLittleMocker.html)
- [TDD - The Bad Parts](https://youtu.be/xPL84vvLwXA)

Related resources:

- [Clean Coders Website](https://cleancoders.com)
- [Clean Coder Blog](http://blog.cleancoder.com)
- [The Three Laws of Test Driven Development](http://programmer.97things.oreilly.com/wiki/index.php/The_Three_Laws_of_Test-Driven_Development)
- [Red Green Refactor](http://www.jamesshore.com/Blog/Red-Green-Refactor.html)
- [Behavior Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development)
