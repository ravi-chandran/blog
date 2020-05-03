
![](/images/python/cmdtimedemo.gif)

It's amazing how a trivial search for something that should obviously exist spirals out of control...

It all started with this. I was tinkering with some Python machine learning scripts which took a while to run, and wanted to know how long they took to run. Unfortunately, I was running my scripts on a Windows command prompt so that Tensorflow could access my PC's NVIDIA GPU. Under Linux via VirtualBox, GPU access doesn't work. Hence my need to use Windows (or get another native Linux machine...)

Under Linux, there's the well known `time` utility that can tell you how long a command took to run. However, I was surprised to find no equivalent utility available in the Windows 10 command prompt. Yes, there are a myriad of solutions for this: use PowerShell, or install Cygwin, or install Bash for Linux, or install one of several old binaries that provide the solution that used to exist on some long-forgotten Windows version. I'm wary of installing anything, especially old binaries of dubious origin. Of course, there's also the more cumbersome approach of simply adding the timing functionality into the Python scripts themselves. It'd be just a few lines of code.

Hey, why not just create a Python console script that does something like what the Linux `time` utility does? It'd only be a few lines of code, right? And so, that's how this little detour started...

<!-- TEASER_END -->

## Let's `twine` It To PyPI
I've called this little utility `cmdtime`. It's based on a tiny Python [script](https://github.com/ravi-chandran/cmdtime/blob/master/cmdtime/cmdtime.py) that's only about a dozen lines long. Once `pip` installed, you can time a Python script or any other terminal command by just prefixing the command with `cmdtime`. I decided to make it open source via [GitHub](https://github.com/ravi-chandran/cmdtime) and available via [PyPI](https://pypi.org/project/cmdtime/). Actually, I checked on PyPI first to make sure the name `cmdtime` wasn't already taken. And also made sure there were no other older utilities on Windows with the same name to avoid confusion. In addition, I tried it out on [TestPyPI](https://test.pypi.org/) first before [`twine`](https://github.com/pypa/twine)-ing it to its final resting place at [PyPI](https://pypi.org/project/cmdtime/).

## Let's `pytest` It
Okay, now that I have that, wouldn't it be nice to add unit tests just as an exercise?

So I did that with [`pytest`](https://github.com/pytest-dev/pytest). There was a little bit of research to figure out how best to test a console application. I settled on using Python `subprocess.run()` to do it. The resulting [test script](https://github.com/ravi-chandran/cmdtime/blob/master/tests/test_cmdtime.py) is quite simple.

## Let's Test Multiple Python Versions With Travis CI
The next question was: Wouldn't it be nice if this was automatically tested under multiple Python versions via [Travis CI](https://travis-ci.org/)?

So I ventured into that and noticed that Travis CI only supported Python under Linux at present. Although Windows testing is what would make sense for this application, I added the tests under Linux anyway. This almost doubled the number of unit tests. Each test had to be crafted specifically for the given OS as the syntax of the terminal commands that I timed were slightly different between Windows and Linux.

In the interest of time (no pun intended), I gave up on further adapting the tests for Python versions below 3.7 as the `subprocess.run()` functionality had changed enough between 3.5, 3.6 and 3.7 to make it painful.

## What About Python Testing Under Windows With Travis CI?
`cmdtime` is meant for Windows users. So it would have been a tragedy indeed if the unit tests were only possible under Linux. Luckily, I found [this](https://gist.github.com/shaypal5/7fd766933fb265af6f71a88cb91dd08c) which explained how Travis CI could be used to test Python under Windows. One simply had to use [Chocolatey](https://chocolatey.org/) to install Python on the Windows VMs.

With the necessary updates to the [Travis CI configuration file](https://github.com/ravi-chandran/cmdtime/blob/master/.travis.yaml), I finally attained the  glorious 100% tests passing state across both Windows and Linux for multiple Python versions, albeit for a trivial application. And never mind that no one's going to use this little utility under Linux... (And at least one known user for Windows...)

## How About Those Flashy GIFs For The README?
After coming this far, you can't just stop there. Wouldn't it be nice to add in a dynamic GIF to the `README` that provides a little demo of what this does? 

[Lots of tools are available](https://intoli.com/blog/terminal-recorders/) to do this but almost all for Linux. And nothing Python-based that did what I wanted. Eventually, I settled on [Terminalizer](https://terminalizer.com/) which seemed to be actively maintained and did it's job under Windows well.

I had to experiment with the settings to change the default which used PowerShell for recording. I got it to launch the Windows command prompt instead. Also had to tinker with the settings to get rid of the Mac look which would've been a really weird way to showcase this Windows-focused utility. My only real gripe about Terminalizer is that it automatically assumes you want to upload the recording and I had to break out of that every time. I didn't find an option to disable this annoyance. It's use of YAML for the recording is a great idea and allowed me to adjust the delays between events in the recording. The result was the snappy little demo GIF at the beginning of this blog.

## Finishing Touches
Add in the cool "build passing" badge from Travis CI [![Build Status](https://travis-ci.org/ravi-chandran/cmdtime.svg?branch=master)](https://travis-ci.org/ravi-chandran/cmdtime) and hope that it continues to pass...

Only one lonely badge? To keep it company, let's add in the MIT License badge:
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Okay, I'm done now. I promise.
