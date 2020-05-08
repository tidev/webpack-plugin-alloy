# @titanium-sdk/webpack-plugin-alloy

> Titanium Alloy Plugin for Appcd Webpack

## Installation

To install this plugin in an existing project, run the following command in your project root:

```sh
npm i @titanium-sdk/webpack-plugin-alloy alloy alloy-compiler
```

Be sure to follow the migration steps below when enabling Webpack in an existing Titanium Classic project, as well as the general [migration guideline](https://github.com/appcelerator/appcd-plugin-webpack/blob/develop/migration.md).

## Migration

### Remove old Alloy plugin

Since Webpack now compiles your Alloy app the default Alloy plugin is not required anymore. You can safely delete `plugins/ti.alloy` and remove it from the `plugins` section of your `tiapp.xml`.

### Create required folders

Webpack requires that your project includes all possible Alloy sub-folders, even if they are empty. This is due to the way Webpack resolves dynamic requires and it needs to scan folders like `app/controllers` or `app/widgets`. Make sure your project structure looks like this and create folders if neccessary:

```txt
app/
├── assets/
├── controllers/
├── i18n/
├── lib/
├── models/
├── platform/
├── styles/
├── views/
├── widgets/
├── alloy.js
└── config.json
```

### No automatic processing of assets/lib/vendor folders

When you are migrating from an Alloy app without Webpack, you are probably used to the fact all content from the following directories is copied to your app:

- `app/assets`
- `app/lib`
- `app/vendor`

This is only true for `app/assets` by default when you are using Webpack. All files from this directory will be copied directly into your app as-is. Source files in `app/lib` and `app/vendor` **need** to be `require`/`import`'ed to be bundled by Webpack so they are included in your app.

Usage of the `app/vendor` directory is discuraged with Webpack. It is recommended to install all your third-party libraries as Node modules and let Webpack process them from there.

### Code changes

In addition to the changes described in the [general guidelines](https://github.com/appcelerator/appcd-plugin-webpack/blob/develop/migration.md), there are a couple of Alloy specific changes that your need to apply to your project.

#### Replace `WPATH` with `@widget`

Requires in widgets need to use `@widget` instead of `WPATH`

```js
// without webpack
require(WPATH('utils'))

// with webpack
require('~widget/utils')
```

#### Use ES6 `export` in Models

Models **need** to use ES6 `export`. To migrate, symply change `exports.definition =` to `export const definition =`.

```js
// without webpack
exports.definition = {
  config: {
    // ...
  }
}

// with webpack
export const definition = {
  config: {
    // ...
  }
}
```

### Notes

A few use cases from the original Alloy build are not supported yet when using Webpack. There are also some gotchas when you are coming from a legacy Alloy project that you need to be aware of when migrating your Alloy app to Webpack.

- Place **all** your JS source code that needs to be bundled by Webpack in `app/lib`. Remember that Webpack will only bundle your JS files when your actually `require`/ `import` them. They will **not** automatically be copied into the app.
- Only the `app/assets` folder will be copied to your app directly. The same applies to widget's `assets` directory.
- JavaScript files in `app/assets` will be copied as-is and will not be transpiled via Babel. Keep this in mind when you are trying to use them in a WebView, for example.
- Source files in `app/lib` and `app/vendor` **need** to be `require`/`import`'ed to be bundled by Webpack so they are included in your app.
- Usage of the `app/vendor` directory is discuraged with Webpack. Consider NPM packages to use third-party dependencies with Webpack, or move existing source code to `app/lib` if you can't rely on NPM packages for a specific dependency.

### Known limitations

- No support for Alloy JS makefiles (JMK).
- No support for `DefaultIcon.png` from themes.
- Views always need to have a matching file in `controllers`. The controller file can be empty, but it needs to be there for Webpack to properly discover the component.

## Webpack configuration

This plugin will add/modify the following Webpack options:

### Resolve

- Aliases
  - `@`: `./src`

### Plugins

- `plugin('copy-assets')` (copy files from `./src/assets` into `./Resources`)
