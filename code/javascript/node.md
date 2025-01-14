# Node

https://nodejs.org/en/  
https://github.com/nodejs/node  

Node.js® is a [JavaScript](index.md) runtime built on Chrome's V8 JavaScript engine.

Useful for running Javascript on a server with access to local system resources.

## Running

```
node
```

### Parameters / Args

```
var args = process.argv.slice(2);
```

https://stackoverflow.com/questions/4351521/how-do-i-pass-command-line-arguments-to-a-node-js-program  
javascript - How do I pass command line arguments to a Node.js program? - Stack Overflow  

For more involved command line parameter parsing:

https://github.com/substack/minimist  
GitHub - substack/minimist: parse argument options  

https://nodejs.org/docs/latest/api/process.html#process_process_argv  
Process | Node.js v17.1.0 Documentation  

https://duckduckgo.com/?t=ffab&q=node+pass+parameters+to+script&ia=web  
node pass parameters to script at DuckDuckGo  


### Sample Script

See also [files](files.md)

```js
#!/usr/bin/env node

var fs = require('fs');

var args = process.argv.slice(2);
console.log("Parameters passed in: ", args);

// require arguments
if (args.length == 0) {
  console.log("Usage:");
  console.log("template.js [source] [destination]");
  process.exit(1);
}

console.log();
console.log("Directory contents:", args[0]);
console.log();
// for scanning a directory
fs.readdir(args[0], (err, files) => {
  files.forEach((file) => {
    console.log(`[${file}](${file})  `);
  });
  console.log();
});

// For system calls
// const { exec } = require("child_process");

// exec(`ls -la ${args[0]}`, (error, stdout, stderr) => {
//     if (error) {
//         console.log(`error: ${error.message}`);
//         return;
//     }
//     if (stderr) {
//         console.log(`stderr: ${stderr}`);
//         return;
//     }
//     console.log(`stdout: ${stdout}`);
// });

// For reading files and loading them as JSON

// var obj;
// fs.readFile(args[0], 'utf8', (err, data) => {
//   if (err) throw err;

//   // parse data as json:
//   // obj = JSON.parse(data);

//   // iterate over lines:
//   const lines = data.split('\n');
//   for(var i=0; i < lines.length; i++) {
//     console.log(lines[i]);
//   }
// });


```

https://www.codegrepper.com/code-examples/shell/node+run+shell+command


### Import vs Require

```
Cannot use import statement outside a module
```

To use ESModules syntax, need to switch package. (Note: all code will need to follow this pattern)

```
"type": "module"
```

https://stackoverflow.com/questions/62488898/node-js-syntaxerror-cannot-use-import-statement-outside-a-module

Or Use commonjs syntax instead of es module syntax:

```

module.exports.func = function (){
    console.log("Hello World");
}
```

```
const myMod = require("./mod")
myMod.func()
```

ES6 modules use the import syntax. Seems like the way to go moving forward.

https://stackoverflow.com/questions/46677752/the-difference-between-requirex-and-import-x  
node.js - The difference between "require(x)" and "import x" - Stack Overflow  

https://stackoverflow.com/questions/31354559/using-node-js-require-vs-es6-import-export  
javascript - Using Node.js require vs. ES6 import/export - Stack Overflow  

https://duckduckgo.com/?t=ffab&q=javascript+require+vs+import&ia=web  
javascript require vs import at DuckDuckGo  


### Local Development

https://duckduckgo.com/?t=ffab&q=work+on+npm+module+in+development&ia=web  
work on npm module in development at DuckDuckGo  
https://nodesource.com/blog/an-absolute-beginners-guide-to-using-npm/  
An Absolute Beginner's Guide to Using npm - NodeSource  
https://www.deadcoderising.com/how-to-smoothly-develop-node-modules-locally-using-npm-link/  
How to smoothly develop node modules locally using npm link  


### Module Not Found

Make sure you've actually installed the module if you get a message like `MODULE_NOT_FOUND`

https://nodejs.org/api/modules.html#modules_all_together

https://gist.github.com/MattGoldwater/78f89ea93b9f1dfc19d3440e172cfa49

