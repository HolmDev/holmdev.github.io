---
title: "Creating an RGB Keyboard Legend"
date: 2022-06-15
author: "Fredholm"
draft: false
---
I recently found [this](https://libreddit.spike.codes/r/unixporn/comments/hgba3b/i3_razer_blade_stealth_highlighting_shortcuts_and/) post, where user u/Rasie1 had created a script for displaying their i3 keybinds when certain modifier keys were pressed. I thought it looked really dope and that it would fit nicely with the rest of my keyboard-centric setup, so I made my own.

I decided to use [OpenRGB](https://openrgb.org/) for controlling my Blackwidow V2's RGB, since it fits my needs very well and has been my main RGB solution since the beginning of my Linux journey. I also decided to write the program in python, since I could finish it quite quickly then.

I decided to tackle this task by letting OpenRGB switch between different modes corresponding to certain combinations of modifier keys. That way I could customize the lights with a GUI instead of messing with a YAML/JSON file that would get really messy really fast.

During a first run, the program will create the needed profiles in OpenRGB. After that, you can go in and customize them in the OpenRGB application.

### Important {.important}
You need to have the OpenRGB server running for this program to work. Launch it with the `--server` option to start the server.

The code is:

``` python
#!/bin/python
from openrgb import OpenRGBClient
from pynput.keyboard import Key, Listener

class Handler():
	def __init__(self):
		self.mod = False
		self.ctrl = False
		self.shift = False
		self.rgb = OpenRGBClient()

		for i in ["Mod", "Mod+Shift", "Mod+Ctrl", "None"]:
			try:
				self.rgb.load_profile(i)
			except ValueError:
				self.rgb.save_profile(i)

	def updateKeys(self, key, press):
		if key == Key.cmd:
			self.mod = press
		if key == Key.ctrl:
			self.ctrl = press
		if key == Key.shift:
			self.shift = press
		self.choose_profile()

	def choose_profile(self):
		if self.mod:
			if not (self.shift or self.ctrl):
				self.rgb.load_profile("Mod")
			elif self.shift:
				self.rgb.load_profile("Mod+Shift")
			elif self.ctrl:
				self.rgb.load_profile("Mod+Ctrl")
		else:
			self.rgb.load_profile("None")

if __name__ == "__main__":
	h = Handler()
	def on_press(key):
		h.updateKeys(key, True)

	def on_release(key):
		h.updateKeys(key, False)

	with Listener(on_press=on_press, on_release=on_release) as listener:
		listener.join()
```

Result:
![Result](result.gif)

### Note {.note}
It is also quite nice to have it launch on startup, so I made a special .desktop file that launches OpenRGB with the `--server` option alongside this program and placed it in `~/.config/autostart/`.
