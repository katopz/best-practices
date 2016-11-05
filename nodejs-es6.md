# Don’t Transpile JavaScript for Node.js
> Ref : http://vancelucas.com/blog/dont-transpile-javascript-for-node-js/

Transpiling JavaScript on the client is pretty much required for wide browser support, but transpiling JavaScript when running it server-side with Node.js is entirely optional, and I strongly recommend against doing it.

## Node.js ES6/ES2015 Pains

On a current project I am working on, I started out with full ES6/ES2015 all the way using import/export, etc. transpiling everything in my **src/** folder with babel into a **build/** folder before running it with node. As I continued working and saving files, however, it became incredibly tedious waiting for the babel compile step before I could see any changes. **It felt like I was using a language with static typing instead of the dynamic, fluid JavaScript I know and love.** Using a tools like [nodemon][1] became a chore, and always with a built-in delay and extra learning curve.

Some people resort to (and even recommend!) using [babel-node][2] as a way around this, blissfully running their new ES6 hotness, but inevitably [run into the "Can I use this in production?" question][3] when it comes time to deploy their code (hint_: No. You shouldn't. babel-node is not meant for production use_). Congratulations. You have now created a different runtime environment for development vs. production, which is a huge devops mistake, and a certain cause of future headaches.

## Server-Side Debugging Headaches

One of the main complaints about using [CoffeeScript][4] (Yeah – remember CoffeeScript? I know it's been a few years.) was the difficulty in debugging and deciphering transpiled code you didn't write. Well, welcome to 2016 and ES6. Here we are doing the same thing, only this time it's okay (promise!), because it's _just JavaScript_, right? Wrong. **You will run into all the same headaches and issues of debugging transpiled code, only it will be _worse_ since you are trying to fix a critical issue on the server side that affects _all your users everywhere_** – not just a subset of users with a certain browser and operating system combination.

## Import require(), and export ES6 modules

**There is no JavaScript engine yet that natively supports ES6 modules** – not even the bleeding-edge v8 engine in Chrome and Node.js or [Microsoft's new Chakra engine][5]. If you are using import/export, babel is converting these statements to require and module.exports anyways. You should just go ahead and use require() and module.exports instead in Node.js. I know there are more features that are possible with import/export like [tree-shaking][6] (hat tip to [Rollup.js][7]), but these features are much more valuable on the client side in reducing bundle sizes than on the server with Node.js. Again – code clarity and ease of debugging and development are more important on the server side than being on the bleeding edge of ES6.

## You Can Still Use Most ES6 Features

The important thing to know given my advice about not transpiling your JavaScript for Node.js is that **you can still use a [bunch of great ES6 features][8] with Node.js without the need to transpile your code**. It's not like your choices are to use the new ES6 hotness or be stuck in 2009. The only real thing you will have to give up is import/export, which isn't a huge sacrifice given how confusing some of the import/export rules are to newcomers, and the fact that the statements will be converted to require/module.exports anyways.

Step 1 is to add a strict mode declaration at the top of each file:

    'use strict';

And then maybe some feature flags, depending on which other ES6 features you require (and even this is temporary, given the new v8 engine already supports both [destructuring][9] and [default parameters][10], and so will be [enabled by default in Node.js v6.0][11]). I personally use the –harmony_default_parameters and –harmony_destructuring flags:

    node --harmony_default_parameters --harmony-destructuring src/server/index.js

Or using `nodemon` for automatic server reloading:

    nodemon --harmony_default_parameters --harmony-destructuring src/server/index.js

## Save Transpiling for the Client

The bottom line here is to save transpiling for the client-side where it is still needed for full browser support. You should never add complications or additional steps to run your code where it is not necessary, and the server is one of those places for JavaScript. Keep your server-side development cycles fast, lean, and simple. Your future self will thank you.

[1]: http://nodemon.io/
[2]: https://babeljs.io/docs/usage/cli/#babel-node
[3]: http://stackoverflow.com/questions/30773756/is-it-okay-to-use-babel-node-in-production
[4]: http://coffeescript.org/
[5]: https://en.wikipedia.org/wiki/Chakra_(JavaScript_engine)
[6]: https://medium.com/@Rich_Harris/tree-shaking-versus-dead-code-elimination-d3765df85c80
[7]: http://rollupjs.org/
[8]: https://nodejs.org/en/docs/es6/
[9]: https://www.chromestatus.com/features/4588790303686656
[10]: https://www.chromestatus.com/features/5275456790069248
[11]: https://github.com/nodejs/node/issues/5355
