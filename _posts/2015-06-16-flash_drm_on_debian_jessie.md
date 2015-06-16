---
tags: debian, hal, flash, drm, amazon
title: Watch DRM-Protected Content on Debian Jessie
---

I have recently installed Debian 8.0 (codenamed Jessie) and like many other distros I have to gradually fix problems that I came across. Here is how I managed to watch Amazon's prime videos using Iceweasel.


Trying to watch Amazon's videos which are DRM-protected results in an error pages in Iceweasel on Debian. Flash is installed on my machine but DRM content
on Flash requires HAL to play. The problem is that HAL is deprecated and is not used anymore by major distributions including Debian.

To make it work first I had to install a modified libhub library. This library does not provide full HAL functionality (which I don't need) instead it has the bare minimum to let the Flash to decode the DRM content. This package is called [hal-flash](https://aur.archlinux.org/packages/hal-flash/) and is available for [download](https://github.com/cshorler/hal-flash/archive/v0.3.0.tar.gz).

    $ tar xzf hal-flash-0.3.0.tar.gz
    $ cd hal-flash-0.3.0

This package has a few dependencies that should be installed first. Please pay attention that I have already installed other build related packages on my machine so you might experience other errors that I did not have.

    $ sudo apt-get install dbus udisks2 autoconf git pkgconfig

At this stage I tried autoconf to generate configure script but it failed. So I found a [workaround](https://bbs.archlinux.org/viewtopic.php?pid=1258673#p1258673) which suggests doing the following to have a working configure script.

    $ sudo apt-get install pkg-config xorg-server-devel libtool automake
    $ libtoolize --force
    $ aclocal
    $ autoheader
    $ automake --force-missing --add-missing
    $ autoconf

At this stage I had a working configure script.

    $ ./configure
    $ make

I'm used to install compiled source codes with `checkinstall` script. It let the package manager to track the installed files.

    $ sudo apt-get install checkinstall
    $ sudo checkinstall make install

Make sure to change the version number, otherwise the install script will complain that version number can't be a string. If everything goes well we should have the libhal.so installed in system library path.

    $ ls -l /usr/local/lib/libhal.so
    lrwxrwxrwx 1 root staff 15 Jun 13 11:01 /usr/local/lib/libhal.so -> libhal.so.1.0.0

Before trying the Amazon videos we need to remove existing Flash files and make sure that the udisks2 service is running.

    $ systemctl start udisks2.service
    $ systemctl status udisks2.service
    ● udisks2.service - Disk Manager
       Loaded: loaded (/lib/systemd/system/udisks2.service; static)
       Active: active (running) since Sa 2015-06-13 10:59:43 CEST; 1 day 2h ago
         Docs: man:udisks(8)
     Main PID: 26013 (udisksd)
       CGroup: /system.slice/udisks2.service
               └─26013 /usr/lib/udisks2/udisksd --no-debug

And removing the files.

    $ rm -rf ~/.adobe/Flash_Player/*

I did a `killall iceweasel` and after that I was able to watch the videos. I hope it works well for you as well!

At the end I have to give credit to awesome people at Archlinux for their perfect documentations.
