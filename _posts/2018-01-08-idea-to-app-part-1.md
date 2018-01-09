---
published: true
author: Mike Hoskins
layout: post
---
# Introduction

Happy New Year!  2018 is here, and for the first blog post of the new year I wanted to start by exploring the creative process and **one possible** technology stack ([Node.js](https://nodejs.org), [Express](https://expressjs.com), [React](https://reactjs.org)) behind turning an idea into an app.  Since we need to cover a lot of ground to get from start to finish, this is going to be my first multi-part series...  Here are the currently planned sections:

- Ideation (you are here)
- UI, UX and client-side concerns
- Backend and APIs
- Deployment

Since there are so many great _How To_ articles, I want to start by giving you context so you understand where this one fits...  I am not a professional developer, though my background is CSCE and I work in technology (SRE).  I am new at building modern web applications, new at working with Node and Express (an [online bootcamp](https://www.udemy.com/the-web-developer-bootcamp) and a few months of tinkering), still learning [React](https://reactjs.org) (via [another online course](https://www.udemy.com/the-advanced-web-developer-bootcamp)), and have yet to actually publish a mobile app.

There will undoubtedly be imperfections and room for optimization in everything you read here -- keep in mind **this series is about sharing the overall process with an absolute beginner** vs something that will help seasoned professionals.  If you are a pro and have advice, please share comments or unicast me.  I am happy to share credit and adjust content as needed so future readers have the most helpful experience possible!

# Ideation

No matter how fun your tech stack, it's hard to produce something useful if you don't start with a good idea...  [Wikipedia defines _ideation_](https://en.wikipedia.org/wiki/Ideation_(creative_process)) as the generation of new ideas.  This sounds obvious and easy until you find yourself brainstorming and hitting a brick wall!

> _Ideation is the creative process of generating, developing, and communicating new ideas, where an idea is understood as a basic element of thought that can be either visual, concrete, or abstract._ -Wikipedia

There are schools of thought around how to generate new ideas, one of my favorites is [design thinking](https://en.wikipedia.org/wiki/Design_thinking).  One technique is using the [double diamond approach](https://medium.com/seek-blog/design-thinking-101-the-double-diamond-approach-ii-4c0ce62f64c7).  The key concept is using both divergent and convergent thinking to generate many possible solutions within a problem space.  This is 180 degrees from a common mistake -- jumping straight to a solution -- which can often blind you to related but potentially more interesting (or valuable) problems to solve.

# Finding Our Idea

If we were a team brainstorming from a blank slate, double diamond in conjunction with customer conversations would be a natural starting point...  for the sake of brevity, I already have an idea to offer up.  Before getting into the _what_ of my idea, how did I come up with the _why_?

> _Very few people or companies can clearly articulate WHY they do WHAT they do...people don’t buy WHAT you do, they buy WHY you do it._ -Simon Sinek

Equal parts education and inspiration, Simon Sinek's [Start with Why: How Great Leaders Inspire Everyone to Take Action](https://smile.amazon.com/Start-Why-Leaders-Inspire-Everyone/dp/1591846447) gives lots of insight into the power of understanding _why_...  as you use double diamond or other techniques to generate ideas, those with the strongest _why's_ will often resonate the loudest -- can you identify with the use case(s), do you feel excited about using the idea in day-to-day life (not just the fun of developing it)?

## Necessity is Your Friend

As frequent travelers and foodies, my wife and I were early adopters of the now ubiquitous food review site [Yelp](http://yelp.com).  As the site grew in popularity, more businesses and reviews were added.  While good in other ways, information overload began setting in -- what started as a way to quickly find good food began to get bogged down.

A scenario played itself out repeatedly...  we would be in a new city, usually just off a flight and starved to death, then leverage [Yelp](http://yelp.com) to find some chow.  Most of the time we would struggle to narrow down cuisine, try to improve our chances by prioritizing places with better reviews, and often end up deadlocked in endless opines over a handful of places that all seemed equally pleasant.

Wouldn't it be nice if there was an app that could optimize this scenario for speed?  Ideally allowing minimal user input (so it's not cumbersome on small screens), and abstracting away all the typical things we care about (good reviews, open now, etc.)?

## Birth of an App

Fleshing it out a bit more, I envisioned something like this:

- The app would need to be aware of user location (search nearby)
- The main feature should be "good food in a single click" (optimize _Time To Food_)
- Minimal user input, should be optional (_walk or drive_? _cheap or expensive_? etc.)
- Present random choice from top-rated options that are open
- Allow other choices if random choice is not satisfactory
- Make it easy to call or route to choices

It's amazing how the imagination runs wild as you start to think through a new idea.  I can almost see the workflow...  One other thing I wanted to tie in was some sort of _[potential for social good](https://youtu.be/SDetZaIKmPI)_.  I wasn't sure how yet, but I wanted it to be easy for anyone using the app to help feed others through charitable donations.

> _As programmers, we have a magic power, and with great power comes great responsibility.  We need to think about the good we can do with this ability._ -Aaron Swartz

Okay, so what to call it?  While not something to get hung up on, apps need a name, logo, and social profile if they have any chance of being found useful.  While not impossible to change, settling on a high level _brand_ for your app early will mean less re-work and confusion later.  My first unoriginal idea was simply calling it _Hungry_, but partially to avoid annoying [my friends](http://hungry.com), and partially at the urging of my wife (_"It's cuter!"_), opted for _ChowChow_ as the name (unfortunately chowchow.io was taken, I am soooo bummed).

<img style="float: left; margin: 1em" width=256 height=256 src="{{site.baseurl}}/media/chowchow_big.png">

Depending on what you're building, a logo and social presence could require real investment...  Since this is a simple app with no financial motivation, I cribbed [a _logo_ from iconarchive.com](http://www.iconarchive.com/show/windows-8-icons-by-icons8/Animals-Dog-Bone-icon.html) (it's available in several sizes, and will double as a good favicon), created [a GitHub repo](https://github.com/deadlysyn/chowchow), and moved along.

# Building a Mental Model

Now that we have our _why_ and _what_, it's time to think about _how_.  Don't bog yourself down unnecessarily (we'll get into implementation detail later), but do you understand the _big pieces_ of your vision at _a high level_?  Is it just stitching together readily available APIs?  Is there a lot of client-side logic, or is it mostly back-end?  Are there known patterns you can re-use to solve parts of the problem, or will you need to create a lot from scratch?

For me, there were a few big questions:

- How will we get the user's location?
- Does Yelp have an API?
    - If so, how do I access it?
    - What information is available?
- Once a business is found, what's the best way to plot a route?
- From a UX perspective, how can we allow unobtrusive user input?
- For charitable donations, can I do that over an API?

If I could answer those, it would increase my confidence the idea was realistic...  we'll answer each of these as we work through the series, but be aware that I took time to define and reflect on these questions.   You should for your idea as well.

> _Technology is notorious for engrossing people so much that they don't always focus on balance..._ -Paul Allen

Like so many things in life, it is about balance...  don't get stuck in analysis paralysis.  However, a little extra time spent upfront will avoid the frustration of getting bogged down later by letting you focus on clearly defined pieces of the problem and ensuring you don't head in an entirely wrong direction architecturally.  That said, if you do find yourself in a bog...  don't panic.  We've all been in a similar place.  Take notes, find a better way, learn from it, share your experience!

# Defining the Workflow

At this point, I found myself thinking a lot about workflow.  My goal was to hone existing skills ([Node](https://nodejs.org), [Express](https://expressjs.com) by building a web-based version of the app.  Later (as I improve in [React](https://reactjs.org), and figure out [Reactive-Native](http://facebook.github.io/react-native)), I want to produce a mobile version.

I had a few concerns:

- What should the "happy path" actually look like?
- How would I build a web app requiring minimal refactoring for mobile?
- How could I ensure any controls were easy to use on small screens?

I began working through the first with ye olde pen and paper.  Whiteboards are great too.  The point is really to step back from the keyboard for a moment and brainstorm the user experience.  You could mock this out in your favorite graphics editor, but I prefer a medium where you can quickly erase and rework.  This consumed a few pages for me, including several that got tossed, but the picture shows how informal and messy it can be...anything to solidify the idea of what you are trying to build.

![A Messy Sketch]({{site.baseurl}}/media/workflow_sketch.jpg)

You can see simple sketches of the main app screens with some zoom detail, and notes on _must haves_ vs _roadmap features_.  With this, I was able to visually step through the path a typical user would follow.  This is a starting point that will be continuously refined, but helps the idea start to feel tangible.  Sketches of individual screens, elements, arrows/flow between screens, dialogs or other callouts -- anything is fair game, but use several pages to ensure they are legible.  Nothing is worse than forgetting what your awesome diagram actually meant a few days later.  For larger apps, a single sheet of paper might represent a page, so you can tape them to the wall, rearrange as needed, and visualize the big picture.

> Throughout brainstorming you will have countless ideas...write them down, and organize by _must haves_ vs _roadmap_.  Holding items in your head risks forgetting them, and clutters mental space best focused on developing what must come next.

With a happy path in mind, I started thinking more about the last two items since they are intricately related...  While the back-end would continue to evolve, I wanted to avoid constantly refactoring the UI.  The way I wanted to avoid major refactoring was by building a web UI that mimicked the mobile experience as much as possible.  [I made some hard choices about UI dimensions](http://www.idev101.com/code/User_Interface/sizes.html), content (nothing extraneous) and controls (everything has to work easily on a small touch screen).

I mostly worried about smaller screens because I knew the most useful place for the app would be on mobile devices...  This isn't an enterprise app with a need to seamlessly support every known platform, but I did want a smooth experience whether in a modern browser or mobile device.  With the above in mind, I decided the best way to really tackle this was to ship something fast and start testing on real mobile devices.  Sure beats worrying!

# Mocking the UI

At this point I was ready to move past workflow and really start defining look and feel.  Based on your skillset, you might prefer to work _bottom up_ (focus on the back-end first), but I've always found a _top down_ process to be more natural.  With this approach, we start by generating key UI elements, then wrap code around them.

I used the git repo while working through this part (laying things out where I thought they would naturally live in production), but want to share the project skeleton and GitHub repo in the next article when we setup our Docker development environment.  Hopefully that's not too confusing.  You might not even have a repo ready at this point, aren't really writing code (just focusing on presentation via HTML and CSS), and directory structure could even change as you figure out what components to use and how to stitch things together.

## Home Sweet Home

For me, the obvious focus was the index page...the literal starting point.  If I could get this halfway decent, it could serve as a template for the rest of the app (while content will change on each screen, we want general UI consistency).  It also offered an opportunity to tackle one of the key challenges, figuring out what user input would be allowed and control types that would be easy to navigate on small screens with touch controls.

A lot could be said here, but since I was developing for the web at this point simple HTML and [the CSS box model](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model) were my tools of choice.  I started by simply laying out a few divs and experimenting with size and position.  A common trick I find useful at this point is giving all elements a hideous border (usually something like _border: 1px dashed red_) to make subtleties for things like floats and overflows easier to see.  While useful, be aware these extra pixels will take up space in your design.

![CSS Box Model]({{site.baseurl}}/media/box_model.jpg)

While working around browser bugs or platform inconsistencies can be infuriating (we'll touch on that in a later part of this series), I find initial UI prototyping rewarding.  By now you've got a solid idea, and get to slap Legos together until they match your mental picture.  It takes patience, but the satisfying part is instant feedback as you add HTML, adjust CSS, reload, repeat...  make small changes, and verify often.  Goes well with coffee.  :-)

While working through the layout, make friends with your browser's native web developer tools (_cmd-alt-i_ for Mac users).  This is not only useful for Javascript debugging in the console (we'll use that later!), but also identifying styling discrepancies, specificity conflicts, or other oddities via the element inspector.

![Element Inspector]({{site.baseurl}}/media/chowchow_inspector.jpg)

Don't spend so much time on the UI you never get the back-end working.  It sounds silly, but this is an easy black hole to get stuck in.  You will almost always think of small tweaks, so remember the Pareto Principle -- the average user will only touch ~20% of your UI.  Put another way, users will derive 80% of their value from 20% of your development.  If you hope to make the 20% of your code users actually care about 80% better, you need to ship sooner so tweaks can be based on feedback.

> You will almost always think of small tweaks, so remember the Pareto Principle -- the average user will only touch ~20% of your UI.

At this point I made a mistake that wasn't the end of the world, but might be useful to share...  Since I wanted a mobile-friendly, responsive design I jumped to a familiar framework ([Bootstrap 4](https://getbootstrap.com)).  After all, I'd just gone through [a class where we used it heavily](https://www.udemy.com/the-web-developer-bootcamp), [my own website uses it](http://deadlysyn.com), and it is both mobile-first and full-featured.

_Full-featured_ was the problem...  this isn't anti-Bootstrap, it's a great framework.  This also isn't a Bootstrap problem, it's a general concern with frameworks.  There's a time and a place.  About halfway through this app I realized what I'd done: I'd pulled an 800-pound dependency into my project when all I needed was [CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout).  Sometimes it's not obvious, but as I worked through the rest of the UI, I found myself using framework elements, overriding styles, replacing elements with completely custom versions since the resulting CSS was shorter, then wondering if I was committing sacrilege by not only using native components -- it shouldn't be this hard, I was fighting the framework.  Oh well, lesson learned.  Add one to the _roadmap_ list!

![Yes, that's EJS, forgive me. :-)]({{site.baseurl}}/media/chowchow_divs.jpg)

Now that I had a handful of responsive divs laid out, I wanted to apply some color and texture.  Since I'm not a graphic designer by trade, I jumped over to [uigradients.com](https://uigradients.com) (grabbed a background gradient for my container div) and [heropatterns.com](http://www.heropatterns.com) (grabbed a conveniently themed SVG fill pattern).  While browsing free resources, I ran what CSS I had through [autoprefixer](https://autoprefixer.github.io).  For UI icons, I pulled in [Font Awesome 5](https://fontawesome.com/get-started).

Last but not least, after a bit of research, I decided simple _toggle switches_ would be aesthetically pleasing and easy to use on small screens.  I came across [a CSS-native implementation](https://github.com/ghinda/css-toggle-switch) and decided to give it a try.  We'll go over importing and using css-toggle-switch in the next article of this series.

With basic HTML, CSS, and a handful of free utilities I had the skeleton of a UI...  here's a teaser of the home screen we'll get to build next time:

![ChowChow Home Screen]({{site.baseurl}}/media/chowchow_home.jpg)

# Conclusion

Starting only with a simple desire to bolster existing skills by building an app, we considered user needs (in this case our own) to identify a valuable idea (find our _why_).  Finding an idea was easier for us since it was based on personal experience, but it's worth noting it resonated because it was based on an understood problem...a place we as programmers could leverage technology to meet a need.  Finding such opportunities generally requires talking to and/or observing your target users.

With our shiny new idea in hand, we fleshed out how it could work at a high level.  Next, we took a moment to think about the major components of our app and did research to understand if our idea was feasible.  Finally, we hammered out a sensible workflow based on what we wanted the app to do, and did simple mocks in HTML and CSS...now we have something to start wrapping code around!

That was quite a journey for an introduction...  but it should lay a nice foundation for the next article in the series where we will go deeper into the UI design and address other client-side concerns like location and user agent detection.  We'll also devote a small section to project setup (sharing our directory structure to make following code examples easier), and discuss the simple Docker-based development environment.

I hope you'll follow the series as our simple app idea continues to evolve...

Thanks for reading!