---
layout: post
category: tutorials
tags: [unity, tutorial, unit-testing]
---

There a lot of tutorials about Unity3d; in fact, there's an overabundance of them. However, *most* of these tutorials are aimed at absolute beginners and there aren't enough tutorials and guides about doing things the right way. We're also be spending a lot of time explaining things.

So, is this tutorial for you? If you consider yourself experience programmer, and you don't need to be explained about what a decorator pattern is, for example, then you know enough to get in.

I assumed that you downloaded unity and that your unity version somewhat is 4.6 or relatively close. we won't be using the features that are likely to change much between the versions, but unity may make some minor technical chances.

# Creating new project

So, first things first — let's create an empty unit project.  
Launch Unity editor, open "file" menu and select "new project...", and you'll be presented with a dialog that looks something like this:

![creating project]({{ site.url }}/assets/unity-tetris/create-project.png)

Unity isn't just a library that you can link and use; it's a complete package, with IDE and all kinds of editors built in, so you have to use the editor to create new projects.

There are three things that you should note in the wizrd:

1. You must specify a folder that doesn't exist yet — Unity will create it and populate with basic stuff that you will need
2. Unity has a built-in package system. Unlike package managers like NPM, bower and Cargo, it doesn't use any kind of version control; to update your package, you must get a new version of a binary file. But since a lot of packages consist mostly of game assets, and there are a lot of close-source paid packages, it makes sense. You can also use Unity Asset Store to browse and buy packages, although we won't be covering it here.
3. With the new support for 2D projects, Unity now has a lot of interface improvements for them. This switch doesn't actually affect content of your project, only the editor's interface.

Right after you created the project, open it's folder. It should look something like this:

![empty project]({{ site.url }}/assets/unity-tetris/empty-project.png)

So, what's there?

* **Assets** is the place where the majority of your stuff will live. All the scripts that you write, (except may be some native plugins — never mind this stuff for now), all assets that you import, all levels that you create, everything goes here, or Unity won't even see it. And since you only created the project, it is completely empty.
* **Project Settings** is a folder just for it says on the lid — your global project settings. There are a bunch of files that must be present in any project to tell Unity basic things about it. You're most likely won't ever edit anything there by hand; there are useful menus in the editor for that.
* **Temp** is pretty obvious too — Unity's temporary files that it needs for various tasks live here.
* And, finally, **Library** is a very special folder. Every time you add an asset to your project, like an image, audio, or a 3d model, Unity converts it's to approprite format and saves that converted version here — and when you decide to switch current active platform, it will take the source asset once again and reconvert it, since the conversion settings will usually be different. For example, you can save hundreds of textures in **Assets** folder just as raw PSD files, and they'll can automatically converted to 1024x1024 32-bit for Windows standalone and as 256x256 16-bit for Android devices if you so choose.

Next, we want to put our project for version control. git is today's standard for VCS, and it is awesome, so that's what I'm going to use. If you're on windows, you may have to install cygwin or use a GUI client instead of console version; I won't be holding your hand here.

Open terminal, navigate to the project's folder and init a repository:

{% highlight bash %}
$ git init
Initialized empty Git repository in /Users/golergka/Projects/Unity/tetris-tutorial/.git/
{% endhighlight %}

But before we create the first commit, we must place an appropriate **.gitignore** file in the project's root, so that various system and Unity files won't be check in version cotrol. Thankfully, there's already a great **.gitignore** ready for Unity projects:

{% gist AndrewAlexMac/8623191 %}

Create **.gitignore** file in your project root, copy the contents of the above file there, and create the initial commit:

{% highlight bash %}
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore
	ProjectSettings/

nothing added to commit but untracked files present (use "git add" to track)
$ git add --all
$ git commit -m "Init commit"
[master (root-commit) f19b309] Init commit
 14 files changed, 91 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 ProjectSettings/AudioManager.asset
 create mode 100644 ProjectSettings/DynamicsManager.asset
 create mode 100644 ProjectSettings/EditorBuildSettings.asset
 create mode 100644 ProjectSettings/EditorSettings.asset
 create mode 100644 ProjectSettings/GraphicsSettings.asset
 create mode 100644 ProjectSettings/InputManager.asset
 create mode 100644 ProjectSettings/NavMeshLayers.asset
 create mode 100644 ProjectSettings/NetworkManager.asset
 create mode 100644 ProjectSettings/Physics2DSettings.asset
 create mode 100644 ProjectSettings/ProjectSettings.asset
 create mode 100644 ProjectSettings/QualitySettings.asset
 create mode 100644 ProjectSettings/TagManager.asset
 create mode 100644 ProjectSettings/TimeManager.asset
{% endhighlight %}

Congradulations! You now have a simple Unity project properly set up. Now we can start making the actual game.

# The game loop

And here we are — with our favorite editor opened on a blank file. May be it's Unity's Monodevelop, or may be you use Vim like me; but first, I want to step aside from a keyboard and talk about what we're going to do and why we're going to do it that way. If you want, you can skip the next section altogether, of course.

## Little rant about game development

Writing code is always a game of trade-offs. If you aim for perfomance first, you may end with arcane-looking C enchantments that will only work on specific hardware configuration and will require about hour per line to read and understand. If you decide that you want the most bug-free possible, you'll porbably end up installing a lot of testing processes and regulations, and changing a single little feature in your specifications will take up 6 months of commitee meetings, before you'll get the green light to change 6 lines of actual code.

All these practices actually have their place in time; but in game development, I'd rather focus on maintainability and readability. When we develop games, we change stuff and work on a pretty complicated systems that have a lot of [emergent complexity](http://en.wikipedia.org/wiki/Emergent_gameplay) — so it's neccessary to keep things simple, since they'll probably get more complex in the future anyway.

You don't write game code for yourself. You don't write it for your boss or your players; in the end, the most important person you're writing code is **the maintainer**. It may be you from the future, or a random dude that will be hired after you'll leave the project behind. Don't count on explaining him things. Don't expect him to be smart, or to have any additional time; always expect the poor fella to be on crunch time. He'll be working without proper documentation, and he'll ceirtanly won't have any time or patience to write it himself. If you give him two ways to do something, he will mix them up at best or even create a new hack. Every time he will encounter a beautiful abstractoin, he will probably break it; every time there's a possibility of doing something quick and dirty, he will do it that exact way. And believe me, you *will* be that guy in near future, on this project or the other; that is just a reality of game industry.

What you need is to create a [pit of success](http://blog.codinghorror.com/falling-into-the-pit-of-success/) for him. Make it easy to do the right thing. Make the conventions obvious and easy to follow. Make the APIs that simple that his code will become self-documenting. And most importantly, make your code as easy to read and browse as possible.

## What is Tetris about?


