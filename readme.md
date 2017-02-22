
### 1. webpack.DllPlugin

```js
 "scripts": {
    "clean": "rimraf dist",//rimraf , oppsite of node-mkdirp
    "build:dll": "webpack --config webpack.dll.js",
    "watch": "npm run build:dll && webpack --config webpack.dev.js --watch --progress"
    //in watch mode, we will always build with webpack.dll.js to generate a new vendor.manifest.json for webpack DllReferencePlugin!
  }
```

webpack.dll.js as follows:

```js
var path = require("path");
var webpack = require("webpack");
module.exports = {
    entry: {
        vendor: [path.join(__dirname, "client", "vendors.js")]
        //vendors.js is where for vendor files, it is also an entry file
    },
    output: {
        path: path.join(__dirname, "dist", "dll"),
        filename: "dll.[name].js",
        library: "[name]"
        //Combine this plugins with output.library option to expose the dll function i. e. into the global scope.
    },
    //output file is in dist/dll folder
    plugins: [
        new webpack.DllPlugin({
            path: path.join(__dirname, "dll", "[name]-manifest.json"),
            //The manifest is very important, it gives other Webpack configurations 
            //a map to your already built modules. Path represent where to generate manifest file
            name: "[name]",
            // the name is the name of the entry
            context: path.resolve(__dirname, "client")
            // context of requests in the manifest file, defaults to the webpack context
        }),
        // new webpack.optimize.OccurenceOrderPlugin(),
        // webpack2 has already built-in
        new webpack.optimize.UglifyJsPlugin()
    ],
    resolve: {
        // root: path.resolve(__dirname, "client"),
        // modulesDirectories: ["node_modules"]
        // webpack2 use modules to replace all this configurations
        modules:["node_modules",path.resolve(__dirname, "client")]
    }
};

```

Combine this plugins with output.library option to expose the dll function i. e. into the global scope.

vendor-manifest.json file contents:

```js
{
  "name": "vendor",
  "content": {
    "./vendors.js": {
      "id": 0,
      "meta": {
        "harmonyModule": true
      }
    },
    //jquery
    "../node_modules/jquery/dist/jquery.js": {
      "id": 1,
      "meta": {}
    },
    //mustache
    "../node_modules/mustache/mustache.js": {
      "id": 2,
      "meta": {}
    }
  }
}
```

That is because we input jquery and mustache in our path.join(__dirname, "client", "vendors.js").

```js
import mustache from 'mustache';
import jquery from 'jquery';
```

dist/dll.vendor.js will include both jquery and mustache file conent!


### 2. webpack.DllReferencePlugin
  
We can simply put our dll/vendor-manifest.json to webpack.DllReferencePlugin configuration.

```js
 new webpack.DllReferencePlugin({
            context: path.join(__dirname, "client"),
            //same as DLLPlugin
            manifest: require("./dll/vendor-manifest.json")
        }),
```

.context: (absolute path) context of requests in the manifest (or content property)

.scope (optional): prefix which is used for accessing the content of the dll
manifest (object): an object containing content and name

.name (optional): the name where the dll is exposed (defaults to manifest.name) (see also externals)

.sourceType (optional): the type how the dll is exposed (defaults to "var") (see also externals)

.content (optional): the mappings from request to module id (defaults to manifest.content)


SOURCE FILES：   

[OPTIMIZING WEBPACK FOR FASTER REACT BUILDS](http://engineering.invisionapp.com/post/optimizing-webpack/)

[rimraf](https://github.com/liangklfang/rimraf)

[list-of-plugin](http://webpack.github.io/docs/list-of-plugins.html#dllplugin)