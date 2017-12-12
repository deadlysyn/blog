---
published: true
author: Mike Hoskins
layout: post
category: blog
---
* TOC
{:toc}

## Background

My academic background is CSCE.  Since then, I've been in the Internet industry for over twenty years.  Many of those years were on cross-functional infrastructure teams that were either small enough not to give much thought at all to categorizing how we planned or worked, and several were in larger companies which followed strict Waterfall (the norm back then).  As non-Agile as you can get, and to be honest I had little interest in learning other ways to work at the time.  I am a UNIX and coding geek at heart, and if learning something required looking away from a terminal it was hard to get my attention in those formative years.

### Agile Transformation

Somewhere around seven to eight years ago, I was on a team that underwent a transformation from Waterfall to Agile.  Unfortunately for us, it was Scrum.  I say "unfortunately", because while Scrum in and of itself isn't good or bad (like any methodology, success largely depends on the people and context), it never felt quite right.  Part of that was a certain amount of cynicism combined with lack of experience.

Another part was being too dogmatic in ceremony...  while I latched onto the "there is no ONE way to do Agile" mantra early in this journey (any methodology should be adapted to a team's and the business' needs), early implementers of Agile often turned cookie cutter practices into religion.  After all, they wanted something to "copy and paste" that would provide immediate results.

