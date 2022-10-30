---
title: Fetching Xresources made simple
date: 2022-10-30
author: Fredholm
draft: false
---

As can be seen in previous posts, I use a standalone window manager. I really love being in control of every little small thing and building things myself. I have also really grown to like when systems work together and especially when they work together *dynamically*. One of these systems is Xresources, which have allowed me to sync my themes over multiple applications/WMs. However, not every application/WM has support for accessing and trying to hack together a solution for connecting them is suboptimal. This is an excerpt from one of my previous Qtile configs.

```python
# {{{ Xresources hack
import subprocess

xfetch = subprocess.Popen("sh -c 'xrdb -merge ~/.Xresources && xrdb -q'", stdout=subprocess.PIPE, shell=True)
(xoutput, xerr) = xfetch.communicate()
xfetch.wait()
xrdb = xoutput.decode().splitlines()
xdict = {}
for i in xrdb:
	temp = i.split(":\t")
	xdict[temp[0]] = temp[1]
xrdb = xdict
# }}}

widget_defaults = dict(
	font=xrdb["*font_type"],
	fontsize=int(xrdb["*font_size"]),
	padding=3,
	background=xrdb["*background"],
	foreground=xrdb["*foreground"],
)
```

There are several problems with this. First of, it's just ugly. Secondly, it creates a dependency of using shell subprocesses. Thirdly and most importantly, it lacks the support for wildcards, which is one of Xresources most beautiful features. But what is the solution to this. Should we just accept our fates, doomed to manually update our configs when we want to switch color schemes?

No, because I have created a tool for fetching Xresources in style.

Introducing, [xrcat](https://pypi.org/project/xrcat/), a CLI tool and Python module combo that allow you to access Xresources with a single command.

To install it just run:

```bash
$ pip install xrcat
```

Here you can see its usage as a CLI tool:

```bash
$ xrcat dk.background
#1d2021
```

Or in a python scripts:

```python
from xrcat import xrcat

# Load current resources into xrcat
xrcat.updateResources()

# Get the resource
print(xrcat.getResource("qtile.background"))
# Output: '#1d2021'
```

The nice thing with this tool is that it support wildcards. This means that if you for example want `dwm.background` and the value of it is None, xrcat will match it to `*.background`. Matches are prioritized based on length so for the example above `*.background` will be chosen instead of `*background` even though both are matches. This behavior is implemented to ensure that program specific resources are selected before general resources.

This mean that my abysmal Qtile config from above can be trimmed down to:

``` python
from xrcat import xrcat

xrcat.updateResources()

widget_defaults = dict(
	font=xrcat.getResource("qtile.font_type"),
	fontsize=int(xrcat.getResource("qtile.font_size")),
	padding=3,
	background=xrcat.getResource("qtile.background"),
	foreground=xrcat.getResource("qtile.foreground"),
)
```

I also use it in my dk config like this:

```sh
dkcmd set border \
	width=$(xrcat dk.border_width) \
	colour \
	focus=$(xrcat dk.color4) \
	unfocus=$(xrcat dk.color0) \
	urgent=$(xrcat dk.color1)
```

While I haven't tried out converting every last non-compliant application to use xrcat, I reckon quite a few could be configured with it. Of course this won't cover all possible applications, but combining Xresources, xrcat, pywal, and Oomox/Themix probably covers a vast majority of what you would want to configure.

Feel free to check out the project on [Github](https://github.com/HolmDev/xrcat) if you would like to contribute!
