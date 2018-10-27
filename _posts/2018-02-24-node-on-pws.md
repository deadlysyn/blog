---
published: true
author: Mike Hoskins
layout: post
category: development
---
# Migrating a Node.js app to PWS

In [a prior post](http://deadlysyn.com/blog//development/2018/02/11/idea-to-app-part-3) we built a responsive web app atop [Node.js](https://nodejs.org), [Express](https://expressjs.com) and [Heroku](https://devcenter.heroku.com)...  To give us more time to focus on building the application, we decided to deploy using a _PaaS_ ([Platform as a Service](https://en.wikipedia.org/wiki/Platform_as_a_service)).  Further limiting our choices, we wanted something with a reasonable _free tier_ to support experimentation.

I was eager to give [Heroku](https://devcenter.heroku.com) a spin since many respected colleagues have raved about it in the past.  The thorough documentation, CLI, git-based workflow support and native buildpacks made getting our idea published on the Internet quick and easy. Heroku also came up with [The Twelve-Factor App](https://12factor.net), which practically became a _holy grail_ of DevOps and best practice for anyone shipping modern applications.

Recently I started working for [Pivotal](https://pivotal.io) focused on cloud native architecture and service delivery via [Cloud Foundry](https://www.cloudfoundry.org/the-foundry/pivotal-cloud-foundry) (_PCF_). Specifically, the _Platform Reliability Team_ helps customers understand, adopt and apply [Site Reliability Engineering principles](https://landing.google.com/sre/book/chapters/part2.html) within their organizations.

To better understand the PCF developer experience, I decided to see how hard it would be to migrate [my Heroku-based application](https://github.com/deadlysyn/chowchow) to [Pivotal Web Services](https://run.pivotal.io) (_PWS_). _PWS_, or _P-Dubs_ as we affectionately call it, is a fully-managed version of PCF hosted atop public cloud infrastructure very similar to Heroku.  There's a polished UI, CLI, [extensive documentation](https://docs.run.pivotal.io), and [buildpacks for many popular languages](https://docs.run.pivotal.io/buildpacks/index.html) at [reasonable prices](https://run.pivotal.io/pricing).

# Getting Started

To start experimenting, [sign up for a free PWS account](https://try.run.pivotal.io/gettingstarted).  This is quick and easy (email, password, confirm SMS, define your _organization_ name).  With that, you can [walk through their tutorial to push your first app](https://pivotal.io/platform/pcf-tutorials/getting-started-with-pivotal-cloud-foundry/introduction).

The tutorial is based on a simple app using the [Java Build Back](https://docs.run.pivotal.io/buildpacks/java/index.html) (_JBP_), but lets you install and exercise the CLI.  You don't have to worry too much about buildpacks since they are typically auto-detected, but in our case we'll be using [the appropriate version for Node.js](https://docs.run.pivotal.io/buildpacks/node/index.html).

# Setup

Not having migrated from one PaaS to another before, I wasn't entirely sure what obstacles I would encounter.  The good news is, it was fairly painless.  Following twelve-factor principles from the start meant no application refactoring was required, and configuration was easily passed around through the environment.

The first new concept I needed to grasp was a [deployment manifest](https://docs.run.pivotal.io/devguide/deploy-apps/manifest.html).  This is a [YAML](https://en.wikipedia.org/wiki/YAML) configuration, typically named _manifest.yml_, allowing you to control almost every aspect of your application's deployment.  While something extra to keep track of, this is similar to other PaaS-specific metadata like Heroku's _Procfile_.

It's best to start small, then iterate to add parameters as needed...  To get started, I just grabbed the skeleton from [the tutorial app's repo](https://github.com/cloudfoundry-samples/cf-sample-app-spring) and adjusted a couple parameters:

```
---
applications:
- name: chowchow
  memory: 512M
  instances: 1
  random-route: true
```

`random-route` enables behavior similar to Heroku's default of randomly-generating the host-part of the application's FQDN.  Without that, Cloud Foundry will try and use the application name...which may be what you want, or may lead to collisions in a shared name space like we have with the free tier's top-level application domain.

Another thing I needed to do was effectively pin my Node and NPM versions.  I had neglected that when initially deploying to Heroku, but it's a good idea for any production app lest deploying a build suddenly pull in unexpected (or expected but broken) dependencies.  To do that, I simply added a couple lines to `package.json` ([see the full version](https://github.com/deadlysyn/chowchow/blob/master/package.json)):

```
"engines": {
  "node": "8.9.4",
  "npm": "5.6.0"
}
```

Technically you don't have to lock down `npm`, if left undefined it will use the version that ships with the selected `node`.  I like to be explicit.

# Going Live

With that, I felt like I was in pretty good shape based on the tutorial...  After a `cf login -a https://api.run.pivotal.io` (using the email address and password used when signing up), I fired off `cf push` to start a deployment, but got an error message:

```
The app upload is invalid: Symlink(s) point outside of root folder
```

Luckily, [a little Google engineering quickly led to answers](https://github.com/cloudfoundry/cli/issues/1299)...  a few related suggestions there.  The one I went with was simply creating `.cfignore` at the top level of my application repo and adding the `node_modules` directory.  With that, `cf push` worked like magic...  reading my deployment manifest, auto-detecting the proper buildpack, and bringing up a public instance with a random route name:

```
Pushing from manifest to org deadlysyn-org / space development as x@y.z...
Using manifest file ./chowchow/manifest.yml
Getting app info...
Updating app with these attributes...
  name:                chowchow
  path:                ./chowchow
  disk quota:          1G
  health check type:   port
  instances:           1
  memory:              512M
  stack:               cflinuxfs2
  routes:
    chowchow-fantastic-waterbuck.cfapps.io

Updating app chowchow...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...

...

   -----> Nodejs Buildpack version 1.6.18
   -----> Installing binaries
          engines.node (package.json): 8.9.4
          engines.npm (package.json): 5.6.0
          
...

Waiting for app to start...

name:              chowchow
requested state:   started
instances:         1/1
usage:             512M x 1 instances
routes:            chowchow-fantastic-waterbuck.cfapps.io
last uploaded:     Sat 24 Feb 18:05:09 EST 2018
stack:             cflinuxfs2
buildpack:         nodejs
start command:     node app.js

     state     since                  cpu    memory      disk      details
#0   running   2018-02-24T23:05:40Z   0.0%   0 of 512M   0 of 1G  
```

We now have a working Node/Express app available at [chowchow-fantastic-waterbuck.cfapps.io](https://chowchow-fantastic-waterbuck.cfapps.io), and like Heroku it is automatically served securely since the app instances are fronted by a routing tier doing TLS offloading (using a wildcard cert for the shared `cfapps.io` domain).  Pretty neat!

You can get status of the application using the CLI:

```
$ cf apps
Getting apps in org deadlysyn-org / space development as x@y.z...
OK

name       requested state   instances   memory   disk   urls
chowchow   started           1/1         512M     1G     chowchow-fantastic-waterbuck.cfapps.io
```

The UI is also lightweight and responsive:

![pwsDashboard.jpg]({{site.baseurl}}/media/pwsDashboard.jpg)

# Cloud Foundry Specifics

If you want to go deeper on Cloud Foundry specifics, [the documentation is the place to start](https://docs.run.pivotal.io)...  A couple key concepts we glossed over above were [routes](https://docs.run.pivotal.io/devguide/deploy-apps/routes-domains.html) and [spaces](https://docs.cloudfoundry.org/concepts/roles.html).  Since they are so central to hosting an application, I wanted to briefly describe both of those here.

The term _route_ within the PCF ecosystem usually refers to the hostname portion of a FQDN ([Fully Qualified Domain Name](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)).  In our example above, the FQDN was `chowchow-fantastic-waterbuck.cfapps.io` and the route was `chowchow-fantastic-waterbuck`.  It's also possible to have _context specific routes_ which allow different micro-services hosted under the same top-level domain name to be reached via URIs.  An example of that would be _example.com/foo_ and _example.com/bar_ where _/foo_ and _/bar_ are routed to different applications.  This is highly flexible, and allows you to easily scale-out specific parts of your service regardless of how you chose to present it to the Internet.

_Spaces_ are part of Cloud Foundry's authorization scheme.  This is a hierarchy...  Every project will have one or more _organizations_ (in our example this was `deadlysyn-org`), which in turn have one or more _spaces_, each of which have one or more _users_ and _applications_ all governed by RBAC ([Role Based Access Control](https://en.wikipedia.org/wiki/Role-based_access_control)).  We deployed to the `development` space which was created for us by default, but this is again as flexible as you need it to be in complex multi-tenant environments.

You can manage all of this from the CLI, and within the web UI it's easy to see what organization and space you are working in.  Assuming you have the right permissions, you can also create new spaces (perhaps for other environments like staging and production, or for service teams).

![pwsOrgs.jpg]({{site.baseurl}}/media/pwsOrgs.jpg)

# Conclusion

I encountered one _oops_ during this journey...  While `cf push` worked, in my haste to get everything going I'd forgotten to properly set up the environment.  We could [add environment variables to our deployment manifest](https://docs.run.pivotal.io/devguide/deploy-apps/manifest.html#env-block), but that is really for non-sensitive information (think things like `NODE_ENV`).

For sensitive bits you don't want checked into source control, keep them out of the deployment manifest...  The mechanism I used instead was `cf env`.  This is similar to Heroku's `config vars`.  You can define variables at deploy time, adjust them while the service is running, and they persist across deployments (so you don't lose settings when orchestrating new instances).

In our case, we just need to ensure a `SECRET` variable exists in the environment so [express-session](https://www.npmjs.com/package/express-session) can pick up a proper session key.  Setting environment variables is easy via the CLI:

```
$ cf set-env chowchow SECRET someRandomString
Setting env variable 'SECRET' to 'someRandomString' for app chowchow in org deadlysyn-org / space development as x@y.z...
OK
TIP: Use 'cf restage chowchow' to ensure your env variable changes take effect 
```

Look at that, it even reminds us how to get our app to pick up the change...  Who says CLIs can't be friendly?  Let's ensure our new variable was properly set, then [restage](http://cli.cloudfoundry.org/en-US/cf/restage.html) the application:

```
$ cf env chowchow
Getting env variables for app chowchow in org deadlysyn-org / space development as x@y.z...
OK

System-Provided:
{
 "VCAP_APPLICATION": {
  "application_id": "f35f84b5-39b2-4937-bc39-4c04ac539e87",
  "application_name": "chowchow",
  "application_uris": [
   "chowchow-fantastic-waterbuck.cfapps.io"
  ],
  "application_version": "bc1591bc-7d40-49be-86a3-e4b9fd745ed7",
  "cf_api": "https://api.run.pivotal.io",
  "limits": {
   "disk": 1024,
   "fds": 16384,
   "mem": 512
  },
  "name": "chowchow",
  "space_id": "900772c4-cccd-4255-8e21-f599733f4b86",
  "space_name": "development",
  "uris": [
   "chowchow-fantastic-waterbuck.cfapps.io"
  ],
  "users": null,
  "version": "bc1591bc-7d40-49be-86a3-e4b9fd745ed7"
 }
}

User-Provided:
SECRET: someRandomString

No running env variables have been set

No staging env variables have been set


$ cf restage chowchow
Waiting for app to start...

name:              chowchow
requested state:   started
instances:         1/1
usage:             512M x 1 instances
routes:            chowchow-fantastic-waterbuck.cfapps.io
last uploaded:     Sat 24 Feb 18:05:09 EST 2018
stack:             cflinuxfs2
buildpack:         nodejs
start command:     node app.js

     state     since                  cpu    memory          disk          details
#0   running   2018-02-25T04:49:05Z   0.0%   14.3M of 512M   73.6M of 1G
```

With that, we have a properly configured version of our app up and running! The maturity of the platform, excellent documentation, and large community made it easy to get started and enabled us to find answers when we got stuck.

Best practices like [Twelve-Factor](https://12factor.net) ensured that everything mostly just worked after applying configuration by adjusting the runtime environment.  We did have to learn about _deployment manifests_ and make minor adjustments to `package.json`, but these were relatively minor changes that are well-documented and not unlike specifics we would have to learn when embracing and customizing any PaaS.

Overall, I'm happy to report migrating our simple application to a new PaaS was relatively straightforward... though admittedly, the simple app used here just scratched the surface of PWS capabilities. They support custom DNS domains, SSL as a service, and a variety of service brokers for backing stores and other dependencies more complex services would require.

Have you experimented with [Pivotal Web Services](https://run.pivotal.io)?