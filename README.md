# Kotlin-Javascript Boilerplate

[![Awesome Kotlin Badge](https://kotlin.link/awesome-kotlin.svg)](https://github.com/KotlinBy/awesome-kotlin)

An _extremely_ barebones template project for getting started with Javascript as a build target for Kotlin.

## Building

Build using `./gradlew build`, which compiles the `Main.kt` file into the javascript file at `build/js/module.js`.
This is configurable, but I'll cover that further down. 

The built file is a [UMD](http://dontkry.com/posts/code/browserify-and-the-universal-module-definition.html) javascript module, which has a dependency on the kotlin standard library. In order to run in
javascript, kotlin itself needs to be in javascript, and is needed as a dependency of any kotlin code compiled to 
javascript. You can configure the build task of this project to output the standard library, or you could pull 
it in using npm/yarn (I recommend [yarn](https://yarnpkg.com), but that's a different topic).

Since I've done the latter in this project, you'll need to do an `npm install`/`yarn install` (see below for 
instructions how to avoid this step and have the kotlin compiler provide the standard library as it builds).

## Running

### Using Node

Now that you've got your kotlin compiled to javascript, it's time to run it. This example has targeted UMD so it can run
server-side using `node`, as well as in the browser.

    $ node build/js/module.js
    $ Hello World!

I've also included a script in the `package.json`, so you can also run `npm start`/`yarn start` if you don't want to
remember/type the whole thing every time.

### In The Browser

Since the module type when compiling was UMD, this can run in the browser as well. If you run `yarn serve` from the root
of the project (or `python -m SimpleHTTPServer` if you don't have yarn and/or prefer that method) and open it in the browser
(`yarn serve` serves on `http://localhost:5000` and `python -m SimpleHTTPServer` serves on `http://localhost:8080`), you'll 
see the `index.html` file, telling you to look in the console, where you'll see the same `'Hello World'!` message printed.

Also, since we've enabled sourcemaps, you can open kotlin files in the Sources browser, and place debugging breakpoints in the
original source code, much like any other compile-to-javascript language.

Looking at the `index.html` file's source code, we see some basic styling (I couldn't deal with the terrible unstyled page)
and a couple of `<script>` tags. Remember from above that the kotlin standard library is still a dependency of our compiled
javascript, and therefore needs to be pulled in first (you'll get an error message if your ordering is incorrect, or if you
forget it entirely). This is the equivalent of `require('kotlin')` in the browser world, albeit much less powerful and more
confusing. (This is why tools like [webpack](https://webpack.github.io) were made, to allow us to write frontend javascript
as if it were CommonJS. I go over the different types of modules later on.)

## Configuration

In the `build.gradle` file, there's a `compileKotlin2Js` block which provides some configuration options for the kotlin
to javascript compiler, such as:

- The output file: where the compiled javascript files should be written

- The module type: one of `commonjs`, `amd`, `umd`, `plain` (although, if you're going to use this, you may as well just use `umd`)

- Whether or not source maps should be used (very useful/necessary for debugging purposes)

The configuration in this project includes:

```gradle
compileKotlin2Js {
    kotlinOptions.outputFile = "${projectDir}/build/js/module.js"
    kotlinOptions.moduleKind = "umd"
    kotlinOptions.sourceMap = true
}
```

### Module Types

The module types correspond to different module loading systems in javascript, explained [here](http://stackoverflow.com/a/16522990):

- CommonJS is the paradigm used in node, [browserify](http://browserify.org), and [webpack](https://webpack.github.io), and looks something like this:

```javascript
// magic-number.js
const _ = require('lodash');
    
module.exports.magicNumber = _.random();
    
// some-other-file.js
const { magicNumber } = require('./magic-number');
console.log(magicNumber);
```

- AMD is more well-suited for the browser, and is the paradigm used in libraries like [require.js](http://requirejs.org).

- UMD (Universal Module Definition) is a way of wrapping up your module to be used in CommonJS, AMD, or just attached to 
`window` as global variables, depending on the environment in which it's run. [This blog post](http://dontkry.com/posts/code/browserify-and-the-universal-module-definition.html)
does a good job summarizing UMD. When in doubt, this is probably what you should use.

### The Standard Library

As I mentioned above, the kotlin javascript standard library is going to be a dependency of the javascript which the compiler
outputs. You can install this via `npm`/`yarn`, or you can have the compiler output it for you, so you don't need to initialize
your project as a node project (although, this probably shouldn't be a concern of anyone anyway, since any other javascript 
dependencies should just be pulled from the npm registry anyway, unless you're using a CDN).

By default, the gradle doesn't make the `kotlin.js` file visible for some reason. So we'll need to add the following to our `build.gradle`:
```gradle
build.doLast {
  configurations.compile.each { File file ->
    copy {
      includeEmptyDirs = false

      from zipTree(file.absolutePath)
      into "${projectDir}/build/js"
      include { fileTreeElement ->
        def path = fileTreeElement.path
        path.endsWith(".js") && (path.startsWith("META-INF/resources/") || !path.startsWith("META-INF/"))
      }
    }
  }
}
```

This looks like arcane gradle magic, but all it does it place the `kotlin.js` file in the same place our other built javascript
files end up.

---

I plan on adding more to this over time, like getting set up with React (and maybe some server-side rendering of React?) or building a module in kotlin and calling it from javascript. I'd definitely recommend you check out the 
[official kotlin tutorials](https://kotlinlang.org/docs/tutorials/javascript/getting-started-gradle/getting-started-with-gradle.html), as that's where I learned a lot of this stuff anyway.
