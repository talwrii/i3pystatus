<!--
    Always edit README.tpl.md and create README.md by running
    python -m i3pystatus.mkdocs
    You can also let the maintainer do the latter :)
-->

# i3pystatus

i3pystatus is a (hopefully growing) collection of python scripts for 
status output compatible to i3status / i3bar of the i3 window manager.

## Installation

Note: i3pystatus requires Python 3.2 or newer and is not compatible with
Python 2.x.

### From PyPI package [i3pystatus](https://pypi.python.org/pypi/i3pystatus)

    pip install i3pystatus

### Packages for your OS

* [Arch Linux](https://aur.archlinux.org/packages/i3pystatus-git/)

### Release Notes

#### 3.28

* **If you're currently using the `i3pystatus` command to run your i3bar**:
    Replace `i3pystatus` command in your i3 configuration with `python ~/path/to/your/i3pystatus.py`
* Improved error handling
* Removed `i3pystatus` binary
* pulseaudio: changed context name to "i3pystatus_pulseaudio"
* Code changes

#### 3.27

* Add weather module
* Add text module
* PulseAudio module: Add muted/unmuted options

#### 3.26

* Add mem module

#### 3.24

**This release introduced changes that may require manual changes to your
configuration file**

* Introduced TimeWrapper
* battery module: removed remaining_\* formatters in favor of TimeWrapper,
as it can not only reproduce all the variants removed, but can do much more.
* mpd: Uses TimeWrapper for song_length, song_elapsed

## Configuration

The config file is just a normal Python script.

A simple configuration file could look like this (note the additional dependencies
from network, wireless and pulseaudio in this example):

    # -*- coding: utf-8 -*-

    import subprocess

    from i3pystatus import Status

    status = Status(standalone=True)

    # Displays clock like this:
    # Tue 30 Jul 11:59:46 PM KW31
    #                          ^-- calendar week
    status.register("clock",
        format="%a %-d %b %X KW%V",)

    # Shows the average load of the last minute and the last 5 minutes
    # (the default value for format is used)
    status.register("load")

    # Shows your CPU temperature, if you have a Intel CPU
    status.register("temp",
        format="{temp:.0f}°C",)

    # The battery monitor has many formatting options, see README for details

    # This would look like this, when discharging (or charging)
    # ↓14.22W 56.15% [77.81%] 2h:41m
    # And like this if full:
    # =14.22W 100.0% [91.21%]
    #
    # This would also display a desktop notification (via dbus) if the percentage
    # goes below 5 percent while discharging. The block will also color RED.
    status.register("battery",
        format="{status}/{consumption:.2f}W {percentage:.2f}% [{percentage_design:.2f}%] {remaining:%E%hh:%Mm}",
        alert=True,
        alert_percentage=5,
        status={
            "DIS": "↓",
            "CHR": "↑",
            "FULL": "=",
        },)

    # This would look like this:
    # Discharging 6h:51m
    status.register("battery",
        format="{status} {remaining:%E%hh:%Mm}",
        alert=True,
        alert_percentage=5,
        status={
            "DIS":  "Discharging",
            "CHR":  "Charging",
            "FULL": "Bat full",
        },)

    # Displays whether a DHCP client is running
    status.register("runwatch",
        name="DHCP",
        path="/var/run/dhclient*.pid",)

    # Shows the address and up/down state of eth0. If it is up the address is shown in
    # green (the default value of color_up) and the CIDR-address is shown
    # (i.e. 10.10.10.42/24).
    # If it's down just the interface name (eth0) will be displayed in red
    # (defaults of format_down and color_down)
    #
    # Note: the network module requires PyPI package netifaces-py3
    status.register("network",
        interface="eth0",
        format_up="{v4cidr}",)

    # Has all the options of the normal network and adds some wireless specific things
    # like quality and network names.
    #
    # Note: requires both netifaces-py3 and basiciw
    status.register("wireless",
        interface="wlan0",
        format_up="{essid} {quality:03.0f}%",)

    # Shows disk usage of /
    # Format:
    # 42/128G [86G]
    status.register("disk",
        path="/",
        format="{used}/{total}G [{avail}G]",)

    # Shows pulseaudio default sink volume
    #
    # Note: requires libpulseaudio from PyPI
    status.register("pulseaudio",
        format="♪{volume}",)

    # Shows mpd status
    # Format:
    # Cloud connected▶Reroute to Remain
    status.register("mpd",
        format="{title}{status}{album}",
        status={
            "pause": "▷",
            "play": "▶",
            "stop": "◾",
        },)

    status.run()

Also change your i3wm config to the following:

    # i3bar
    bar {
        status_command    python ~/.path/to/your/config/file.py
        position          top
        workspace_buttons yes
    }

### Formatting

All modules let you specifiy the exact output formatting using a
[format string](http://docs.python.org/3/library/string.html#formatstrings), which
gives you a great deal of flexibility.

Some common stuff:

* If a module gives you a float, it probably has a ton of uninteresting decimal
places. Use `{somefloat:.0f}` to get the integer value, `{somefloat:0.2f}` gives
you two decimal places after the decimal dot

#### formatp

Some modules use an extended format string syntax (the mpd module, for example).
Given the format string below the output adapts itself to the available data.

    [{artist}/{album}/]{title}{status}

Only if both the artist and album is known they're displayed. If only one or none
of them is known the entire group between the brackets is excluded.

"is known" is here defined as "value evaluating to True in Python", i.e. an empty
string or 0 (or 0.0) counts as "not known".

Inside a group always all format specifiers must evaluate to true (logical and).

You can nest groups. The inner group will only become part of the output if both
the outer group and the inner group are eligible for output.

#### TimeWrapper

Some modules that output times use TimeWrapper to format these. TimeWrapper is
a mere extension of the standard formatting method.

The time format that should be used is specified using the format specifier, i.e.
with some_time being 3951 seconds a format string like `{some_time:%h:%m:%s}`
would produce `1:5:51`

* `%h`, `%m` and `%s` are the hours, minutes and seconds without leading zeros
(i.e. 0 to 59 for minutes and seconds)
* `%H`, `%M` and `%S` are padded with a leading zero to two digits, i.e. 00 to 59
* `%l` and `%L` produce hours non-padded and padded but only if hours is not zero.
If the hours are zero it produces an empty string.
* `%%` produces a literal %
* `%E` (only valid on beginning of the string) if the time is null, don't format
anything but rather produce an empty string. If the time is non-null it is
removed from the string.
* When the module in question also uses formatp, 0 seconds counts as "not known".
* The formatted time is stripped, i.e. spaces on both ends of the result are removed

## Modules

System:
[Clock](#clock) - 
[Free space](#disk) - 
[System load](#load) - 
[Memory usage](#mem)

Audio:
[ALSA](#alsa) -
[PulseAudio](#pulseaudio)

Hardware:
[Battery](#battery) -
[Screen brightness](#backlight) -
[CPU temperature (Intel)](#temp)

Network:
[Wired](#network) -
[Wireless](#wireless)

Other:
[Unread mail](#mail) -
[Tracking parcels](#parcel) -
[pyLoad](#pyload) -
[Weather](#weather) -
[Music Player Daemon (MPD)](#mpd) -
[Simple text](#text)

Advanced:
[Rip info from files](#file) -
[Regular expressions](#regex) -
[Run watcher](#runwatch)

!!module_doc!!

## Contribute

To contribute a module, make sure it uses one of the Module classes. Most modules
use IntervalModule, which just calls a function repeatedly in a specified interval.

The output attribute should be set to a dictionary which represents your modules output,
the protocol is documented [here](http://i3wm.org/docs/i3bar-protocol.html).

**Patches and pull requests are very welcome :-)**

### The README

The README.md file is generated from the README.tpl.md file; only edit the latter
and run `python -m i3pystatus.mkdocs`.
