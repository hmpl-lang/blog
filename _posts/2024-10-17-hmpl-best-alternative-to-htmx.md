---
layout: post
title: "HMPL - best alternative to HTMX"
date: 2024-10-17 2:10 PM
categories: blog
---

Hello everyone! In this article we will consider such a javascript module as HMPL and how it can replace HTMX in a project. Also consider their differences, advantages and disadvantages.

When further comparing the two modules, it is worth considering that one is a template language, while the other is a set of tools for working with HTML, implemented through attributes and more.

Let's start with the general concept for the two modules.

## The concept of reducing javascript code by moving user interface components to the server

The HMPL module is similar in concept to HTMX. We can also take HTML from the server via API, thus replacing modern frameworks and libraries for creating UI. Let's take a small example illustrating the work of HMPL and HTMX, as well as a framework such as Vue.js:

### Vue.js example:

```javascript
createApp({
  setup() {
    const count = ref(0);
    return {
      count,
    };
  },
  template: `<div>
        <button @click="count++">Click!</button>
        <div>Clicks: {{ count }}</div>
    </div>`,
}).mount("#app");
```
_Size: 226 bytes (4KB on disk)_

### HMPL example:

```javascript
document.querySelector("#app").append(
  hmpl.compile(
    `<div>
        <button>Click!</button>
        <div>Clicks: {{ "src": "/api/clicks", "after": "click:button" }}</div>
    </div>`
  )().response
);
```
_Size: 206 bytes (4KB on disk)_

### HTMX example:

```html
<div>
  <button hx-post="/api/increment" hx-target="#counter" hx-swap="outerHTML">Click!</button>
  <div id="counter">Clicks: 0</div>
</div>
```
_140 bytes (4 KB on disk)_

Using a simple clicker as an example, we can see (with some caveats regarding server-side and client-side data, as well as html and js markup, but that's not the main idea) that we get the same interface, although the file sizes on the client will be completely different. This is precisely the main advantage of the approach to creating a ready-made, or template UI component on the server side, so that the site user can load it faster while preserving the result.

Now, let's remember how large applications today, well, or at least earlier (when server-side rendering was not so popular), could be obtained using frameworks and libraries. The same SPA (Single Page Application) generates all content via javascript, when in html we have literally one line, but the joke is that with 10 kilobyte html we get a javascript file of several tens of megabytes. Such a site, the first time users visit it, can take a long time to load.

For example, if a potential client wants to quickly order flowers, he will not wait 10-15 seconds for the delivery store website to load, he will go to another website where the website will load faster.

There are many more practical examples of how websites work that can influence the sales funnel. But the point is that the main thing is the speed and convenience of the interface, and here there are already differences in approaches. But it is better to do this in a separate article. Here we also consider a comparison of HMPL and HTMX.

## Why use HMPL and what are its advantages over HTMX?

In this section I will try to tell you about several main reasons why you may choose HMPL instead of HTMX in some cases:

1.With a similar idea of ​​reducing code, the two modules differ in concepts. In the case of HTMX, on the one hand, we get a convenient tool for working with an existing DOM, but on the other, all this happens through HTML and is updated literally in real time. With great difficulty, through non-standard solutions, we can work more or less through javascript, and in fact, working with javascript is almost completely absent. In the case of HMPL, on the one hand, we can easily work with javascript; generate a custom RequestInit, create thousands of separate DOM nodes with the same server UI support as on HTMX, but on the other - all the work is done with code, which is not always convenient when you want to create projects faster. Let's take an example of code:

### HMPL example:

```javascript
import { compile ) from "hmpl-js";

const templateFn = compile(
  `<div>
  <form onsubmit="function prevent(e){e.preventDefault();};return prevent(event);" id="form">
    <div class="form-example">
      <label for="name">Enter your email: </label>
      <input type="text" name="email" id="email" required />
    </div>
    <div class="form-example">
      <input type="submit" value="Register!" />
    </div>
  </form>
  <p id="notification">{
    {
      "src":"/api/register",
      "after":"submit:#form",
    }
  }</p>
</div>`
);
const initFn = (ctx) => {
  const event = ctx.request.event;
  
  return {
    body: new FormData(event.target, event.submitter),
    credentials: "same-origin"
  };
};
const obj = templateFn(initFn);
wrapper.appendChild(obj.response);
```

### HTMX Example:

```html
<script src="/path/to/htmx.min.js"></script>
<div>
  <form hx-post="/api/register" hx-target="#notification" hx-swap="outerHTML">
    <div class="form-row">
      <label for="name">Enter your email: </label>
      <input type="text" name="email" id="email" required />
    </div>
    <div class="form-row">
      <input type="submit" value="Register!" />
    </div>
  </form>
  <p id="notification"></p>
</div>
```

This example clearly shows that HTMX is more about maximizing the speed and shortening of code, while HMPL is something combined between HTMX and a modern framework or library for creating UI. We can say that we get a somewhat similar result, but taking into account that we can customize the request to the server. This is very important, because customization of the request in conjunction with fetch and work in javascript will allow you to work with a microfrontend, or in conjunction with another framework, or even with tests.

2.The HMPL syntax is also an advantage in its own way, because the request objects are not tied to any tag. When rendering, they are replaced with comments that do not clutter the DOM with unnecessary tags. Example syntax:

### HMPL syntax:

```hmpl
<div>some text {{ "src": "/api/getSomeText" }} some text</div>
```

### HTMX syntax:

```html
<div>some text <span hx-put="/api/getSomeText" hx-swap="outerHTML"></span> some text</div>
```

In some cases, it is not possible to assign an attribute to achieve the minimum file size with just short tags like p or s. Sometimes, you will have to use the template tag in the same table, and it, in turn, takes up a lot of characters in the file. In the hmpl syntax, there are always single curly brackets and then an object.

3.HMPL is built entirely on fetch requests, which were introduced as a standard in 2015. HTMX, for IE13 support, uses XMLHTTPRequest by default, which was introduced in 2000. The fetch function allows you to use modern javascript features in browsers, such as AbortController, signals, and more.

And, there are still a fair number of advantages, like a separate .hmpl file extension when working with webpack and others, but in my opinion, those that I highlighted are the most important. Webpack config example:

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.hmpl$/i,
        use: ["hmpl-loader"],
      },
    ],
  },
};
```

## Disadvantages of HMPL

Also, hmpl has a number of disadvantages that I would like to talk about:

1. HMPL does not yet support WeBsockets, which may complicate the implementation of code into the project. In HTMX, this support is present.

2. Since fetch is used, the layout will not be supported in some older browser versions.

3. HMPL is a new module, it can sometimes have bugs. HTMX, on the contrary, is tested due to its widespread use.

Conclusion
The HMPL template language can replace HTMX in cases where flexible request customization is required, as well as direct work with nodes via JavaScript; If you, for example, want to create a cycle of 1000 identical nodes, and at the same time have the advantages of a server-oriented UI, then it will also suit the task. If the goal is to completely minimize work with JavaScript, or to use an established tested module and a simple connection to the server with a minimum amount of HTML code, then HTMX is good here.

Thank you all for reading this article!

Links:

[https://hmpl-lang.github.io](https://hmpl-lang.github.io)

[https://htmx.org](https://htmx.org)
