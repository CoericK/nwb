## Configuration

nwb's default setup can get you developing, testing and building production-ready apps and npm-ready components out of the box without any configuration.

If you need to tweak the default setup to suit your project's needs, or you want to use some of the other features the Babel, Karma and Webpack ecosystems have to offer, you can provide a configuration file.

> You can also add new functionality by installing a [plugin module](/docs/Plugins.md#plugins).

### Configuration File

By default, nwb will look for an `nwb.config.js` file in the current working directory for configuration.

You can also specify a configuration file using the `--config` option:

```
nwb --config ./config/nwb.js
```

This file should export either a configuration object...

```js
module.exports = {
  // ...
}
```

...or a function which returns a configuration object when called:

```js
module.exports = function(args) {
  return {
    // ...
  }
}
```

If a function is exported, it will be passed an object with the following properties:

- `command`: the name of the command currently being executed, e.g. `'build'` or `'test'`
- `webpack`: nwb's version of the `webpack` module, giving you access to the other plugins webpack provides

#### Example Configuration Files

- [react-hn's `nwb.config.js`](https://github.com/insin/react-hn/blob/concat/nwb.config.js) is a simple configuration file with minor tweaks to Babel and Webpack config.
- [React Yelp Clone's `nwb.config.js`](https://github.com/insin/react-yelp-clone/blob/nwb/nwb.config.js) configures Babel, Karma and Webpack to allow nwb to be dropped into an existing app to handle its development tooling, [reducing the amount of `devDependencies` and configuration](https://github.com/insin/react-yelp-clone/compare/master...nwb) which need to be managed.

### Configuration Object

The configuration object can include the following properties:

- nwb Configuration
  - [`type`](#type-string-required-for-generic-build-commands)
- [Babel Configuration](#babel-configuration)
  - [`babel`](#babel-object)
  - [`babel.loose`](#loose-boolean) - enable loose mode for Babel plugins which support it
  - [`babel.plugins`](#plugins-array) - extra Babel plugins to be used
  - [`babel.presets`](#plugins-array) - extra Babel presets to be used
  - [`babel.runtime`](#runtime-string--boolean) - enable the `transform-runtime` plugin with different configurations
  - [`babel.stage`](#stage-number--false) - control which experimental and upcoming JavaScript features can be used
- [Webpack Configuration](#webpack-configuration)
  - [`webpack`](#webpack-object)
  - [`webpack.aliases`](#aliases-object) - rewrite certain import paths
  - [`webpack.autoprefixer`](#autoprefixer-string--object) - options for Autoprefixer
  - [`webpack.compat`](#compat-object) - enable Webpack compatibility tweaks for commonly-used modules
  - [`webpack.define`](#define-object) - options for `DefinePlugin`, for replacing certain expressions with values
  - [`webpack.extractText`](#extracttext-object) - options for `ExtractTextPlugin`
  - [`webpack.html`](#html-object) - options for `HtmlPlugin`
  - [`webpack.install`](#install-object) - options for `NpmInstallPlugin`
  - [`webpack.loaders`](#loaders-object) - tweak the configuration of the default Webpack loaders
    - [Default Loaders](#default-loaders)
  - [`webpack.postcss`](#postcss-array--object) - a custom list of PostCSS plugins
  - [`webpack.uglify`](#uglify-object) - options for Webpack's `UglifyJsPlugin`
  - [`webpack.vendorBundle`](#vendorbundle-boolean) - control creation of a separate vendor bundle
  - [`webpack.extra`](#extra-object) - an escape hatch for extra Webpack config, which will be merged into the generated config
- [Karma Configuration](#karma-configuration)
  - [`karma`](#karma-object)
  - [`karma.browsers`](#browsers-arraystring--plugin) - browsers tests are run in
  - [`karma.frameworks`](#frameworks-arraystring--plugin) - testing framework
  - [`karma.plugins`](#plugins-arrayplugin) - additional Karma plugins
  - [`karma.reporters`](#reporters-arraystring--plugin) - test results reporter
  - [`karma.testContext`](#testcontext-string) - point to a Webpack context module for your tests
  - [`karma.testDirs`](#testdirs-string--arraystring) - directories containing test code which should be ignored in code coverage
  - [`karma.testFiles`](#testfiles-string--arraystring) - files containing tests to be run
  - [`karma.extra`](#extra-object-1) - an escape hatch for extra Karma config, which will be merged into the generated config
- [npm Build Configuration](#npm-build-configuration)
  - [`npm`](#npm-object)
  - [`npm.jsNext`](#jsnext-boolean)
  - UMD build
    - [`npm.umd`](#umd-string--object) - enable a UMD build which exports a global variable
      - [`umd.global`](#global-string-required-for-umd-build)
      - [`umd.externals`](#externals-object)
    - [`package.json` fields](#packagejson-umd-banner-configuration)

#### `type`: `String` (required for generic build commands)

nwb uses this field to determine which type of project it's working with when generic build commands like `build` are used.

It must be one of:

- `'react-app'`
- `'react-component'`
- `'web-app'`
- `'web-module'`

### Babel Configuration

#### `babel`: `Object`

[Babel](https://babeljs.io/) configuration can be provided in a `babel` object, using the following properties.

For Webpack builds, any Babel config provided will be used to configure `babel-loader` - you can also provide additional configuration in [`webpack.loaders`](#loaders-object) if necessary.

##### `loose`: `Boolean`

Some Babel plugins have a [loose mode](http://www.2ality.com/2015/12/babel6-loose-mode.html) in which they output simpler, potentially faster code rather than following the semantics of the ES6 spec closely.

Loose mode also turns off some useful errors which are present in normal mode, so you might find it useful to develop with normal mode and use loose mode as a production optimisation.

e.g. to enable loose mode only for production builds:

```js
module.exports = {
  babel {
    loose: process.env.NODE_ENV === 'production'
  }
}
```

##### `plugins`: `Array`

Additional Babel plugins to use.

nwb commands are run in the current working directory, so if you need to configure additional Babel plugins or presets, you can install them locally, pass their names and let Babel import them for you.

e.g. to install and use the [babel-plugin-react-html-attrs](https://github.com/insin/babel-plugin-react-html-attrs#readme) plugin:

```
npm install babel-plugin-react-html-attrs
```
```js
module.exports = {
  babel: {
    plugins: ['react-html-attrs']
  }
}
```

##### `presets`: `Array`

Additional Babel presets to use.

##### `runtime`: `String | Boolean`

Babel's [runtime transform](https://babeljs.io/docs/plugins/transform-runtime/) does 3 things:

1. It *always* imports small helper modules from `babel-runtime` instead of duplicating **helpers** in every module which needs them.
2. By default, it imports a local **polyfill** for new ES6 builtins (`Promise`) and static methods (e.g. `Object.assign`) when they're used in your code.
3. By default, it imports the **regenerator** runtime required to use `async`/`await` when needed.

If you want to enable all of these, set `runtime` to `true` .

```js
module.exports = {
  babel: {
    runtime: true
  }
}
```

If you want to pick and choose, you can use `'helpers'`, `'polyfill'` or `'regenerator'`.

e.g. if you use `async`/`await` (which is a Stage 3 feature, so enabled by default) but you're already handling polyfilling of ES6 built-ins you need, use `'regenerator'`:

```js
module.exports = {
  babel: {
    runtime: 'regenerator'
  }
}
```

e.g. if you're polyfilling all the features your code needs globally and you just want to import `babel-runtime` helpers instead of duplicating them, use `'helpers'`:

```js
module.exports = {
  babel: {
    runtime: 'helpers'
  }
}
```

##### `stage`: `Number | false`

*(A Babel 6 equivalent of Babel 5's `stage` config)*

Controls which Babel preset will be used to enable use of experimental, proposed and upcoming JavaScript features in your code, grouped by the stage they're at in the TC39 process for proposing new JavaScript features:

| Stage | TC39 Category | Features |
| ----- | ------------- | -------- |
| [0](https://babeljs.io/docs/plugins/preset-stage-0) | Strawman, just an idea |`do {...}` expressions, `::` function bind operator |
| [1](https://babeljs.io/docs/plugins/preset-stage-1) | Proposal: this is worth working on | class properties, export extensions, `@decorator` syntax ( using the [Babel Legacy Decorator plugin](https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy)) |
| [2](https://babeljs.io/docs/plugins/preset-stage-2) | Draft: initial spec | object rest/spread syntax - **enabled by default** |
| [3](https://babeljs.io/docs/plugins/preset-stage-3) | Candidate: complete spec and initial browser implementations | trailing function commas, `async`/`await`, `**` exponentiation operator |

e.g. if you want to use decorators in your app, you should set `stage` to `1`:

```js
module.exports = {
  babel {
    stage: 1
  }
}
```

Stage 2 is enabled by default - to disable use of a stage preset entirely, set `stage` to `false`:

```js
module.exports = {
  babel {
    stage: false
  }
}
```

### Webpack Configuration

#### `webpack`: `Object`

[Webpack](https://webpack.github.io/) configuration can be provided in a `webpack` object, using the following properties:

##### `aliases`: `Object`

Configures [Webpack aliases](https://webpack.github.io/docs/resolving.html#aliasing), which allow you to control module resolution. Typically aliases are used to make it easier to import certain modules from within any depth of nested directories in an app.

e.g.:

```js
module.exports = {
  webpack: {
    aliases: {
      // Enable use of 'img/file.png' paths in JavaScript and
      // "~images/file.png" paths in stylesheets to require an image from
      // src/images from anywhere in the the app.
      'img': path.resolve('src/images'),
      // Enable use of require('src/path/to/module.js') for top-down imports
      // from anywhere in the app, to promote writing location-independent
      // code by avoiding ../ directory traversal.
      'src': path.resolve('src')
    }
  }
}
```

You should be careful to avoid creating aliases which conflict with the names of Node.js builtins or npm packages, as you will then be unable to import them.

##### `autoprefixer`: `String | Object`

Configures [Autoprefixer options](https://github.com/postcss/autoprefixer#options) for nwb's default PostCSS configuration.

If you just need to configure the range of browsers prefix addition/removal is based on (Autoprefixer's own default is `'> 1%, last 2 versions, Firefox ESR'`), you can use a String.

e.g. if you want to make sure Autoprefixer also adds or keeps prefixes required for iOS 8 devices:

```js
module.exports = {
  webpack: {
    autoprefixer: '> 1%, last 2 versions, Firefox ESR, ios >= 8'
  }
}
```

Use an Object if you need to set any of Autoprefixer's other options.

e.g. if you also want to disable removal of prefixes which aren't required for the configured range of browsers:

```js
module.exports = {
  webpack: {
    autoprefixer: {
      browsers: '> 1%, last 2 versions, Firefox ESR, ios >= 8',
      remove: false,
    }
  }
}
```

##### `compat`: `Object`

Certain libraries require specific configuration to play nicely with Webpack - nwb can take care of the details for you if you use a `compat` object to tell it when you're using them. The following libraries are supported:

###### `enzyme`: `Boolean`

Set to `true` for [Enzyme](http://airbnb.io/enzyme/) compatibility - this assumes you're using the latest version of React (v15).

###### `json-schema`: `Boolean`

Set to `true` to prevent a transitive [json-schema](https://github.com/kriszyp/json-schema) dependency from [breaking your Webpack build](https://github.com/kriszyp/json-schema/issues/59). Failure in this case manifests itself something like so:

```
Error: define cannot be used indirect

webpack:///(webpack)/buildin/amd-define.js
```

###### `moment`: `Object`

If you use [Moment.js](http://momentjs.com/) in a Webpack build, all the locales it supports will be imported by default and your build will be about 139KB larger than you were expecting!

Provide an object with a `locales` Array specifying language codes for the locales you want to load.

###### `sinon`: `Boolean`

Set to `true` for [Sinon.js](http://sinonjs.org/) 1.x compatibility.

---

Here's an example config showing the use of every `compat` setting:

```js
module.exports = {
  webpack: {
    compat: {
      enzyme: true,
      'json-schema': true,
      moment: {
        locales: ['de', 'en-gb', 'es', 'fr', 'it']
      },
      sinon: true
    }
  }
}
```

##### `define`: `Object`

By default, nwb will use Webpack's [`DefinePlugin`](https://webpack.github.io/docs/list-of-plugins.html#defineplugin) to replace all occurances of `process.env.NODE_ENV` with a string containing `NODE_ENV`'s current value.

You can configure a `define` object to add your own constant values.

e.g. to replace all occurrences of `__VERSION__` with a string containing your app's version from its `package.json`:

```js
module.exports = {
  webpack: {
    define: {
      __VERSION__: JSON.stringify(require('./package.json').version)
    }
  }
}
```

##### `extractText`: `Object`

Configures [options for `ExtractTextWebpackPlugin`](https://github.com/webpack/extract-text-webpack-plugin#readme).

This can be used to control whether or not CSS is extracted from all chunks in an app which uses code splitting, or only the initial chunk:

```js
module.exports = {
  webpack: {
    extractText: {
      allChunks: true
    }
  }
}
```

##### `html`: `Object`

Configures [options for `HtmlWebpackPlugin`](https://github.com/ampedandwired/html-webpack-plugin#readme).

For apps, nwb will look for a `src/index.html` template to inject `<link>` and `<script>` tags into for each CSS and JavaScript bundle generated by Webpack.

Use `template`config if you have an HTML file elsewhere you want to use:

```js
module.exports = {
  webpack: {
    html: {
      template: 'html/index.html'
    }
  }
}
```

If you don't have a template at `src/index.html` or specify one via `template`, nwb will fall back to using a basic template which has the following properties you can configure:

- `title` - contents for `<title>`

  > Default: the value of `name` from your app's `package.json`

- `mountId` - the `id` for the `<div>` provided for your app to mount itself into

  > Default: `'app'`

```js
module.exports = {
  webpack: {
    html: {
      mountId: 'root',
      title: 'Unimaginative documentation example'
    }
  }
}
```

Other `HtmlWebpackPlugin` options can also be used. e.g. if you have a `favicon.ico` in your `src/` directory, you can include it in the `index.html` generated when your app is built and have it copied to the output directory like so:

```js
module.exports = {
  webpack: {
    html: {
      favicon: 'src/favicon.ico'
    }
  }
}
```

##### `install`: `Object`

Configures [options for `NpmInstallPlugin`](https://github.com/ericclemmons/npm-install-webpack-plugin#usage), which will be used if you pass `--auto-install` flag to `nwb serve`.

##### `loaders`: `Object`

Each [Webpack loader](https://webpack.github.io/docs/loaders.html) configured by default has a unique id you can use to customise it.

To customise a loader, add a prop to the `loaders` object matching its id with a configuration object.

Refer to each loader's documentation (linked to for each [default loader](#default-loaders) documented below) for configuration options which can be set.

Generic loader options such as `include` and `exclude` can be configured alongside loader-specific query options - you can also use an explicit `query` object if necessary to separate this configuration.

e.g. to enable [CSS Modules][CSS Modules] for your app's CSS, the following loader configs are equivalent:

```js
module.exports = {
  webpack: {
    loaders: {
      css: {
        modules: true,
        localIdentName: '[hash:base64:5]'
      }
    }
  }
}
```
```js
module.exports = {
  webpack: {
    loaders: {
      css: {
        query: {
          modules: true,
          localIdentName: '[hash:base64:5]'
        }
      }
    }
  }
}
```

If a loader supports configuration via a top-level webpack configuration property, this can be provided as a `config` prop. This is primarily for loaders which can't be configured via query parameters as they have configuration which can't be serialised, such as instances of plugins.

e.g. to use the `nib` plugin with the [Stylus](http://learnboost.github.io/stylus/) preprocessor provided by [nwb-stylus](https://github.com/insin/nwb-stylus):

```js
var nib = require('nib')

{
  webpack: {
    loaders: {
      stylus: {
        config: {
          use: [nib()]
        }
      }
    }
  }
}
```

Alternatively, you can also add new properties directly to the top-level Webpack config using [`extra`](#extra-object)

###### Default Loaders

Default loaders configured by nwb and the ids it gives them are:

- `babel` - handles `.js` (and `.jsx`) files with [babel-loader][babel-loader]

  > Default config: `{exclude: /node_modules/}`

- `css-pipeline` - handles your app's own`.css` files by chaining together a number of loaders:

  > Default config: `{exclude: /node_modules/}`

  Chained loaders are:

  - `style` - (only when serving) applies styles using [style-loader][style-loader]
  - `css` - handles URLs, minification and CSS Modules using [css-loader][css-loader]
  - `postcss` - processes CSS with PostCSS plugins using [postcss-loader][postcss-loader]; by default, this is configured to automatically add vendor prefixes to CSS using [Autoprefixer][autoprefixer]

- `vendor-css-pipeline` - handles `.css` files required from `node_modules/`, with the same set of chained loaders as `css-pipeline` but with a `vendor-` prefix in their id.

  > Default config: `{include: /node_modules/}`

- `graphics` - handles `.gif` and `.png` files using using [url-loader][url-loader]

  > Default config: `{limit: 10240}`

- `jpeg` - handles `.jpeg` files using [file-loader][file-loader]

- `fonts` - handles `.otf`, `.svg`, `.ttf`, `.woff` and `.woff2` files using [url-loader][url-loader]

  > Default config: `{limit: 10240}`

- `eot` - handles `.eot` files using [file-loader][file-loader]

- `json` - handles `.json` files using [json-loader][json-loader]

##### `postcss`: `Array | Object`

By default, nwb configures the `postcss-loader` in each style pipeline to automatically add vendor prefixes to CSS rules.

Use `postcss` configuration to provide your own list of PostCSS plugins to be used for each pipeline, which will completely overwrite nwb's default configuration.

If you're *only* configuring PostCSS plugins for your app's own CSS, you can just provide a list:

```js
module.exports = {
  webpack: {
    postcss: [
      require('precss')(),
      require('autoprefixer')()
    ]
  }
}
```

Use an object if you're configuring other style pipelines. When using an object, PostCSS plugins for the default style pipeline (applied to your app's own CSS) must be configured using a `defaults` property:

```js
var autoprefixer = require('autoprefixer')
var precss = require('precss')
module.exports = {
  webpack: {
    postcss: {
      defaults: [
        precss(),
        autoprefixer()
      ],
      vendor: [
        autoprefixer({add: false})
      ]
    }
  }
}
```

Plugins for other style pipelines are configured using their prefix as a property name: `vendor` for anything imported out of `node_modules/`, `sass` if you're using the `nwb-sass` preprocessor plugin, etc.

Your app is responsible for managing its own PostCSS plugin dependencies - between the size of the PostCSS ecosystem and the number of different configuration options `postcss-loader` supports, PostCSS could do with its own equivalent of nwb to manage dependencies and configuration!

It's recommended to create instances of PostCSS plugins in your config, as opposed to passing a module, in case you ever need to make use of debug output (enabled by setting a `DEBUG` environment variable to `nwb`) to examine generated config.

##### `uglify`: `Object`

Configures [options for Webpack's `UglifyJsPlugin`](https://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin), which will be used when creating production builds.

Any additional options provided will be merged into nwb's defaults, which are:

```js
module.exports = {
  compress: {
    screw_ie8: true,
    warnings: false,
  },
  mangle: {
    screw_ie8: true,
  },
  output: {
    comments: false,
    screw_ie8: true,
  },
}
```

##### `vendorBundle`: `Boolean`

Setting this to `false` disables extraction of anything imported from `node_modules/` into a `vendor` bundle.

##### `extra`: `Object`

Extra configuration to be merged into the generated Webpack configuration using [webpack-merge](https://github.com/survivejs/webpack-merge#webpack-merge---merge-designed-for-webpack) - see the [Webpack configuration docs](https://webpack.github.io/docs/configuration.html) for the available fields.

Note that you *must* use Webpack's own config structure in this object - e.g. to add an extra loader which isn't managed by nwb's own `webpack.loaders` config, you would need to provide a list of loaders at `webpack.extra.module.loaders`.

```js
var path = require('path')

function(nwb) {
  return {
    type: 'react-app',
    webpack: {
      extra: {
        // Example of adding an extra loader which isn't managed by nwb,
        // assuming you've installed html-loader in your project.
        module: {
          loaders: [
            {test: /\.html$/, loader: 'html'}
          ]
        },
        // Example of adding an extra plugin which isn't managed by nwb
        plugins: [
          new nwb.webpack.optimize.MinChunkSizePlugin({
            minChunkSize: 1024
          })
        ]
      }
    }
  }
}
```

### Karma Configuration

nwb's default [Karma](http://karma-runner.github.io/) configuration uses the [Mocha](https://mochajs.org/) framework and reporter plugins for it, but you can configure your own preferences.

#### `karma`: `Object`

Karma configuration can be provided in a `karma` object, using the following properties:

##### `browsers`: `Array<String | Plugin>`

> Default: `['PhantomJS']`

A list of browsers to run tests in.

PhantomJS is the default as it's installed by default with nwb and should be able to run in any environment.

The launcher plugin for Chrome is also included, so if you want to run tests in Chrome, you can just name it:

```js
module.exports = {
  karma: {
    browsers: ['Chrome']
  }
}
```

For other browsers, you will also need to supply a plugin and manage that dependency yourself:

```js
module.exports = {
  karma: {
    browsers: ['Firefox'],
    plugins: [
      require('karma-firefox-launcher')
    ]
  }
}
```

nwb can also use the first browser defined in a launcher plugin if you pass it in `browsers`:

```js
module.exports = {
  karma: {
    browsers: [
      'Chrome',
      require('karma-firefox-launcher')
    ]
  }
}
```

##### `frameworks`: `Array<String | Plugin>`

> Default: `['mocha']`

Karma testing framework plugins.

You must provide the plugin for any custom framework you want to use and manage it as a dependency yourself.

e.g. if you're using a testing framework which produces [TAP](https://testanything.org/) output (such as [tape](https://github.com/substack/tape)). this is how you would use `frameworks` and `plugins` props to configure Karma:

```
npm install --save-dev karma-tap
```
```js
module.exports = {
  karma: {
    frameworks: ['tap'],
    plugins: [
      require('karma-tap')
    ]
  }
}
```

nwb can also determine the correct framework name given the plugin itself, so the following is functionally identical to the configuration above:

```js
module.exports = {
  karma: {
    frameworks: [
      require('karma-tap')
    ]
  }
}
```

If a plugin module provides multiple plugins, nwb will only infer the name of the first plugin it provides, so pass it using `plugins` instead and list all the frameworks you want to use, for clarity:

```js
module.exports = {
  karma: {
    frameworks: ['mocha', 'chai', 'chai-as-promised'],
    plugins: [
      require('karma-chai-plugins') // Provides chai, chai-as-promised, ...
    ]
  }
}
```

**Note:** If you're configuring frameworks and you want to use the Mocha framework plugin managed by nwb, just pass its name as in the above example.

##### `testContext`: `String`

Use this configuration to point to a [Webpack context module](/docs/Testing.md#using-a-test-context-module) for your tests if you need to run code prior to any tests being run, such as customising the assertion library you're using, or global before and after hooks.

If you provide a context module, it is responsible for including tests via Webpack's  `require.context()` - see the [example in the Testing docs](/docs/Testing.md#using-a-test-context-module).

If the default [`testFiles`](#testfiles-string--arraystring) config wouldn't have picked up your tests, you must also configure it so they can be excluded from code coverage.

##### `testDirs`: `String | Array<String>`

> Default: `['test', 'tests', '**/__tests__/']`

Globs for directories containing test code.

These are used to exclude testing utility code from code coverage.

##### `testFiles`: `String | Array<String>`

> Default: `['**/*.spec.js', '**/*.test.js', '**/*-test.js']`

Globs for files containing tests.

If [`testContext`](#testcontext-string) is not being used, this controls which files Karma will run tests from.

The defaults use file suffixes to identify tests and are not rooted to a particular directory, allowing you to place your tests wherever you want.

##### `plugins`: `Array<Plugin>`

A list of plugins to be loaded by Karma - this should be used in combination with [`browsers`](#browsers-arraystring--plugin), [`frameworks`](#frameworks-arraystring--plugin) and [`reporters`](#reporters-arraystring--plugin) config as necessary.

##### `reporters`: `Array<String | Plugin>`

> Default: `['mocha']`

Customising reporters follows the same principle as frameworks, just using the `reporters` prop instead.

For built-in reporters, or nwb's versfon of the Mocha reporter, just pass a name:

```js
module.exports = {
  karma: {
    reporters: ['progress']
  }
}
```

For custom reporters, install and provide the plugin:

```
npm install --save-dev karma-tape-reporter
```
```js
module.exports = {
  karma: {
    reporters: [
      require('karma-tape-reporter')
    ]
  }
}
```

##### `extra`: `Object`

Extra configuration to be merged into the generated Karma configuration using [webpack-merge](https://github.com/survivejs/webpack-merge#webpack-merge---merge-designed-for-webpack).

Note that you *must* use Karma's own config structure in this object.

e.g. to tweak the configuration of the default Mocha reporter:

```js
module.exports = {
  karma: {
    extra: {
      mochaReporter: {
        divider: '°º¤ø,¸¸,ø¤º°`°º¤ø,¸,ø¤°º¤ø,¸¸,ø¤º°`°º¤ø,¸',
        output: 'autowatch'
      }
    }
  }
}
```

### npm Build Configuration

#### `npm`: `Object`

By default, `nwb build` creates an ES5 build of your React component or vanilla JS module's code for publishing to npm. Additional npm build configuration is defined in a `npm` object, using the following fields:

##### `jsNext`: `Boolean`

Determines whether or not nwb will create an ES6 modules build for tree-shaking module bundlers when you run `nwb build` for a React component or web module.

```js
module.exports = {
  npm: {
    jsNext: true
  }
}
```

##### `umd`: `String | Object`

Configures creation of a UMD build when you run `nwb build` for a React component or web module.

If you just need to configure the global variable the UMD build will export, you can use a String:

```js
module.exports = {
  npm: {
    umd: 'MyLibrary'
  }
}
```

If you also have some external dependencies to configure, you must use an Object containing the following properties:

###### `global`: `String` (*required* for UMD build)

The name of the global variable the UMD build will export.

###### `externals`: `Object`

A mapping from `peerDependency` module names to the global variables they're expected to be available as for use by the UMD build.

e.g. if you're creating a React component which also depends on [React Router](https://github.com/rackt/react-router), this configuration would ensure they're not included in the UMD build:

```js
module.exports = {
  build: {
    umd: true,
    global: 'MyComponent',
    externals: {
      'react': 'React',
      'react-router': 'ReactRouter'
    }
  }
}
```

#### `package.json` UMD Banner Configuration

A banner comment added to UMD builds will use as many of the following `package.json` fields as are present:

- `name`
- `version`
- `homepage`
- `license`

If all fields are present the banner will be in this format:

```js
/*!
 * nwb 0.6.0 - https://github.com/insin/nwb
 * MIT Licensed
 */
```

[autoprefixer]: https://github.com/postcss/autoprefixer/
[babel-loader]: https://github.com/babel/babel-loader/
[CSS Modules]: https://github.com/css-modules/css-modules/
[css-loader]: https://github.com/webpack/css-loader/
[file-loader]: https://github.com/webpack/file-loader/
[isparta-loader]: https://github.com/deepsweet/isparta-loader/
[json-loader]: https://github.com/webpack/json-loader/
[npm-install-loader]: https://github.com/ericclemmons/npm-install-loader/
[postcss-loader]: https://github.com/postcss/postcss-loader/
[style-loader]: https://github.com/webpack/style-loader/
[url-loader]: https://github.com/webpack/url-loader/
