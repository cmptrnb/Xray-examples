# How to install i3 on Debian

## Step 1. Make the console font larger

```
sudo dpkg-reconfigure console-setup
```

Here are some suggested settings:

* Encoding to use on the console: UTF-8
* Character set: Latin1 and Latin5
* Font for the console: Terminus Bold
* Font Size: 11x22	


## Step 2. Install the required packages

```
sudo apt install xorg i3 kitty firefox-esr
```

Notice that installing the kitty package automatically updates the preferred alternative terminal emulator to be kitty.



## Step 3. Make X execute i3

Copy the model xinitrc file into your home directory:

```
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

Edit your personal xinitrc file:

```
vi ~/.xinitrc
```

Append a line to execute i3 when X starts:

```
exec i3
```

Write the file to disk, and quit the editor.


## Step 4. Start i3 for the first time

```
startx
```

1. Press Enter to generate the i3 configuration
2. Press Enter again to use the "Window" (or "Super") key as your modifier key in i3

To exit i3, press Win + Shift + e, and confirm it by clicking the confirmation button.



## Step 5. Make the kitty font larger

Copy the model kitty configuration file into your home directory:

```
cp /usr/share/doc/kitty/examples/kitty.conf ~/.config/kitty/
```

Edit your kitty configuration:

```
vi ~/.config/kitty/kitty.conf
```

Change the font size, for example:

```
font_size 14.0
```

Write the file to disk, and quit the editor.



## Step 6. Make the i3 font larger

Edit your i3 config file:

```
vi .config/i3/config
```

Change the font size, for example:

```
font pango:monospace 12
```

Change the effect of Ctrl + d so that only desktop apps are displayed on the dynamic menu:

```
bindsym $mod+d exec --no-startup-id i3-dmenu-desktop
```

Write the file to disk, and quit the editor.



## Step 7. Learn the basics of i3

Start X again with your new settings:

```
startx
```

Press Win + Enter to open a terminal emulator window.

Press Win +  d to bring up the dynamic menu, then press Enter on firefox-esr to open a Firefox window.

Use Win + the arrow keys on your keyboard to move between windows.

Use Win + Shift + the arrow keys on your keyboard to move the windows themselves.


## Note on how to copy and paste in i3

Highlight text to copy it to the clipboard.

Press Shift + Insert to paste from the clipboard.
