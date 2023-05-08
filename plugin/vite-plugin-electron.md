# vite-plugin-electron

---

Run your vite program in Electron.

## Features

- 🚀 Fully compatible with Vite and Vite's ecosystem <sub><sup>(Based on Vite)</sup></sub>
- 🎭 Flexible configuration
- 🐣 Few APIs, easy to use
- 🔥 Hot restart

## Quick Setup

1. Add the following dependency to your project

```sh
npm i -D vite-plugin-electron
```

2. Add `vite-plugin-electron` to the `plugins` section of `vite.config.ts`

```js
import electron from 'vite-plugin-electron'

export default {
  plugins: [
    electron({
      entry: 'electron/main.ts',
    }),
  ],
}
```

3. Create the `electron/main.ts` file and type the following code

```js
import { app, BrowserWindow } from 'electron'

app.whenReady().then(() => {
  const win = new BrowserWindow({
    title: 'Main window',
  })

  // You can use `process.env.VITE_DEV_SERVER_URL` when the vite command is called `serve`
  if (process.env.VITE_DEV_SERVER_URL) {
    win.loadURL(process.env.VITE_DEV_SERVER_URL)
  } else {
    // Load your file
    win.loadFile('dist/index.html');
  }
})
```

4. Add the `main` entry to `package.json`

```diff
{
+ "main": "dist-electron/main.js"
}
```

That's it! You can now use Electron in your Vite app ✨

## API <sub><sup>(Define)</sup></sub>

`electron(config: Configuration | Configuration[])`

```ts
export interface Configuration {
  /**
   * Shortcut of `build.lib.entry`
   */
  entry?: import('vite').LibraryOptions['entry']
  vite?: import('vite').InlineConfig
  /**
   * Triggered when Vite is built every time -- `vite serve` command only.
   * 
   * If this `onstart` is passed, Electron App will not start automatically.  
   * However, you can start Electroo App via `startup` function.  
   */
  onstart?: (this: import('rollup').PluginContext, options: {
    /**
     * Electron App startup function.  
     * It will mount the Electron App child-process to `process.electronApp`.  
     * @param argv default value `['.', '--no-sandbox']`
     */
    startup: (argv?: string[]) => Promise<void>
    /** Reload Electron-Renderer */
    reload: () => void
  }) => void | Promise<void>
}
```

## Recommend Structure

Let's use the official [template-vanilla-ts](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-vanilla-ts) created based on `create vite` as an example.

```diff
+ ├─┬ electron
+ │ └── main.ts
  ├─┬ src
  │ ├── main.ts
  │ ├── style.css
  │ └── vite-env.d.ts
  ├── .gitignore
  ├── favicon.svg
  ├── index.html
  ├── package.json
  ├── tsconfig.json
+ └── vite.config.ts
```

## [Examples](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples)

- [quick-start](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/quick-start)
- [multiple-renderer](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/multiple-renderer)
- [bytecode](https://github.com/electron-vite/vite-plugin-electron/tree/main/examples/bytecode)

## JavaScript API

`vite-plugin-electron`'s JavaScript APIs are fully typed, and it's recommended to use TypeScript or enable JS type checking in VS Code to leverage the intellisense and validation.

- `Configuration` - type
- `defineConfig` - function
- `resolveViteConfig` - function, Resolve the default Vite's `InlineConfig` for build Electron-Main
- `withExternalBuiltins` - function
- `build` - function
- `startup` - function

**Example**

```js
import { build, startup } from 'vite-plugin-electron'

const isDev = process.env.NODE_ENV === 'development'
const isProd = process.env.NODE_ENV === 'production'

build({
  entry: 'electron/main.ts',
  vite: {
    mode: process.env.NODE_ENV,
    build: {
      minify: isProd,
      watch: isDev ? {} : null,
    },
    plugins: [{
      name: 'plugin-start-electron',
      closeBundle() {
        if (isDev) {
          // Startup Electron App
          startup()
        }
      },
    }],
  },
})
```

## How to work

It just executes the `electron .` command in the Vite build completion hook and then starts or restarts the Electron App.

## Be aware

- 🚨 By default, the files in `electron` folder will be built into the `dist-electron`
- 🚨 Currently, `"type": "module"` is not supported in Electron
- 🚨 In general, Vite may not correctly build Node.js packages, especially C/C++ native modules, but Vite can load them as external packages. So, put your Node.js package in `dependencies`. Unless you know how to properly build them with Vite.

  ```js
  electron({
    entry: 'electron/main.ts',
    vite: {
      build: {
        rollupOptions: {
          // Here are some C/C++ plugins that can't be built properly.
          external: [
            'serialport',
            'sqlite3',
          ],
        },
      },
    },
  }),
  ```

## Next

If you need to use `node.js` in the rendering process, then you need to install [vite plugin electron renderer](/plugin/vite-plugin-electron-renderer.html).
  