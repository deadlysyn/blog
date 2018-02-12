---
published: true
author: Mike Hoskins
layout: post
tags:
  - frontend
  - javascript
  - web
category: development
---
# Introduction

We were pretty meta in [part one of this series](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) -- running through a typical thought process and common techniques for turning ideas into something tangible...  including how to come up with ideas in the first place, thinking about workflow once you have one, and mocking a UI that lets us start visualizing a [Minimum Viable Product](https://en.wikipedia.org/wiki/Minimum_viable_product)).

This is where things start to get fun!  We can finally delve deeper into _code_ (just in case CSS can't actually be called code, we'll start a bit of JavaScript as well).  As we said in [Part 1](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1), there's a lot of ground to cover...  Just in case you forgot, here's the planned breakdown:

- [Ideation](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) (Part 1)
- UI, UX and client-side concerns (you are here)
- [Backend and APIs](http://deadlysyn.com/blog//2018/02/11/idea-to-app-part-3)
- Deployment

Let's get started!

# Slight Detour: Dev Environment

In [Part 1](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) I promised to share details on the _dev environment_ we'll be using for this project.  The first thing you need to check out is the GitHub repo.  [Go browse that now if you'd like.](https://github.com/deadlysyn/chowchow).  I'll wait.  When you're done, clone it so you can follow along:

```Bash
$ mkdir -p ~/src; cd ~/src
$ git clone https://github.com/deadlysyn/chowchow.git
```

Back?  Cool.  This has been documented by others far better than I ever could, so we'll keep it brief...  If you have questions, there are lots of great references available for [Docker in general](https://docs.docker.com) (one of the most amazingly well-documented projects around!), as well as specific guides for [Node.js](https://nodejs.org) development using [Docker](https://www.docker.com/docker-community)...like these, just to share a few:

- [The Official Guide](https://nodejs.org/en/docs/guides/nodejs-docker-webapp)
- [A Great Medium Article on Docker+Node](https://medium.com/@sunnykay/docker-development-workflow-node-express-mongo-4bb3b1f7eb1e) (Though we're not using MongoDB here)
- [Another Awesome Guide for Larger Projects](http://paislee.io/the-ultimate-nodejs-development-setup-with-docker)
- [Instant Node.js Development Environment](https://medium.com/@stapait/instant-node-js-development-environment-with-docker-cd2de2cc0e90?source=userActivityShare-9cdc41953872-1516484723)

Compared to those, ours is quite simple...but we have a couple things key things in common.  [Here's our Dockerfile](https://github.com/deadlysyn/chowchow/blob/master/Dockerfile):

```Dockerfile
FROM node:latest

RUN mkdir -p /app
WORKDIR /app

COPY package.json /app/
RUN npm install
RUN npm install -g nodemon

EXPOSE 3000

CMD [ "npm", "start" ]
```

Key things to note:

- We just pull the latest [Node.js](https://nodejs.org) image as a starting point (you might want to lock this down to a specific version for a production app)
- We create `/app` within the container as our working directory (more on this later)
- We copy `package.json` from the current directory (e.g. our cloned GitHub repo) and install all project dependencies with `npm install`
- We carefully put dependency bits toward the top of the `Dockerfile` to [maximize cache hits](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#sort-multi-line-arguments)
- We `EXPOSE` port `3000` within the container (more on this later)

Next we use what may be the world's tiniest `Makefile` to make building and running our image easier (isn't it amazing how useful _old_ tools can be?):

```Makefile
DOCKER=/usr/local/bin/docker
IMG=chowchow

build:
	$(DOCKER) build . -t $(IMG)

run:
	$(DOCKER) run --rm -v $(PWD):/app -p 3000:3000 \
		-e IP="0.0.0.0" \
		-e API_KEY="${API_KEY}" $(IMG)

```

This lets us `make build` to build an image from our `Dockerfile`, and `make run` to bring up our app in the container on our machine.  Note how we use `-p` to map port `3000` on our machine to port `3000` in the container.  This means you can browse `localhost:3000` to access the application during development.

`IP=0.0.0.0` just gets the application listening on all available network interfaces (if you simply bind to `127.0.0.1` aka _loopback_, you won't be able to access the application from outside the container).  `API_KEY` deserves more explanation, but I want to save that for _Part 3: Backend and APIs_ so we don't muddy the waters too much here.  The big takeaway for now is that you can pass environment variables (defined in your shell when running [Docker](https://www.docker.com/docker-community)) into the container to control application behavior.

The `--rm` avoids leaving old versions of our container around (we want to rebuild each time we make a change), and `-v` maps our current directory into the container's `/app` we created in the `Dockerfile`.

The last piece of this puzzle is understanding the _start command_ in our `package.json`.  Let's look at that:

```JSON
{
  "name": "chowchow",
  "version": "1.0.0",
  "description": "find food fast",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon app.js"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/deadlysyn/chowchow.git"
  },
  "author": "Mike Hoskins",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/deadlysyn/chowchow/issues"
  },
  "homepage": "https://github.com/deadlysyn/chowchow#readme",
  "dependencies": {
    "body-parser": "^1.18.2",
    "ejs": "^2.5.7",
    "express": "^4.16.2",
    "express-session": "^1.15.6",
    "memorystore": "^1.6.0"
  }
}
```

When you first `npm init` a new project, there will be no `start` command.  We've added `"start": "nodemon app.js"` which causes the `npm start` command in our `Dockerfile` to run our application under supervision of `nodemon` (If you haven't heard of `nodemon`, see [here](https://nodemon.io) and [here](https://www.npmjs.com/package/nodemon)).  This causes the application to restart anytime we change JavaScript source within the project so we can easily preview changes.

For the most part you can ignore the rest of the fields for now, but if you haven't worked with `package.json` before and want a deep-dive, [check out the official documentation](https://docs.npmjs.com/files/package.json).

# Project Hierarchy

It always helps to know how a project is laid out so you can follow along, and we'll delve deeper into this in the next part of the series where we focus on [Node.js](https://nodejs.org) and [Express](https://expressjs.com) particulars.

For now, since this is a small project, you can just [browse the repository](https://github.com/deadlysyn/chowchow) to get a feel for things.  The main thing is awareness of what a view is (loosely, a template rendered when users access specified URIs or routes served by our application), where those live (in the aptly named `views` directory), and where assets like CSS reside (inside the `public` directory).  Here's a simplified tree view of the project, with only areas we'll work with in this part of the series:

```
.
├── Dockerfile
├── Makefile
├── app.js
├── package.json
├── public
│   ├── css
│   │   ├── main.css
│   ├── js
│   │   ├── bowser-191.min.js
│   │   ├── fontawesome-all-5.js
│   │   ├── home.js
│   │   ├── list.js
│   │   └── random.js
│   └── media
│       ├── chowchow.png
│       ├── favicon.png
└── views
    ├── catchall.ejs
    ├── home.ejs
    ├── list.ejs
    ├── partials
    │   ├── footer.ejs
    │   └── header.ejs
    └── random.ejs
```

# Our First Refactor

In [Part 1](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) we discussed how I started with [Bootstrap](https://getbootstrap.com) when mocking the UI because it was familiar.  [The bootcamp I went through](https://www.udemy.com/the-web-developer-bootcamp) leveraged it heavily, it's one of the most popular CSS/JS frameworks, and it's well-documented and easy to use...  but for such a simple app, it was a huge dependency to pull in.

I also found myself fighting the framework.  The main requirement we have is a responsive design.  Responsive design is largely about ensuring your application works well on a variety of screen sizes (desktop, mobile, etc.).

> Responsive web apps use technologies like media queries and viewport to make sure that their UIs will fit any form factor: desktop, mobile, tablet, or whatever comes next. -[MDN](https://developer.mozilla.org/en-US/Apps/Progressive)

Since we are building a web application on a journey toward native mobile (where our app would be most useful), responsiveness is key.  While [Bootstrap](https://getbootstrap.com) provides responsiveness, it's achievable with native CSS as well.  It felt a bit silly to pull in [Bootstrap](https://getbootstrap.com) for what boiled down to a single row and column.  I found myself with something like this in each view:

```HTML
<div class="container">
    <div class="row">
        <div class="col">
        </div>
    </div>
</div>
```

I decided to rip out the framework and use this as an opportunity to jump on the [CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout) bandwagon.  Nothing against [Bootstrap](https://getbootstrap.com), but there is a time and a place for frameworks.  I also simplified the interface quite a bit, removing another dependency ([css-toggle-switch](https://ghinda.net/css-toggle-switch/)) which is amazing in itself but felt too heavyweight for our needs.

> Dependencies are a way of life (who wants to re-invent the wheel), but it's important to be mindful of what you pull into your project.  Make judgement calls on whether the added complexity makes sense in your context.

In the end, it turned into a bit more work than originally thought...  mostly because I had started using things like special formatting and classes to define buttons and other UI elements.  What started as _I'll just rip out a few divs_ turned into quite a journey, but the end result was a responsive UI, fewer dependencies, FAR less CSS users are forced to download (a whopping ***96.43% less CSS*** to be exact!), and a lot of learning.  :-)

# Getting Stylish

As mentioned, we're going to use CSS Grid to layout our simple app.  Be warned, I'm learning with you and not a CSS master by any means.  I'm sure everything I show you can be done more efficiently -- if you spot something, unicast me or share in comments.  I've refactored things to reduce duplication in some places, but know it could be cleaner.

We'll also be using another CSS layout mechanism called [Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox).  Before going further, take time to review the following resources so it's easier to follow along:

- [Per Harald Borgen's](https://medium.com/@perborgen) [AMAZING CSS Grid Tutorial](https://scrimba.com/g/gR8PTE)
- [A Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox)
- [Jen Simmons' Reading List](http://jensimmons.com/post/feb-27-2017/learn-css-grid)

You also want to get familiar with your browser's _developer tools_.  In Firefox, you can simply hit `F12` to toggle the developer tools.  The _DOM and Style Inspector_ (the _Inspector_ tab along the top menu bar of the development tools window) and JavaScript _Console_ will be invaluable friends during your web development journey.

![Inspector in Action]({{site.baseurl}}/media/chowchow_inspector_big.jpg)

The Inspector has a lot of useful features, the screenshot highlights a few I use regularly:

- Responsive design mode lets you mimic a variety of screen sizes and bandwidths
- You can select the small pointer icon to focus on elements you click
- For grid layouts, you can click the small grid icon beside `display: grid` to show an outline of the grid (very useful!)
- You can `shift-click` color swatches to see alternate formats like `rgb` and `hsl`

Before we dive into the restyle...  let's look at the original UI we mocked in Part 1:

![ChowChow UI 0.1]({{site.baseurl}}/media/chowchow_original_ui.jpg)

Pretty hideous, huh?  Well, I felt pretty good about my design to start -- after all, it was my brainchild...  but looking closer it had a lot of shortcomings (other than the 90's-flashback punch-out effect I felt so clever devising).

> Fight the urge to get attached to your designs.  In the spirit of Agile/Lean, be courageous -- experiment and change often!

Look and feel aside, it simply wasn't responsive...and didn't work at all on smaller screens.  Even if it had looked amazing, it didn't meet our primary requirement (finding decent food quickly while traveling with a ***mobile device***).  Loading the app in the _Inspector's Responsive Design Mode_ with a resolution of 320 x 480 was the final nail in the coffin.

![Aww so sad...  :-(]({{site.baseurl}}/media/chowchow_broken_ui.jpg)

Let's see how we can improve the mobile experience...

# Defining our Views

In [Part 1](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) we defined a workflow consisting of three views...  the _home view_ (or index page) which allowed users to modify purposefully-minimal (e.g. not cumbersome on mobile) inputs, a _random view_ to display information about a randomly selected restaurant, and finally a _list view_ which allows users to browse additional choices if the random selection is not up to par.  Here's an artist's rendition (hopefully you've caught on to the inherent snark):

![Getting Sketchy]({{site.baseurl}}/media/chowchow_ui_sketch.jpg)

Let's walk through the HTML and CSS to build each of these in a [responsive manner](https://en.wikipedia.org/wiki/Responsive_web_design)...  Keep in mind that while I am trying to separate this series into sections for convenience, back-end bits inevitably bleed into front-end.  You will see a bit of [EJS template](http://www.ejs.co) detail and even helper functions as we work through the markup...  I'll explain important bits, but we'll also go deeper into all that in the next part of this series.

## Common Elements

Before we get into specific views, I want to share some common elements all views will leverage.  There's more we could do, but the idea is to factor out anything we can as reusable classes to reduce duplication in our CSS.  I want to share this now since it will avoid confusion later when we reference some of these in our markup:

```CSS
/*
 * common elements
 */

.button {
    align-items: center;
    box-sizing: border-box;
    color: #d32323;
    display: flex;
    font-size: 4vh;
    font-weight: 700;
    justify-content: center;
    letter-spacing: 2px;
    text-align: center;
}

.button:hover {
    box-shadow: inset 0px 0.1em 0.1em rgba(0, 0, 0, 0.4),
                inset 0px -0.1em 0.1em rgba(0, 0, 0, 0.4),
                inset 0.1em 0px 0.1em rgba(0, 0, 0, 0.4),
                inset -0.1em 0px 0.1em rgba(0, 0, 0, 0.4);
    color: #000;
    transition: all 0.25s;
}

.footer {
    grid-area: footer;
    opacity: 0.7;
}

.text-shadow {
    text-shadow: 0px 2px 4px rgba(211, 35, 35, 0.7),
                 0px -2px 4px rgba(211, 35, 35, 0.7),
                 2px 0px 4px rgba(211, 35, 35, 0.7),
                 -2px 0px 4px rgba(211, 35, 35, 0.7);
}
```

Nothing special here, we simply define a bit of custom CSS to get back the concept of a _button_ we lost when we ditched [Bootstrap](https://getbootstrap.com) and define the `footer` which will be a common element on every page.

Note the use of the `transition` property to make the hover effect feel a bit better (to me at least).  Also note the use of a relative `font-size` based on display size (using the `vh` unit which we'll discuss later).  That's key to getting the responsiveness we need.  `grid-area` won't make sense just yet, read on!

## Home

The _home view_ gives our users a starting point.  In the next part of this series we'll talk about details of defining and rendering the view in the [Express](https://expressjs.com) framework, here we want to focus on the style.  The first thing we want to do is use CSS Grid to define a layout we can map our UI elements onto:

```CSS
#home {
  display: grid;
  grid-template-columns: auto;
  grid-template-rows: repeat(4, auto) 5vh;
  grid-template-areas: "find" "price" "drive" "donate" "footer";
}
```

Be sure to review the [CSS Grid Tutorial](https://scrimba.com/g/gR8PTE) shared above so this makes sense...  I've simply defined an `id` we can use in our HTML which defines a grid consisting of five rows and a single column.  I then name each of the rows for easy reference later.  You might be more familiar with another format for `grid-template-areas`, something like:

```CSS
grid-template-areas:
    "find"
    "price"
    "drive"
    "donate"
    "footer";
```

While useful for visualizing multi-column layouts, it seems a bit unwieldy for simple pages like this where we only have a single column so I merged everything onto one line.  Either way works just fine.  Which do you prefer?

Why do this in an `id`?  Because we will have a specific grid layout per view, and only need to use each layout once (a requirement for CSS ids vs classes like Bootstrap's `row` and `column` which can be re-used).

OK, great...so how do we use this?  While refactoring, I got over-zealous (I might regret it later) and decided to eliminate as many extraneous wrappers as possible.  I didn't want the equivalent of `<div class="container"></div>`, so chose to map the grid definitions to my `<body>` elements.  Here's what we see in `views/home.ejs`:

```HTML
<% include partials/header %>
<body id="home">
  ...
```

I like the way this effectively uses the body as the container, but it could be less flexible based on your requirements.  Just keep in mind that this is just one way to define our grid, and [not even one I thought up myself](https://medium.com/flexbox-and-grids/how-to-efficiently-master-the-css-grid-in-a-jiffy-585d0c213577?source=userActivityShare-9cdc41953872-1516481482)...I just found it novel.  Do you, or is it setting off alarm bells?  :-)

Now that we've got a grid, and it's mapped onto our page...  let's add some content.  We do that by assigning elements to grid areas.  Our simplified _home view_ is just four buttons (_find food_, _adjust price range_, _toggle walk/drive_, or _make a donation_) which should fill the screen and be responsive:

```CSS
/*
 * home view styling
 */

#find {
    background-color: rgba(255, 0, 0, 0.4);
    grid-area: find;
    height: 21vh;
    max-height: 180px;
    width: 100%;
}

#price {
    background-color: rgba(255, 125, 0, 0.4);
    grid-area: price;
    height: 21vh;
    max-height: 180px;
    width: 100%;
}

#drive {
    background-color: rgba(255, 255, 0, 0.4);
    grid-area: drive;
    height: 21vh;
    max-height: 180px;
    width: 100%;
}

#donate {
    background-color: rgba(125, 255, 125, 0.4);
    grid-area: donate;
    height: 21vh;
    max-height: 180px;
    width: 100%;
}
```

We use `grid-area` to map elements onto our grid, and relative sizes for `height` and `width` to get the desired responsiveness (`vh` stands for _[view height](https://web-design-weekly.com/2014/11/18/viewport-units-vw-vh-vmin-vmax)_).  This mostly gets the desired behavior on the platforms I can easily test (Chrome, Firefox and Safari on MacOS, as well as mobile Safari on an iPhone 7), but is almost certainly not the best way to do it (finding all the neato ways designs break on various devices is part of the fun of being a web developer).

When I first tested this, I tweaked the heights to look just right in _developer tools_ at 320 x 480.  When I tested on the iPhone, it was close but didn't quite fit the screen...  Not a major surprise given resolution differences, but don't assume every device is happy just because something looks good in a simulator.  Be aware it can take some patience and a bit of trickery to find a good middle ground that mostly works on all devices.  :-)

Let's look at the home view's markup:

```HTML
<% include partials/header %>
<body id="home">
    <script defer src="/js/home.js"></script>
    <form action="/random" method="post" id="form">
        <input type="hidden" name="price" value="$$">
        <input type="hidden" name="drive" value="true">
        <input type="hidden" name="latitude" value="0" min="0" step="Any" id="latitude" required>
        <input type="hidden" name="longitude" value="0" min="0" step="Any" id="longitude" required>
    </form>
    <div class="button" id="find">
        <span id="loading">
            <i class="fas fa-spinner fa-pulse fa-lg" aria-hidden="true"></i>
        </span>
    </div>
    <div class="button" id="price">
        <i class="fas fa-dollar-sign fa-lg" aria-hidden="true"></i>
        <i class="fas fa-dollar-sign fa-lg" aria-hidden="true"></i>
    </div>
    <div class="button" id="drive">
        <i class="fas fa-dot-circle fa-lg"></i>
    </div>
    <div class="button" id="donate">
        GIVE <i class="fas fa-heart fa-lg" aria-hidden="true"></i> FOOD
    </div>
<% include partials/footer %>
```

The `price` and `drive` inputs are controlled by JavaScript associated with the divs which have similarly-named ids.  We'll look at more of that code in a future part of this series, or you can [check it out in GitHub](https://github.com/deadlysyn/chowchow/blob/master/public/js/home.js).  As we'll see below, we auto-fill the `latitude` and `longitude` inputs using results we get back from `geolocation`.  When the user submits the form, these will be passed along to Yelp's API.  A future feature could be allowing the user to enter a zip code if location is unavailable.

Overall, we can see how removing [Bootstrap](https://getbootstrap.com) has really simplified our layout.  Our markup now neatly corresponds to design elements.  No more extraneous `container`s, `row`s or `column`s let alone class names like `col-xs-12` which are not at all obvious unless you've spent time with the documentation!

Those weird `<% %>` tags signify EJS ([Embedded JavaScript templates](http://www.ejs.co))...  The small amount of EJS here is pretty clear, we simply include a header and a footer to avoid having to type common bits like `<html></html>` in every view.  Where you see things like `<%= var %>`, we're simply including a variable from our application in the template.  One thing we need to understand is a bit of magic included in `views/partials/header.ejs`:

```HTML
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, height=device-height, initial-scale=1.0, maximum-scale=1.0">
        <title>chowchow</title>
        <link rel="icon" type="image/png" href="/media/chowchow.png">
        <link rel="stylesheet" href="/css/main.css">
        <script src="/js/bowser-191.min.js"></script>
        <script defer src="/js/fontawesome-all-5.js"></script>
    </head>
```

A key part of our responsive design is the `viewport` we define via a `meta` tag (which should be the first `meta` tag in your `head` after `charset`).  This allows the UI to scale based on the target device's screen-size.  If you want to learn more about that, [check out the MDN documentation]( https://developer.mozilla.org/en-US/docs/Mozilla/Mobile/Viewport_meta_tag), [as well as this oldie but goodie](http://www.allenpike.com/2010/choosing-a-viewport-for-ipad-sites).

This all works reasonably well on a small screen, but looks a bit odd in a larger desktop browser.  To fix that, we'll need to leverage [CSS media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) a few places in our design.  You could get as fancy as you like here, I simply add some spacing and give our buttons a different shape on larger screens:

```CSS
@media screen and (min-width: 600px) and (min-height: 800px) {
    html {
        padding-top: 5vh;
    }

    body > div {
        border-radius: 10px 40px;
        margin-top: 1vh;
    }
}
```

With that, we have something that looks decent on small and large screen sizes:

![Media queries adjust our layout based on display size.]({{site.baseurl}}/media/chowchow_ui_side_by_side.jpg)

We've covered a lot of ground in this section, but luckily for us many of these concepts will be reused in each view.  This will save us a lot of time as we work through the remaining sections.

## Random

In `views/random.ejs` we'll use CSS grid for our layout again...but define a completely different grid suited to our needs.  The main feature will be a large content area, followed by a few buttons (_route_, _call_, _home_, _list_).  Going further, we'll re-define the grid based on screen size for an optimal layout.  Here's the _random view's_ CSS:

```CSS
/*
 * random view styling
 */

#random {
    display: grid;
    grid-template-columns: auto;
    grid-template-rows: repeat(4, auto) 5vh;
    grid-template-areas:
        "content content"
        "route route"
        "call call"
        "back list"
        "footer footer";
}

@media screen and (min-width: 600px) and (min-height: 800px) {
    #random {
        grid-template-rows: repeat(5, auto) 5vh;
        grid-template-areas:
            "content"
            "route"
            "call"
            "back"
            "list"
            "footer";
    }
}

#random-content {
    grid-area: content;
    height: 60vh;
    max-height: 350px;
    overflow: auto;
    width: 100%;
}

#random-biz {
    background-color: rgba(0, 0, 0, 0.4);
    color: #fff;
    font-weight: 700;
    line-height: 0.5em;
    overflow: hidden;
    padding: 1em 0;
    width: 100%;
}

#js-random-route {
    background-color: rgba(255, 0, 0, 0.4);
    grid-area: route;
    height: 11.66vh;
    max-height: 100px;
    width: 100%;
}

#js-random-call {
    background-color: rgba(255, 125, 0, 0.4);
    grid-area: call;
    height: 11.66vh;
    max-height: 100px;
    width: 100%;
}

#js-random-home {
    background-color: rgba(255, 255, 0, 0.4);
    grid-area: back;
    height: 11.66vh;
    max-height: 100px;
    width: 100%;
}

#js-random-list {
    background-color: rgba(125, 255, 125, 0.4);
    grid-area: list;
    height: 11.66vh;
    max-height: 100px;
    width: 100%;
}
```

(_Psst...I see duplication sneaking into our code.  Make a note on the refactor list!_)

One thing worth noting is the `js-*` naming convention for some of the elements.  That's a signal those elements are tied to JavaScript (which we'll get to later).  In a project of this size it's not a big deal, but will make maintenance easier as our codebase grows.  It's [another fine idea I didn't have to come up with myself](https://medium.freecodecamp.org/css-naming-conventions-that-will-save-you-hours-of-debugging-35cea737d849)!  Here's the markup:

```HTML
<% include partials/header %>
<body id="random">
    <script defer src="/js/random.js"></script>
    <% if (biz.image_url) { %>
    <div id="random-content" style="background-image: url('<%= biz.image_url %>'); background-position: center; background-size: cover">
    <% } else { %>
    <div id="random-content" style="border: 1px solid #000">
    <% } %>
        <div id="random-biz" class="text-shadow">
            <p><%= shorten(biz.name) %></p>
            <p><%= biz.location.display_address[0] %></p>
            <p><%= parseFloat(biz.distance / 1609.344).toFixed(1) %> miles</p>
            <p><a href="<%= biz.url %>"><%- stars(biz.rating) %></a> (<%= biz.review_count %> reviews)</p>
        </div>
    </div>
    <div class="button" id="js-random-route">
        <i class="fas fa-map-marker-alt fa-lg"></i>
    </div>
    <div class="button" id="js-random-call">
        <i class="fas fa-phone fa-lg"></i>
    </div>
    <div class="button" id="js-random-home">
        <i class="fas fa-home fa-lg"></i>
    </div>
    <div class="button" id="js-random-list">
        <i class="fas fa-th-list fa-lg"></i>
    </div>
    <script>
    window.onload = function() {
        updateMapURL("<%= biz.location.display_address %>")
        updatePhone("<%= biz.phone %>")
    }
    </script>
<% include partials/footer %>
```

We're starting to leverage EJS more and more...  working around the fact that `biz.image_url` may or may not exist (sometimes a business doesn't have an image defined), so we avoid render errors by wrapping associated output in a conditional.  We also use a helper function (more on those later) and `parseFloat` along with `toFixed` to convert meters to miles.

On the last few lines, we take the `biz` object's (a Yelp API search result) address and phone number to pass into a couple JavaScript functions we've created to generate the correct links for routing to or calling selected businesses.  Since there is only one result displayed on this page, we use the `window.onload` event to set everything up when the view is rendered.  Be sure to check out `updateMapURL` and `updatePhone` in [public/js/random.js](https://github.com/deadlysyn/chowchow/blob/master/public/js/random.js).

Overall, this is still a fairly clean view and achieves the desired responsiveness:

![Random View]({{site.baseurl}}/media/chowchow_random_view.jpg)

## List

Last but not least, we want to let the user browse additional choices when the randomly-selected restaurant is unsatisfactory.  Enter the list view.  What happens behind the scenes is our app takes the _top five_ restaurants near the user (perhaps less, if there are not actually five, adequately-ranked, open choices nearby!), displays a random choice, and then saves the remaining choices for later use if the user wishes to browse through them (we don't need to hit the API again).

By now CSS Grid is old-hat, here's our styling for the list view:

```CSS
/*
 * list view styling
 */

#list {
    display: grid;
    grid-template-columns: auto;
    grid-template-rows: auto auto 5vh;
    grid-template-areas: "content" "home" "footer";
}

#list-content {
    align-content: center;
    display: flex;
    flex-wrap: wrap;
    grid-area: content;
    height: 70vh;
    max-height: 700px;
    width: 100%;
}

#js-list-home {
    background-color: rgba(255, 255, 0, 0.4);
    grid-area: home;
    height: 15vh;
    max-height: 100px;
    width: 100%;
}

.list-item {
    color: #fff;
    display: flex;
    font-weight: 700;
    height: 17vh;
    justify-content: center;
    max-height: 150px;
    width: 100%;
}

.list-item-route {
    background-color: rgba(0, 125, 0, 0.4);
    display: flex;
    height: 100%;
    width: 15%;
}

.list-item-content {
    align-content: center;
    display: flex;
    flex-wrap: wrap;
    width: 70%;
}

.list-item-content > div {
    display: flex;
    justify-content: center;
    width: 100%;
}

.list-item-call {
    align-items: center;
    background-color: rgba(0, 0, 125, 0.4);
    display: flex;
    height: 100%;
    justify-content: center;
    width: 15%;
}

@media screen and (min-width: 600px) and (min-height: 800px) {
    .list-item {
        border-radius: 10px 40px;
    }

    .list-item-route {
        border-radius: 10px 0 0 40px;
    }

    .list-item-call {
        border-radius: 0 40px 10px 0;
    }
}
```

This is still evolving.  Simple things like the color of our buttons should be consistent for good UX (in my opinion), but some of the colors we chose in our initial views don't look good when directly overlaid on our background images.  I'm still tweaking this.  :-)  Also note the slightly different syntax for `grid-template-rows`.  `auto auto 5vh` was simply shorter in this case, so I didn't use `repeat`.

The main thing to note is how we are mixing the familiar grid layout with `display: flex`.  This lets us easily map higher-level containers onto the grid, then neatly arrange the list item components (_route button_, _business detail_, _call button_) within those containers.  Nothing groundbreaking, but keep in mind you can mix and match layout styles as needed.

Here's our markup:

```HTML
<% include partials/header %>
<body id="list">
    <script defer src="/js/list.js"></script>
    <div id="list-content">
        <% results.forEach(function(biz) { %>
        <% if (biz.image_url) { %>
        <div class="list-item" style="background-image: url('<%= biz.image_url %>'); background-position: center; background-size: cover">
        <% } else { %>
        <div class="list-item" style="background-color: rgba(0, 0, 0, 0.5);">
        <% } %>
            <div class="button list-item-route" onclick='route("<%= biz.location.display_address %>");return false'>
                <i class="fas fa-map-marker-alt fa-fw fa-lg"></i>
            </div>
            <div class="list-item-content text-shadow">
                <div><%= shorten(biz.name, 20) %> (<%= parseFloat(biz.distance / 1609.344).toFixed(1) %>mi)</div>
                <div><a href="<%= biz.url %>"><%- stars(biz.rating) %></a></div>
            </div>
            <div class="button list-item-call" onclick='call("<%= biz.phone %>");return false'>
                <i class="fas fa-phone fa-fw fa-lg"></i>
            </div>
        </div>
        <% }) %>
    </div>
    <div class="button" id="js-list-home">
        <i class="fas fa-home fa-lg"></i>
    </div>
<% include partials/footer %>
```

We do a similar trick as seen in the _random view_ to feed business details to `route` and `call` functions, but use `onclick` since we'll build up a list of many results.  Check those out over in [public/js/list.js](https://github.com/deadlysyn/chowchow/blob/master/public/js/list.js).

> Don't let the pursuit of perfection make you afraid to ship anything! Ship and iterate.

By now, you've likely noticed some styling sneaking into our markup (things like the `background-color` on the `list-item` div).  This is an anti-pattern we'd like to avoid, but we'll just make a note to refactor later.  This was an easy way to get started.

![List View]({{site.baseurl}}/media/chowchow_list_view.jpg)

# Client-Side JavaScript

Phew, that was quite a tour!  We are almost ready for a break so you can digest all this a bit more.  Better yet, clone the repo and play with it yourself!  Before we part, I want to start our journey into the world of JavaScript...  aside from the bits of EJS you've already seen, we actually need some client-side code to make our app useful.

First, we need to be able to obtain the user's location so we can search nearby.  Second, we need a way to know what platform our app is running on so we can do things like plot routes in native map applications vs using the browser.  Last but not least, since we've already had them shoved in our face a few times while working through the views, let's go over the template helper functions (`shorten` and `stars`) just to see what they do.

### Geolocation

Luckily for me, geolocation in browsers has matured a lot since I last experimented.  For our needs we can simply leverage `navigator.geolocation`, wrap that in a conditional to ensure location is enabled (if not, redirect to a view which gives a helpful message), and provide a couple callbacks to define behavior on success and failure.

Peeking inside [public/js/home.js](https://github.com/deadlysyn/chowchow/blob/master/public/js/home.js), we see:

```JavaScript
if (!navigator.geolocation || !navigator.cookieEnabled) {
    location.href = '/nolocation'
} else {
    navigator.geolocation.getCurrentPosition(geoSuccess, geoError, geoOpts)
}

function geoSuccess(position) {
    latitude.value = position.coords.latitude
    longitude.value = position.coords.longitude
    loading.innerHTML = 'FIND <i class="fas fa-utensils fa-lg" aria-hidden="true"></i> FOOD'
}

function geoError(error) {
    loading.innerHTML = '<p style="font-size: 0.7rem">Unable to retrieve location:<br> "' + error.message + '"</p>'
}
```

It's not strictly required, but we also define a `geoOpts` object to configure geolocation behavior:

```JavaScript
var geoOpts = {
        enableHighAccuracy: true,
        maximumAge: 300000,
        timeout: 20000
    }
```

For more details on geolocation, [be sure to check out the MDN docs](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/Using_geolocation) which give useful examples and go over available options.  It starts to feel like 80% of our time as developers will be split across Google and MDN.  :-)

When the _home view_ is loaded, this ensures geolocation is enabled, attempts to get a high accuracy location result, displays a spinning icon while waiting, updates some form elements on the page with the latitude and longitude if successful (we'll go over feeding that into the Yelp API later), otherwise displays a useful error message.  Pretty cool, huh?

### Platform Detection

When the user clicks or taps a route button, we want to use the native map application vs opening everything in the browser.  This should give a richer user experience, and also gets us closer to feeling like a mobile app.  To do that, we need two things...

First, we include [bowser](https://github.com/lancedikson/bowser) to help us detect what platform the end user is running (this adds a new dependency, but one that meets a specific requirement and avoids re-inventing a rather complicated wheel).

Second, we use [a bit of Google-engineered knowledge](http://www.anexinet.com/blog/opening-native-mapping-app-from-your-mobile-hybrid-app) to manipulate the URL assigned to the route buttons in our app based on detected platform.  Let's put these together:

```JavaScript
function updateMapURL(address) {
    let maps = 'https://maps.google.com?q='

    if (bowser.iphone) {
        maps = 'maps:q='
    } else if (bowser.android) {
        maps = 'geo:0,0?q='
    }

    route.addEventListener("click", function() {
        location.href = maps + encodeURIComponent(address)
    })
}
```

As we saw above, we receive `address` from the rendered views (`random` and `list` do similar things), and `encodeURIComponent` ensures it's in valid URI format.

### Template Helpers

In the EJS templates above, I'm sure you noticed references to the `shorten` and `stars` functions.  We make these available in our templates by attaching them to `app.locals`, [an Express feature](https://expressjs.com/en/api.html#app.locals) allowing us to leverage the power of custom functions within our templates.

These let us shorten the length of strings used in our templates (some can get quite long causing misbehavior on small screens) and represent ratings as graphical stars.  Note the use of a default value for `shorten`'s `len` argument, that way things mostly work as expected if we forget to pass a second argument:

```JavaScript
/*
 * template helpers
 */

// truncate long strings; only add ellipsis on truncation
app.locals.shorten = function(str, len = 30) {
    if (str.length > len) {
        str = str.substring(0,len).trim() + '...'
    }
    return str
}

// convert rating number to svg stars courtesy of fontawesome.io
app.locals.stars = function(num) {
    let stars = ''
    let style = 'style="color: #FFD700"' // "gold"

    for (let i = 0; i < num-1; i++) {
        stars += '<i class="fas fa-star fa-lg" ' + style + '></i>'
    }

    if (num%1 != 0) {
        stars += '<i class="fas fa-star-half fa-lg" ' + style + '></i>'
    }

    return stars
}
```

# Final Thoughts

With that, we've come to the end of Part 2...  If you've read this far, your tenacity is commendable.  :-)  Factoring out [Bootstrap](https://getbootstrap.com) was a bit more work than anticipated, but with only a few hundred lines of custom CSS (a significant reduction) we were able to build a responsive UI that is functional on a variety of devices.  In exchange for our effort, we've got fewer dependencies to worry about and greatly simplified markup.

We saw that media queries are a powerful tool in our quest for responsive design, but just scratched the surface.  As it exists, we've only tested a small number of potential device types and window sizes.  There is still a lot of room for improvement, but at least we're headed in the right direction!

We still have a sort of _hybrid web app_ vs a true mobile application, but it's a journey with plenty of milestones ahead...  Also, while we accomplished a lot, we just built an interface -- how does it actually do real work?  Next time around we'll dive into the back-end and talk about integrating with Yelp's API.  Their API is so complete and well-documented it will be a simple case, but we'll get to touch on some common themes which will carry over when building more complex applications interacting with multiple back-end services.

In case you haven't already figured it out, the SVG background pattern we're using comes from [heropatterns.com](http://www.heropatterns.com) (which I came across while listening to the [Full Stack Radio](http://www.fullstackradio.com) podcast).  The icons we leverage for our app's buttons come from [Font Awesome 5](https://fontawesome.com/get-started).

[Continue to the next part of this series...](http://deadlysyn.com/blog//2018/02/11/idea-to-app-part-3)
