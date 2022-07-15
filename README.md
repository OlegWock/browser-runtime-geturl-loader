# browser-runtime-geturl-loader

[![npm][npm]][npm-url]

This is a fork of [file-loader](https://github.com/webpack-contrib/file-loader) which was optimized for use in browser extensions.

The `browser-runtime-geturl-loader` resolves `import`/`require()` on a file into an absolute url (using `browser.runtime.getURL`) and emits the file into the output directory.

All options are same as in [file-loader](https://github.com/webpack-contrib/file-loader).

## Getting Started

To begin, you'll need to install `browser-runtime-geturl-loader`:

```console
$ npm install browser-runtime-geturl-loader --save-dev
```

Import (or `require`) the target file(s) in one of the bundle's files:

**file.js**

```js
import img from './file.png';
```

Then add the loader to your `webpack` config. For example:

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                    },
                ],
            },
        ],
    },
};
```

And run `webpack` via your preferred method. This will emit `file.png` as a file
in the output directory (with the specified naming convention, if options are
specified to do so) and returns the public URI of the file.

> ℹ️ By default the filename of the resulting file is the hash of the file's contents with the original extension of the required resource.

## Options

### `name`

Type: `String|Function`
Default: `'[contenthash].[ext]'`

Specifies a custom filename template for the target file(s) using the query
parameter `name`. For example, to emit a file from your `context` directory into
the output directory retaining the full directory structure, you might use:

#### `String`

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    name: '[path][name].[ext]',
                },
            },
        ],
    },
};
```

#### `Function`

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    name(resourcePath, resourceQuery) {
                        // `resourcePath` - `/absolute/path/to/file.js`
                        // `resourceQuery` - `?foo=bar`

                        if (process.env.NODE_ENV === 'development') {
                            return '[path][name].[ext]';
                        }

                        return '[contenthash].[ext]';
                    },
                },
            },
        ],
    },
};
```

> ℹ️ By default the path and name you specify will output the file in that same directory, and will also use the same URI path to access the file.

### `outputPath`

Type: `String|Function`
Default: `undefined`

Specify a filesystem path where the target file(s) will be placed.

#### `String`

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    outputPath: 'images',
                },
            },
        ],
    },
};
```

#### `Function`

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    outputPath: (url, resourcePath, context) => {
                        // `resourcePath` is original absolute path to asset
                        // `context` is directory where stored asset (`rootContext`) or `context` option

                        // To get relative path you can use
                        // const relativePath = path.relative(context, resourcePath);

                        if (/my-custom-image\.png/.test(resourcePath)) {
                            return `other_output_path/${url}`;
                        }

                        if (/images/.test(context)) {
                            return `image_output_path/${url}`;
                        }

                        return `output_path/${url}`;
                    },
                },
            },
        ],
    },
};
```

### `publicPath`

Type: `String|Function`
Default: [`__webpack_public_path__`](https://webpack.js.org/api/module-variables/#__webpack_public_path__-webpack-specific-)+outputPath

Specifies a custom public path for the target file(s).

#### `String`

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    publicPath: 'assets',
                },
            },
        ],
    },
};
```

#### `Function`

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    publicPath: (url, resourcePath, context) => {
                        // `resourcePath` is original absolute path to asset
                        // `context` is directory where stored asset (`rootContext`) or `context` option

                        // To get relative path you can use
                        // const relativePath = path.relative(context, resourcePath);

                        if (/my-custom-image\.png/.test(resourcePath)) {
                            return `other_public_path/${url}`;
                        }

                        if (/images/.test(context)) {
                            return `image_output_path/${url}`;
                        }

                        return `public_path/${url}`;
                    },
                },
            },
        ],
    },
};
```

### `postTransformPublicPath`

Type: `Function`
Default: `undefined`

Specifies a custom function to post-process the generated public path. This can be used to prepend or append dynamic global variables that are only available at runtime, like `__webpack_public_path__`. This would not be possible with just `publicPath`, since it stringifies the values.

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpg|gif)$/i,
                loader: 'browser-runtime-geturl-loader',
                options: {
                    publicPath: '/some/path/',
                    postTransformPublicPath: (p) =>
                        `__webpack_public_path__ + ${p}`,
                },
            },
        ],
    },
};
```

### `context`

Type: `String`
Default: [`context`](https://webpack.js.org/configuration/entry-context/#context)

Specifies a custom file context.

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            context: 'project',
                        },
                    },
                ],
            },
        ],
    },
};
```

### `emitFile`

Type: `Boolean`
Default: `true`

If true, emits a file (writes a file to the filesystem). If false, the loader
will return a public URI but **will not** emit the file. It is often useful to
disable this option for server-side packages.

**file.js**

```js
// bundle file
import img from './file.png';
```

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            emitFile: false,
                        },
                    },
                ],
            },
        ],
    },
};
```

