---
title: "Let's make a fighting game #0.5: Preparations"
date: 2023-01-09 21:08:30 -0000
category: fgtutorial
author: minogame
toc: true
---

[<< Prev: #0: The heart](/fgtutorial/2023/01/06/article0-heart.html)

Good day. In this series of articles, I'll be attempting to program a simple 2D fighting game using
the Python game development library Pygame. 

Even after sitting through the previous article, we still have a bit of preparation to complete 
before we can get started. Let's get our prerequisites sorted out.

<!--more-->

## Install Python (3.10)

[https://www.python.org/downloads/](https://www.python.org/downloads/)

> **Important:** Install Python **3.10**, as Pygame does not run on Python 3.11 at the time of writing
[(2022/12/31, confirm on Pygame website)](https://www.pygame.org/wiki/GettingStarted).

Confirm the installation by running 

```
python3 --version
```

in a terminal.

### Aside: A first roadblock (Strategies for bugs)
```
'python' is not recognized as an internal or external command,
operable program or batch file.
```

You might run into your first error here. Don't panic, errors are going to happen a lot.

- The instructions you're following may be out-of-date, or even outright wrong.  
- You may have simply mistyped something; maybe you read a 0 (zero) as an O (capital o),
or your Mac autocorrected a `--` (two hyphens, often used as a flag ) into a -- (em dash).
- Maybe you just did something wrong. If it won't take too long, run it again from the beginning
just to be sure. *(Do a sanity check)*
- Sometimes your computer is just very strange.   

Whatever the case, here's some of the basic steps you should try to solve an error:

1. Try to reproduce the error.
  - If you can figure out some of the possible conditions that cause it, you might understand why it happens.
2. Paste the error text into Google.
  - The exact error text will often have bit specific to your program or run that 
obviously won't appear in others' questions, so you will likely need to pare down the text a bit.
  - Sometimes the error readout/stack trace will have multiple errors in it. More often than not, you
only want the one closest to the bottom since it's the one causing the rest of errors, but give the others a skim.
3. [Read the error again, more carefully this time.](https://twitter.com/b0rk/status/1570463473011920897)
4. After giving the problem an honest try, ask others for help on StackOverflow or a relevant Discord.
  - [Just the act of trying to cleanly articulate what's wrong may give you insight.](https://en.wikipedia.org/wiki/Rubber_duck_debugging)

Advice for this error, if you can't figure it out:
<p class="spoiler">
On Windows, this is usually caused by the newly-installed Python executable being absent from the PATH 
environment variable. Try searching for "how to add python path environment variable windows". 
You may need to restart your computer for the environment variable changes to take effect.
</p>

### Additional reading
- [StackOverflow's How to Ask](https://stackoverflow.com/help/how-to-ask)
- [How to Report Bugs Effectively](https://www.chiark.greenend.org.uk/~sgtatham/bugs.html)
- [How to Ask Questions the Smart Way](http://catb.org/~esr/faqs/smart-questions.html)

The latter two are a bit lengthy and harsh to the reader, but if you're willing to swallow your pride,
all three of these articles teach you best practices fir how to ask for help in a way that will improve your 
chances of getting a good answer.

I recommend giving them a read when you have a bit of time.


## Install Pygame

[https://www.pygame.org/wiki/GettingStarted](https://www.pygame.org/wiki/GettingStarted)

(2023/01/09 - The Pygame site appears to be down currently; you can install pygame by typing
`pip install pygame` into a terminal window.)

<aside>
Although Python 2 is no longer updated, some computers might still default to 2 when running `python`.
From now on, I will be assuming that `python ...` commands run Python 3.
You might wish to uninstall Python 2 from your computer or 
<a href="https://stackoverflow.com/questions/20530996/aliases-in-windows-command-prompt">create an
alias to correct python to python3</a> if you must keep it.
</aside>

Again, as a sanity check, confirm the installation as detailed on the website.


```
python -m pygame.examples.aliens
```


## Install and configure your IDE of choice

As mentioned in #0, don't settle for accepting pain points, try to eliminate or reduce them 
however you can. An IDE (Integrated Development Environment) should be the program you write most, if not
all of your code in, as they provide much-needed quality-of-life features (eg. text autocompletion, 
bracket highlighting, test runners plugins, Git integration) for development. I recommend 
[Visual Studio Code](https://code.visualstudio.com/download), an excellent, free pick with robust 
support from its developers and plugin writers. 

Another option I found for Python is Spyder IDE + Anaconda. I haven't tried this myself, but it was
recommended to me by [another Pygame tutorial](https://www.patternsgameprog.com/discover-python-and-patterns-1)
I found very helpful.

Optional things you might like to configure at this point:
- Browse the IDE's interface for useful-looking Python plugins to install.
- [Configure a Python linter](https://code.visualstudio.com/docs/python/linting) to run whenever you
save your file.


## (Optional) Set up Version Control Software (Git)

You don't need to set up VCS to write code, but having it set up will help you keep track of 
your project changes. There are a number of VCS options out there, but Git is the one I'm familiar with.

### Motivations for using VCS
- You'd like to segment your changes into nice, discrete chunks instead of letting everything 
blend together into a continuous stream of changes. *(git commit)*
- You'd like to be able to see which parts of your program you've made changes to since your last commit. 
*(git diff, should also be displayed passively in IDEs with Git integration)*
- You'd like the ability to view, and in a worst-case scenario, revert back to, previous versions 
of your codebase. *(git revert)*
- Similarly, being able to easily and safely apply and unapply a set of temporary changes to
see what the difference looks like. *(git stash)*
- You'd like to be able to fluidly upload and download a record of your changes to/from the web,
in the off-chance your filesystem explodes. *(remote repository like GitHub)*

I would consider having the ability to do all of these an integral part of development QOL. 
Better to have it and not need it than the other way around, after all.

<aside>
* There is a GUI (Graphical User Interface) version of Git available for users who don't wish to 
learn the command line commands. I find it much faster to use the command line instead, but it is an option 
available to you to help you learn the basic workflow.
</aside>

Git can be intimidating to learn 
due to the combination of having a very long command list available and being invoked through the 
command line\*, but these are things that can be overcome. Take a bit of time, learn something new,
and you'll have a handy new tool in your toolbox.

Here is a summary of the basic Git workflow most of your Git usage will resemble:

```sh
# In the root of your project folder, generate a Git repository
# Only need to do this once
git init

# Get a look at the list of changed/uncommitted files
git status

# Add a single file to the list of files to be committed
git add filename.txt

# OR Use a pattern and the wildcard * to add all files starting with "filename."
# to the list of files to be committed
git add filename.* 

# OR What the heck, add all files
git add . 

# Look again, check which files have been staged
git status 

# Ok, not that file, put that one back
# Unstage this file from the list of files to be included in the next commit
# Maybe write a .gitignore file to automatically prevent this one from ever getting added
git restore -f passwords.txt

# Tie it all up in a bow with a message
git commit -m "Descriptive message summarizing this chunk of changes"

# View the newly-updated commit log for your sanity
git log
```

Some other useful commands include:
```sh
# Split into a new development branch (eg. for experimental feature
# or a new version release)
# For your personal work, you might not need this, but it is an option
git branch
git branch list
git checkout

# Revert to last commit, saving all uncommitted changes away
# on an invisible stack saved by Git
git stash

# Put the last-stashed set of changes back into play, popping it off
# the stack of set-of-changes stored by Git
git stash pop

# Quickly undo all uncommitted changes to a file
# Same as the command to unstage a file from the list of files to be included in a commit
# but with no -f flag, try to avoid using this accidentally
# ! This can erase all your changes even if saved, and can't be undone, be VERY careful
git restore filename.txt

# Read more about this and soft/hard reverts at your own leisure
git revert
```

### Additional reading:
- [https://docs.github.com/en/get-started/using-git/about-git](https://docs.github.com/en/get-started/using-git/about-git)
- [https://docs.github.com/en/get-started/quickstart/set-up-git](https://docs.github.com/en/get-started/quickstart/set-up-git)
- [https://training.github.com/downloads/github-git-cheat-sheet/](https://training.github.com/downloads/github-git-cheat-sheet/)


## At Last

Excellent. The necessary software has been installed, and your dev tools are all in place.
You're ready to write some code.

Let's get started.

...Next time.

![](/assets/images/article05/pewgf.png)

[TODO: EDIT LINK TO #1]
<a href="" align="right"> >> Next: #1: Inputs</a>
