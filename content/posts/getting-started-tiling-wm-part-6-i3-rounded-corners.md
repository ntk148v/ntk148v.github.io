---
title: "Getting Started Tiling Wm [Part 6] I3 Rounded Corners"
date: 2022-01-10T13:45:01+07:00
tags: ["tech", "tiling-vm", "linux"]
comments: true
---

According to [Reddis post](https://www.reddit.com/r/unixporn/comments/7h0rm0/what_would_you_want_in_a_wm/) and [i3-gaps issue](https://github.com/Airblader/i3/issues/167), it seems like a lot of people would like this. But Airblade - i3-gaps maintainer [doesn't like it](https://github.com/Airblader/i3/issues/167#issuecomment-328562433). But nevermind, we still have two ways to achieve it.

## 1. Rounded i3-gaps

[Resloved](https://github.com/resloved) have an awesome [fork](https://github.com/resloved/i3) to implement rounded corners. I fork it again to keep it up-to-date with the upstream i3-gaps. You can check it [here](https://github.com/ntk148v/i3).

- Install i3-gaps.

```bash
# Dependencies
sudo apt install git libxcb1-dev libxcb-keysyms1-dev libpango1.0-dev \
    libxcb-util0-dev libxcb-icccm4-dev libyajl-dev \
    libstartup-notification0-dev libxcb-randr0-dev \
    libev-dev libxcb-cursor-dev libxcb-xinerama0-dev \
    libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev \
    autoconf libxcb-xrm0 libxcb-xrm-dev automake i3status \
    ninja-build meson libxcb-shape0-dev build-essential -y
git clone https://github.com/ntk148v/i3.git
cd i3/
# Compile
mkdir -p build && cd build
meson ..
ninja
sudo ninja install
```

- Configure rounded corners by adding this to your config.

```
border_radius 10
```

- Result, but you can see that these corners look so jagged (check out this [comment](https://github.com/Airblader/i3/issues/167#issuecomment-485263770)).

{{< figure class="figure" src="/photos/getting-started-tiling-wm-part-6/rounded-corners-1.png" >}}

{{< figure class="figure" src="/photos/getting-started-tiling-wm-part-6/rounded-corners-4.png" >}}

## 2. picom

- You can also achieve rounded corners by using [picom](https://github.com/yshui/picom).
- Install picom.
- Configure rounded corners by changing `corner_radius` value.

```
# Path ~/.config/picom.conf
corner_radius: 10;
```

- Result looks much smoother.

{{< figure class="figure" src="/photos/getting-started-tiling-wm-part-6/rounded-corners-2.png" >}}

{{< figure class="figure" src="/photos/getting-started-tiling-wm-part-6/rounded-corners-3.png" >}}
