---
title: "Want to increase productivity! Start with managing your screen real-estate first "
description: "Manage your desktop screen effeciently with yabai and skhd"
date: 2021-01-08T17:51:47+11:00
tags: [devops, SRE, productivity]
categories: [DevOps, SRE]
draft: false
---
Are you using all the real-estate that screen has to offer ?

Do you want your screen to look from this mess
 <!--more  -->

![Unmanaged](/img/unmanaged.png)

To this fully utilized clutter free
![Managed](/img/managed.png)


I am sure people who have used i3 know the importance of managing screen real-estate to its otmost potential. While i3 is a great tiling window manager of Linux and is really great for managing your workspaces. 

The basic idea is that you should be using all the real-estate your desktop has to offer, all the time. 

In principle, whenever you run a program, it should take all of the screen, and if you open another program, screen space should be cut in half, so that both programs will take half.  You can continue this as much as you can and as much you find it suitable.

While mac does not support i3, there are tools which you can use to replicate that. These tools are called:

* Yabai 
* skhd 

### Yabai

Yabai is a good application that allows you to do tiling and manage window sizing. The main function of yabai is to manage your tiling window, automatically modify your window layout using binary space partitioning algoritgh.

### skhd

skhd is a simple hot key daeomon for macOS that create keyboard shortcuts. Like alt + W will create a new window. 

## Install yabai

It is really simple. 

Some tutorial will ask you to disable system integrity protection, you can just ignore that. I have been doing the same, and has not affected me a bit.

download and install brew if you do not have

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

Install Yabai
```
brew install koekeishiya/formulae/yabai
```

Start yabai service
```
brew services start yabai
```

It will then ask you to allow yabai accessibility permission. Go to your security and privacy > Privacy > Accessibility and allow yabai access by checking it.


## Install skhd

To install skhd just run
```
brew install koekeishiya/formulae/skhd
```

Start skhd service
```
brew services start skhd

```

This is alos ask you security permission to allow skhd accessibility permission. Go to your security and privacy > Privacy > Accessibility and allow skhd access by checking it.

Once you are done with that, now we need to configure it.

## Configuration

### Confguring yabai

follow this command
```
touch ~/.yabairc
chmod +x ~/.yabairc
```

Add following code to .yabairc file
```
#!/usr/bin/env sh

# the scripting-addition must be loaded manually if
# you are running yabai on macOS Big Sur. Uncomment
# the following line to have the injection performed
# when the config is executed during startup.
#
# for this to work you must configure sudo such that
# it will be able to run the command without password
#
# see this wiki page for information:
#  - https://github.com/koekeishiya/yabai/wiki/Installing-yabai-(latest-release)
#
# sudo yabai --load-sa
# yabai -m signal --add event=dock_did_restart action="sudo yabai --load-sa"

# global settings
yabai -m config mouse_follows_focus          off
yabai -m config focus_follows_mouse          off
yabai -m config window_placement             second_child
yabai -m config window_topmost               off
yabai -m config window_shadow                on
yabai -m config window_opacity               off
yabai -m config window_opacity_duration      0.0
yabai -m config active_window_opacity        1.0
yabai -m config normal_window_opacity        0.90
yabai -m config window_border                off
yabai -m config window_border_width          6
yabai -m config active_window_border_color   0xff775759
yabai -m config normal_window_border_color   0xff555555
yabai -m config insert_feedback_color        0xffd75f5f
yabai -m config split_ratio                  0.50
yabai -m config auto_balance                 off
yabai -m config mouse_modifier               fn
yabai -m config mouse_action1                move
yabai -m config mouse_action2                resize
yabai -m config mouse_drop_action            swap

# general space settings
yabai -m config layout                       bsp
yabai -m config top_padding                  12
yabai -m config bottom_padding               12
yabai -m config left_padding                 12
yabai -m config right_padding                12
yabai -m config window_gap                   06

echo "yabai configuration loaded.."

```


### Configuring skhd
```
touch ~/.skhdrc
chmod +x ~/.skhdrc
```

Add following code to the .skhdrc file
```

# Navigation
alt - h : yabai -m window --focus west
alt - j : yabai -m window --focus south
alt - k : yabai -m window --focus north
alt - l : yabai -m window --focus east

# Moving windows
shift + alt - h : yabai -m window --warp west
shift + alt - j : yabai -m window --warp south
shift + alt - k : yabai -m window --warp north
shift + alt - l : yabai -m window --warp east

# Make window native fullscreen
alt - f         : yabai -m window --toggle zoom-fullscreen
shift + alt - f : yabai -m window --toggle native-fullscreen

# toggle window split type
alt - e : yabai -m window --toggle split

# Move focus container to workspace
shift + alt - m : yabai -m window --space last && yabai -m space --focus last
shift + alt - p : yabai -m window --space prev && yabai -m space --focus prev
shift + alt - n : yabai -m window --space next && yabai -m space --focus next
shift + alt - 1 : yabai -m window --space 1 && yabai -m space --focus 1
shift + alt - 2 : yabai -m window --space 2 && yabai -m space --focus 2
shift + alt - 3 : yabai -m window --space 3 && yabai -m space --focus 3
shift + alt - 4 : yabai -m window --space 4 && yabai -m space --focus 4
shift + alt - 5 : yabai -m window --space 5 && yabai -m space --focus 5
shift + alt - 6 : yabai -m window --space 6 && yabai -m space --focus 6

# Kill a container
shift + alt - q : yabai -m window --close

# Float or unfloat a window
shift + alt - space : yabai -m window --toggle float
```


## Restart yabai and skhd
```
brew services restart skhd
brew services restart yabai
```

Now you are ready to properly use all of you screen tiles. Do not worry, it will take some time to get hang of it, but after one two days you will get used to it.