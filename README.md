# CAPS2ESC <sub>a linux port</sub>

_Transforming the most useless key **ever** in the most useful one._
<sub>_For vi/Vim/NeoVim addicts at last_.</sub>

<a href="http://www.catonmat.net/blog/why-vim-uses-hjkl-as-arrow-keys/">

![ADM-3A terminal](http://www.catonmat.net/images/why-vim-uses-hjkl/lsi-adm3a-full-keyboard.jpg)

</a>

## What is it?

- **Put what's useless in its place**  
  <sub>_By moving the CAPSLOCK function to the far ESC location_</sub>
- **Make what's useful comfortably present, just below your Pinky**  
  <sub>_By moving both ESC and CTRL functions to the CAPSLOCK location_</sub>

## Why?!

Because CAPSLOCK is just "right there" and making it CTRL when key-chording and
ESC when pressed alone is quite handy, specially in vi.

## Dependencies

#### Arch Linux
- [libevdev][]

#### Ubuntu
- [libudev-dev][]
- [libevdev-dev][]

## Building

`gcc caps2esc.c -o caps2esc -I/usr/include/libevdev-1.0 -levdev -ludev`

## Execution

The following daemonized sample execution increases the application priority
(since it'll be responsible for a vital input device, just to make sure it stays
responsive):

`sudo nice -n -20 ./caps2esc >caps2esc.log 2>caps2esc.err &`

> Note the `./caps2esc` which assumes it is in the current directory, if you want to avoid cluttering your home directory you can place the `caps2esc` binary somewhere that is in your `PATH` environment variable (viewable by `echo $PATH`). I use `~/.local/bin/` so I copy the binary to that folder with `cp caps2esc ~/.local/bin/`. If the directory doesn't exist you can create it `mkdir -p ~/.local/bin/`.

If you want to run caps2esc with high priority on startup you can create a `~/.xprofile` file with the following contents:

```
#!/bin/bash
sudo nice -n -20 $(which caps2esc) > >(logger -s -t $(`which basename` $0) -p user.info ) 2> >(logger -t $(`which basename` $0) -p user.warn) &
```

> In order to run this `~/.xprofile` without being prompted for your password (i.e. automatically on startup) you will need to add the `nice` command to a file in `/etc/sudoers.d/`, I typically create a new one for the application that is requiring the permission, so in this case `/etc/sudoers.d/caps2esc`.
> The bits about `logger` are using this system built-in to write to the syslog/journald logging mechanism so that you can easily search for it using `grep -r caps2esc /var/log/syslog` or `journalctl | grep caps2esc`.

Use this command to edit an alternate sudoers file with validation of the syntax (you can break your system otherwise):
`sudo visudo -f /etc/sudoers.d/caps2esc`
> If there is an error with the syntax when saving and quitting you should edit the file again to correct this, for Vim this means hitting `e`, you can also open another sudoers file in another terminal window/editor to check for example syntax (if using Vim you can do it in a new pane in the current terminal using <kbd>Ctrl W</kbd><kbd>Ctrl S</kbd> then <kbd>:e /etc/sudoers</kbd> or <kbd>:e /etc/sudoers.d/OtherSudoersFile</kbd> if you have another custom file.

Final `/etc/sudoers.d/caps2esc` contents if you follow the hints above and put `caps2esc` in your `PATH`:
```
%users ALL=NOPASSWD: /usr/bin/nice -n -20 $(which caps2esc)
```

> There are some concerns with letting all `%users` use `nice` which is why I've limited it


## Installation

I'm maintaining an Archlinux package on AUR:

- <https://aur.archlinux.org/packages/caps2esc>

It wraps the executable in a systemd service that can be easily started, stopped
and enabled to execute on boot.

## How it works

Executing `caps2esc` without parameters (with the necessary privileges to access
input devices) will make it monitor any devices connected (or that gets
connected) that produces CAPSLOCK or ESC events.

Upon detection it will fork and exec itself now passing the path of the detected
device as its first parameter. This child instance is then responsible for
producing an uinput clone of such device and doing the programmatic keymapping
of such device until it disconnects, at which time it ends its execution.

## Caveats

As always, there's always a caveat:

- It will "grab" the detected devices for itself.
- If you tweak your key repeat settings, check whether they get reset.  
  Please check [this report][key-repeat-fix] about the resolution.

## History

I can't recall when I started using CAPSLOCK as both ESC and CTRL but it has
been quite some time already. It started when I was on OS X where it was quite
easy to achieve using the [Karabiner][], which already provides an option to
turn CTRL into CTRL/ESC (which can be coupled with OS X system settings that
turn CAPSLOCK into CTRL).

Moving on, permanently making Linux my home, I searched and tweaked a similar
solution based on [xmodmap][] and [xcape][]:

- <https://github.com/alexandre/caps2esc>

It's a simple solution but with many annoying drawbacks I couldn't stand in the
end:

- It resets any time a device change happens (bluetooth, usb, any) or the
  laptop lid is closed or when logging off and needs to be re-executed.
- It depends on [X][]. Doesn't work on TTY (bare terminal based machine,
  CTRL-ALT F2, etc).

Meanwhile on Windows land, I had a definitive solution based on my
[Interception library][interception] that always works perfectly, no hiccups.

It made me envy enough, so I ported the
[Windows Interception caps2esc][caps2esc-windows] sample to Linux based upon
evdev, udev and uinput.

## License

<a href="http://www.gnu.org/copyleft/gpl.html">

![GPL v3](https://www.gnu.org/graphics/gplv3-127x51.png)

</a>

Copyright © 2016 Francisco Lopes da Silva.

[caps2esc-windows]: https://github.com/oblitum/Interception/blob/master/samples/caps2esc/caps2esc.cpp
[libevdev]: https://www.freedesktop.org/software/libevdev/doc/latest/index.html
[libudev-dev]: http://packages.ubuntu.com/search?keywords=libudev-dev
[libevdev-dev]: https://launchpad.net/ubuntu/+source/libevdev
[karabiner]: https://pqrs.org/osx/karabiner/
[xmodmap]: https://www.x.org/releases/X11R7.7/doc/man/man1/xmodmap.1.xhtml
[xcape]: https://github.com/alols/xcape
[x]: https://www.x.org
[interception]: https://github.com/oblitum/Interception
[key-repeat-fix]: https://github.com/oblitum/caps2esc/issues/1
