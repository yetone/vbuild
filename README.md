<p align="center">
  <img src="https://cloud.githubusercontent.com/assets/8784712/23060768/1e3bea76-f53a-11e6-8735-998ee5f87238.png" alt="preview" />
</p>

## Badges

[![NPM version](https://img.shields.io/npm/v/vbuild.svg?style=flat-square)](https://npmjs.com/package/vbuild) [![NPM downloads](https://img.shields.io/npm/dm/vbuild.svg?style=flat-square)](https://npmjs.com/package/vbuild) [![Build Status](https://img.shields.io/circleci/project/egoist/vbuild/master.svg?style=flat-square)](https://circleci.com/gh/egoist/vbuild) [![donate](https://img.shields.io/badge/$-donate-ff69b4.svg?maxAge=2592000&style=flat-square)](https://github.com/egoist/donate) [![twitter](https://img.shields.io/badge/twitter-@vbuildjs-1da1f2.svg?style=flat-square)](https://twitter.com/vbuildjs)

## tl;dr

```bash
vbuild whatever.js --dev
# it just works
```

Develop Vue apps with no build configuration until you need.

<h2 id="toc"></h2>

<!-- toc -->

- [Install](#install)
- [How to use](#how-to-use)
- [Commands](#commands)
- [Config file](#config-file)
  * [Shorthand](#shorthand)
  * [Arguments](#arguments)
    + [options](#options)
    + [req](#req)
  * [Babel](#babel)
    + [Transpile modules](#transpile-modules)
  * [PostCSS](#postcss)
    + [Custom CSS preprocessors](#custom-css-preprocessors)
    + [CSS modules](#css-modules)
  * [Webpack entry](#webpack-entry)
  * [Hot reloading](#hot-reloading)
  * [Code splitting](#code-splitting)
  * [Webpack](#webpack)
  * [Custom HTML output](#custom-html-output)
  * [Custom output filename](#custom-output-filename)
  * [Clean dist files](#clean-dist-files)
  * [Extracting CSS](#extracting-css)
  * [Copy static files](#copy-static-files)
  * [Define constants at compile time](#define-constants-at-compile-time)
  * [Load env variables](#load-env-variables)
  * [Proxy API request](#proxy-api-request)
  * [Dev server](#dev-server)
    + [port](#port)
    + [host](#host)
  * [Custom server logic](#custom-server-logic)
  * [Custom build process](#custom-build-process)
- [JavaScript API](#javascript-api)
- [Recipes](#recipes)
- [Limitations](#limitations)
- [FAQ](#faq)
- [Contributing](#contributing)
- [Author](#author)

<!-- tocstop -->

## Install

It works with both Yarn(>=0.17) and npm(>=3):

```bash
yarn global add vbuild
# You can also install it locally
# yarn add vbuild --dev
```

## How to use

<details><summary>The simple way</summary>

```bash
vbuild init <folder>
```

Then follow the instructions in terminal.
</details>

Or manually, populate an entry file, let's say `index.js`:

```js
import Vue from 'vue'

new Vue({
  el: '#app',
  render: h => <h1>Hello World!</h1>
})
```

Run app in dev mode:

```bash
vbuild index.js --dev
```

So far we get:

- Automatic transpilation and bundling (with webpack and babel/postcss)
- Hot code reloading
- Static file in `./static/` will be copied to `./dist/`

To see how simple it is, check out [our official website](https://github.com/egoist/vbuild.js.org) which is built with vbuild itself.

Build app in production mode (default mode):

```bash
vbuild index.js
```

[⬆ back to top](#toc)

## Commands

- `vbuild`: Default command, run build process
- `vbuild init <folder>`: Generate a template (app template/electron template/component template) 

## Config file

All CLI options and advanced options can be set here:

```js
module.exports = (options, req) => ({
  port: 5000
  // Other options
})

// Note that you can directly export an object too:
// module.exports = {port: 5000}
```

To use it, you can add `--config [path]` in CLI arguments. If no path was speified, it will use `vbuild.config.js` if it exists.

[⬆ back to top](#toc)

### Shorthand

To set different config for different mode, you may use `options.dev` like:

```js
module.exports = options => ({
  webpack(cfg) {
    if (options.dev) {}
    else {}
    return cfg
  }
})
```

To simplify this, we provide a shorthand to do this:

```js
module.exports = {
  production: {}, // used in `!options.dev`
  development: {} // used in `options.dev`
}
```

The `production` or `development` config will be assigned into base config using `Object.assign`.

[⬆ back to top](#toc)

### Arguments

#### options

CLI options.

[⬆ back to top](#toc)

#### req

The `require` function but context directory is the path to `node_modules/vbuild/lib`, which means you can use it to load vbuild's dependencies, like `webpack`.

[⬆ back to top](#toc)

### Babel

JS files and `script` tags in single-file components are transpiled by Babel. We only use one preset by default: [babel-preset-vue-app](https://github.com/egoist/babel-preset-vue-app).

vbuild will use `.babelrc` if it exists, you can also set `babelrc` option in config file to disable config file, check out [full reference](https://babeljs.io/docs/usage/api/#options) for `babel` option.

[⬆ back to top](#toc)

#### Transpile modules

By default babel will only transpile files outside `node_modules` folder into ES5, but you may use some npm packages that are written in ES2015, then you can tell babel to transpile them as well:

```js
module.exports = {
  transpileModules: ['element-ready']
}
```

[⬆ back to top](#toc)

### PostCSS

Standalone `.css` files and `style` tags in single-file components are transpiled by PostCSS, the only plugin we use by default is `autoprefixer`, and you can use `autoprefixer` option in config file to adjust it, here's the config with default value:

```js
module.exports = {
  autoprefixer: {
    browsers: ['ie > 8', 'last 4 versions']
  }
}
```

You can use PostCSS config file like `postcss.config.js` or whatever [postcss-load-config](https://github.com/michael-ciniawsky/postcss-load-config) supports. `postcss` option is also available in config file.

Note that we only add autoprefixer when you use an `Array` or `Object` as postcss option.

#### Custom CSS preprocessors

Supported preprocessors: `sass` `scss` `stylus` `less`, the workflow of CSS is `custom css preprocessor` -> `postcss-loader` -> `css-loader`.

To use a custom CSS preprocessor, you can directly install relevant loader and dependency. For example, to use `scss`:

```bash
yarn add node-sass sass-loader --dev
```

#### CSS modules

To use CSS modules in single-file component, you can set `module` attribute in `<style></style>` tag.

To use CSS modules in standalone css files, you can set `cssModules` to `true` in config file.

[⬆ back to top](#toc)

### Webpack entry

Type: `string` `Array` `Object`

You can set webpack entry from CLI option or `entry` property in config file. If it's an array or string, we add it into `webpackConfig.entry.client` entry, otherwise it will totally override `webpackConfig.entry`

[⬆ back to top](#toc)

### Hot reloading

By default we add HMR client to `client` entry, you can change it by:

```js
module.exports = {
  hmrEntry: ['other-name']
}
```

[⬆ back to top](#toc)

### Code splitting

We enabled code splitting for vendor code and app code by default in production mode, you can set `vendor` option to `false` to disable it. And by default all required modules in `node_modules` will be split.

[⬆ back to top](#toc)

### Webpack

Mutate webpack config as you wish:

```js
module.exports = options => ({
  webpack(webpackConfig) {
    if (options.dev) {
      // Apply some changes to webpackConfig
    }
    return webpackConfig
  }
})
```

The value of `webpack` could also be a plain object, this way it will be merged into default webpack config using [webpack-merge](https://github.com/survivejs/webpack-merge).

[⬆ back to top](#toc)

### Custom HTML output

Type: `Object` `Array` `boolean`

[html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) options, use this option to customize `index.html` output, default value:

```js
module.exports = {
  html: {
    title: 'VBuild',
    template: path.join(__dirname, '../lib/template.html')
  }
}
```

Check out the [default template](/lib/template.html) file we use. To disable generating html file, you can set `html` to `false`.

[⬆ back to top](#toc)

### Custom output filename

Set custom filename for js css static files:

```js
module.exports = {
  filename: {
    js: 'index.js',
    css: 'style.css',
    static: 'static/[name].[ext]'  
  }
}
```

[⬆ back to top](#toc)

### Clean dist files

The files inside dist folder will be removed before you run vbuild in production mode, because in most cases the output filename will contain `[hash]`, we need to remove old files to keep the directory clean.

However in some cases you don't need this, then you can disable it by:

```js
module.exports = {
  cleanDist: false
}
```

[⬆ back to top](#toc)

### Extracting CSS

The `extract` option is `true` by default in production mode, however you can also set it manually to overrde:

```js
module.exports = {
  // always disable extracting css
  extract: false
}
```

[⬆ back to top](#toc)

### Copy static files

By default, all files inside `./static` folder will be copied to dist folder, you can set it to `false` to disable this.

If your want it to copy other folders, use an array instead:

```js
module.exports = {
  // copy ./public to ./dist/public
  copy: [{from: './public', to: './public'}]
}
```

See more options about this at [copy-webpack-plugin](https://github.com/kevlened/copy-webpack-plugin#pattern-properties).

[⬆ back to top](#toc)

### Define constants at compile time

`define` is a short-hand to add webpack's [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) for settings global constants which can be configured at compile time.

```js
module.exports = options => ({
  define: {
    __DEV__: JSON.stringify(options.dev)
  }
})
```

Then use it in your app code:

```js
if (__DEV__) {
  // perform something only in development mode a.k.a `--dev`
}
```

The default constants we add is `process.env.NODE_ENV`. The variables you defined here are only available in app code.

[⬆ back to top](#toc)

### Load env variables

Your project can consume variables declared in your environment as if they were declared locally in your JS files. By default you will have `NODE_ENV` defined.

There're three way to define env variables:

- CLI options `--env.VARIABLE_NAME xxx`
- `env` option in config file
- `.env` file via [dotenv](https://github.com/motdotla/dotenv)

You can use them in `index.html` and app code:

```bash
# .env file
VUE_APP_DESCRIPTION=my awesome project
```

In template html file which uses [ejs](http://ejs.co) syntax, you can write:

```html
<meta name="description" content="<%= htmlWebpackPlugin.options.env.VUE_APP_DESCRIPTION %>" />
```

In app code you access it by:

```js
const key = process.env.VUE_APP_KEY
```

To totally disable loading env variables, you can set `env` to `false`.

[⬆ back to top](#toc)

### Proxy API request

To tell the development server to serve any `/api/*` request to your API server in development, use the `proxy` options:

```js
module.exports = {
  proxy: 'http://localhost:8080/api'
}
```

This way, when you fetch `/api/todos` in your Vue app, the development server will proxy your request to `http://localhost:8080/api/todos`.

We use [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware) under the hood, so the `proxy` option can also be an object:

```js
module.exports = {
  proxy: {
    '/api/foo': 'http://localhost:8080/api',
    '/api/fake-data': {
      target: 'http://jsonplaceholder.typicode.com',
      changeOrigin: true,
      pathRewrite: {
        '^/api/fake-data': ''
      }
    }
  }
}
```

Keep in mind that proxy only has effect in development.

[⬆ back to top](#toc)

### Dev server

#### port

Type: `number`<br>
Default: `4000`

Port of dev server.

#### host

Type: `string`<br>
Default: `localhost`

Host of dev server.

[⬆ back to top](#toc)

### Custom server logic

Perform some custom logic to development server:

```js
module.exports = {
  setup(app) {
    app.get('/api', (req, res) => {
      res.end('This is the API')
    })
  }
}
```

[⬆ back to top](#toc)

### Custom build process

Insead of letting vbuild run webpack as the build process, you can perform a custom one by using `run` function in config file:

```js
// For example, run tests with Karma
const Server = require('karma').Server

module.exports = {
  run(webpackConfig) {
    const server = new Server({
      webpack: webpackConfig,
      // ...Other karma options
    })
    server.start()
  }
}
```

[⬆ back to top](#toc)

## JavaScript API

You can use vbuild as a Node.js module:

```js
const vbuild = require('vbuild')

vbuild(cliOptions).then(res => {})
//=> res:
{
  webpackConfig, // final webpack config
  options, // final options
  compiler, // webpack compiler instance
  watcher, // in watch mode, webpack watcher
  server, // in dev mode, an instance of `http.Server` without calling `.listen`
  devMiddleware // in dev mode, the webpack-dev-middleware instance
}

// get webpack config and merged options only
vbuild.getConfig(cliOptions).then(res => {})
//=> res:
{
  webpackConfig,
  options
}

// error catch
function handleError(err) {
  if (err.name === 'AppError') {
    // error occurs in starting the compilation
  } else {
    // other unknown error
  }
}

vbuild(options).catch(handleError)
```

[⬆ back to top](#toc)

## Recipes

- [Minimize and sourcemaps](./docs/recipes/minimize-and-sourcemaps.md)
- [Bundle in CommonJS or UMD format](./docs/recipes/cjs-and-umd-format.md)
- [Progressive web app](./docs/recipes/progressive-web-app.md)
- [Electron app](./docs/recipes/electron-app.md)
- [ESLint](./docs/recipes/eslint.md)
- [Testing](./docs/recipes/testing.md)
- [Deployment](./docs/recipes/deployment.md)
- [Analyze bundle size](./docs/recipes/bundle-analyzer.md)
- [Troubleshooting](./docs/troubleshooting.md)

[⬆ back to top](#toc)

## Limitations

- Server-side rendering: [you can generate server bundle though](./docs/recipes/cjs-and-umd-format.md#server-bundle)
- Eject webpack config: [here's an on-going issue](https://github.com/egoist/vbuild/issues/124)

Feel free to request new features.

## FAQ

<details><summary>Is it like Next.js or Nuxt.js?</summary>

Yes and no, yes is because they all simplified the process of building a complete web app, while `vbuild` is more focusing on building single-page app without the server-side, at least it is for now.
</details>

<details><summary>Is it like vue-cli?</summary>

No, vue-cli is just a boilerplate generator while vbuild is a Webpack wrapper which reduces boilerplate code for you.

You may notice that there's a `vue build` command lying in `vue-cli`, that's actually quite similar to vbuild, but providing less features and vbuild goes far beyond that.
</details>

<details><summary>Is there a `--watch` mode?</summary>

Sure, you can combine the `--watch` mode with default mode and `--dev` mode, when it's combined with `--dev` mode, it will remove the hot-reloading support.
</details>

<details><summary>What are the differences between `--watch` `--dev` and production mode?</summary>

The default mode is production mode, i.e. without `--dev`.

`--dev` mode uses hot reloading by default, when your file does not support hot reloading it fallbacks to live reloading.

`--watch` can be used with/without `-dev` flag:

- with `--dev`: no dev server, no hot reloading, since you may not need to open browser at all. It only rebuilt when file changes, all other features in `dev` are the same.
- without `--dev`: like production mode but it rebuilt due to file changes.
</details>

<details><summary>What is this inspired by?</summary>

Despiting that `vbuild` predates `Next.js` `create-react-app` `nwb` `vue-cli`, we're heavily inspired by these projects.
</details>

[⬆ back to top](#toc)

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D

## Author

**vbuild** © [EGOIST](https://github.com/egoist), Released under the [MIT](./LICENSE) License.<br>
Authored and maintained by egoist with help from contributors ([list](https://github.com/egoist/vbuild/contributors)).

> [egoistian.com](https://egoistian.com) · GitHub [@egoist](https://github.com/egoist) · Twitter [@rem_rin_rin](https://twitter.com/rem_rin_rin)