https://stackoverflow.com/questions/9023672/how-do-i-resolve-cannot-find-module-error-using-node-js


## Package Managers

Package managers ensure that all of the modules that your application depends on are compatible and available to the local code base.

### Types of dependencies

https://classic.yarnpkg.com/en/docs/dependency-types#toc-dev-dependencies

> Dependencies serve many different purposes. Some dependencies are needed to build your project, others are needed when you’re running your program. As such there are a number of different types of dependencies that you can have (e.g. dependencies, devDependencies, and peerDependencies).


### Choosing a package manager

NPM vs PNPM vs Yarn 

The answer? Use what the project is using. Be consistent. 

All are good. No need to get hung up here. See what works best and make a change when necessary. 


### Npm

NPM is the default package manager for Node. In some environments, it may be all you have access to. 

#### Install packages

Install everything as configured in package.json file

```
npm install
```

Calls to

```
npm install --save [package name]
```

or

```
npm install --save-dev [package name]
```

or

```
npm install --save-optional [package name]
```

will update the package.json to list your dependencies.


#### Upgrade / Update packages

To upgrade a package that has already been installed, use either:

```
npm install [package name]@latest
```

or

```
npm update [package name]
```


#### Remove packages

To remove a dependency:

```
npm uninstall <name> --save
```

https://stackoverflow.com/questions/13066532/how-to-uninstall-npm-modules-in-node-js


### pnpm

If you have a bit more control over the hosting environment and can install a user level package manager, pnpm is a good option. 

PNPM uses links to node modules so you don't end up with 100 copies of the same module on your local drive. This is useful in development when you may want to use a large development module in multiple projects. 

https://pnpm.io/


https://pnpm.io/installation

With node:

```
curl -f https://get.pnpm.io/v6.16.js | node - add --global pnpm
```

With npm:

```
npm install -g pnpm
```

npx:

```
npx pnpm add -g pnpm
```

One downside is that it is not included by default in the main docker node image. May not be necessary to use it in a container context. `pnpm` is more helpful on development. 

