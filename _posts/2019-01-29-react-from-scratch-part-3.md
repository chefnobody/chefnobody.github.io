---
title: React from Scratch - React (Part 3)
date: "2019-1-29"
---

[React](https://reactjs.org/) is a framework from Facebook that lets us build user interfaces. It is known for its composability and the way it efficiently manages changes to the DOM.

Before you read any further, I would highly recommend all the [documentation](https://reactjs.org/docs/getting-started.html). It is fantastic and you shouldn't sleep on it.

**React - TL;DR**

- Install React and ReactDOM
- Write some React ⚛️ !!! 
- JSX
- Babel and babel-loader
- Summary

### Install React and React DOM

Let's install `react` and the `react-dom` node packages as primary dependencies:

```terminal
$ npm install react react-dom
```

### Write some React!

Now update `src/index.js` to import React and ReactDOM to create a React Component and render it into the DOM:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  React.createElement('h1', null, 'Hello from React!'),
  document.getElementById('root')
);
```

I know what you're saying, this doesn't make use of that JSX syntatic sugar. JSX is super helpful, but it can confuse you at first if you do not understand what it is or what it represents. More on that below.

Now update `dist/index.html` and add a `<div>` into which to ReactDOM will render our React Components:

```html
<!doctype html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <div id="root">
    
    </div>
    <script src="main.js"></script>
  </body>
</html>
```
**Note:** The order of the `<script>` that's running the JavaScript and the `<div id='root'>` matters. I believe our script that is looking for the `'id=root'` DOM element won't find it if the script is declared _before_ the `<div>`. Be sure to include it after the `<div>`. Try putting it before the `<div>` or even in the `<head>` and see what happens.

Re-run webpack, and refresh your browser:

![Hello world!](/assets/hello-world-react.png)

### JSX

[JSX](https://facebook.github.io/jsx/) is an extension to JavaScript that allows you to write markup in scripts. It is neither a string nor HTML and [Babel](https://babeljs.io/) will compile JSX into `React.createElement()` function calls like we wrote earlier. You can read [more about JSX here](https://reactjs.org/docs/jsx-in-depth.html).

Simply writing JSX in our script won't get us all the way there. webpack will barf because it finds a token in the script that it doesn't yet understand. Try it out:

Update `src/index.js` with some JSX:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

const element = <h1>Hello from React!</h1>;

ReactDOM.render(
  element,  
  document.getElementById('root')
);
```

Then try running webpack:

```terminal
$ npm run build

> basic-react-app@1.0.0 build /Users/aconnolly/workspace/react/basic-react-app
> webpack

Hash: 2b9f3a724dab7114999b
Version: webpack 4.29.0
Time: 332ms
Built at: 01/28/2019 6:36:15 AM
 1 asset
Entrypoint main = main.js
[0] ./src/index.js 238 bytes {0} [built] [failed] [1 error]

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/

ERROR in ./src/index.js 4:16
Module parse failed: Unexpected token (4:16)
You may need an appropriate loader to handle this file type.
| import ReactDOM from 'react-dom';
|
> const element = <h1>Hello from React!</h1>;
|
| ReactDOM.render(
npm ERR! code ELIFECYCLE
npm ERR! errno 2
npm ERR! basic-react-app@1.0.0 build: `webpack`
npm ERR! Exit status 2
npm ERR!
npm ERR! Failed at the basic-react-app@1.0.0 build script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/aconnolly/.npm/_logs/2019-01-28T14_36_15_408Z-debug.log
```

### Babel and babel-loader

To use JSX in our scripts we need to tell webpack use a special Babel "preset" to load our JavaScript files. [Babel is a JavaScript compiler](https://babeljs.io/) and it has syntax presets that we can use to tell webpack that we're writing JSX. 

webpack uses [loaders](https://webpack.js.org/loaders/) to customize compilation for different file types. 

This step is complicated, so there are a few dev dependencies to install:

- [`babel-loader`](https://webpack.js.org/loaders/babel-loader/): Allows compiling JavaScript using Babel
- `@babel/core`: Nuts and bolts of the Babel compiler
- `@babel/preset-react`: Let's us write JSX and includes various other plugins.

Install them and watch `node_modules` swell like a full belly on Thanksgiving! 🤙

```terminal
$ npm install babel-loader @babel/core @babel/preset-react --save-dev
```

Now that all of those things are installed, how do we _tell_ webpack to use a babel-loader for our JavaScript files _and_ that I want to use the react preset? A: In `webpack.config.js` add the loader as a new `module` key with some rules for which file types, folder exclusions, loader and presets:

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.m?js$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-react']
          }
        }
      }
    ]
  }  
};
```

Now run webpack again:

```terminal
$ npm run build

> basic-react-app@1.0.0 build /Users/aconnolly/workspace/react/basic-react-app
> webpack

Hash: 9eb49dcd15e2806ec9bf
Version: webpack 4.29.0
Time: 2509ms
Built at: 01/28/2019 6:56:32 PM
  Asset     Size  Chunks             Chunk Names
main.js  109 KiB       0  [emitted]  main
Entrypoint main = main.js
[3] ./src/index.js 189 bytes {0} [built]
[8] (webpack)/buildin/global.js 472 bytes {0} [built]
    + 7 hidden modules

WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```

No errors! Done.

### Summary

This was interesting because there were a bunch of moving parts. There is `npm`, then `webpack`, then React and finally Babel and its `loader` system to help us with the latest JavaScript jazz like JSX. We worked through a bunch of fairly complicated things pretty quickly and now we have a very basic shell of a React app.

Nice job, time for some oreos.