### `regExp`

Type: `RegExp`
Default: `undefined`

Specifies a Regular Expression to one or many parts of the target file path.
The capture groups can be reused in the `name` property using `[N]`
[placeholder](https://github.com/webpack-contrib/file-loader#placeholders).

**file.js**

```js
import img from './customer01/file.png';
```

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            regExp: /\/([a-z0-9]+)\/[a-z0-9]+\.png$/i,
                            name: '[1]-[name].[ext]',
                        },
                    },
                ],
            },
        ],
    },
};
```

> ℹ️ If `[0]` is used, it will be replaced by the entire tested string, whereas `[1]` will contain the first capturing parenthesis of your regex and so on...

### `esModule`

Type: `Boolean`
Default: `true`

By default, `browser-runtime-geturl-loader` generates JS modules that use the ES modules syntax.
There are some cases in which using ES modules is beneficial, like in the case of [module concatenation](https://webpack.js.org/plugins/module-concatenation-plugin/) and [tree shaking](https://webpack.js.org/guides/tree-shaking/).

You can enable a CommonJS module syntax using:

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            esModule: false,
                        },
                    },
                ],
            },
        ],
    },
};
```

## Placeholders

Full information about placeholders you can find [here](https://github.com/webpack/loader-utils#interpolatename).

### `[ext]`

Type: `String`
Default: `file.extname`

The file extension of the target file/resource.

### `[name]`

Type: `String`
Default: `file.basename`

The basename of the file/resource.

### `[path]`

Type: `String`
Default: `file.directory`

The path of the resource relative to the webpack/config `context`.

### `[folder]`

Type: `String`
Default: `file.folder`

The folder of the resource is in.

### `[query]`

Type: `String`
Default: `file.query`

The query of the resource, i.e. `?foo=bar`.

### `[emoji]`

Type: `String`
Default: `undefined`

A random emoji representation of `content`.

### `[emoji:<length>]`

Type: `String`
Default: `undefined`

Same as above, but with a customizable number of emojis

### `[hash]`

Type: `String`
Default: `md4`

Specifies the hash method to use for hashing the file content.

### `[contenthash]`

Type: `String`
Default: `md4`

Specifies the hash method to use for hashing the file content.

### `[<hashType>:hash:<digestType>:<length>]`

Type: `String`

The hash of options.content (Buffer) (by default it's the hex digest of the hash).

#### `digestType`

Type: `String`
Default: `'hex'`

The [digest](https://en.wikipedia.org/wiki/Cryptographic_hash_function) that the
hash function should use. Valid values include: base26, base32, base36,
base49, base52, base58, base62, base64, and hex.

#### `hashType`

Type: `String`
Default: `'md4'`

The type of hash that the hash function should use. Valid values include: `md4`, `md5`, `sha1`, `sha256`, and `sha512`.

#### `length`

Type: `Number`
Default: `undefined`

Users may also specify a length for the computed hash.

### `[N]`

Type: `String`
Default: `undefined`

The n-th match obtained from matching the current file name against the `regExp`.

## Examples

### Names

The following examples show how one might use `browser-runtime-geturl-loader` and what the result would be.

**file.js**

```js
import png from './image.png';
```

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            name: 'dirname/[contenthash].[ext]',
                        },
                    },
                ],
            },
        ],
    },
};
```

Result:

```bash
# result
dirname/0dcbbaa701328ae351f.png
```

---

**file.js**

```js
import png from './image.png';
```

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            name: '[sha512:hash:base64:7].[ext]',
                        },
                    },
                ],
            },
        ],
    },
};
```

Result:

```bash
# result
gdyb21L.png
```

---

**file.js**

```js
import png from './path/to/file.png';
```

**webpack.config.js**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|gif)$/i,
                use: [
                    {
                        loader: 'browser-runtime-geturl-loader',
                        options: {
                            name: '[path][name].[ext]?[contenthash]',
                        },
                    },
                ],
            },
        ],
    },
};
```

Result:

```bash
# result
path/to/file.png?e43b20c069c4a01867c31e98cbce33c9
```

## License

[MIT](./LICENSE)

[npm]: https://img.shields.io/npm/v/browser-runtime-geturl-loader.svg
[npm-url]: https://npmjs.com/package/browser-runtime-geturl-loader
