---
title: React from Scratch - The Shell (Part 1)
date: '2019-1-26'
---

[`create-react-app`](https://github.com/facebook/create-react-app) is a great way to start working with React in a boostrapped development environment, but what if that didn't exist? What if you didn't know what to do after you hit [`eject`](https://github.com/facebook/create-react-app#philosophy)? How might you build a React app from scratch? I want to work through that, by hand. We'll see that its not terribly complicated. As we proceed, we'll do a high-level review of the things you'll need to build and run a React web app on your machine.

**The Shell - TL;DR**

- Install [`npm`](https://www.npmjs.com/) and Node.js
- Create a directory for the project
- Set up [`git`](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) (optional)
- Create `package.json` to manage node packages
- But Why?

---

### Install `npm` and Node.js

First you'll need to install [Node.js](https://nodejs.org/en/) and [`npm`](https://www.npmjs.com/get-npm). `npm` is a Node.js package manager and generally comes bundled with Node.js. I prefer to download them both from nodejs.org:

* [Download Node.js](https://nodejs.org/en/)

Follow the instructions and then ensure that installs succeeded:

```terminal
$ node -v
v10.13.0

$ npm -v
6.4.1
```

### Create a directory for the project

```terminal
~ $ mkdir basic-react-app
~ $ cd basic-react-app
.../basic-react-app
```

### Set up `git` (optional)

```terminal
$ git init
Initialized empty Git repository in /.../basic-react-app/.git/
```

### Create `package.json` to manage node packages

```terminal
$ npm init
```

Follow along with the prompts from `npm` for your new "package". You can add a different package name, version number, description, entry-point, test command, git repository, keywords, author, license, etc... but for our purposes you can just hit `enter` at each prompt and then type `"yes"` when you see the confirmed `package.json` in the prompt. It should look something like this:

```terminal
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (basic-react-app)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /.../basic-react-app/package.json:

{
  "name": "basic-react-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes)
```

Finally, run `npm install`. This will create a `package-lock.json` which allows you to share your confiuguration for _this_ project with someone else and know with confidence that they will have the exact same set of dependencies.

### But Why?

Why do I have to do all of this just to start building a React app? Well, the short answer is that `npm` is the standard software registry for JavaScript packages. We will install and make use of several packages hosted on `npm`. We want to try and build something that can be a base from which to scale.

You could, however, jump right in and start writing React in your web app by including links to both [CDN-hosted React and ReactDOM](https://reactjs.org/docs/cdn-links.html).

---

In [Part 2]({% post_url 2019-01-26-react-from-scratch-part-2 %}), we will look at setting up [webpack](https://webpack.js.org/)

