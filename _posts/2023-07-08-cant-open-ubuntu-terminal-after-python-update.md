---
published: true
title: Failed to Open Ubuntu Terminal After Upgrading Python Version
tags: Ubuntu Tips
---
Failed to open the terminal on your ubuntu system after upgrading python3 version? Here's a way to potentially heal your pool command line!

(*I tried this on Ubuntu 18.04.6 LTS, but I'm expecting it to work on other versions of Ubuntu.*)

(1) Take a picture of this how article with your phone (or open it on some device other than your pool ubuntu system), since you're about to press ***ctrl + alt + f3*** to open up a TTY window. 

(2) Login to your account (with the same username + password when you login to the ubuntu system).

(3) Type in ***update-alternatives --list python3*** and hit enter, thus you can see all the python3 versions you've installed, keep them in mind.

(4) Type ***sudo vim /usr/bin/gnome-terminal*** and hit enter, so that you can modify the settings of your gnome terminal. 

(5) The first line of this file should be ***!/usr/bin/python3***, you should change it to ***!/usr/bin/python3.x*** where ***x*** is a number from the versions of python3 you've just checked. For example, if in step 3, you saw that python3.6, python3.8 are both installed, here you can type either 6 or 8 as ***x***, one of them should hopefully work. 

(6) Note that you can press ***i*** to switch from normal mode to insert mode in VIM, then make the change. After you're done, press ***esc*** to switch back to normal mode, and then type ***:x*** and hit enter to save and close the file. It's NOT conpulsary to use VIM in step 4, but ***sudo*** is necessary cause it ensures privilege.

(7) Press ***alt + f2*** to switch back to desktop, and try opening the terminal. If failed, you may repeat step 1, 4, 5 to try all of the python3 versions you've installed, hopefully one of them would work. 

(8) After you've fixed the terminal, you can press ***ctrl + alt + f3*** to go back to TTY again, and type ***exit***, hit enter, to logout yourself in TTY. Again, you can then use ***alt + f2*** to switch back to your desktop.

## Background
This post is triggered by the insidence I encountered a few hours ago. After I tried to follow [this video](https://www.youtube.com/watch?v=fEOkpp2uo6g) to upgrade my python3 from 3.6 to 3.8, my terminal suddenly stopped working. Upgrading python3 and pip3 version on ubuntu appears to be an annoying task for me, I haven't figured out a clean way of doing these yet.

## Reference
* https://www.groovypost.com/howto/cant-open-terminal-in-ubuntu-fixes/
