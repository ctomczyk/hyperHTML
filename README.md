# hyperHTML [![Build Status](https://travis-ci.org/WebReflection/hyperHTML.svg?branch=master)](https://travis-ci.org/WebReflection/hyperHTML)

A Fast & Light Virtual DOM Alternative
- - -

The easiest way to describe `hyperHTML` is through [an example](https://webreflection.github.io/hyperHTML/test/tick.html).
```js
// this is React's first tick example
// https://facebook.github.io/react/docs/state-and-lifecycle.html
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}
setInterval(tick, 1000);

// this is hyperHTML
function tick(render) {
  render`
    <div>
      <h1>Hello, world!</h1>
      <h2>It is ${new Date().toLocaleTimeString()}.</h2>
    </div>
  `;
}
setInterval(tick, 1000,
  hyperHTML.bind(document.getElementById('root'))
);
```

## Features

  * Zero dependencies and fits in less than a kilobyte (minzipped)
  * Uses directly native DOM instead of inventing new syntax/APIs, DOM diffing, or virtual DOM
  * Designed for [template literals](http://www.ecma-international.org/ecma-262/6.0/#sec-template-literals), a templating feature built in to JS
  * Compatible with vanilla DOM elements and vanilla JS data structures `*`
  * Compatible 1:1 with Babel transpiled output, hence compatible with every browser you can think of

`*` <sup><sub> actually, this is just a 100% vanilla JS utility, that's why is most likely the fastest and also the smallest. I also fell like I'm writing Assembly these days ... anyway ...</sub></sup>


### ... wait, WAT?
[ES6 Template literals](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals) come with a special feature that is not commonly used: prefixed transformers.

Using such feature to map a template string to a generic DOM node, makes it possible to automatically target and update only the differences between two template invokes and with **no `innerHTML` involved**.

Following [an example](https://webreflection.github.io/hyperHTML/test/article.html):
```js
function update(render, state) {
  return render `
  <article data-magic="${state.magic}">
    <h3>${state.title}</h3>
    List of ${state.paragraphs.length} paragraphs:
    <ul>${
      state.paragraphs
        .map(p => `<li>${p.title}</li>`)
    }</ul>
  </article>
  `;
}

update(
  hyperHTML.bind(articleElement),
  {
    title: 'True story',
    magic: true,
    paragraphs: [
      {title: 'touching'},
      {title: 'incredible'},
      {title: 'doge'}
    ]
  }
);
```

Since most of the time templates are 70% static text and 30% or less dynamic, `hyperHTML` passes through the resulting string only once, finds all attributes and content that is dynamic, and maps it 1:1 to the node to make updates as cheap as possible for both node attributes and node content.

## Usage
You have a `hyperHTML` function that is suitable for parsing template literals but it needs a DOM node context to operate.

If you want to render many times the same template for a specific node, bind it once and boost up performance for free.
No new nodes, or innerHTML, will be ever used in such case: safe listeners, faster DOM.

### FAQs

  * _how can I differentiate between textContent only and HTML or DOM nodes?_
    If there's any space or char around the value, that'd be a textContent.
    Otherwise it can be strings, used as html, or DOM nodes.
    As summary: ```render`<p>This is: ${'text'}</p>`;``` for text, and ```render`<p>${'html' || node || array}</p>`;``` for other cases.
    An array wil result into html, if its content has strings, or a document fragment, if it contains nodes.
    I've thought a pinch of extra handy magic would've been nice there 😉.

  * _can I use different renders for a single node?_
    Sure thing. However, the best performance gain is reached with nodes that always use the same template string.
    If you have a very unpredictable conditional template, you might want to create two different nodes and apply `hyperHTML` with the same template for both of them, swapping them when necessary.
    In every other case, the new template will create new content and map it once per change.

  * _is this project just the same as [yo-yo](https://github.com/maxogden/yo-yo) or [bel](https://github.com/shama/bel) ?_
    First of all, I didn't even know those projects were existing when I've written `hyperHTML`, and while the goal is quiet similar, the implementation is very different.
    For instance, `hyperHTML` performance seems to be superior than [yo-yo-perf](https://github.com/shama/yo-yo-perf).
    You can directly test [hyperHTML DBMonster](https://webreflection.github.io/hyperHTML/test/dbmonster.html) benchmark and see it goes _N_ times faster than `yo-yo` version on both Desktop and Mobile browsers 🎉.


### Caveats

Following a list of `hyperHTML` caveats <sup><sub>(so far just one)</sub></sup>.

#### Quotes are mandatory for dynamic attributes
To achieve best performance at setup time, a special `<!-- comment -->` is used the first time as template values.

This makes it possible to quickly walk through the DOM tree and setup behaviors, but it's also the value looked for within attributes.

Unfortunately, if you have html such `<div attr=<!-- comment --> class="any"></div>` the result is broken, while using single or double quotes will grant a successful operation. This is the biggest, and so far only, real caveat.

As summary, always write `<p attr="${'OK'}"></p>` instead of `<p attr=${'OK'}></p>`, or the layout will be broken, even if the attribute is a number or a boolean.

In this way you'll also ensure whatever value you'll pass later on won't ever break the layout. It's a bit annoying, yet a win.


## Compatibility
If your string literals are transpiled, this project should be compatible with every single browser, old or new.

If you don't transpile string literals, check the [test page](https://webreflection.github.io/hyperHTML/test/) and wait 'till it's green.

- - -
(C) 2017 Andrea Giammarchi - MIT Style License
