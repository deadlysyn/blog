---
published: true
author: Mike Hoskins
layout: post
---
# Introduction

In the first two parts of this series we bootstrapped our idea and got the basic UI ready for a responsive webapp.  Getting this part out took longer than I'd hoped because of being distracted ramping up at a new job...  I feel like some of the points I wanted to share have slipped into the ether, so this will be a quicker tour than originally hoped.

Just in case you forgot, here's the outline of our _Idea to App_ series:

- [Ideation](http://deadlysyn.com/blog/2018/01/08/idea-to-app-part-1)
- [UI, UX and client-side concerns](http://deadlysyn.com/blog/2018/01/20/idea-to-app-part-2)
- Backend and APIs (you are here)

I'd originally planned a _Deployment_ section as well...but decided this will be the last part in this series.  Perhaps I'll touch on deployment aspects of typical web and mobile apps in a future series.  ChowChow is a simple app deployed atop Heroku, so there is not much to share that isn't already in [their excellent docs](https://devcenter.heroku.com/categories/reference)!

To make up for that, based on how well I manage to split time amongst work (SRE is fun enough it's often hard to tear yourself away!), actual coding, learning new stuff and blogging...  a more interesting topic may be circling back over our finished app and doing some refactoring (add more error handling, make better use of newer ES features such as promises, cleanup based on [Airbnb's style guide](http://airbnb.io/javascript), etc.).

**Remember to [clone the repository](https://github.com/deadlysyn/chowchow) to follow along...**

# Using the Environment

One of the first things we need to take care of when designing modern apps is managing sensitive data.  The common practice is to read these values from the environment, which can then be injected via credential management utilities, set via CI/CD tools, etc.

Leveraging the environment isn't limited to sensitive information you don't want checked into version control...  it can also be used to control behavior (e.g. dev vs prod) or read dynamic information like the listen address.  With [Node.js](https://nodejs.org), we use `process.env` for that.

ChowChow is simple enough we don't have much to worry about, but we do need to read the IP address and port from the environment (so things work on my laptop as well as my hosting provider).  We also have a secret key used by [express-session](https://www.npmjs.com/package/express-session) (provides lightweight session management).

```
var ip = process.env.IP || '0.0.0.0'
var port = parseInt(process.env.PORT, 10) || 3000
var secret = process.env.SECRET || 'some random string'
```

The special address `0.0.0.0` just listens on all available interfaces (which might be my laptop's loopback or ethernet address at home, or a container's virtual NIC atop a platform like Heroku).  I could set this to `127.0.0.1` and have it work just as easily at home, but that would usually break when shipping the app so `0.0.0.0` is easier to manage.

If you are concerned about binding to all available interfaces on your laptop (hopefully you run a firewall), you could use `127.0.0.1` then set the `IP` environment variable on startup.  `PORT` is very similar so I won't dwell on it, just note how you can either pass in a `PORT` environment variable or let it default to `3000`.  You would set these via shell as `export NAME = value` or a mechanism like [Heroku's config vars](https://devcenter.heroku.com/articles/config-vars).

Last but not least, `secret` will default to `some random string` just to make dev easier, but for production we'll set the `SECRET` environment variable in our environment, container, build tool, or hosting provider...this way the real secret is not checked into GitHub.  Easy, right?

# Dev vs Production

I've really been having fun using [Express](https://expressjs.com), but if you are shipping a real app one of the first things you'll need to change (it's no secret, all the docs make it clear!) is [express-session's](https://www.npmjs.com/package/express-session) `MemoryStore`.  It's a great starting point for prototyping, but will leak memory in production (from what I understand, it just doesn't bother with expiry).

[There are a TON of options for backing stores](https://www.npmjs.com/package/express-session#compatible-session-stores) (you might, for example, want sessions stored in MongoDB or a SQL database), but sticking to our theme of simplicity, [memorystore](https://www.npmjs.com/package/memorystore) works very similarly to the default without the memory leaks.  Perfect!  Let's configure that for ChowChow:

```
app.use(session({
  store: new memstore({
      checkPeriod: 3600000 // 1 hour in ms
  }),
  resave: false,
  saveUninitialized: false,
  secret: secret
}))
```

`secret` is the value we read from the environment above.  Adjust the others to taste, based on the [memorystore docs](https://www.npmjs.com/package/memorystore) and [express-session docs](https://www.npmjs.com/package/express-session).

# Middleware

As mentioned earlier in the series, the stack we chose for this experiment was [Node.js](https://nodejs.org) and [Express](https://expressjs.com)...  In this ecosystem, a common theme is keeping code responsible for routes clean by factoring functions doing heavy lifting into [middleware](https://expressjs.com/en/guide/writing-middleware.html).

You can chain middleware functions together for flexibility, passing results around via session or return values.  Let's see a simple case of this in our app...  first, in `app.js`, the `/random` route responsible for showing a randomly selected restaurant near the user looks quite clean:

```
var m = require('./middleware') // import ./middleware/index.js

...

app.post('/random', m.logRequest, m.parseRequest, function(req, res, next) {
  req.session.save(function(err) {
      res.redirect('/random')
  })
})
```

We could just have our `app.post` route contain all the logic in `m.logRequest` and `m.parseRequest`, but that would both make the code harder to read, and cause lots of duplication (not very [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)!) for things like `logRequest` which all our routes currently share.

Let's dig into `parseRequest`...

# API Wrangling

The bulk of our functionality comes from the [Yelp Fusion API](https://www.yelp.com/developers/documentation).  The poorly named `parseRequest` (honest, it felt like a good name at the time for reasons the code _might_ make clear; if not, add an item to our refactor list!) is the _middleware_ responsible for massaging our app's inputs (things like latitude and longitude obtained from `geolocation` discussed in [the previous part of this series](http://deadlysyn.com/blog/2018/01/20/idea-to-app-part-2)) and getting API results.

This is still longer than I'd like, even after factoring out a few lines to a helper function, but here's a look at `parseRequest` as it stands:

```
middleware.parseRequest = function(req, res, next) {
  if (req.body.latitude && req.body.longitude) {
      // build up yelp api query string...
      let q = `?term=${SEARCHTERM}&latitude=${req.body.latitude}&longitude=${req.body.longitude}&limit=${APILIMIT}&open_now=true&sort_by=rating`

      // how much you're willing to spend
      switch (req.body.price) {
          case '$':
              q += '&price=1'
              break
          case '$$':
              q += '&price=1,2'
              break
          case '$$$':
              q += '&price=1,2,3'
              break
          case '$$$$':
              q += '&price=1,2,3,4'
      }

      if (req.body.drive == 'true') {
          // 8000 meters ~= 5 miles
          q += '&radius=8000'
      } else {
          // 500 meters ~= 5 blocks
          q += '&radius=500'
      }

      searchYelp(q, function(results) {
          if (results.businesses) {
              // grab random result
              let randChoice = Math.floor(Math.random() * results.businesses.length)
              req.session.choice = results.businesses[randChoice]
              // save remaining results for list view
              req.session.results = results.businesses.filter(biz => req.session.choice.id != biz.id)
              return next()
          } else {
              req.flash('error', 'No results found: please try again')
              res.redirect('/')
          }
      })
  } else {
      req.flash('error', 'Location error: please try again')
      res.redirect('/')
  }
}
```

This builds up the requisite query string Yelp's API needs to search for food near the user...  Worth noting, that `q` assignment is made _slightly_ more manageable by using [template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

Once we have a suitable query, we call `searchYelp` with a callback (it could take a while) to retrieve our results...  if unexpected things happen, we use the de facto [connect-flash](https://www.npmjs.com/package/connect-flash) to display messages to the user by writing to the session (if you cloned the repo, you can see an example of how that's displayed in `home.ejs` in the `error` div).

This is all pretty standard (the only _trick_ being the use of [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)), so let's wrap up by taking a look at `searchYelp`:

```
function searchYelp(queryString, callback) {
    let options = {
        headers: {'Authorization': 'Bearer ' + APIKEY},
        hostname: APIHOST,
        path: APIPREFIX + queryString,
        port: 443
    }
    https.get(options, function(res) {
        res.setEncoding("utf8")

        let body = ""
        res.on("data", data => {
            body += data
        })

        res.on("end", () => {
            callback(JSON.parse(body))
        })
    })
}
```

First, we build a `options` object to control the behavior of `https.get`.  Most of these are simple `const`s further up in the file, but `APIKEY` is read from the environment (after all, it's sensitive data we don't want checked into git!) via the familiar `process.env.API_KEY`.

The most interesting (perhaps?) part of this is the use of `https.get`.  We could have used any number of options for this request...  my first instinct was to use something familiar; [the request module](https://www.npmjs.com/package/request).  Aside from exercising it in a recent class, it's also similar to [Python's requests library](http://docs.python-requests.org/en/master).  One real down-side for our app (trying to optimize for simplicity) is the sheer number of dependencies...a bit heavy-weight given our simple use case.

Some other options are [Axios](https://www.npmjs.com/package/axios), [wrapping promises around request](https://www.npmjs.com/package/request-promise-native), or [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)...I chose `https.get` because it is part of the standard library, and I found the way it treats responses as streams interesting!  What would you use?

Since the `https.get` response is a stream, we declare `body` as a block-level variable and append data to it until we receive the `end` event.  Then we call back with our response data ready for use.

_NOTE: We didn't even scratch the surface on the many ways you could make HTTP requests with Javascript/Node...  [Check this out](https://www.twilio.com/blog/2017/08/http-requests-in-node-js.html)._

# Summary

That was admittedly a lightening tour, but now we've officially peeked beneath the covers and seen the middleware responsible for talking to Yelp and retrieving the data which makes our app useful (or at least slightly more than useless)...  we also saw how to quickly work around the default `MemoryStore`'s shortcomings, and leverage Node's `process.env` to easily control app behavior and protect sensitive information.

As I said earlier, this will likely be the last official article in this series...  Heroku simply made deployment so simple that there's not much worth writing their docs haven't already covered better than I could ever hope to improve upon!  That said, I am in the process of experimenting with a new development environment including [VSCode](https://code.visualstudio.com), [ESLint](https://www.npmjs.com/package/eslint) and [Airbnb's style guide](http://airbnb.io/javascript)...so may revisit this project to cover refactoring.

The last piece to making this a _real_ app would be something closer to a native or progressive web app...that will take a bit longer for me, since I am still learning [React-Native](https://facebook.github.io/react-native) (not the only way to go, just one I am currently learning).  With any luck, I'll come up with a more interesting premise to explore together once I'm further along and ready to incorporate additional technologies.  :-)

Thanks for reading!
