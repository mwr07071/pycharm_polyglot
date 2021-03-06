# ES6 Imports with Babel

In [Modules with CommonJS](../modules/modules.md) we looked at organizing our code into modules,
with imports similar to Python. That article showed NodeJS's native
module system, called CommonJS. We then showed the use of Webpack to
solve the problem of browsers which don't support modules and module
loaders.

In this section we look at a different system for imports, based on
the emerging new standards in JavaScript. We'll use ES6 imports, then
have the [Babel transpiler](http://babeljs.io) convert our code into
compliant code for Node and the browser.

> #### info::Note
> 
> Why say "ES6" instead of ES2015? It's true that "ES6" is being
> retired as a label, now that the committee managing the
> specifications has switched from a version number to a time-based
> naming scheme. But it turns out that modules and loaders is a
> multi-layered tricky business, with different specs being produced
> over time. We'll say "ES6 Modules" just to stay out of the details.

## Overview

- Install and configure Babel

- Switch modules and imports from CommonJS to ES6

- Get Babel plugged into frontend toolchain (Mocha, Webpack, ESLint)

## Starting Point

For the code in this article, we need a starting point that contains a
few pieces from earlier articles:

- Part webpack, part ESLint, part Mocha

- Ensure ESLint is hooked up in PyCharm

- npm start and npm test, plus Mocha test runner

Our `index.html` file is very simple, from the Webpack article:

[include](../../../src/es6_imports/index.html)

The webpack configuration is also the same:

[include](../../../src/es6_imports/webpack.config.js)

With this, Webpack bundles `app1.js` and `lib1.js` into the bundle
served by `webpack-dev-server`:

[include](../../../src/es6_imports/app1.js)

[include](../../../src/es6_imports/lib1.js)

Finally, we have our Mocha test file from the previous section:

[include](../../../src/es6_imports/test1.js)

## Installation

Let's install Babel, the transpiler for modern JavaScript, along with
a "loader" that lets Webpack transpile during bundling:

```
$ npm install --save-dev babel-preset-es2015 babel-loader
```

This gives us a `package.json` with the following added to the
`devDependencies`:

[include](../../../src/es6_imports/package.json)

## ES6 Modules

Let's change from modules and imports based on CommonJS, to modules based
on ES6. First, we make an `app2.js` that uses a different import syntax:

[include](../../../src/es6_imports/app2.js)

When we do this, PyCharm immediately freaks out. It is expecting
"ECMAScript 5.1" as the JavaScript syntax. We have now given it
non-valid tokens in the syntax. Thus, we need to to change
Preferences -> Languages & Frameworks -> JavaScript ->
JavaScript language version to `ECMAScript 6`.

At this point, ESLint also freaks out. It needs to be told the new syntax.
Fortunately ESLint, like Babel, is a hugely popular and fast-moving
open source project. They make it easy to work with new ES2015,
ES2016, and ES2017 syntax. Let's update our `.eslintrc` file:

[include, lang:"json"](../../../src/es6_imports/.eslintrc)

Let's now make a `lib2.js` which exports our function using ES6
export syntax:

[include](../../../src/es6_imports/lib2.js)

In this case, nothing is importable in this module *except* the
function. Because it is a different, incompatible syntax, we can
avoid the `module.exports` indirection, and put the `export
default` directly on the declaration.

By saying `default` in the export, we are saying that this
function is the one symbol exported. In fact, you could do the
export like this:

.. code-block:: javascript

  export default function (i) {

That is, with an anonymous, un-named function. With default exports,
the name comes from the importer, not the exporter.

We now have modules that use ES6 for exporting and importing. To see
if this works in Node, we can adapt our test to use ES6 imports:

[include](../../../src/es6_imports/test2.js)

Let's give it a whirl:

```
$ npm run test2.js
```

Uh-oh, we get an error:

```
SyntaxError: Unexpected token import
```

It seems that Mocha, which is a Node application, can't parse our
JavaScript. Well, no kidding...it's now written in ES6. We need
something that translates "future" JavaScript (ES6 Modules) into
"current" JavaScript (Node).

## Transpiling with Babel

This translation is called "transpiling" and Babel is the most-used
tool for transpiling. We talk more about transpiling in 
[Pythonic JavaScript with ES2015](../pythonic_js/pythonic_js.md).

We need to tell Mocha to run our JavaScript -- both our application
code *and* our tests , which we also want to write in the new
style -- through Babel. First, Babel has a configuration, since
of course all the other cool kids have one:

[include, lang:"json"](../../../src/es6_imports/.babelrc)

That configures Babel itself. To tell our PyCharm Mocha run configuration
to use Babel, edit the run configuration and in `Extra Mocha options`,
put this:

```
--compilers js:babel-core/register
```

Our Mocha tests now run fine. Which also means our `lib2.js` also
runs. To make this change for our `npm run` script, change `package.json`
to have:

```json
    "test": "mocha --compilers js:babel-core/register test*.js"
```

That covers Node. What about the browser? Do we need to teach Webpack to,
like Mocha, transpile code with Babel when bundling? Yes, and it is
quite easy:

[include](../../../src/es6_imports/webpack.config2.js)

We added a `modules` section and defined a `loader` for our `.js`
files. This loader runs the files through the `babel-loader` software,
which is a Webpack plugin for Babel. `babel-loader` uses the shared
Babel settings in `.babelrc`, preventing Mocha and Webpack from
keeping separate Babel settings.

## Wrapup

We are now using a tiny, wafer-thin mint of ES6: modules and imports. To
get there, we needed to introduce a new tool -- Babel -- into the
frontend toolchain and deal with the ripple effects:

- PyCharm -> ECMAScript 6

- ESLint

- Mocha

- Webpack

As you type, your CPU sure will be kept busy, transpiling and bundling
the universe. There are solutions to this, but for now, trust us...it's
worth it, as you'll see in the next article.