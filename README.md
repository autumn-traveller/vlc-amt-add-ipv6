# Extending VLC's AMT Module for IPv6
Nicolas Zunker
INET Project 2022 Summer Semester

## Overview

This repository contains a fork of VLC's [master](https://code.videolan.org/videolan/vlc) branch.

All the changes are to the [amt module](./modules/access/amt.c) to extend it for IPv6.

The old (master) README.md is [here](./oldREADME.md)

## How to build

This project was built and tested on linux, however it is known to have worked on mac as well. The specific changes introduced by this fork do not affect the build process at all so in principle anyone who has a working setup will not be affected, and anyone building for the first time can follow VLC's [official guide](https://wiki.videolan.org/Category:Building/) for compilation because nothing changed. 

#### Quickstart

(Assuming linux)
If building for the very first time you need to run `./bootstrap` and then `./configure`, finally run `make`

## Execution

After building execute `./vlc [[IPv6 address of source / empty if it can be any source (ASM)]@[IPv6 address of group]] --amt-relay [[IPv6 Address of an AMT relay (IPv6 addresses need to go inside square brackets)]]`

Since all the changes are to the AMT module, in order to test the changes you need to find an IPv6 multicast stream (and probably an AMT relay too).

One way to create such a setup is by running a stream yourself, for example with the `ffmpeg` command: `fmmpeg -re -stream_loop 1000 -i [some .mpg or .mp3 media file, alternatively .mp4] -c copy -f ["mpeg" if using .mpg or mp3, "mpegts" if using .mp4] udp://[some IPv6 group address]:[some port]` . Running this command in a local, multicast enabled, network will let you test that local multicast works. Running this command on a machine in a network which you are not able to reach over multicast will let you test AMT, provided that there is an AMT relay in that network. The AMT relay implementation [here](https://github.com/GrumpyOldTroll/amtrelayd) is recommended - execute with `sudo src/amtrelayd -n inet6 -l inet6 -a [IPv6 address to listen on] -c [interface name]` from the project root.

#### Examples (commands to reproduce the demo)

`./vlc -vvv "amt://[fd00:1::15b]@[ff3e::3]:1234" --amt-native-timeout 1 --amt-relay "[fd00:1::1b1]"` on local machine

`ffmpeg -stream_loop 500 -re -i allegro.mp3 -c copy -f mpeg udp://[ff3e::3]:1234` on streaming machine with IP `fd00:1::15b`

`sudo src/amtrelayd -n inet6 -l inet6 -a fd00:1::1b1  -c enp1s0 -d 1` in the AMT relay machine with IP `fd00:1::1b1`

In the demo there was no way for the local machine to receive native multicast from the streaming machine. This was achieved by using virtual machines on the same host but could also be achieved in other ways...

## Source Code sitemap
```
ABOUT-NLS          - Notes on the Free Translation Project.
AUTHORS            - VLC authors.
COPYING            - The GPL license.
COPYING.LIB        - The LGPL license.
INSTALL            - Installation and building instructions.
NEWS               - Important modifications between the releases.
README             - Project summary.
THANKS             - VLC contributors.

bin/               - VLC binaries.
bindings/          - libVLC bindings to other languages.
compat/            - compatibility library for operating systems missing
                     essential functionalities.
contrib/           - Facilities for retrieving external libraries and building
                     them for systems that don't have the right versions.
doc/               - Miscellaneous documentation.
extras/analyser    - Code analyser and editor specific files.
extras/buildsystem - Different build system specific files.
extras/misc        - Files that don't fit in the other extras/ categories.
extras/package     - VLC packaging specific files such as spec files.
extras/tools/      - Facilities for retrieving external building tools needed
                     for systems that don't have the right versions.
include/           - Header files.
lib/               - libVLC source code.
modules/           - VLC plugins and modules. Most of the code is here. The `access` module contains the `amt.c` file which has all the relevant code/changes from this project
po/                - VLC translations.
share/             - Common resource files.
src/               - libvlccore source code.
test/              - Testing system.
```
