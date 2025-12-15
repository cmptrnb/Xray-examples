# Install i3 tiling window manager on command-line Debian


## Step 1. Install software

Install X, i3, the URxvt terminal emulator, and the Firefox extended support release:

````
sudo apt install xorg i3 rxvt-unicode firefox-esr
````


## Step 2. Configure X

Copy the Xinit model rc file to your own dot file

````
cp /etc/X11/xinit/xinitrc ~/.xinitrc
````

Edit your dot file:

````
vi ~/.xinitrc
````

Append a line to execute i3 when X starts:

````
exec i3
````

Write the file to disk, and quit the editor.

## Step 3. First time into i3

Start X for the first time, creating your i3 configuration dot file, and specifying the Windows key on your keyboard as your i3 mod key:

````
startx
````

## Step 4. Configure i3

Press the Windows key and Enter to open a terminal emulator window.

Edit your i3 config file:

````
vi ~/.config/i3/config
````

Configure i3 to use a larger font: 

````
font pango:monospace 12
````

Configure dmenu to display only programs designed for the desktop. Replace `dmenu` with `i3-dmenu-desktop`.

Write the i3 config file to disk, and quit the editor.


## Step 5. Configure X resources

Edit your X resources file:

````
vi ~/.Xresources
````

Insert lines specifying the font, font size, and colors for the URxvt terminal emulator.

````
URxvt.font: xft:DejaVu Sans Mono:pixelsize=18
URxvt.background: #000000
URxvt.foreground: #00ff00
````

Write the file to disk, and quit the editor.

Incorporate your X resources into the X database:

````
xrdb ~/.Xresources
````

## Step 6. Reboot

```
sudo reboot
```


## Step 7. Launch into i3

Enter i3 with your basic configuration. 

Press the Windows key plus Enter to open a terminal emulator window. 

Press the Windows key plus D to bring up dmenu, then select Firefox to open a new Firefox window.

You can now go ahead and customize i3 to your liking.

## Copy and paste

To copy and paste from Firefox to URxvt:

* In Firefox, just highlight text to copy it to the clipboard
* In URxvt, do Shift + Insert to paste from the clipboard

To copy and paste from URxvy to Firefox:

* In URxvt, select text and do Ctrl + Alt + c to copy it to the clipboard
* In Firefox, do Ctrl + v to paste from the clipboard
