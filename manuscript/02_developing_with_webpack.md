# Developing with Webpack

If you are not one of those people who likes to skip the introductions, you might have some clue what Webpack is. In its simplicity, it is a module bundler. It takes a bunch of assets in and outputs assets you can give to your client.

This sounds simple, but in practice, it can be a complicated and messy process. You definitely don't want to deal with all the details yourself. This is where Webpack fits in. Next, we'll get Webpack set up and your first project running in development mode.

W> Before getting started, make sure you are using a recent version of Node.js. Especially Node.js 0.10 has [issues with css-loader](https://github.com/webpack/css-loader/issues/144). This will save you some trouble.

## Setting Up the Project

Webpack is one of those tools that depends on [Node.js](http://nodejs.org/). Make sure you have it installed and that you have `npm` available at your terminal. Set up a directory for your project, navigate there, hit `npm init` and fill in some details. You can just hit *return* for each and it will work. Here are the commands:

```bash
mkdir kanban_app
cd kanban_app
npm init
# hit return a few times till you have gone through the questions
```

As a result, you should have *package.json* at your project root. You can still tweak it manually to make further changes. We'll be doing some changes through *npm* tool, but it's fine to tweak the file to your liking. The official documentation explains various [package.json options](https://docs.npmjs.com/files/package.json) in more detail. I also cover some useful library authoring related tricks later in this book.

If you are into version control, as you should, this would be a good time to set up your repository. You can create commits as you progress with the project.

If you are using git, I recommend setting up a *.gitignore* to the project root:

**.gitignore**

```bash
node_modules
```

At the very least you should have *node_modules* here as you probably don't want that to end up in the source control. The problem with that is that as some modules need to be compiled per platform, it gets rather messy to collaborate. Ideally your `git status` should look clean. You can extend *.gitignore* as you go.

T> You can push operating system level ignore rules such as *.DS_Store* and *\*.log* to *~/.gitignore*. This will keep your project level rules simpler.

## Installing Webpack

Next, you should get Webpack installed. We'll do a local install and save it as a project dependency. This will allow us to maintain Webpack's version per project. Hit

```bash
npm i webpack --save-dev
```

This is a good opportunity to try to run Webpack for the first time. Hit `node_modules/.bin/webpack`. You should see a version log, a link to the command line interface guide and a long list of options. We won't be using most of those, but it's good to know that this tool is packed with functionality if nothing else.

Webpack works using a global install as well (`-g` or `--global` flag during installation). It is preferred to keep it as a project dependency like this. The arrangement helps to keep your life simpler. This way you have direct control over the version you are running.

We will be using `--save` and `--save-dev` to separate application and development dependencies. The separation keeps project dependencies more understandable. This will come in handy when we generate a vendor bundle later on.

T> There are handy shortcuts for `--save` and `--save-dev`. `-S` maps to `--save` and `-D` to `--save-dev`. So if you want to optimize for characters written, consider using these instead.

## Directory Structure

As projects with just *package.json* are boring, we should set up something more concrete. To get started, we can implement a little web site that loads some JavaScript which we then build using Webpack. Set up a structure like this:

- /app
  - index.js
  - component.js
- package.json
- webpack.config.js

In this case, we'll generate *bundle.js* using Webpack based on our */app*. To make this possible, we should set up some assets and *webpack.config.js*.

## Setting Up Assets

As you never get tired of `Hello world`, we might as well model a variant of that. Set up a component like this:

**app/component.js**

```javascript
module.exports = function () {
  var element = document.createElement('h1');

  element.innerHTML = 'Hello world';

  return element;
};
```

Next, we are going to need an entry point for our application. It will simply `require` our component and render it through the DOM:

**app/index.js**

```javascript
var component = require('./component');
var app = document.createElement('div');

document.body.appendChild(app);

app.appendChild(component());
```

## Setting Up Webpack Configuration

We'll need to tell Webpack how to deal with the assets we just set up. For this purpose we'll build *webpack.config.js*. Webpack and its development server will be able to discover this file through convention.

To keep things simple, we'll generate an entry point to our application using [html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin). We could create *index.html* by hand. Maintaining that could become troublesome as the project grows, though. *html-webpack-plugin* is able to create links to our assets keeping our life simple. Hit

```bash
npm i html-webpack-plugin --save-dev
```

to install it to the project.

To map our application to *build/bundle.js* and generate *build/index.html* we need configuration like this:

**webpack.config.js**

```javascript
var path = require('path');
var HtmlwebpackPlugin = require('html-webpack-plugin');

var ROOT_PATH = path.resolve(__dirname);
var APP_PATH = path.resolve(ROOT_PATH, 'app');
var BUILD_PATH = path.resolve(ROOT_PATH, 'build');

module.exports = {
  entry: APP_PATH,
  output: {
    path: BUILD_PATH,
    filename: 'bundle.js'
  },
  plugins: [
    new HtmlwebpackPlugin({
      title: 'Kanban app'
    })
  ]
};
```

Given Webpack expects absolute paths we have some good options here. I like to use `path.resolve`, but `path.join` would be a good alternative. `path.resolve` is equivalent to navigating the file system through *cd*. `path.join` gives you just that, a join. See [Node.js path API](https://nodejs.org/api/path.html) for the exact details.

If you hit `node_modules/.bin/webpack` now you should see a Webpack build. You can serve */build* through a dummy server such as *serve* (`npm i serve -g`). Examine the results in a browser.

Even though this is nice, it's not useful for development. We can set up something better for development usage.

T> Note that you can pass a custom template to *html-webpack-plugin*. In our case, the default template it uses is fine for our purposes for now.

T> There are other little plugins, such as `html-webpack-plugin`, that you may find useful. For example [open-browser-webpack-plugin](https://www.npmjs.com/package/open-browser-webpack-plugin) allows you to open a browser automatically after Webpack is running.

## Setting Up *webpack-dev-server*

Now that we have the basic building blocks together, we can set up a development server. *webpack-dev-server* is a development server that automatically refreshes content in the browser while you develop your application.

This makes it roughly equivalent to tools such as [LiveReload](http://livereload.com/) or [Browsersync](http://www.browsersync.io/). The greatest advantage Webpack has over these tools is Hot Module Replacement (HMR). We'll discuss it when we go through React.

Hit

```bash
npm i webpack-dev-server --save-dev
```

at the project root to get the server installed. We will be invoking our development server through npm. It allows us to set up `scripts` at *package.json*. The following configuration is enough:

**package.json**

```json
...
"scripts": {
  "start": "webpack-dev-server"
},
...
```

We also need to do some configuration work.

**webpack.config.js**

```javascript
...
var webpack = require('webpack');

var ROOT_PATH = path.resolve(__dirname);
var APP_PATH = path.resolve(ROOT_PATH, 'app');
var BUILD_PATH = path.resolve(ROOT_PATH, 'build');

module.exports = {
  ...
  devServer: {
    historyApiFallback: true,
    hot: true,
    inline: true,
    progress: true
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    ...
  ]
};
```

Hit `npm start` and surf to **localhost:8080**. You should see something familiar there. Try modifying *app/component.js* while the server is running and see what happens. Quite neat, huh?

![Hello world](images/hello_01.png)

Or we can run the application from **localhost:8080/webpack-dev-server/bundle** instead of root. It provides an iframe showing a status bar. It indicates the status of the rebundling process.

T> If you want to use some other port than `8080`, you can pass `port` parameter (e.g., `port: 4000`) to *devServer*.

In addition to **webpack.config.js** it is possible to set *webpack-dev-server* configuration through a command line interface (CLI). For additional configuration options, refer to the WebpackDevServer's [Node.js API](https://webpack.github.io/docs/webpack-dev-server.html#api) documentation.

W> Note that there are [slight differences](https://github.com/webpack/webpack-dev-server/issues/106) between the CLI and Node.js API and they may behave slightly differently at times. This is the reason why some prefer to use solely Node.js API.

We are using a somewhat basic setup here. Beyond defaults we've enabled Hot Module Replacement (HMR) and HTML5 History API fallback. The former will come in handy when we discuss React in detail. The latter allows HTML5 History API routes to work. *inline* setting embeds the *webpack-dev-server* runtime into the bundle allowing HMR to work easily. Otherwise we would have to set up more `entry` paths.

## Refreshing CSS

We can extend the approach to work with CSS. Webpack allows us to change CSS without forcing a full refresh. To load CSS into a project, we'll need to use a couple of loaders. To get started, invoke

```bash
npm i css-loader style-loader --save-dev
```

T> If you are using Node.js 0.10, this is a good time to get a [ES6 Promise polyfill](https://github.com/jakearchibald/es6-promise#auto-polyfill) set up.

Now that we have the loaders we need, we'll need to make sure Webpack is aware of them. Configure as follows.

**webpack.config.js**

```javascript
...

module.exports = {
  entry: APP_PATH,
  output: {
    path: BUILD_PATH,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loaders: ['style', 'css'],
        include: APP_PATH
      }
    ]
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new HtmlwebpackPlugin({
      title: 'Kanban app'
    })
  ]
};
```

The configuration we added means that files ending with `.css` should invoke given loaders. `test` matches against a JavaScript style regular expression. The loaders are evaluated from right to left. In this case, *css-loader* gets evaluated first, then *style-loader*. *css-loader* will resolve `@import` and `url` statements in our CSS files. *style-loader* deals with `require` statements in our JavaScript. A similar approach works with CSS preprocessors, like Sass and Less, and their loaders.

T> Loaders are transformations that are applied to source files, and return the new source. Loaders can be chained together, like using a pipe in Unix. See Webpack's [What are loaders?](http://webpack.github.io/docs/using-loaders.html) and [list of loaders](http://webpack.github.io/docs/list-of-loaders.html).

W> If `include` isn't set, Webpack will traverse all files within the base directory. This can hurt performance! It is a good idea to set up `include` always. There's also `exclude` option that may come in handy.

We are missing just one bit, the actual CSS itself:

**app/main.css**

```css
body {
  background: cornsilk;
}
```

Also, we'll need to make Webpack aware of this file:

**app/index.js**

```javascript
require('./main.css');

...
```

Hit `npm start` now. Point your browser to **localhost:8080** if you are using the default port.

Open up *main.css* and change the background color to something like `lime` (`background: lime`). Develop styles as needed to make it look a little nicer.

![Hello cornsilk world](images/hello_02.png)

## Making the Configuration Extensible

Our current configuration is enough as long as we're interested in just developing our application. But what if we want to deploy it to the production or test our application? We need to define separate **build targets** for these purposes. Given Webpack uses a module based format for its configuration, there are multiple possible approaches. At least the following are feasible:

* Maintain configuration in multiple files and point Webpack to each through `--config` parameter. Share configuration through module imports. You can see this approach in action at [webpack/react-starter](https://github.com/webpack/react-starter).
* Push configuration to a library which you then consume. Example: [HenrikJoreteg/hjs-webpack](https://github.com/HenrikJoreteg/hjs-webpack).
* Maintain configuration within a single file and branch based on npm lifecycle event. That is set when we trigger a script through *npm* (i.e., `npm run test`). I prefer this approach as it allows me to understand what's going on easily. We'll be using this approach.

The idea is that we extract configuration that's common to each target into a structure of its own. When defining the targets, we use a variant of `merge` function that overrides objects and concatenates arrays. This convention works well with Webpack as we'll soon see.

T> Webpack works well as a basis for more advanced tools. I've helped to develop a static site generator known as [Antwar](https://antwarjs.github.io/). It builds upon Webpack and React and hides a lot of the complexity from the user.

### Setting Up Configuration Target for `npm start`

To keep things simple, I've pushed the `merge` function to a package of its own. Hit

```bash
npm i webpack-merge --save-dev
```

to add it to the project. Add `merge` stub as below. The idea is that we detect npm lifecycle event (`start`, `build`, ...) and then branch and merge based on that. We'll expand these in the coming chapters.

To improve the debuggability of the application, we can set up sourcemaps while we are at it. They allow you to see exactly where an error was raised. In Webpack this is controlled through the `devtool` setting. We can use decent defaults as follows:

**webpack.config.js**

```javascript
var path = require('path');
var HtmlwebpackPlugin = require('html-webpack-plugin');
var webpack = require('webpack');
var merge = require('webpack-merge');

var TARGET = process.env.npm_lifecycle_event;
var ROOT_PATH = path.resolve(__dirname);
var APP_PATH = path.resolve(ROOT_PATH, 'app');
var BUILD_PATH = path.resolve(ROOT_PATH, 'build');

var common = {
  entry: APP_PATH,
  output: {
    path: BUILD_PATH,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loaders: ['style', 'css'],
        include: APP_PATH
      }
    ]
  },
  plugins: [
    // important! move HotModuleReplacementPlugin below
    //new webpack.HotModuleReplacementPlugin(),
    new HtmlwebpackPlugin({
      title: 'Kanban app'
    })
  ]
};

if(TARGET === 'start' || !TARGET) {
  module.exports = merge(common, {
    devtool: 'eval-source-map',
    devServer: {
      historyApiFallback: true,
      hot: true,
      inline: true,
      progress: true
    },
    plugins: [
      new webpack.HotModuleReplacementPlugin()
    ]
  });
}
```

`if(TARGET === 'start' || !TARGET) {` provides a default in case we're running Webpack outside of npm.

If you run the development build now using `npm start`, Webpack will generate sourcemaps. Webpack provides many different ways to generate them as discussed in the [official documentation](https://webpack.github.io/docs/configuration.html#devtool). In this case, we're using `eval-source-map`. It builds slowly initially, but it provides fast rebuild speed and yields real files.

W> If `new webpack.HotModuleReplacementPlugin()` is added twice to the plugins declaration, Webpack will return `Uncaught RangeError: Maximum call stack size exceeded` while hot loading!

Faster development specific options such as `cheap-module-eval-source-map` and `eval` produce lower quality sourcemaps. Especially `eval` is fast and is the most suitable for large projects.

It is possible you may need to enable sourcemaps at your browser for this to work. See [Chrome](https://developer.chrome.com/devtools/docs/javascript-debugging) and [Firefox](https://developer.mozilla.org/en-US/docs/Tools/Debugger/How_to/Use_a_source_map) instructions for further details.

Configuration could contain more sections such as these based on your needs. Later on we'll develop another section to generate a production build.

## Linting the Project

I discuss linting in detail in the *Linting in Webpack* chapter. Consider integrating the setup to your project to save some time. It will allow you to pick certain categories of errors earlier.

## Conclusion

In this chapter you learned to build an effective development configuration using Webpack. Webpack deals with the heavy lifting for you now. The current setup can be expanded to support more scenarios. Next, we will see how to expand it to work with React.
