# 10up-toolkit

A collection of bundled scripts for 10up development.

1. [Introduction](#introduction)
2. [Authoring Projects](#projects)
3. [Linting](#linting)
4. [Authoring Libraries](#libraries)
5. [Customizations](#customizations)
6. [CLI Options](#cli)
8. [TypeScript Support](#typescript)

## <a id="introduction"></a>Introduction

10up-toolkit is 10up's official asset bundling tool based on Webpack 5. It comes with support for many things commonly
used across 10up's projects such as:
- JavaScript transpilation through babel
- core-js@3 automatic polyfill injection (project mode)
- PostCSS, SASS and CSS Modules
- ESLint and prettier
- Jest

With 10up-toolkit, engineers can quickly and easily bundle assets for both production and development without having
to worry about config files. 10up-toolkit is also easy to extend to project's specifics needs.

`10up-toolkit` is inspired in tools like `react-scripts`, `kcd-scripts` and `wp-scripts`.

### Installation

To install 10up-toolkit simply run 

```bash{showPrompt}
npm install --save-dev 10up-toolkit
```

### Setting it up

In order to get `10up-toolkit` up and running simply define the `source` and `main` properties in your `package.json` file.
You can also specify a `style` property to tell `10up-toolkit` where to output your compiled css.

```json
{
    "name": "your-package-name",
    "version": "1.0.0",
    "main": "./dist/index.js",
    "source": "./src/index.js",
    "style": "./dist/index.css",
    "scripts": {
        "build": "10up-toolkit build",
        "dev": "10up-toolkit build --watch"
    }
}
```

Then, the following code:
```javascript
// src/index.js
import './styles.css';

export default () => { /* my awesome js code */};
```

will generate a `index.js` and a `index.css` file in the `dist` folder after running `npm run build`.

10up-toolkit can run in two different modes: package mode and project mode:
- **Project Mode**: Allows bundling multiple entry points and automatically includes core-js polyfills. 
- **Package Mode**: Does not include core-js polyfills automatically, assumes one entry point and doesn't include dependencies in the bundle.

By default it will run in package mode (like in the example above) and it works 
well when you're building a package for distribution. 

## <a id="projects"></a>Authoring Projects

When running in **project mode** 10up-toolkit will automatically inject core-js polyfills and also allow for multiple entry points.

10up's [wp-scaffold](https://github.com/10up/wp-scaffold/blob/trunk/themes/10up-theme/package.json) is a good example of 10up-toolkit being used in project mode.

Here's how a `package.json` would look like for using 10up-toolkit this way:

```json
{
  "name": "tenup-theme",
  "version": "1.0.0",
  "scripts": {
    "start": "10up-toolkit build --watch",
    "build": "10up-toolkit build",
    "format-js": "10up-toolkit format-js",
    "lint-js": "10up-toolkit lint-js",
    "lint-style": "10up-toolkit lint-style",
    "test": "10up-toolkit test-unit-jest",
  },
  "devDependencies": {
    "10up-toolkit": "^1.0.0"
  },
  "dependencies": {
    "normalize.css": "^8.0.1",
    "prop-types": "^15.7.2"
  },
  "10up-toolkit": {
    "devURL": "https://my-project.test",
    "entry": {
      "admin": "./assets/js/admin/admin.js",
      "blocks": "./includes/blocks/blocks.js",
      "frontend": "./assets/js/frontend/frontend.js",
      "shared": "./assets/js/shared/shared.js",
      "styleguide": "./assets/js/styleguide/styleguide.js",
      "admin-style": "./assets/css/admin/admin-style.css",
      "editor-style": "./assets/css/frontend/editor-style.css",
      "shared-style": "./assets/css/shared/shared-style.css",
      "style": "./assets/css/frontend/style.css",
      "styleguide-style": "./assets/css/styleguide/styleguide.css",
      "core-block-overrides": "./includes/core-block-overrides.js",
      "example-block": "./includes/blocks/example-block/index.js"
    }
  }
}
```

Note the `10up-toolkit` object in `package.json`. It exposes a few options to configure 10up-toolkit behavior.
The most important is the `entry` option. It's an object where you can specify all of the entry points you need in your project. You can specify both JavaScript/TypeScript files or CSS/PostCSS/SASS files.

When you run `10up-toolkit build` with this configuration, 10up-toolkit will generate compiled assets for every entry point in the dist folder.

By default the compiled assets will be generated on the following diretories:
- `dist/css` - for all css files
- `dist/js` - for all js files.
- `dist/blocks` - for all WordPress Gutenberg blocks.
- `dist/[images,fonts,svg]` - all assets under `assets/images`, `asset/fonts` and `assets/svg` are copied, even those not referenced in code (this behavior is specific to building in project mode).

See the [Customizing build paths](#customize-build-paths) section for changing the structure of the dist folder.

### BrowserSync

10up-toolkit ships with [BrowserSync](https://browsersync.io/) and can be enabled by simply adding a devURL property.

```json
 "10up-toolkit": {
    "devURL": "https://my-project.test",
    "entry": {
        //...
    }
  }
```

In the example above, running `10up-toolkit start` or `10up-toolkit build --watch` will start 10up-toolkit in watch mode and start a browser sync session, proxying the *https://my-project.test* url.

### <a id="customize-build-paths"></a> Customizing build paths 

To change where assets are generated in the `dist` folder, you can create a `filenames.config.js` at the root of your project.

```javascript
// filenames.config.js
module.exports = {
	js: 'js/[name].js',
	jsChunk: 'js/[name].[contenthash].chunk.js',
	css: 'css/[name].css',
    // changing where gutenberg blocks assets are stored.
	block: 'js/blocks/[name]/editor.js',
	blockCSS: 'css/blocks/[name]/editor.css',
};
```

Alternatively you can specify the paths in `package.json`

```json
 "10up-toolkit": {
    "devURL": "https://my-project.test",
    "entry": {
        //...
    },
    "filenames": {
        "block": "js/blocks/[name]/editor.js",
	    "blockCSS": "css/blocks/[name]/editor.css",
    }
  }
```

Note that when overriding via the `filenames.config.js` you must export the filenames for all file types.

### WordPress Editor Styles

By default 10up-toolkit will scope any css file named `editor-styles.css` files with the 
`.editor-styles-wrapper` class. Take a look at the default [postcss config](https://github.com/10up/10up-toolkit/blob/develop/packages/toolkit/config/postcss.config.js#L21) for more information.

## <a id="linting"></a> Linting

10up-toolkit comes with eslint, prettier and stylelint set up out of the box. It uses [10up's eslint config](https://github.com/10up/10up-toolkit/tree/develop/packages/eslint-config) and exposes the following commands:
`10up-toolkit lint-js`,  `10up-toolkit format-js` and `10up-toolkit lint-style`.

10up-toolkit can lint JavaScript, TypeScript and JSX without any additional configuration. It's recommended to add a npm script to your `package.json`.

```json
{
    ...
    "scripts": {
        "build": "10up-toolkit build",
        "lint-js": "10up-toolkit lint-js",
        "format-js": "10up-toolkit format-js",
        "lint-style": "10up-toolkit lint-style"
    }
    ...
}
```

Then you can run `npm run lint-js` and `npm run format-js` to lint and format your codebase respectively.

### IDE Intregration
It's not required, but you might want to create a `.eslintrc.js` and `stylelint.config.js` file at the root of your project to better integrate eslint with your code editor or to extend the linting config.

```javascript
// .eslintrc.js
module.exports = {
	extends: ['@10up/eslint-config/wordpress'],
};
```

You can extend any of the [avaliable configs](https://github.com/10up/10up-toolkit/tree/develop/packages/eslint-config#available-eslint-configs) and enable/disable rules based on your project needs.

```javascript
// stylelint.config.js
const config = {
	extends: ['@10up/stylelint-config'],
};

module.exports = config;
```

### Choosing what to lint

10up-toolkit will lint/format any JavaScript or TypeScript file in your source code directory, you can customize this behavior in two ways:

- Specify the directory manually `10up-toolkit lint-js src/`
- Create a `.eslintignore`

```ignore
# Don't forget to exclude node_module and dist/build dirs
node_modules
build
dist

path-to-code-to-be-ignored/*

```

### Setting up Husky and Lint-Staged

We also strongly recommend that you set up `lint-staged` and `husky` to automate linting your code on every commit.

First, create a `.lintstagedrc.json` file
```json
{
    "*.css": [
      "10up-toolkit lint-style"
    ],
    "*.[tj]s": [
      "10up-toolkit lint-js"
    ],
    "*.[tj]sx": [
      "10up-toolkit lint-js"
    ],
    // If you have php and phpcs
    "*.php": [
      "./vendor/bin/phpcs --extensions=php --warning-severity=8 -s"
    ]
}
```

This will instruct lint-staged to run 10up-toolkit to link css, js and jsx files for the staged files in your commit.

To set up git hooks with husky, create a `.husky` dir and a bash script for the git hooks you want, for example to run lint-staged 
on `pre-commit`, create a `pre-commit` bash script with the following contents:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

Under the `.husky` dir, also add a `.gitgnore` file with the following contents:

```ignore
_
```

Then, finally add the husky and lint-staged deps to your project

```bash
npm install --save-dev husky lint-staged
```

Whenever you commit your code, husky will run lint-staged which will trigger the appropriate commands to lint the staged files and if any of the commands fails, your commit will be aborted.

### VSCode Integration

It's strongly recommended to enable VSCode settings to format your JavaScript code on save. To do so, make sure you have the [vscode-eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) extension and add the following settings to your vscode config:

```json
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true,
    },
    "[javascript]": {
        "editor.defaultFormatter": "dbaeumer.vscode-eslint"
    },
    "[typescript]": {
        "editor.defaultFormatter": "dbaeumer.vscode-eslint"
    },
```

10up's eslint config integrated with prettier through a eslint plugin so having the vscode prettier extension is not needed and in fact, it must be disabled to avoid conflicts when saving and formatting the code.

## <a id="libraries"></a>Authoring Libraries/Packages

10up-toolkit leverages standard package.json fields when running in package mode. Below is a list of all fields 10up-toolkit supports and how it's used.

- **name**: It's used to generate a valid library name to expose the library via a global variable (for UMD builds). 
We recommend always specifying `libraryName` manually when creating UMD builds via `--name=your-library-name` or `10up-toolkit.libraryName` config.
- **source**: It's used to define your package entry point.
- **main**: Where the bundled JavaScript should go. Note that this field is used by NPM to specify your package entry point for consumption.
- **style**: If set, will define where the CSS will be generated.
- **umd:main** or **unpkg**: 10up-toolkit generates a separate umd build by default. This field is used by 10up-toolkit to specify where the UMD bundle should go. The benefit of generating a separate UMD bundle is to avoid including boilerplate code.


```json
{
    "name": "your-package-name",
    "version": "1.0.0",
    "main": "./dist/index.js",
    "source": "./src/index.js",
    "style": "./dist/index.css",
    "umd:main": "./dist/index.umd.js",
    "scripts": {
        "build": "10up-toolkit build",
        "dev": "10up-toolkit build --watch"
    },
    "10up-toolkit": {
        "libraryName": "MyLibraryName"
    }
}
```

Then, the following code:
```javascript
// src/index.js
import './styles.css';

export default () => { /* my awesome js code */};
```

will generate a `index.js`, `index.umd.js` and a `index.css` file in the `dist` folder after running `npm run build`.

### Undertanding Package Mode

It's important to understand how 10up-toolkit behaves when running in package mode. First and foremost, core-js polyfills **will not** be added automatically.

If you package should support older browsers and you want to include core-js polyfills you will need to declare core-js as a dependency, and manually include the polyfills you need, e.g:

```javascript
import 'core-js/es/array/from';
import 'core-js/web/dom-collections';
```

The second difference is that 10up-toolkit **wil not** include the dependencies (or peer dependencies) in the final bundle. 
The reason for this is that, it's responsibility of the consumer bundle to resolve and include dependencies in the final bundle. Doing otherwise could lead to duplication of packages in the application final bundle.

This behavior is inspired in [how microbundle](https://github.com/developit/microbundle/wiki/How-Microbundle-decides-which-dependencies-to-bundle) handle bundling packages.

## <a id="customizations"></a>Customizations

10up-toolkit is very extensible and pretty much all config files can be overridden by simply creating a config file at the root of your project.

### Customizing the Webpack config
In general, we don't recommend customizing the webpack config, the default webpack config and the 10up-toolkits options should provide all that's needed
for most projects. However, in case you need to modify the webpack config you can to so by creating a `webpack.config.js` file at the root of your project.

The example below will update the webpack config so that 10up-toolkit processes and transpiles `@vendor/your-custom-package`. This would be required you publishing an untranspiled package.

```javascript
// webpack.config.js

const config = require('10up-toolkit/config/webpack.config.js');

config.module.rules[0].exclude =
	/node_modules\/(?!(@10up\/block-components)|(@vendor\/your-custom-package)\/).*/;

module.exports = defaultConfig;
```

### Customizing eslint and styling

To customize eslint, create a supported eslint config file at the root of your project. Make sure to extend the from `@10up/eslint-config` package.

If you're writing tests with jest for example, you will need to include the rules for jest.

```javascript
// .eslintrc.js

module.exports = {
	extends: ['@10up/eslint-config/wordpress', '@10up/eslint-config/jest'],
    rules: {
        /* add or modify rules here */
    }
};
```

Similarly, for customizing stylelint, create a `stylelint.config.js` file.

```javascript
// stylelint.config.js
const config = {
	extends: ['@10up/stylelint-config'],
    rules: {
        /* add or modify rules here */
    }
};

module.exports = config;
```

Refer to <Link to="/10up-toolkit/linting">linting docs</Link> for more information.

### Customizing PostCSS

To customize the postcss config, create a `postcss.config.js` at the root of your project. When overriding the postcss config, keep in mind
that the default config is exported as a **function**.

The example below modifies the ignored list of the `editor-styles` plugin when processing the `editor-styles.css` file.

```javascript
const baseConfig = require('10up-toolkit/config/postcss.config.js');
const path = require('path');

module.exports = (props) => {
	const config = baseConfig(props);
	const { file } = props;

	if (path.basename(file) === 'editor-style.css') {
		config.plugins['postcss-editor-styles'].ignore = [
			...config.plugins['postcss-editor-styles'].ignore,
			'.mh-personalization-segment-picker',
		];
	}
	return config;
};
```

## <a id="cli"></a> CLI Options

10up-toolkit supports several CLI options that can be used to override settings.

### Source and Output
To set the source and main/ouput path via the CLI you can use the `-i` and `-o` (or `--input` and `--output` options)

```bash
10up-toolkit build -i=src/index.js -o=dist/index.js
```

This can be useful if you have multiple entry points if you want to create a test application for your package.

```javascript
// app.js
// index is the entry point of the package
import { Accordion } from './index';

new Accordion('.accordion');
```

Then you can instruct 10up-toolkit to use your app.js file and spin up a dev server in a separate npm script.
```json
"start": "10up-toolkit start -i=src/app.js --dev-server",
```

### Dev Server

<blockquote>This option was added in 10up-toolkit 1.0.8</blockquote>

The `--dev-server` allows you to quickly spin up a development server. The default port is 8000 and can be changed via `--port`.

If you need to override the default html template, create a `index.html` file under a `public` dir.

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8"/>
    <title>Library Test</title>
  </head>
  <body>
        <!-- You can add any custom markup --->
  </body>
</html>
```

**Note**: You don't need to manually include the css and js in the html template, webpack will handle that for you.

### WP Mode

10up-toolkit is optimized for WordPress development and by default will include several WordPress specific settings. To disable this behavior
you can pass `--wp=false`.

### Format

The format option controls how webpack will generate your bundle. The supported options are:
- all (default)
- commonjs
- umd

The default value will generate both a commonjs bundle and a umd bundle.

To override, use the `-f` or `--format` option

```bash
10up-toolkit build -f=commonjs
```

### Externals

This option is only useful in package mode and is used to override the webpack externals definitions. In package mode, the 
default externals will be set based on your dependencies and peer dependencies.

```bash
10up-toolkit build --external=react,react-dom
```

An interesting use case for this is when you want to generate a UMD bundle that ships with all depedencies so that it
can be used by simply loading the JavaScript in the browser.

As an example, consider the following `package.json`

```json
{
  "name": "@10up/component-accordion",
  "version": "2.0.1",
  "author": "10up",
  "description": "Accessible accordion component.",
  "main": "dist/index.js",
  "umd:main": "dist/index.umd.js",
  "source": "src/index.js",
  "style": "./dist/index.css",
  "scripts": {
    "watch": "concurrently \"npm run build:modern -- --watch\" \"npm run build:umd -- --watch\"",
    "start": "10up-toolkit start -i=src/app.js --dev-server",
    "build": "npm run build:modern && npm run build:umd",
    "build:modern": "10up-toolkit build -f=commonjs",
    "build:umd": "10up-toolkit build -f=umd -i=src/index.umd.js --name=TenUpAccordion --external=none"
  },
  "depedencies": {
      "core-js": "^3.0.0",
  },
  "devDependencies": {
    "10up-toolkit": "1.0.7",
    "concurrently": "^5.3.0"
  },
}
```

Running `npm run build:modern` will only generate a bundle suitable for bundlers consumption and `npm run build:umd` will generate
a bundle that's suitable for both bundlers and direct inclusion in browsers, note that `---exernal=none` is being passed and that effectively tells
10up-toolkit to inline all of the dependencies. So someone loading `index.umd.js` don't need to load `core-js`.

The UMD bundle could then be used like so: 
```html
<script src="https://unpkg.com/@10up/component-accordion@2.0.1/dist/index.umd.js"></script>
<script type="text/javascript">
    const myAccordion = new window.TenUpAccordion.Accordion( '.accordion', {
        onCreate: function() {
            console.log( 'onCreated' )
        },
        onOpen: function() {
            console.log( 'onOpen' )
        },
        onClose: function() {
            console.log( 'onClose' )
        },
        onToggle: function() {
            console.log( 'onToggle' )
        }
    } );
</script>
```

## <a id="typescript"></a> TypeScript Support


10up-toolkit provides support for TypeScript out of the box. Simply create files with `.ts` or `.tsx` (for react components) and 10up-toolkit will
compile them just fine as well as lint and format them.

To enable better support for linting with VSCode and other IDE's we recommend the following `.eslintrc.js` file

```javascript
module.exports = {
	parser: '@typescript-eslint/parser',
	extends: ['@10up/eslint-config/react'], // or @10up/eslint-config/wordpress
	plugins: ['@typescript-eslint'],
};
```

## Support Level

**Active:** 10up is actively working on this, and we expect to continue work for the foreseeable future including keeping tested up to the most recent version of WordPress.  Bug reports, feature requests, questions, and pull requests are welcome.

## Like what you see?

<a href="http://10up.com/contact/"><img src="https://10up.com/uploads/2016/10/10up-Github-Banner.png" alt="Work with 10up, we create amazing websites and tools that make content management simple and fun using open source tools and platforms"></a>
