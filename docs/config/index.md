---
title: Configuring Frontron for React
---

# Configuring Frontron for React

If you want to use Frontron in your current project, you can use npm install.

```bash
npm install frontron
```

However, The `install` command needs some configuration in your package and directory.

The `install` command will prompt you to set some fields in your config.
There are a few rules to follow for the purposes of this tutorial:

* `entry point` should be `main.cjs`.
* `author` and `description` can be any value, but are necessary for app packaging(`description` is optional but recommended).

Your `package.json` file should look something like this:

```json
"name": "my-frontron-app",
"version": "0.0.1",
"main": "src/frontron/main.cjs",
"author": "your developer name",
```

> Note: If you're encountering any issues with installing frontron, please
> Report to frontron issues.

Finally, you need to be able to execute your Frontron app. In the `scripts`
field of your `package.json` config, add a `app` command like so:

```json
"scripts": {
    "app": "concurrently \"npm run dev\" \"wait-on http://localhost:3000 && cross-env NODE_ENV=development electron .\""
}
```

If your local development server is running on a different port, use port 3000.

This adjustment ensures that wait-on correctly waits for your development server to be ready before starting the Frontron application.

This `app` command will let you open your app in development mode.

```bash
npm run app
```

> Note: This script tells Electron to run on your project's root folder. At this stage,
> your app will immediately throw an error telling you that it cannot find an app to run.

### Run the main process

The entry point of any Frontron application is its `main` script. This script controls the
**main process**, which runs in a full Node.js environment and is responsible for
controlling your app's lifecycle, displaying native interfaces, performing privileged
operations, and managing renderer processes (more on that later).

During execution, Electron will look for this script in the `main`
field of the app's `package.json` config, which you should have configured during the
app scaffolding step.

To use `frontron` components in the `main` script, create an empty file named `main.cjs` in the `frontron` folder
of your project's src directory.

> Note: If you run the `start` script again at this point, your app will no longer throw
> any errors! However, it won't do anything yet because we haven't added any code into
> `main.cjs`.

### Opening your web page in a browser window

Now that you have a web page, load it into an application window. To do so, you'll
need a Frontron module:

* The `mainWindow` module, which controls your application's All events lifecycle.

Because the main process runs Node.js, you can import these as CommonJS
modules at the top of your `main.cjs` file:

```js
const { mainWindow } = require('frontron/react')
```

### Access Node.js from the renderer with a preload script

This is where attaching a **preload** script to your renderer comes in handy.
A preload script runs before the renderer process is loaded, and has access to both
renderer globals (e.g. `window` and `document`) and a Node.js environment.

Create a new script named `preload.js` as such:

```js
const { contextBridge } = require('frontron/preload')
```

## Package and distribute your application

The fastest way to distribute your newly created app is using
[Electron Builder](https://www.electron.build).

Add a description to your `package.json` file, otherwise npmbuild will fail. Blank description are not valid.

Add a build field to your package.json like this:.

```json
"build": {
  "appId": "my-frontron-app",
  "mac": {
    "icon": "public/icon.png"
  },
  "win": {
    "icon": "public/icon.png"
  },
  "productName": "my-frontron-app",
  "copyright": "Copyright © your developer name",
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true
  },
  "files": [
    "node_modules/**/*",
    "src/electron/**/*",
    "public/**/*",
    "dist/**/*"
  ],
  "directories": {
    "buildResources": "assets",
    "output": "dist_app"
  }
}
```

add a `build` command like so:

```json
"scripts": {
  "build": "vite build && electron-builder"
},
```

Electron Builder creates the `dist_app` folder where your package will be located:

```plain
// Example for Windows
dist_app/
├── dist_app/my-frontron-app Setup 0.0.1.exe
└── ...

// Example for macOS
dist_app/
├── dist_app/make/zip/darwin/x64/my-frontron-app-darwin-x64-1.0.0.zip
├── dist_app/my-electron-app-darwin-x64/my-frontron-app.app/Contents/MacOS/my-frontron-app
└── ...
```