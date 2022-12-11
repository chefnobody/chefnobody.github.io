---
title: React from Scratch - webpack (Part 2)
date: "2019-1-26"
---

Now that we have a shell for our node modules, we will set up tooling to bundle and serve our JavaScript. This will make things easier for us as our app grows in size and complexity. We will turn to [`webpack`](https://webpack.js.org/) to help us with this!

At a high level webpack ... (ahem) "packages"  all your JavaScript code and assets into bundles.

- The resulting bundles can then be referenced from an HTML Document.
- You create bundles for different assets whether that is JavaScript, CSS, images, an ES6 module through "loaders".

**webpack - TL;DR**

- Install webpack
- Configure webpack
- Create a JavaScript file to bundle with webpack
- Create a HTML document to host the webpacked JavaScript bundle
- Run webpack!
- But Why?

### Install webpack

webpack and its cli tool come as a npm packages. Install them as development dependencies like this:

```terminal
$ npm install webpack webpack-cli --save-dev
```

There should now be a ton of node packages in the local `node_modules` directory.😍

### Configure webpack

Remember that webpack wants to bundle your assets and then put that bundle somewhere. To do that it needs to know where to look to start bundling and where to drop the payload when it is done. We tell webpack where to start by defining [entry](https://webpack.js.org/concepts/entry-points/) and [output](https://webpack.js.org/concepts/output/) points. We will be specific and define them with a new file named `webpack.config.js`:

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

webpack will look in the `src` directory for an `index.js` to build its big dependency graph and do its bundling. This also tells webpack to put the resulting bundled payload into `dist/main.js`.

### Create a JavaScript file for webpack to bundle

Create a new directory named `src` and add a JavaScript file named `index.js`:

```javascript
function helloWorld() {
  let element = document.createElement('div');
  element.innerHTML = 'Hello world!';
  return element;
}

document.body.appendChild(helloWorld());
```

### Create a static HTML Document to host the webpacked JavaScript bundle

Create a new directory `dist` and add a new HTML Document there named `index.html`. Add to it the following HTML:

```html
<!doctype html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

### Run webpack!

We're ready to let webpack do its thing. Type: `npx webpack`.

```terminal
$ npx webpack
Hash: bfd0e898782088551b28
Version: webpack 4.29.0
Time: 69ms
Built at: 01/25/2019 3:04:15 PM
  Asset      Size  Chunks             Chunk Names
main.js  1.02 KiB       0  [emitted]  main
Entrypoint main = main.js
[0] ./src/index.js 170 bytes {0} [built]
```

You should now have new file in `dist` named `main.js`. 

We can take this a step further and add a `build` script in `package.json` to do this for us.

```json
{
  ...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  ...
}

```

Now we can run this through `npm run build`. This will be a convenience when we add flags to the webpack calls.

### Profit

Open `dist/index.html` in your browser and you should see something like this:

![Hello world!](/assets/hello-world.png)

Hey now! 😵 

### But Why?

Now is a good time to ask "But why are we doing this?". We're using webpack because manually compiling, obfuscating, compressing and bundling all our JavaScript assets is a ton of work. webpack makes this a snap. Feel free to do that by hand, though. 😎

---

In [Part 3]({% post_url 2019-01-29-react-from-scratch-part-3 %}), we will finally write some [React](https://reactjs.org/)! ⚛️