Worse, in a large organization this was usually something like SAFe ([Scaled Agile Framework](https://en.wikipedia.org/wiki/Scaled_agile_framework)), LeSS ([Large Scale Scrum](https://en.wikipedia.org/wiki/Scrum_(software_development)#Large-scale_scrum)) or DAD ([Disciplined Agile Delivery](https://en.wikipedia.org/wiki/Disciplined_agile_delivery)).  While any of these could likely be pragmatically applied to large teams to improve productivity...  they were often crammed whole-hog down the throats of the team by over-zealous consultants.  In my humble opinion, trying to adopt everything from any of these frameworks blindly is a bad idea for anyone.  That is, however, an opinion with the privilege of hindsight.

One common theme for me during these early years of Agile experimentation, was being strictly within technical individual contributor roles.  I was occasionally asked to help interview candidates, but it was largely an after-thought.  Most often I was being interviewed, and focused more on "passing the test" than how the interviews themselves were conducted.

### Getting Extreme

Over the past couple years, I've been on a sanctifying journey from Scrum to Lean/XP.  Being pragmatic in adoption of new methods while focusing on value and minimizing waste is truly enlightening.  Part of this came with the buzzwordification of DevOps (with a common analogy to [Lean Manufacturing](https://en.wikipedia.org/wiki/Lean_manufacturing), for better or worse), but a larger part was being on an infrastructure engineering team over the past fourteen months which practiced [extreme programming](https://en.wikipedia.org/wiki/Extreme_programming) (XP).  Were we perfect in our practice?  No way.  However, striving to practice XP daily showed me that, while I still have a lot to learn, it is a refreshing Kool-Aid well worth drinking.

If you've been engaged in intergalactic travel over the past decade and not heard of XP, a full overview is too much to cover sufficiently here...instead, hop over to Amazon and grab copies of Kent Beck's [_Extreme Programming Explained_](https://www.amazon.com/Extreme-Programming-Explained-Embrace-Change/dp/0321278658) and [_Test Driven Development By Example_](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530).  They're quick reads.

For the purpose of this post, we only need to understand one XP concept...

## XP Values

XP practices center around a set of shared values.  The key thought is that, by taking these values to heart and letting them guide our daily practices, we can both pragmatically apply the common patterns (TDD, incremental design, continuous feedback, etc.) and invent new patterns when needed.  It is this pragmatism that initially sold me on XP.

You can dig into the precise meaning of these values in the books referenced above, or by spending time in your favorite search engine.  For now, a concise overview will do:

- Simplicity: Do what is needed, but no more.
- Communication: Communicate daily.
- Feedback: Early and often trumps late or rarely.
- Respect: Every member of the team is valuable.
- Courage: Tell the truth, adapt to change.

### Extrapolating the Values

While drinking the XP Kool-Aid, I also transitioned from technical IC to pager-carrying technical manager.  While still in the trenches, I had to think about hiring a lot more.  The team grew beyond [the mythical two pizzas](https://www.fastcompany.com/3037542/productivity-hack-of-the-week-the-two-pizza-approach-to-productive-teamwork), got split along clear fracture lines, and grew again.  At this time in my career, I was fortunate enough to have senior management who talked about the importance of good hiring practices...  however, the talk mostly boiled down to "take it seriously", "do better", or "delight the candidate".  Good advice, but how?

Partially because of needing to directly drive the hiring process, and partially because of the XP immersion...I started wondering if the XP values could be used to guide how we hired.  After all, the XP literature itself talks about applying these principles in everything we do, particularly when we're faced with inadequate or non-existent practices and need to develop alternatives.  As the name implies, XP has a purposeful focus (simplified scope) on the engineering side of an organization...but the thought leaders in XP happily admit the transformational effects can and often must span organizational boundaries.

There is a lot of welcome buzz around diversity and fairness of late, a large part of which is how companies acquire talent...  Many companies struggling with this already have engineering teams, many of which practice XP in some form.  With that in mind, let's see how the XP values (as a convenient starting point already familiar to many) can help improve the hiring process.

## Recent Examples

While the values speak very well for themselves (a nod to the wisdom of those who helped choose them), I wanted to take a quick tour through a recent job search and give concrete examples where the presence of practices inspired by XP values caused delight, or the absence of such practices caused something else entirely.

As usual, take this with a grain of salt -- it is just one person's experience.  It is based on ~30 conversations over a three- to four-week period, ranging from awesome to infuriating.  My hope is that real-world examples (with names changed to protect the innocent) will make a better case for an industry shift toward _Extreme Hiring_ than just skimming the values themselves.

### Simplicity

The hiring process itself should be just enough.  Note that "just enough" doesn't mean "too little" any more than it means "too much".  The right balance requires careful thought, and you will need to be pragmatic when designing the hiring process for your specific organization.

One of the companies I talked to erred on the side of too little.  After one technical conversation, they made an offer.  I have to admit, there was an immediate dopamine hit on the call and I felt really proud of myself.  I asked for 24-48 hours to consider, and when we hung up the doubt set in...  Why were they moving so quickly?  Did they really know I was a good fit based on one conversation?  Did I?

Several other companies erred on the side of too much...  I expect enough intelligent conversation to get to know the role and team, but some processes were ran more like academic drills than honest conversations.  "When we say jump, you ask how high!"

I'll give another side to this in "Respect" below, but too draconian of a focus on rote recitation of technical facts easily found on Google or in man pages probably isn't the best way to screen candidates in the limited time you have together.  Even if you come out feeling like a rock-star because you chirped static facts, did you really get to know anyone?  Did they get to know you?

### Communication

XP encourages daily communication amongst team members.  This is seen in many forms, including daily standups and pair programming.  This communication not only leads to better software, it also builds better relationships.  It's hard to delight candidates without good communication.

Daily communication with candidates may seem pointless (if nothing has changed), and is undoubtedly difficult (everyone's busy).  To a candidate who is eagerly awaiting news on a position they are excited about (you do believe people are excited to join your company, right?), frequent and concise communication can mean the difference between delight and frustration.

One company I spoke to made an amazing first impression.  The HR person was charismatic and helpful, going so far as to reassure me that even if one role didn't work out...they would guide me amongst other internal openings until they found one that did.  I left the call feeling elated, waiting on next steps.  A day passed...should I follow up?  A couple days passed, crickets.  I wrote the HR person, but got no response.  A couple days later (when I'd already started talking to other companies), I got a calendar invite for a technical exercise.  The exercise itself went well, then...crickets again.  I followed up, three times in total every few days.  I still heard nothing.  Two weeks later I got told they'd hired another candidate.  Why?  Good question, there was no explanation.  There was plenty of frustration.

Another similar example started with a seemingly positive first conversation, followed by three weeks before an email asking to schedule a technical test (while initially excited, I'd already received other offers).

Done right, communication doesn't require much overhead but provides comfort to candidates.  A simple, "Thanks for the conversation, we are working to schedule the next call/exercise/whatever" update is enough.  Understandably, people get busy and booking time for conversations across teams can be difficult...  ping candidates with lightweight updates every other day.  Several companies I spoke to were good at this.  Guess which ones I've chased for offers?

Aside from progress updates, this is obviously a very big part of the interviews themselves...  we really should practice interviewing like we do technical skills.  This could be another post in itself.  In my experience, good communication comes naturally very infrequently (no matter how good you think you are).  More so for knowledge workers and technical managers.  Incoherent, poorly-connected questions and awkward silences are the proof.  Invest in improved communication.

### Feedback

Much like creating a new piece of software, interviewing is a learning experience for all parties involved.  Even if a candidate is not a good fit today, by providing feedback that helps them grow, you help someone who could be an employee tomorrow.

Sadly, even when asked for feedback (whether in cover letters, technical interviews, direct to HR representatives, or all of the above) I've never found a company that provides feedback when things don't go as planned.  This is the perfect time for feedback, think retrospective!  Would you respect an engineering team that never provided feedback on failure?

Initially I thought this was to protect the investment in hiring procedures (e.g. giving away technical details to tests or similar exercises)...  However, at this point you've already ran through the process, seen the test, etc.  Unfortunately, it feels more like a case of "cutting losses" and writing off those who didn't fit the current requirements as less than human.

If we are to believe all the Agile/Lean/XP learnings around the quality of retrospectives, and seriously going to tout being more humane in business...take the time to give quality feedback throughout the hiring process.  Yes, this especially means a thoughtful "retro" with the candidates that didn't work out.  Who knows, today's "failures" may be critical to tomorrow's successes.

### Respect

It seems like we see headlines in the news every day about business leaders that have apparently forgotten how to show respect.  Often this behavior is inadequately justified with, "It's not personal, it's just business!"  What is business, beside a lot of relationships with people?  In honesty, it doesn't get any more personal!

Respect comes in many forms, but I want to talk about two situations where I've encountered both amazingly professional and less than adequate behavior.

First, let's talk about a very simple idea...  timeliness.  We've all had coworkers who were always late to meetings, apparently more important than anyone else in the room.  While this is most often seen as a concern for the interviewee, I argue it's just as important for the interviewer.  We all have busy schedules, and will be late for good reasons at some point in our life.  However, don't use that as an excuse.  Improve your process to make being late an exception rather than the rule (leave gaps between meetings to allow for breaks, don't schedule certain types of calls too close together, etc.).

On a recent interview, the interviewer showed up fifteen minutes late.  This is always bad for a candidate (did they forget about me?), but this was a technical test in a time box...as a result, the entire test felt more pressured.  To make matters worse, toward the end of the call, before asking if I had questions (and we should always have questions!) they prefixed the question with "I'm running late for the next call."  OK, don't want to hold you up!  Understandable, but certainly not delighted.

In another experience, I joined a video call ten minutes early...  and the interviewer was already there standing by.  I was trying to be early and make a good impression, but instead received the impression this person actually cared about talking to me!  In yet another example of mindfulness, the person politely interjected 15 minutes before the end of the call to remind me to save time for any questions of my own (vs just answering theirs).  I won't forget that call, and hope I can be as thoughtful when I next interview a candidate.

This is already getting long, so I won't add examples here...  but another important thing in the roadhouse of respect is remembering candidates are nervous.  Talking to a bunch of strangers who are doing their best to judge you does not bring out the best in most people.  As an interviewer, you should take it as a responsibility to help put candidates at ease.  Focus on thought process and dig into past accomplishments rather than focusing on recitation of technical facts easily found in the top three results of a Google search.  This should be a humane process, not a round of Jeopardy.  Respect people as humans, not robots.  The candidate should be seen as part of the interivew team.

### Courage

In XP, courage takes many forms...  one of which is the willingness not to become too attached to a piece of code (after all, it may get thrown out during refactoring).

In hiring, this also takes many forms...  so I'll give one important example -- honestly communicate bad news quickly.  Don't put off communicating bad news because it's hard.  Waiting is harder for the candidate.  Don't wait until you've interviewed a handful of candidates to ensure you have better options.  Have clear evaluation criteria, and if someone doesn't meet them, say so.  No need for more examples here, I've described one already in _Communication_ above.  When you have something to do that seems hard, be courageous.

## Summary

None of these principles are rocket science.  That's what makes them so appealing.  They are easy to understand, but take a lifetime of practice to consistently apply.  Just like XP, you don't have to adopt all of them...any can help.  Pick one that will have the most impact, practice until it is second nature, then pick the next most impactful.  Track progress, and strive to improve.  Hold one another accountable.

Much like a programmer who has learned TDD, but reverts to former habits of banging out large changes quickly without tests...  you will fall off the horse in your journey to "Extreme Hiring".  Don't be dismayed, failure is a chance to learn.  Hold a retrospective.  Make a commitment as an organization to continuously improve, and let the values guide you.

Overall this is nothing new...  the core values of XP, with a few simple examples based on personal experience.  However, much like we see when applied to programming, I truly believe being mindful of these values and applying them rigorously to one's hiring process is a path to happier people and a better industry that benefits everyone.  While not all-inclusive, my hope is this can serve as a convenient starting point that is a bit more concrete than "do better."  Be Extreme.
