+++
title = "Set Animated Gif as Wallpaper"
date = 2021-01-14T09:36:34+07:00
lastmod = 2021-01-14T09:36:34+07:00
tags = ["linux", "tips", "tech"]
comments = true
toc = true
+++

> **NOTE**: Environment Ubuntu 20.04

## Dependencies

- [Xwinwrap](https://github.com/ujjwal96/xwinwrap):

```bash
sudo apt-get install xorg-dev build-essential libx11-dev x11proto-xext-dev libxrender-dev libxext-dev
git clone https://github.com/ujjwal96/xwinwrap.git
cd xwinwrap
make
sudo make install
make clean
```

- Gifsicle:

```bash
sudo apt install gifsicle
```

## The helper script

A helper script to setup animated .gif in dual monitors.

```bash
#!/bin/bash
# Uses xwinwrap to display given animated .gif in dual monitors.
if [ $# -ne 1 ]; then
    echo 1>&2 Usage: $0 image.gif
    exit 1
fi
gif=$1
killall -9 xwinwrap
killall -9 gifview
# Get monitors resolution
SCR1=`xrandr | awk '/primary/ && /connected/ { print $4 }'`
SCR2=`xrandr | awk '!/primary/ && /connected/ { print $3 }'`

xwinwrap -g $SCR1 -ov -ni -s -nf -- gifview -w WID $gif -a &
xwinwrap -g $SCR2 -ov -ni -s -nf -- gifview -w WID $gif -a &
```

If you want to run xwinwrap by yourself, here is the example:

```bash
xwinwrap -g 1920x1080 -ov -ni -s -nf -- gifview -w WID /full/path/to/gif -a
```
