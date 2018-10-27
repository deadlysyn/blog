---
published: true
author: Mike Hoskins
layout: post
category: development
---
# The Notorious FSC

I've been fortunate enough to have time to start learning
[React](https://reactjs.org).
It is an amazing piece of technology for any builder to have in their toolbox.
After working through
[the tutorial](https://reactjs.org/tutorial/tutorial.html)
and experimenting a bit on my own, I picked up a copy of Robin Wieruch's
_[The Road to Learn React](https://www.robinwieruch.de/the-road-to-learn-react)_.
If you are interested in React, you've likely read it too -- if not, do
yourself a favor and check it out! It is to React what
_[You Don't Know JS](https://github.com/getify/You-Dont-Know-JS)_
is to Javascript.

I have a bad habit of getting carried away and publishing overly-dense (read:
too long) blog posts. As part of my React journey, one goal is to blog about
key things I learn...improving my posts as I go.  Each one should have enough
learning to be fun, but remain shorter than a novella.  We'll see.  :-)

Today I want to talk about the ominously named "FSC" or Functional Stateless
Components in React. Not from the position of an expert, but as someone just
learning about them myself. What are they? When should you use them? We'll
answer those questions, and refactor a simple component along the way...

# What, When, Why?

Let's unpack "FSC" -- Functional, Stateless, Component. It turns out there's a
lot in a name.  Unlike ES6 Class Components which are built atop
[ES6 Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes),
FSCs are composed using...wait for it...functions! Hmm, OK. We use functions
all the time, that's not so scary. So we have two ways of representing
React components... How do we pick one?  Well, classes are from ES6 so they
must be better right?  Read on!

Especially with modern tooling like
[create-react-app](https://github.com/facebook/create-react-app) which
handles transpiling for you, I think it is safe to say there is nothing
you can do with functional components that you can't do with class components.
Often you will see simple, stateless tutorials that are entirely class based.
One advantage to this is consistency -- components have a common look and feel,
so you can argue there is less cognitive load when browsing code. I suppose it
could also be easier to template or generate boilerplate in some cases.

The reverse is not true...  you can do things with class components
that are not supported with FSCs.  The clue is in the name: stateless. FSCs
only have access to [props](https://reactjs.org/docs/components-and-props.html),
and do not maintain [local state](https://reactjs.org/docs/state-and-lifecycle.html)
(`this.state`). This also means you loose access to lifecycle methods (the FSC
itself, or its returned JSX, is in effect the render method).

Just as one could argue using a consistent component approach makes code
easier to read, the flip side is FSCs require fewer lines of code (less
potential for bugs) and present less "noise" when skimming code. You will loose
"consistency" if your app requires state, because you will still need one or
more class components.

Thinking too much about performance is certainly premature optimization at my
point in the game, but large projects may see gains by refactoring class
components as FSCs -- if there's no need for state, you avoid the overhead
of managing its lifecycle, and should see smaller bundle sizes as well.

Note that even in a FSC, you can still do work between receiving `props` and
returning the JSX. You can add custom functions or other code inside
your FSCs, they just won't be linked to specific phases in the
[component lifecycle](https://reactjs.org/docs/state-and-lifecycle.html)
as with `constructor()`, `componentDidMount()`, `componentWillUnmount()`, etc.

# Getting Real

A snippet is worth a thousand blogs, so let's refactor a simple component...
[Thinking in React](https://reactjs.org/docs/thinking-in-react.html) is a
fun exercise which walks you through turning a mock into a real live component.
In [the example solution](https://codepen.io/gaearon/pen/BwWzwm),
everything is consistently implemented as ES6 classes. A wise choice
for teaching (maybe even your personal preference), but lets refactor one
component as a FSC to solidify the theory above.

```javascript
class ProductCategoryRow extends React.Component {
  render() {
    const category = this.props.category;
    return (
      <tr>
        <th colSpan="2">
          {category}
        </th>
      </tr>
    );
  }
}
```

I am a beginner, and this is a sample component from a tutorial, written by
folks way smarter than me...  As you might expect, it's already quite easy
to read.  Keep an open mind as we refactor, and try to imagine any small
gains we observe magnified across a large project.

Before we begin mangling code, **is this a good candidate for refactoring?** Based
on what we know so far, we see that `ProductCategoryRow` does not reference
`this.state` or any lifecycle methods.  We can safely turn it into a function:

```javascript
function ProductCategoryRow(props) {
  const category = props.category;
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}
```

It works the same, now we just receive `props` as a function parameter.
Aside from that minor change, our old render method just became our FSC! This
is already fewer lines of code to reason about, but ES6 syntactic sugar
helps us remove even more visual clutter. Combining
_[arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)_,
_[destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)_ and
_[concise bodies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Function_body)_
yields a simplified function which draws the eye to inputs and outputs:

```javascript
const ProductCategoryRow = ({ category }) => (
  <tr>
    <th colSpan="2">
      {category}
    </th>
  </tr>
);
```

The props are being destructured in the function signature, so if you are
passing more just add parameters. Default values are OK too.
Technically you can go for concise gold and leave off the return's
parens, but it drives some syntax highlighting crazy. :-)

```javascript
const ProductCategoryRow = ({ category = 'javascript', children }) =>
  <tr>
    <th colSpan="2">
      {category}
    </th>
    {children}
  </tr>
```

# Conclusion

If you're a React novice like me, this hopefully helps clarify what FSCs are,
when they are appropriate and how to use them. We took a very simple component
from _twelve_ lines of code to _seven_ (_>40% reduction_), and that's given a
line for adding functionality (passing in `children`). When envisioning
how the tests wrapped around production code could be simplified to match,
it's a real eye opener! The ES6 syntax really improves readability and
maintainability, and should be available to most thanks to tools like
[create-react-app](https://facebook.github.io/create-react-app)
and
[babel](https://babeljs.io).

One size definitely does not fit all...in the real world you will likely
have apps combining ES6 class and functional stateless components. For
beginners or modest sized projects where it doesn't matter as much, FSCs may be
a premature optimization. You need to be aware of the trade offs, and pick
the right tool for the job. For some, a more concise codebase with
fewer lines to hide bugs may be enough to justify a refactor.

