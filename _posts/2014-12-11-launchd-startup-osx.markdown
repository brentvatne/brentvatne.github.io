---
layout: post
title: launchd for startup scripts on OSX - dark dock, light toolbar
class: launchd-osx-startup
description: Learn how to use launchd/launch to write startup scripts, and make OSX Yosemite look prettier in the process.
---

I held off upgrading to Mavericks because I didn't want to spend the
several hours fixing various development tools, but Yosemite finally got
me because two of my favorite tools -
[Sketch](http://bohemiancoding.com/sketch/) and
[Droplr](https://droplr.com/) made their newest versions exclusive to
Yosemite. Overall I'm very happy with the update - it looks better and
runs faster (subjective and anecdotal, I know, YMMV). One of the first
system preferences I attempted to change upon install was the dock
color: it is now has a white background, which makes it stand out a bit
too much, and overall just offends my right brain in a way that its
unable to articulate. But you can't change the dock color without also
changing the system toolbar color!

A quick Googling revealed that [this is simple to fix through a few
terminal commands](http://robservatory.com/yosemite-dark-dock-and-app-switcher-with-light-menu-bar/),
which can just be called as a shell script. But how do I get this to run
automatically each time I restart my machine?

### launchd, launchctl

> launchd manages the daemons at both a system and user level. Similar
> to xinetd, launchd can start daemons on demand. Similar to watchdogd,
> launchd can monitor daemons to make sure that they keep running.
> launchd also has replaced init as PID 1 on Mac OS X and as a result it
> is responsible for starting the system at boot time.

> launchctl is a command line application which talks to launchd using
> IPC and knows how to parse the property list files used to describe
> launchd jobs, serializing them using a specialized dictionary protocol
> that launchd understands. launchctl can be used to load and unload
> daemons, start and stop launchd controlled jobs, get system
> utilization statistics for launchd and its child processes, and set
> environment settings.

Source: [Wikipedia](http://www.wikiwand.com/en/Launchd)

You've probably used this before when installing tools that you'd like
to run constantly, such `postgres`. This seems like a good fit. Let's do
this.

- Download [dock.sh](https://gist.github.com/brentvatne/632041136e2fb40527ee) to wherever you'd like
to store custom startup scripts.
- Create a `.plist` file to load with `launchctl` in
  `~/Library/LaunchAgents` - mine is
`~/Library/LaunchAgents/com.brent.dock.plist` and copy
[this](https://gist.github.com/brentvatne/6ba6a5ff7bac76dfbf6e) into it,
replacing the text where necessary for the name of the service and the
path to the script.

### Using this for other things

If you take a look at the files, you will see that this is pretty simple
stuff and very flexible. In `dock.sh` we just do some shell commands,
you could just as easily make this a Ruby or Node script.

In `com.brent.dock.plist` the key (pun not intended) key is `RunAtLoad`
which just tells `launchd` to execute the service when it loads it. We
specify the script path in the `ProgramArguments`.

More details on `plist` configuration for `launchctl` can be found in
the [Daemons and Services Programming Guide in the Mac Developer
Library](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html#//apple_ref/doc/uid/10000172i-SW7-BCIEDDBJ)
