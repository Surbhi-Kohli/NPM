# NPM Link
src:https://medium.com/@alexishevia/the-magic-behind-npm-link-d94dcb3a81af

Node.js has a very simple module loading strategy. Whenever you require() a module, the following steps are executed in order:

If it is a core module, load the core module.
If it is a relative path, load the module from the relative path.
Look for the module in the ./node_modules directory. If it is not there, recursively search in the 
parent directories’ ./node_modules until either the module is found or the root of the file system is reached.
Note: You can specify a NODE_PATH environment variable to make Node.js search for modules in other folders, but it is not recommended.

Loading a local module
The npm link command is special because it allows you to load a module from anywhere on your computer.

Here is an example:

1. Create (or download) an npm module to your computer:

cd ~/Desktop
git clone git@github.com:klughammer/node-randomstring.git (consider like gscope-core)
2. Run npm link inside the module’s root folder:

cd ~/Desktop/node-randomstring
npm link     //this will make the randomstring module available for linking:npm creates a symbolic
            //link from your “global node_modules” directory to the local module’s directory.
3. In a different directory, run npm link <module_name>:

mkdir ~/Desktop/my-project
cd ~/Desktop/my-project
npm link randomstring
4. Now, you can require the linked module from within your project:

 ~/Desktop/my-project/app.js
const randomstring = require("randomstring");
console.log(randomstring.generate());
The cool thing about npm link is that any change you make in your required module will be immediately reflected in your project.

For example, let us replace the generate() function with a slightly less useful one:

 ~/Desktop/node-randomstring/lib/randomstring
exports.generate = function(){
  return 4; // chosen by fair dice roll.
            // guaranteed to be random.
}
If we run our app again, we will see a result of 4.

Being able to load a local module is really useful if you want to make changes on the required module and test 
them immediately in the context of your project, without going through a publish/install cycle.

How does it work?
It might seem like npm link is doing some magic to side-step Node’s module loading strategy, but the truth is there is nothing special about it:
npm link just creates two symbolic links.

1. The global symlink

When you run npm link in a module’s root directory, npm creates a symbolic link from your “global node_modules” directory to the 
local module’s directory.

The “global node_modules” directory is a special directory where all modules installed with npm install -g are stored. You can 
find the path to your global 
node_modules directory by running npm root -g.

The “global node_modules” directory is a special directory where all modules installed with npm install -g are stored. 
You can find the path to your global node_modules directory by running npm root -g.

You can see the symlink that was created by running:

> ls -al $(npm root -g)
lrwxrwxrwx  1 alexishevia alexishevia   43 Mar 22 21:19 randomstring -> /home/alexishevia/Desktop/node-randomstring

2. The local symlink

When you run npm link <module_name> in a project’s directory, npm creates a symbolic link 
from ./node_modules/<module_name> to <global_node_modules>/<module_name>.

You can see the symlink that was created by running:
> ls -al ./node_modules
lrwxrwxrwx 1 alexishevia alexishevia   64 Mar 22 21:19 randomstring -> ../../../.nvm/versions/node/v6.9.5/lib/node_modules/randomstring

Since I am using nvm, my global node_modules folder is: /home/alexishevia/.nvm/versions/node/v6.9.5/lib/node_modules

That’s it: no magic, just symbolic links.

Bonus
You can “undo” the effects of npm link by simply removing the symbolic links. But there is a built in command for it, aptly called: npm unlink.

Just run npm unlink --no-save <module_name> on your project’s directory to remove the local symlink, and run npm unlink on the module’s 
directory to remove the global symlink.
