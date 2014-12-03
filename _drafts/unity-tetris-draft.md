---
layout: post
category: tutorials
tags: [unity, tutorial, unit-testing]
---

There a lot of tutorials about Unity3d; in fact, there's an overabundance of them. However, *most* of these tutorials are aimed at absolute beginners and there aren't enough tutorials and guides about doing things the right way. We're also be spending a lot of time explaining things. All explanations will be contained in block quotes; just ignore them if you want to code more and read less.

So, is this tutorial for you? If you consider yourself experience programmer, and you don't need to be explained about what a decorator pattern is, for example, then you know enough to get in.

I assumed that you downloaded unity and that your unity version somewhat is 4.6 or relatively close. we won't be using the features that are likely to change much between the versions, but unity may make some minor technical chances.

# Creating new project

So, first things first — let's create an empty unit project.  
Launch Unity editor, open "file" menu and select "new project...", and you'll be presented with a dialog that looks something like this:

![creating project]({{ site.url }}/assets/unity-tetris/create-project.png)

> Unity isn't just a library that you can link and use; it's a complete package, with IDE and all kinds of editors built in, so you have to use the editor to create new projects.
> There are three things that you should note in the wizrd:

1. You must specify a folder that doesn't exist yet — Unity will create it and populate with basic stuff that you will need
2. Unity has a built-in package system. Unlike package managers like NPM, bower and Cargo, it doesn't use any kind of version control; to update your package, you must get a new version of a binary file. But since a lot of packages consist mostly of game assets, and there are a lot of close-source paid packages, it makes sense. You can also use Unity Asset Store to browse and buy packages, although we won't be covering it here.
3. With the new support for 2D projects, Unity now has a lot of interface improvements for them. This switch doesn't actually affect content of your project, only the editor's interface.

Right after you created the project, open it's folder. It should look something like this:

![empty project]({{ site.url }}/assets/unity-tetris/empty-project.png)

So, what's in this folders?

* **Assets** is the place where the majority of your stuff will live. All the scripts that you write, (except may be some native plugins — never mind this stuff for now), all assets that you import, all levels that you create, everything goes here, or Unity won't even see it.
* **Project Settings** is a folder just for that — your global project settings. There are a bunch of files that must be present in any project to tell Unity basic things about it. In 99% of the case 
