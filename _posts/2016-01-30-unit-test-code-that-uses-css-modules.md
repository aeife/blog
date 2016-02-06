---
layout: post
title: Unit test code that uses CSS Modules
comments: true
---

## How to fix that syntax error
Did you ever work with CSS and were annoyed of global variables, classes and accidental overwriting of your rules? *CSS Modules* is a great solution to this problems, as it allows importing CSS files into JavaScript. Especially in the *React*[^react] ecosystem the usage is widely spread. If you have not heard about *CSS Modules* yet, go and check it out here[^css-modules].

While using it is really cool, it often leads to a problem a lot of people are confronted with. Running unit tests for code that uses *CSS Modules* most likely will result in a *SyntaxError*, because JavaScript can’t handle the CSS syntax. In this article I demonstrate multiple ways of solving this problem, so unit testing code that imports CSS should be no problem anymore.

## CSS Modules
First of all let’s take a closer look this thing called *CSS Modules*. It’s a spec that describes a way of importing CSS files into JavaScript. By doing this you gain the possibility to reference your style content right in your application code. Here is an example:

```js
import styles from "./style.css";
// import { className } from "./style.css";

element.innerHTML = '<div class="' + styles.className + '">';
```

Importing and using CSS in JavaScript has a lot of advantages. You don’t need to remember your CSS syntax because you access it via an object and your IDE can suggest existing (and only existing) rules. An even bigger advantage is that all your classes can automatically be prefixed and modified. This is possible, because you don’t reference your CSS via simple strings, but instead via references to the imported object. That is actually a big deal as you can easily realize locally scoped and modular CSS.

The CSS Module spec is currently implemented via *css-loader*[^css-loader] for Webpack[^webpack], which is also the most common use case, although it’s not limited to this. Webpack and the *css-loader* handle the CSS imports, local scoping and automatic prefixing of class names.

## The Problem
While this is all very cool, there is one problem a lot of people are having with *CSS Modules* and finding a solution can be surprisingly tricky. The problem is that JavaScript can’t handle CSS syntax. So when you are running unit tests, the test will try to import the CSS file without Webpack resolving the import before. Most likely this will result in an error looking something like this:

```js
SyntaxError: Heading.scss: Unexpected token (3:0)
  1 | $color: green;
  2 |
> 3 | .header {
    | ^
  4 |   color: $color;
  5 | }
  6 |
```

As you can see, the test imported the content of the CSS file but can’t actually handle the syntax and therefore a *SyntaxError* is thrown. Fortunately there are multiple ways to handle this.

## Use a custom mocha compiler
Mocha allows the definition of compilers. This can be used to write a custom compiler for the JS files:

```js
function noop () {
 return null;
}
require.extensions['.css'] = noop;
```

This will handle every import that matches the defined extension. This way you can also handle SCSS, LESS or every other type of files. If you start mocha with the custom compiler, the tests will run without any problem.
mocha --compilers js:./tests/customCompiler.js

## Let Webpack handle it
If you are managing your tests via Webpack (e.g. with *mocha-loader*[^mocha-loader]), there is a way to fix it in Webpack itself. For this you need the *null-loader*[^null-loader] and the following definition in your Webpack config:

```js
{
    test: /(\.css|\.less|.\scss)$/,
    loader: 'null-loader'
}
```

With this configuration Webpack will load null for every encountered CSS file. To still load the CSS normally for the application it is recommended to specify a separate Webpack config for the tests.

## Use ignore-styles
There is an even simpler solution than the two previous ones, which only requires an additional npm module. First of all you need to install the module *ignore-styles*[^ignore-styles] in your project.

```js
npm install --save-dev ignore-styles
```

All you need to do after this is to require the module when running the tests:

```js
mocha --require ignore-styles
```

Easy enough. All it does is basically making sure every import of a CSS file is ignored. It also works when using preprocessors, e.g. imported SCSS files.

## Summary
As seen there are various methods to solve the issue of running unit tests for code that uses *CSS Modules*. A more manual one by defining a custom compiler by hand, a ready to go solution via the npm module *ignore-styles* and a Webpack solution via the *null-loader*.

Personally I think the solution with *ignore-styles* works best as it’s really easy to use, it does not require manual work and does not necessarily need Webpack to run the tests.

[^react]: [React](https://facebook.github.io/react/)
[^css-modules]: [CSS Modules](https://github.com/css-modules/css-modules)
[^css-loader]: [css-loader](https://github.com/webpack/css-loader)
[^webpack]: [Webpack](https://webpack.github.io/)
[^mocha-loader]: [mocha-loader](https://github.com/webpack/mocha-loader)
[^null-loader]: [null-loader](https://github.com/webpack/null-loader)
[^ignore-styles]: [ignore-styles](https://github.com/bkonkle/ignore-styles)
