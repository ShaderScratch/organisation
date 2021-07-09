# ShaderScratch

ShaderScratch is an Open-Source project that aims to introduce beginners to Shader programming in a similar fashion to that of [Scratch](https://scratch.mit.edu/). It also aims to bring some of the neat features of [Shadertoy](https://www.shadertoy.com/).

# setup

Getting the scratch codebase to run locally is a bit of a hassle, with
[instructions](https://github.com/ShaderScratch/scratch-gui#developing-alongside-other-scratch-repositories)
[spread](https://github.com/LLK/scratch-gui/wiki/Getting-Started)
across many different files, repositories,
[wikis](https://github.com/LLK/scratch-blocks/wiki)
and
[forums](https://scratch.mit.edu/discuss/topic/289503/), so I thought it'd be a good idea to outline how I managed get everything to work here. I am by no means an expert when it comes to this and have no idea if what worked for me will also work for you, but it should give you a general idea of what you'll need to do.

## prerequisites

First of, you'll need [node.js](https://nodejs.org/en/) to run this, so head over there and grab the installer for the latest recommended version. In the installer, be sure to also check the following checkbox:

![Node_Setup](https://user-images.githubusercontent.com/43812953/125100051-f7ce9200-e0d8-11eb-8582-e43a42b608ef.png)

This will install a few different tools required by some node.js-packages, it seems, including the latest version of python 3. You'll also need [python 2](https://www.python.org/downloads/release/python-2716/) however, as the build script for scratch-blocks is not compatible with python 3. So, install that too, add the installation folder to your windows path (if you know linux, idk, you'll have to figure something out yourself) and rename the `python.exe` file there to `python2.exe` so you'll then be able to use python 3 as `python` and python 2 as `python2` in the command line:
![grafik](https://user-images.githubusercontent.com/43812953/125101773-e2f2fe00-e0da-11eb-8049-ae8ca131584b.png)

## installing

Next, open git bash in whatever folder you want to use for this and clone the repositories there (with `--depth=10`, because some large files have accidentally been committed in the past):

    git clone https://github.com/ShaderScratch/scratch-gui scratch-gui --depth=10
    git clone https://github.com/ShaderScratch/scratch-vm scratch-vm --depth=10
    git clone https://github.com/ShaderScratch/scratch-blocks scratch-blocks --depth=10
    git clone https://github.com/ShaderScratch/scratch-render scratch-render --depth=10

If you run `npm install` and then `npm start` in scratch-gui now, you should already be able to see the gui locally under [localhost:8601](http://localhost:8601). At the moment, this only mirrors you changes to scratch-gui though - to test your changes to the other repositories as well, we'll have to build these too and then link them to here.

First of, let's finish setting up the other repositories as well. For scratch-vm and scratch-render, just running `npm install` there too should already do the trick; for scratch-blocks however, things are a little more complicated. Doing this there should instead give out an error that blockly is incompatible with python 3:

![grafik](https://user-images.githubusercontent.com/43812953/125105231-8abdfb00-e0de-11eb-8075-385f6f0c98b2.png)

To fix this, open `package.json` in that folder and replace `python` with `python2` in line 16 to run the build script in the older python version you installed previously instead. Now running `npm install` again should greet you with a different error message instead:

![grafik](https://user-images.githubusercontent.com/43812953/125106317-cb6a4400-e0df-11eb-80f6-5dd2e8fdb43a.png)

This is because the total length of the arguments passed to some sort of subprocess or something exceeds 8 kilobytes here, which is the maximum length a command can have in windows (it should work fine on linux though I think, so if you use that you can probably skip this step). We can fix this by replacing all occurences of `node_modules\google-closure-library` with a shorter path to a symlink - that will bring the length back down below the limit. To do this, run the following command in the main folder you cloned the repositories into:

    mklink /D cl scratch-blocks\node_modules\google-closure-library

This should create a symlink named `cl` that points to the google closure library in the `node_modules`-folder of scratch-blocks. Now, to shorten the command using this symlink, add the following code as line 331 into `scratch-blocks\build.py` (that is, just above `proc = ...`):

    args = map(lambda s:s.replace("node_modules\google-closure-library","..\cl"),args)

Now, running `npm install` in scratch-blocks again should finish without errors too.

## linking

Now that all four repositories are successfully installed, the only thing remaining to do is telling npm to use the code from all of those folders in scratch-gui, so our local changes to all of those repositories can be seen and tested there.

To do this, just run `npm link` in scratch-blocks, scratch-vm and scratch-render each; then, navigate to scratch-gui and run `npm link scratch-blocks scratch-vm scratch-render` there. Now, running `npm start` again should show you all of your changes on [localhost:8601](http://localhost:8601).
