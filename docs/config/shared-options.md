# Shared Options

Unless noted, the options in this section are applied to all dev, build, and preview.

## root

- **Type:** `string`
- **Default:** `process.cwd()`

Project root directory (where `index.html` is located). Can be an absolute path, or a path relative to the current working directory.

See [Project Root](/guide/#index-html-and-project-root) for more details.

## base

- **Type:** `string`
- **Default:** `/`
- **Related:** [`server.origin`](/config/server-options.md#server-origin)

Base public path when served in development or production. Valid values include:

- Absolute URL pathname, e.g. `/foo/`
- Full URL, e.g. `https://bar.com/foo/` (The origin part won't be used in development so the value is the same as `/foo/`)
- Empty string or `./` (for embedded deployment)

See [Public Base Path](/guide/build#public-base-path) for more details.

## mode

- **Type:** `string`
- **Default:** `'development'` for serve, `'production'` for build

Specifying this in config will override the default mode for **both serve and build**. This value can also be overridden via the command line `--mode` option.

See [Env Variables and Modes](/guide/env-and-mode) for more details.

## define

- **Type:** `Record<string, any>`

Define global constant replacements. Entries will be defined as globals during dev and statically replaced during build.

Vite uses [esbuild defines](https://esbuild.github.io/api/#define) to perform replacements, so value expressions must be a string that contains a JSON-serializable value (null, boolean, number, string, array, or object) or a single identifier. For non-string values, Vite will automatically convert it to a string with `JSON.stringify`.

**Example:**

```js
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify('v1.0.0'),
    __API_URL__: 'window.__backend_api_url',
  },
})
```

::: tip NOTE
For TypeScript users, make sure to add the type declarations in the `vite-env.d.ts` file to get type checks and Intellisense.

Example:

```ts
// vite-env.d.ts
declare const __APP_VERSION__: string
```

:::

## plugins

- **Type:** `(Plugin | Plugin[] | Promise<Plugin | Plugin[]>)[]`

Array of plugins to use. Falsy plugins are ignored and arrays of plugins are flattened. If a promise is returned, it would be resolved before running. See [Plugin API](/guide/api-plugin) for more details on Vite plugins.

## publicDir

- **Type:** `string | false`
- **Default:** `"public"`

Directory to serve as plain static assets. Files in this directory are served at `/` during dev and copied to the root of `outDir` during build, and are always served or copied as-is without transform. The value can be either an absolute file system path or a path relative to project root.

Defining `publicDir` as `false` disables this feature.

See [The `public` Directory](/guide/assets#the-public-directory) for more details.

## cacheDir

- **Type:** `string`
- **Default:** `"node_modules/.vite"`

Directory to save cache files. Files in this directory are pre-bundled deps or some other cache files generated by vite, which can improve the performance. You can use `--force` flag or manually delete the directory to regenerate the cache files. The value can be either an absolute file system path or a path relative to project root. Default to `.vite` when no package.json is detected.

## resolve.alias

- **Type:**
  `Record<string, string> | Array<{ find: string | RegExp, replacement: string, customResolver?: ResolverFunction | ResolverObject }>`

Will be passed to `@rollup/plugin-alias` as its [entries option](https://github.com/rollup/plugins/tree/master/packages/alias#entries). Can either be an object, or an array of `{ find, replacement, customResolver }` pairs.

When aliasing to file system paths, always use absolute paths. Relative alias values will be used as-is and will not be resolved into file system paths.

More advanced custom resolution can be achieved through [plugins](/guide/api-plugin).

::: warning Using with SSR
If you have configured aliases for [SSR externalized dependencies](/guide/ssr.md#ssr-externals), you may want to alias the actual `node_modules` packages. Both [Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias) and [pnpm](https://pnpm.io/aliases/) support aliasing via the `npm:` prefix.
:::

## resolve.dedupe

- **Type:** `string[]`

If you have duplicated copies of the same dependency in your app (likely due to hoisting or linked packages in monorepos), use this option to force Vite to always resolve listed dependencies to the same copy (from project root).

:::warning SSR + ESM
For SSR builds, deduplication does not work for ESM build outputs configured from `build.rollupOptions.output`. A workaround is to use CJS build outputs until ESM has better plugin support for module loading.
:::

## resolve.conditions <NonInheritBadge />

- **Type:** `string[]`
- **Default:** `['module', 'browser', 'development|production']` (`defaultClientConditions`)

Additional allowed conditions when resolving [Conditional Exports](https://nodejs.org/api/packages.html#packages_conditional_exports) from a package.

A package with conditional exports may have the following `exports` field in its `package.json`:

```json
{
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js"
    }
  }
}
```

Here, `import` and `require` are "conditions". Conditions can be nested and should be specified from most specific to least specific.

`development|production` is a special value that is replaced with `production` or `development` depending on the value of `process.env.NODE_ENV`. It is replaced with `production` when `process.env.NODE_ENV === 'production'` and `development` otherwise.

Note that `import`, `require`, `default` conditions are always applied if the requirements are met.

## resolve.mainFields <NonInheritBadge />

- **Type:** `string[]`
- **Default:** `['browser', 'module', 'jsnext:main', 'jsnext']` (`defaultClientMainFields`)

List of fields in `package.json` to try when resolving a package's entry point. Note this takes lower precedence than conditional exports resolved from the `exports` field: if an entry point is successfully resolved from `exports`, the main field will be ignored.

## resolve.extensions

- **Type:** `string[]`
- **Default:** `['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json']`

List of file extensions to try for imports that omit extensions. Note it is **NOT** recommended to omit extensions for custom import types (e.g. `.vue`) since it can interfere with IDE and type support.

## resolve.preserveSymlinks

- **Type:** `boolean`
- **Default:** `false`

Enabling this setting causes vite to determine file identity by the original file path (i.e. the path without following symlinks) instead of the real file path (i.e. the path after following symlinks).

- **Related:** [esbuild#preserve-symlinks](https://esbuild.github.io/api/#preserve-symlinks), [webpack#resolve.symlinks
  ](https://webpack.js.org/configuration/resolve/#resolvesymlinks)

## html.cspNonce

- **Type:** `string`
- **Related:** [Content Security Policy (CSP)](/guide/features#content-security-policy-csp)

A nonce value placeholder that will be used when generating script / style tags. Setting this value will also generate a meta tag with nonce value.

## css.modules

- **Type:**
  ```ts
  interface CSSModulesOptions {
    getJSON?: (
      cssFileName: string,
      json: Record<string, string>,
      outputFileName: string,
    ) => void
    scopeBehaviour?: 'global' | 'local'
    globalModulePaths?: RegExp[]
    exportGlobals?: boolean
    generateScopedName?:
      | string
      | ((name: string, filename: string, css: string) => string)
    hashPrefix?: string
    /**
     * default: undefined
     */
    localsConvention?:
      | 'camelCase'
      | 'camelCaseOnly'
      | 'dashes'
      | 'dashesOnly'
      | ((
          originalClassName: string,
          generatedClassName: string,
          inputFile: string,
        ) => string)
  }
  ```

Configure CSS modules behavior. The options are passed on to [postcss-modules](https://github.com/css-modules/postcss-modules).

This option doesn't have any effect when using [Lightning CSS](../guide/features.md#lightning-css). If enabled, [`css.lightningcss.cssModules`](https://lightningcss.dev/css-modules.html) should be used instead.

## css.postcss

- **Type:** `string | (postcss.ProcessOptions & { plugins?: postcss.AcceptedPlugin[] })`

Inline PostCSS config or a custom directory to search PostCSS config from (default is project root).

For inline PostCSS config, it expects the same format as `postcss.config.js`. But for `plugins` property, only [array format](https://github.com/postcss/postcss-load-config/blob/main/README.md#array) can be used.

The search is done using [postcss-load-config](https://github.com/postcss/postcss-load-config) and only the supported config file names are loaded. Config files outside the workspace root (or the [project root](/guide/#index-html-and-project-root) if no workspace is found) are not searched by default. You can specify a custom path outside of the root to load the specific config file instead if needed.

Note if an inline config is provided, Vite will not search for other PostCSS config sources.

## css.preprocessorOptions

- **Type:** `Record<string, object>`

Specify options to pass to CSS pre-processors. The file extensions are used as keys for the options. The supported options for each preprocessor can be found in their respective documentation:

- `sass`/`scss`:
  - Uses `sass-embedded` if installed, otherwise uses `sass`. For the best performance, it's recommended to install the `sass-embedded` package.
  - [Options](https://sass-lang.com/documentation/js-api/interfaces/stringoptions/)
- `less`: [Options](https://lesscss.org/usage/#less-options).
- `styl`/`stylus`: Only [`define`](https://stylus-lang.com/docs/js.html#define-name-node) is supported, which can be passed as an object.

**Example:**

```js
export default defineConfig({
  css: {
    preprocessorOptions: {
      less: {
        math: 'parens-division',
      },
      styl: {
        define: {
          $specialColor: new stylus.nodes.RGBA(51, 197, 255, 1),
        },
      },
      scss: {
        importers: [
          // ...
        ],
      },
    },
  },
})
```

### css.preprocessorOptions[extension].additionalData

- **Type:** `string | ((source: string, filename: string) => (string | { content: string; map?: SourceMap }))`

This option can be used to inject extra code for each style content. Note that if you include actual styles and not just variables, those styles will be duplicated in the final bundle.

**Example:**

```js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `$injectedColor: orange;`,
      },
    },
  },
})
```

## css.preprocessorMaxWorkers

- **Type:** `number | true`
- **Default:** `true`

Specifies the maximum number of threads CSS preprocessors can use. `true` means up to the number of CPUs minus 1. When set to `0`, Vite will not create any workers and will run the preprocessors in the main thread.

Depending on the preprocessor options, Vite may run the preprocessors on the main thread even if this option is not set to `0`.

## css.devSourcemap

- **Experimental:** [Give Feedback](https://github.com/vitejs/vite/discussions/13845)
- **Type:** `boolean`
- **Default:** `false`

Whether to enable sourcemaps during dev.

## css.transformer

- **Experimental:** [Give Feedback](https://github.com/vitejs/vite/discussions/13835)
- **Type:** `'postcss' | 'lightningcss'`
- **Default:** `'postcss'`

Selects the engine used for CSS processing. Check out [Lightning CSS](../guide/features.md#lightning-css) for more information.

::: info Duplicate `@import`s
Note that postcss (postcss-import) has a different behavior with duplicated `@import` from browsers. See [postcss/postcss-import#462](https://github.com/postcss/postcss-import/issues/462).
:::

## css.lightningcss

- **Experimental:** [Give Feedback](https://github.com/vitejs/vite/discussions/13835)
- **Type:**

```js
import type {
  CSSModulesConfig,
  Drafts,
  Features,
  NonStandard,
  PseudoClasses,
  Targets,
} from 'lightningcss'
```

```js
{
  targets?: Targets
  include?: Features
  exclude?: Features
  drafts?: Drafts
  nonStandard?: NonStandard
  pseudoClasses?: PseudoClasses
  unusedSymbols?: string[]
  cssModules?: CSSModulesConfig,
  // ...
}
```

Configures Lightning CSS. Full transform options can be found in [the Lightning CSS repo](https://github.com/parcel-bundler/lightningcss/blob/master/node/index.d.ts).

## json.namedExports

- **Type:** `boolean`
- **Default:** `true`

Whether to support named imports from `.json` files.

## json.stringify

- **Type:** `boolean | 'auto'`
- **Default:** `'auto'`

If set to `true`, imported JSON will be transformed into `export default JSON.parse("...")` which is significantly more performant than Object literals, especially when the JSON file is large.

If set to `'auto'`, the data will be stringified only if [the data is bigger than 10kB](https://v8.dev/blog/cost-of-javascript-2019#json:~:text=A%20good%20rule%20of%20thumb%20is%20to%20apply%20this%20technique%20for%20objects%20of%2010%20kB%20or%20larger).

## esbuild

- **Type:** `ESBuildOptions | false`

`ESBuildOptions` extends [esbuild's own transform options](https://esbuild.github.io/api/#transform). The most common use case is customizing JSX:

```js
export default defineConfig({
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment',
  },
})
```

By default, esbuild is applied to `ts`, `jsx` and `tsx` files. You can customize this with `esbuild.include` and `esbuild.exclude`, which can be a regex, a [picomatch](https://github.com/micromatch/picomatch#globbing-features) pattern, or an array of either.

In addition, you can also use `esbuild.jsxInject` to automatically inject JSX helper imports for every file transformed by esbuild:

```js
export default defineConfig({
  esbuild: {
    jsxInject: `import React from 'react'`,
  },
})
```

When [`build.minify`](./build-options.md#build-minify) is `true`, all minify optimizations are applied by default. To disable [certain aspects](https://esbuild.github.io/api/#minify) of it, set any of `esbuild.minifyIdentifiers`, `esbuild.minifySyntax`, or `esbuild.minifyWhitespace` options to `false`. Note the `esbuild.minify` option can't be used to override `build.minify`.

Set to `false` to disable esbuild transforms.

## assetsInclude

- **Type:** `string | RegExp | (string | RegExp)[]`
- **Related:** [Static Asset Handling](/guide/assets)

Specify additional [picomatch patterns](https://github.com/micromatch/picomatch#globbing-features) to be treated as static assets so that:

- They will be excluded from the plugin transform pipeline when referenced from HTML or directly requested over `fetch` or XHR.

- Importing them from JS will return their resolved URL string (this can be overwritten if you have a `enforce: 'pre'` plugin to handle the asset type differently).

The built-in asset type list can be found [here](https://github.com/vitejs/vite/blob/main/packages/vite/src/node/constants.ts).

**Example:**

```js
export default defineConfig({
  assetsInclude: ['**/*.gltf'],
})
```

## logLevel

- **Type:** `'info' | 'warn' | 'error' | 'silent'`

Adjust console output verbosity. Default is `'info'`.

## customLogger

- **Type:**
  ```ts
  interface Logger {
    info(msg: string, options?: LogOptions): void
    warn(msg: string, options?: LogOptions): void
    warnOnce(msg: string, options?: LogOptions): void
    error(msg: string, options?: LogErrorOptions): void
    clearScreen(type: LogType): void
    hasErrorLogged(error: Error | RollupError): boolean
    hasWarned: boolean
  }
  ```

Use a custom logger to log messages. You can use Vite's `createLogger` API to get the default logger and customize it to, for example, change the message or filter out certain warnings.

```ts twoslash
import { createLogger, defineConfig } from 'vite'

const logger = createLogger()
const loggerWarn = logger.warn

logger.warn = (msg, options) => {
  // Ignore empty CSS files warning
  if (msg.includes('vite:css') && msg.includes(' is empty')) return
  loggerWarn(msg, options)
}

export default defineConfig({
  customLogger: logger,
})
```

## clearScreen

- **Type:** `boolean`
- **Default:** `true`

Set to `false` to prevent Vite from clearing the terminal screen when logging certain messages. Via command line, use `--clearScreen false`.

## envDir

- **Type:** `string | false`
- **Default:** `root`

The directory from which `.env` files are loaded. Can be an absolute path, or a path relative to the project root. `false` will disable the `.env` file loading.

See [here](/guide/env-and-mode#env-files) for more about environment files.

## envPrefix

- **Type:** `string | string[]`
- **Default:** `VITE_`

Env variables starting with `envPrefix` will be exposed to your client source code via `import.meta.env`.

:::warning SECURITY NOTES
`envPrefix` should not be set as `''`, which will expose all your env variables and cause unexpected leaking of sensitive information. Vite will throw an error when detecting `''`.

If you would like to expose an unprefixed variable, you can use [define](#define) to expose it:

```js
define: {
  'import.meta.env.ENV_VARIABLE': JSON.stringify(process.env.ENV_VARIABLE)
}
```

:::

## appType

- **Type:** `'spa' | 'mpa' | 'custom'`
- **Default:** `'spa'`

Whether your application is a Single Page Application (SPA), a [Multi Page Application (MPA)](../guide/build#multi-page-app), or Custom Application (SSR and frameworks with custom HTML handling):

- `'spa'`: include HTML middlewares and use SPA fallback. Configure [sirv](https://github.com/lukeed/sirv) with `single: true` in preview
- `'mpa'`: include HTML middlewares
- `'custom'`: don't include HTML middlewares

Learn more in Vite's [SSR guide](/guide/ssr#vite-cli). Related: [`server.middlewareMode`](./server-options#server-middlewaremode).

## future

- **Type:** `Record<string, 'warn' | undefined>`
- **Related:** [Breaking Changes](/changes/)

Enable future breaking changes to prepare for a smooth migration to the next major version of Vite. The list may be updated, added, or removed at any time as new features are developed.

See the [Breaking Changes](/changes/) page for details of the possible options.
