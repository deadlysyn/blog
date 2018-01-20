---
published: false
author: Mike Hoskins
layout: post
---
# Introduction

Welcome back.  We were pretty meta in [part one of this series](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) -- running through a typical thought process and common techniques for turning ideas into something tangible...  including how to come up with ideas in the first place, thinking about workflow once you have one, and mocking a UI that lets us start visualizing that workflow and eventually a final product (or at least a [Minimum Viable Product](https://en.wikipedia.org/wiki/Minimum_viable_product)).

This is where things start to get fun, as we can finally delve deeper into  _code_ (just in case CSS can't actually be called code, we'll start a bit of Javascript as well).  As we said in Part 1, there's a lot of ground to cover.  Just in case you forgot, here's the planned breakdown:

- [Ideation](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) (Part 1)
- UI, UX and client-side concerns (you are here)
- Backend and APIs
- Deployment
- Final Thoughts (we'll do some refactoring, no app is complete without it!)

We'll also share enough code to get our points across (hopefully), but avoid multi-page long snippets since that might look cool but gets overwhelming (for me at least).  All the code is in GitHub, so you can always [browse the full source](https://github.com/deadlysyn/chowchow) there if needed (better yet, clone the repo and submit PRs with bug fixes or new features).

So buckle up, and come along for the ride...

# Slight Detour: Dev Environment

In [Part 1](http://deadlysyn.com/blog//2018/01/08/idea-to-app-part-1) I promised to share details on the _dev environment_ we'll be using for this project.  The first thing you need to check out is the GitHub repo.  [Go browse that now if you'd like.](https://github.com/deadlysyn/chowchow).  I'll wait.  When you're done, go ahead and clone it so you can follow along:

```
$ mkdir -p ~/src; cd ~/src
$ git clone https://github.com/deadlysyn/chowchow.git
```

Back?  Cool.  This has been documented by others far better than I ever could, so we'll keep it brief...  If you have questions, there are lots of great references available for [Docker in general](https://docs.docker.com) (one of the most amazingly well-documented projects around!), as well as specific guides for Node.js development using Docker...like these, just to share a few:

- [The Official Guide](https://nodejs.org/en/docs/guides/nodejs-docker-webapp)
- [A Great Medium Article on Docker+Node](https://medium.com/@sunnykay/docker-development-workflow-node-express-mongo-4bb3b1f7eb1e) (Though we're not using MongoDB here)
- [Another Awesome Guide for Larger Projects](http://paislee.io/the-ultimate-nodejs-development-setup-with-docker)

Compared to those, ours is quite simple...but we have a couple things key things in common.  [Here's our Dockerfile](https://github.com/deadlysyn/chowchow/blob/master/Dockerfile):

```
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

- We just pull the latest Node.js image as a starting point (you might want to lock this down to a specific version for a production app)
- We create `/app` within the container as our working directory (more on this later)
- We copy `package.json` from the current directory (e.g. our cloned github repo) and install all project dependencies with `npm install`
- We carefully put dependency bits toward the top of the `Dockerfile` to [maximize cache hits](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#sort-multi-line-arguments)
- We `EXPOSE` port 3000 within the container (more on this later)

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

This simply lets us `make build` to build an image from our `Dockerfile`, and `make run` to bring up our app in the container on our machine.  Note how we use `-p` to map port 3000 on our machine to port 3000 in the container.  This means you can browse to `localhost:3000` to access the application during development.  `IP=0.0.0.0` just gets the application listening on all available network interfaces (if you simply bind to `127.0.0.1` aka _loopback_, you won't be able to access the application from outside the container).  `API_KEY` deserves more explanation, but I want to save that for _Part 3: Backend and APIs_ so we don't muddy the watters too much here.  The big takeaway for now is that you can pass environment variables (defined in your shell when running Docker) into the container to control application behavior.

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

When you first `npm init` a new project, there will be no `start` command.  We've added `"start": "nodemon app.js"` which causes the `npm start` command in our Dockerfile to run our application under supervision of `nodemon` (If you haven't heard of `nodemon`, see [here](https://nodemon.io) and [here](https://www.npmjs.com/package/nodemon)).  This causes the application to restart anytime we change Javascript source within the project, so we can easily preview changes.

For the most part you can ignore the rest of the fields for now, but if you haven't worked with `package.json` before and want a deep-dive, [check out the official documentation](https://docs.npmjs.com/files/package.json).

# Project Hierarchy

It always helps to know how a project is laid out so you can follow along, and we'll delve deeper into this in the next part of the series where we focus on [Node.js](https://nodejs.org) and [Express](https://expressjs.com) particulars.

For now, particularly since this is a small project, you can just [browse the repository](https://github.com/deadlysyn/chowchow) to get a feel for things.  The main thing is awareness of what a view is (loosely, a template rendered when users access specified URIs or routes served by our application), where those live (in the aptly named `views` directory), and where assets like CSS reside (inside the `public` directory).  Here's a simplified tree view of the project, with only the areas we'll work with in this section:

```
.
├── Dockerfile
├── Makefile
├── app.js
├── package.json
├── public
│   ├── css
│   │   ├── main.css
│   ├── fonts
│   │   ├── exo-v6-latin-700.woff
│   │   ├── exo-v6-latin-700.woff2
│   │   ├── exo-v6-latin-regular.woff
│   │   └── exo-v6-latin-regular.woff2
│   ├── js
│   │   ├── bowser-191.min.js
│   │   ├── fontawesome-all-5.js
│   │   ├── home.js
│   │   ├── list.js
│   │   └── random.js
│   └── media
│       ├── chowchow.png
│       ├── favicon.png
│       └── feeding_america.png
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

In Part 1 we discussed how I started with [Bootstrap](https://getbootstrap.com) when mocking the UI because it was a famliar starting place.  [The bootcamp I went through](https://www.udemy.com/the-web-developer-bootcamp) leveraged it heavily, it's one of the most popular CSS/JS frameworks, and it's well-documented and easy to use...  but for such a simple app, it was a huge dependency to pull in.

I also found myself fighitng the framework in some areas.  The main requirement we have is a responsive design.  Responsive design is largely about ensuring your application works well on a variety of screen sizes (desktop, mobile, etc.).

> Responsive web apps use technologies like media queries and viewport to make sure that their UIs will fit any form factor: desktop, mobile, tablet, or whatever comes next. -[MDN](https://developer.mozilla.org/en-US/Apps/Progressive)

Since we are building a web applicaiton on a journey toward native mobile (where our app would be most useful), responsiveness is key.  While Bootstrap provides responsiveness, it's achievable with native CSS as well.  It felt a bit silly to pull in Bootstrap for what boiled down to a single row and column.  I found myself with something like this in each view:

```HTML
<div class="container">
	<div class="row">
        <div class="col">
        </div>
    </div>
</div>
```

I decided to rip out the framework and use this as an opportunity to jump on the [CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout) bandwagon.  Nothing against Bootstrap, but there is a time and a place for frameworks.  I also simplified the interface quite a bit, removing another dependency ([css-toggle-switch](https://ghinda.net/css-toggle-switch/)) which is amazing in itself but felt too heavyweight for our needs.

> Dependencies are a way of life (who wants to re-invent the wheel), but it's important to be mindful of what you pull into your project.  Make judgement calls on whether the added complexity makes sense in your context.

Of course when I decided to rip out the framework, it turned into a bit more work than originally thought...  mostly because I had started using things like special formatting and classes to define buttons and other UI elements.  What started as _I'll just rip out a few divs_ turned into quite a journey, but the end result was a responsive UI, fewer dependencies, FAR less un-used CSS users were forced to download, and a lot of learning.  :-)

***give line number reduction on CSS***

# Getting Stylish

As mentioned, we're going to use CSS Grid to layout our simple app.  Be warned, I'm learning with you and not a CSS master by any means.  I'm sure everything I show you can be done more efficiently -- if you spot something, unicast me or share in comments.  I've refactored things to reduce duplication in some places, but know it could be cleaner.  We'll also be using another CSS layout mechanism called [Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox).  Before going further, take time to review the following resources so it's easier to follow along:

- [Per Harald Borgen's](https://medium.com/@perborgen) [AMAZING CSS Grid Tutorial](https://scrimba.com/g/gR8PTE)
- [A Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox)

You also want to get familiar with your browser's developer tools.  In Firefox, you can simply hit `F12` to toggle the developer tools.  The _DOM and Style Inspector_ (or simply the _Inspector_ tab along the top menu bar of the development tools window) and _Javascript Console_ will be invaluable friends during your web development journey, particularly when doing UI layouts.

**insert inspector picture inspector.jpg **

The Inspector has a lot of useful features, the screenshot highlights a few I use regularly:

- Responsive design mode lets you mimic a variety of screen sizes and bandwidths
- You can browse elements or select the small pointer icon to focus on specific elements you click
- For grid layouts, you can click the small grid icon beside `display: grid` to show an outline of the grid
- You can shift-click color swatches to see alternate formats (Have a hex code and want the rgb?  Easy without leaving your browser!)

Before we dive into the restyle...  let's look at the original UI we mocked in Part 1:

**insert picture of original UI**

I felt pretty good about my hiedeous design to start, afterall it was my baby...  but looking closer it had a lot of shortcomings (other than the 90's-flashback cutout look I felt so clever devising).

> Fight the urge to get attached to your design.  In the spirit of Agile/Lean, be courageous -- experiment and change often!

Look and feel aside, it simply wasn't responsive...and didn't work at all on smaller screens.  Even if it had looked amazing, it didn't meet our primary requirement and target use case (finding decent food quickly while traveling with a mobile device).  Loading the app in the Inspector's Responsive Design Mode with a resolution of 320 x 480 was the final nail in the coffin.

**insert picture of broken UI on small screen**

Let's see how we can improve the mobile experience...

# Defining our Views

In Part 1 we defined a workflow consisting of three views...  the home view (or index page) which allowed users to modify purposefully-minimal (e.g. not cumbersome on mobile) inputs, a random view to display information about a randomly selected restauraunt, and finally a list view which allows users to browse additional choices if the random selection is not up to par.  Here's an artist's rendition (hopefully you've caught on to the inheirent snark):

**insert picture of sketched views ui_sketch.jpg**

Let's walk through the HTML and CSS to build each of these in a responsive manner...  Keep in mind that while I am trying to separate this series into sections for convenience, back-end bits inevitable bleed into front-end.  You will see a bit of [EJS template](http://www.ejs.co) detail and even helper functions as we work through the markup...  I'll mostly gloss over those here since they are not our current focus, but we'll go deeper on all that in the next part of this series.

## Common Elements

Before we get into specific views, I want to share some common elements all of the views will leverage.  There's more we could do, but the idea is to factor out common elements as classes to reduce duplication in our CSS.  I want to share this now since it will avoid confusion later when we reference some of these in our markup.  We'll go into even more of the CSS in a later section, but here are some we need to understand now:

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

Nothing too special here, we simply define a bit of custom CSS to get back the concept of a _button_ we lost when we ditched bootstrap and define the `footer` which will be a common element on every page.  Note the use of the `transition` property to make the hover effect feel a bit better (to me at least).  Also note the use of a relative `font-size` based on display size (using the `vh` unit which we'll discuss later).  That's key to getting the responsiveness we need.  `grid-area` won't make sense just yet, read on!

## Home

The home view gives our users a starting point.  In the next section we'll talk about details of defining and rendering the view in the Express framework, here we want to focus on the style.  The first thing we want to do is use CSS Grid to define a layout we can map our UI elements onto.  Let's do that:

```CSS
#home {
  display: grid;
  grid-template-columns: auto;
  grid-template-rows: repeat(4, auto) 5vh;
  grid-template-areas: "find" "price" "drive" "donate" "footer";
}
```

Be sure to review the [CSS Grid Tutorial](https://scrimba.com/g/gR8PTE) shared above so this makes sense...  I've simply defined an `id` we can use in our HTML which defines a grid consisteting of five rows and a single column.  I then name each of the rows for easy reference later.  You might be more familiar with another format for `grid-template-areas`, something like:

```CSS
grid-template-areas:
    "find"
    "price"
    "drive"
    "donate"
    "footer";
```

While useful for visualizing multi-column layouts, it seems a bit unweidly for simple pages like this where we only have a single column -- so I simply merged everything onto one line.  Either way works just fine.  Which do you prefer?

Why do this in an `id`?  Because we will have a specific grid layout per view, and only need to use each layout once (a requirement for CSS `id`s vs `class`es like Bootstrap's `row` and `column` which can be re-used).  OK, great...so how do we use this?  While ripping out Bootstrap, I got over-zealous (I might regret it later) and decided to eliminate as many extraneous _wrapper_ divs as possible.  I didn't want the equivalent of `<div class="container"></div>`, so chose to map the grid definitions to my `<body>` elements.  Here's what we see in `views/home.ejs`:

```HTML
<% include partials/header %>
<body id="home">
  ...
```

I like the way this effectively uses the body as the container, but it could be less flexible based on your design.  Just keep in mind that this is just one way to define our grid, and [not even one I thought up myself](https://medium.com/flexbox-and-grids/how-to-efficiently-master-the-css-grid-in-a-jiffy-585d0c213577?source=userActivityShare-9cdc41953872-1516481482)...I just found it novel.  Do you, or is it setting off alarm bells?  :-)

Now that we've got a grid, and it's mapped onto our page...  let's add some content.  We do that by assigning elements to grid areas.  Our simplified home view is just four buttons (find food, adjust price range, toggle walk/drive, or make a donation) which should fill the screen and be responsive:

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

We use `grid-area` to map elements onto our grid, and relative sizes for `height` and `width` to get the desired responsiveness (`vh` or _[view height](https://web-design-weekly.com/2014/11/18/viewport-units-vw-vh-vmin-vmax)_ for `height`, and percentages for `width`) -- again, I am not a CSS guru.  We are on a learning journey together.  This mostly gets the desired behavior on the platforms I can easily test (Chrome, Firefox and Safari on MacOS, as well as mobile Safari on an iPhone 7) but is almost certainly not the best way to do it (finding all the neato ways designs break on varoius devices is part of the fun of being a web developer).  Hey it works, and perfect is the enemy of the good -- let's move on and refactor this later.  :-)

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

You can ignore the form for now (we'll get to that later).  Comparing everything from there down, we can see how removing Bootstrap has really simplified our layout.  Our markup now neatly corresponds to design elements.  No more extraneous `container`s, `row`s or `column`s let alone class names like `col-xs-12` which are not at all obvious unless you've spent time with the Bootstrap documentation!

You can also ignore the EJS details (stuff wrapped in `<% %>`), we'll get there in the next part of this series.  The small amount of EJS here is pretty clear though, we simply include a header and a footer to avoid having to type common bits like `<html></html>` in every view.  One thing we need to understand is a bit of magic in the included `views/partials/header.ejs`:

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

This all mostly works reasonable well on a small screen, but looks a bit odd in a larger desktop browser window.  To fix that, we'll need to leverage [CSS media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) a few places in our design.  You could get as fancy as you like here, I simply add some spacing and give our buttons a fancier shape on bigger screens:

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

***insert side-by-side view of home page on mobile and desktop***

We've covered a lot of ground in this section along, but luckily for us many of these concepts (`viewport`, `view-height`, `@media` queries) will be re-used in each view.  This will save us a lot of time as we work through the remaining views!

## Random

In `views/random.ejs` we'll use CSS grid for our layout again...but define a completely different grid suited to our needs.  The main feature will be a large content area, followed by a few buttons (route, call, home, list).  Going further, we'll re-define the grid based on screen size for an optimal layout.  Here's the random view's CSS:

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

(Psst...I see duplication sneaking into our code.  Make a note on our refactor list!)

We'll gloss over our old friends media queries, view height and percentages...  One thing worth noting is the `js-*` naming convention for some of the elements.  That's a signal that those elements are tied to Javascript (which we'll get to later).  In a project of this size it's not a big deal, but could be very useful as our codebase grows.  It's [another fine idea I didn't have to come up with myself](https://medium.freecodecamp.org/css-naming-conventions-that-will-save-you-hours-of-debugging-35cea737d849)!  Here's the markup:

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

It's starting to get really hard to ignore the EJS bits, huh?  Don't worry, we'll talk more about those and the Javascript later on.  One thing you'll see us working around is the fact that `biz.image_url` may or may not exist (some times a business doesn't have an image defined), so we avoid errors by wrapping associated output in a conditional.  Overall, this is still a fairly clean view and achieves the desired responsiveness.

***insert picture of random view on mobile and desktop***

## List

Last but not least, we want to let the user browse additional choices when the randomly-selected restauraunt is not satisfactory.  Enter the list view.  What happens behind the scenes is our app takes the _top five_ restauraunts near the user (perhaps less, if there are not actually five, adequately-ranked, open choices nearby!), displays a random choice, and then saves the remaining choices for later use if the user wishes to browse through them.

By now CSS grid is old-hat, here's our styling for the list view:

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

This is still evolving.  Simple things like the color of our buttons should be consistent (in my opion) for good UX, but some of the colors we chose in our initial views don't look good when directly overlaid on our background images.  I'm still tweaking this.  :-)

The main thing to note is how we are mixing the familiar grid layout with `display: flex` items.  This lets us easily achive the desired results, mapping higher-level containers onto the grid, then neatly arranging the list item components (route button, business detail, call button).  Again, nothing groundbreaking here -- but keep in mind you can mix and match layout styles as needed to build the layouts you are after!

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

By now, you've likely noticed some styling sneaking into our markup (things like the `background-color` on the `list-item` div).  This is an anti-pattern we'd like to avoid, but we'll just make a note to refactor later.  For now, this was an easy way to get started.

> Don't let the pursuit of perfection make you afraid to ship anything!

# Client-Side Javascript

Phew, that was quite a tour!  Luckily, we are almost ready for a break so you can digest all this a bit more.  Better yet, clone the repo and play with it on yourself!  Before we part, I want to start our journey into the world of JavaScript...  aside from the bits of EJS you've already seen, we actually need some client-side code to make our app useful.

First, we need to be able to obtain the user's location so we can search nearby.  Second, we need a way to know what platform our app is running on so we can do things like plot routes to business in native map applications vs just using the browser (this gets us slightly closer to our goal of being a mobile vs just a web applicaiton).  Last but not least, since we've already had them shoved in our face a few times while working through the views, let's go over the template helper functions (`shorten` and `stars`) just to see what they do.



# Conclusion

few hundred lines of CSS
simplifed markup
fewer dependencies
mostly functional web app (not mobile yet, but it's a journey)
now we understand how to make it look how we want, but how does it really wokr?
next time backend/apis and then we'll be ready to deploy
