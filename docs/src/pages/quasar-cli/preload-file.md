---
title: Preload file
desc: Managing your startup code in a Quasar app.
---

A common use case for Quasar applications is to **run code before the root Vue app instance is instantiated**, like injecting and initializing your own dependencies (examples: Vue components, libraries...) or simply configuring some startup code of your app.

Since you won't be having access to any `/main.js` file (so that Quasar CLI can seamlessly initialize and build the same codebase for SPA/PWA/SSR/Cordova/Electron) Quasar provides an elegant solution to that problem by allowing users to define so-called boot files.  See the [Boot files](/quasar-cli/boot-files) documentation.

Boot files work after the store and router have been setup which makes them problematic if there are requirements to execute code prior to creation of either of those two resources.  That is where the preload file comes into play allowing the app developer to run generic code at the very startup of the application.  Keep in mind the Quasar concept of encouraging developers to write maintainable and elegant cross-platform applications; the preload function should not be a cluttered and hard to maintain collection of code  and thus should be focused and only used on an as needed basis if the boot files do not suffice.  

## Anatomy of a preload file

A preload file is a simple JavaScript file which exports an async function. Quasar will then call the exported function when it boots the application.  There are no properties passed to this function.

```js
export default async () => {
  // something to do
}
```

## When to use boot files
::: warning
Please make sure you understand what problem boot files solve and when it is appropriate to use them, to avoid applying them in cases where they are not needed.  Only if the problem is not addressed by the boot files, look to tuse the preload file.
:::

Preload file fulfill one special purpose: they run code **before** the App's store, router *and* Vue root component is instantiated.  It cannot be used to interfere with Vue Router, inject Vue prototype or inject the root instance of the Vue app.  However, it can be used to preload a library needed by the store, router or Vue root component creation.

### Example of unneeded usage of the preload file
* For plain JavaScript libraries like Lodash, which don't need any initialization prior to their usage.

## Usage of preload file
The first step is always to generate a new preload file using Quasar CLI:

```bash
$ quasar new preload preload [--format ts]
```

This command creates a new file: `/src/boot/preload.js` with the following content:

```js
// import something here

// "async" is optional!
// remove it if you don't need it
export default async () => {
  // something to do
}
```

You can also return a Promise:

```js
// import something here

export default () => {
  return new Promise((resolve, reject) => {
    // do something
  })
}
```

### Quasar App Flow
In order to better understand how the preload file works and what it does, you need to understand how your website/app boots:

1. Quasar is initialized (components, directives, plugins, Quasar i18n, Quasar icon sets)
2. Quasar Extras get imported (Roboto font -- if used, icons, animations, ...)
3. Quasar CSS & your app's global CSS are imported
4. Preload file is import; if a function is loaded successfully, the function is executed
5. App.vue is loaded (not yet being used)
6. Store is imported (if using Vuex Store in src/store)
7. Router is imported (in src/router)
8. Boot files are imported
9. Router default export function executed
10. Boot files get their default export function executed
11. (if on Electron mode) Electron is imported and injected into Vue prototype
12. (if on Cordova mode) Listening for "deviceready" event and only then continuing with following steps
13. Instantiating Vue with root component and attaching to DOM

Further reading on syntax: [ES6 import](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/import), [ES6 export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export).