One downside is the addition of `_tmp_*` directories. These are easy to ignore in [VSCode](/system/editors/vs-code/vs-code.md#ignore-files)


### Yarn

It should be available in most Node containers.

Once you have `yarn` available, you can add packages as a requirement with:

    yarn add <name>

or as a dev dependency with:

    yarn add <package...> [--dev/-D]

https://classic.yarnpkg.com/en/docs/cli/add/  
https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-dev-d  

To install a package globally, the order of the parameters is important:

    yarn global <add/bin/list/remove/upgrade> [--prefix]
    
To remove a dependency

     yarn remove <name>
     
Note: manually deleting from `package.json` removes the dependency from the project, but will not remove the files from `node_modules` of the local instance.


#### Reinstall Modules

Reinstalling a package after just deleting the node module works with:

    yarn install --check-files

[via](https://stackoverflow.com/questions/41864099/how-do-i-force-yarn-to-reinstall-a-package)


#### Install / update yarn:

    npm install -g yarn

or use a system level package manager

    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -

    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

then, with apt-get:

    sudo apt-get install --no-install-recommends yarn

via https://yarnpkg.com/lang/en/docs/install/




## Lock files

Is it really a good idea to track lock files under version control?
Seems like it adds a lot of noise to the process

Yes, it can add noise if you manage it manually. However, without it you may run into situations where deployments to a different environment may not work due to different versions of modules being installed. It's better to test with a set of locked versions. In that case, lock files are necessary.

### Resolve yarn.lock conflicts

It's as easy as running `yarn` or `yarn install` to automatically resolve version control conflicts in a yarn.lock file. Sweet!

### Resolving package-lock.json conflicts

Occasionally, two separate calls to `npm install` will create package locks that cause merge conflicts in source control systems. As of npm@5.7.0, these conflicts can be resolved by manually fixing any package.json conflicts, and then running npm install [--package-lock-only] again. npm will automatically resolve any conflicts for you and write a merged package lock that includes all the dependencies from both branches in a reasonable tree. If --package-lock-only is provided, it will do this without also modifying your local node_modules/.

To make this process seamless on git, consider installing npm-merge-driver, which will teach git how to do this itself without any user interaction. In short:

    $ npx npm-merge-driver install -g

will let you do this, and even works with pre-npm@5.7.0 versions of npm 5, albeit a bit more noisily. Note that if package.json itself conflicts, you will have to resolve that by hand and run npm install manually, even with the merge driver.

[via](https://docs.npmjs.com/cli/v6/configuring-npm/package-locks)

See also
https://docs.npmjs.com/cli/v6/configuring-npm/package-lock-json


## Process Monitoring

If you have a service running live using a node based (e.g. [express](../api/express.md)) server, a monitoring tool can make sure it stays up. Options include...


### Docker

[Docker](/system/virtualization/docker.md) is a great option.


### PM2

https://www.google.com/search?client=ubuntu&channel=fs&q=pm2&ie=utf-8&oe=utf-8  
pm2 - Google Search  
https://pm2.keymetrics.io/  
PM2 - Home  

### Nodemon

https://github.com/remy/nodemon  
GitHub - remy/nodemon: Monitor for any changes in your node.js application and automatically restart the server - perfect for development  
https://www.google.com/search?client=ubuntu&channel=fs&q=nodemon+vs+pm2&ie=utf-8&oe=utf-8  
nodemon vs pm2 - Google Search  


## Environment Variables (.env) dotenv

Variables set in a `.env` file are automatically loaded by node and made available via `process.env.*` variables. 

Try out a ui/.env file. Is the value available via process.env.whatever?

Many frameworks leverage these variables for configuration. 


## Install

See what the latest versions of node are:

https://nodejs.org/en/about/releases/

### NVM

Node Version Manager - Simple bash script to manage multiple active node.js versions

https://github.com/creationix/nvm

Visit the site for the latest version of the command

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

Open a new shell and verify with:

```
command -v nvm
```

Clean up old (non-nvm) node version(s):

```
npm ls -gp --depth=0 | awk -F/ '/node_modules/ && !/\/npm$/ {print $NF}' | sudo xargs npm -g rm
```

Install Node via NVM:

```
nvm install node
```

To install a specific version of node, use:

```
nvm install 14
```

To see what version is currently installed

```
nvm ls # this should be equivalent to `node -v`
```

To see versions available to be installed via `nvm`

```
nvm ls-remote
```

https://github.com/nvm-sh/nvm#listing-versions

To install the latest lts version of node:

```
nvm install --lts
```

https://stackoverflow.com/questions/64002438/installing-node-lts-with-nvm-on-windows

https://www.linode.com/docs/guides/how-to-install-use-node-version-manager-nvm/

I needed to add the following to my `.bashrc` file in order for VSCode to see the new version of node:

```
PATH="/usr/local/bin:$(getconf PATH)"
```

https://stackoverflow.com/questions/44700432/visual-studio-code-to-use-node-version-specified-by-nvm


### Nodesource

Nodesource is another popular way to install node

https://github.com/nodesource/distributions/blob/master/README.md


### Docker

[Docker containers](../../system/virtualization/docker.md) are a great way to ensure you're running the node environment that you think you're running. 

Containers for Node applications are maintained here

https://hub.docker.com/_/node/

In `docker-compose.yml`, this is a good place to start

```
image: node:lts
```

If you're using a container that does not have Node installed (e.g. Centos), installing from nodesource.com seems like the best option

```
RUN curl -sL https://rpm.nodesource.com/setup_10.x | bash # for node version 10.x
RUN yum -y install nodejs
RUN node --version # optional to check that it worked
RUN npm --version # optional to check that it worked
```

NVM is tricky to use in a container:

```
# nvm environment variables
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 12.16.2

# Install NVM for installing node
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

RUN source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default

```


#### NPM Packages & Docker

Official documentation for dockerizing node applications:

https://nodejs.org/en/docs/guides/nodejs-docker-webapp/

May be possible to minimize the number of npm packages pulled down during an image build:

https://itnext.io/npm-install-with-cache-in-docker-4bb85283fa12

Looks like Seth has another tactic for this here:

https://github.com/City-of-Bloomington/myBloomington/blob/master/Dockerfile


