---
title: React Refs
date: "2018-12-02"
---

### What is a `ref` in React?
_Stumble with me through React, a web browser and the fuzzy grey area between._

#### Background ####
Recently, I decided to read through all of the <a href="https://reactjs.org/docs/hello-world.html">React documentation</a> to better understand the framework and to try and pick up things I might have missed. Though I had worked with React a bit, I had not ventured into the documentation until I needed to know something specific. In hindsight, I wish I read through them sooner because they are fantastic.

When I came to <a href="https://reactjs.org/docs/refs-and-the-dom.html">Refs and the DOM</a> I wondered why anyone would need this functionality. React Components are an abstraction of the native DOM elements that your browser renders along with their attributes and various functions. React has a normal "data flow" thorugh these components managed by `props` and `state` and so it seemed that with them you could drive the values of any property on any `HTMLElement` about and get the behavior you want.

#### The Confusion

Check out the methods for the <a href="https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement#Methods">`HTMLElement`</a>. This means, that in the example from the <a href="https://reactjs.org/docs/refs-and-the-dom.html">Refs and the DOM</a> an `input` `HTMLElement` does not have a `focused` property, rather it has a `.focus()` function. You can't easily use `props`, `state` or the view lifecycle to manipulate a custom `input` Component to auto-focus. In React, it is useful to have a reference to this DOM element so that you can call `.focus()` on it. I think this is what is meant by an "escape hatch".

#### References

<ul>
  <li><a href="https://reactjs.org/docs/rendering-elements.html">Rendering Elements</a></li>
  <li><a href="https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html">React Components, Elements and Instances</a></li>
</ul>
