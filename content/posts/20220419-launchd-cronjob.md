---
draft: False
title: launchd agents for brew upgrade
description: Automatically running brew upgrade using mac native launchd

date: 2022-04-19T20:00:00+00:00
tags: 
    - launchd
    - mac-config
    - homebrew
    - automation
categories:
    - mac-config
    - automation
---
Having just made the jump to mac for my daily driver - I decided `brew upgrade` would be great to run on a schedule. However, as it's my personal device, it needs to be flexible enough to run everyday - even if I've missed the window (e.g - at 1200 or next login)

I'm familiar with `cron` and `anacron`, but some googling turned up an apple native peice `launchd` that maybe is a bit clunkier with it's scheduling than `cron`  and provided the 'run if missed' of `anacron`

So what this doc will cover:
- Setting up a launchd agent
- Making sure it can log all output to file
- Running the script anyway if the scheduled time is missed, e.g mac is asleep.

## Setup the scheduled "agent"

As we're messing with `launchd` in the user context, the official term is an agent, but the docs use agent and daemon interchangably.

Here's the `com.drtbz.brewupd.plist` file added to `~/Library/LaunchAgent`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.drtbz.brewupd</string>
        <key>Program</key>
        <string>/path/to/script/com.drtbz.brewupd</string>
        <key>EnvironmentVariables</key>
        <dict>
            <key>PATH</key>
            <string>/bin:/usr/bin:/usr/local/bin:/opt/homebrew/bin/</string>
        </dict>
        <key>StandardOutPath</key>
        <string>/path/to/logfile/brewupd.log</string>
        <key>StandardErrorPath</key>
        <string>/path/to/logfile/brewupd.log</string>
        <key>WorkingDirectory</key>
        <string>/path/to/workingdir</string>
        <key>StartCalendarInterval</key>
        <dict>
            <key>Hour</key>
            <integer>13</integer>
            <key>Minute</key>
            <integer>15</integer>
        </dict>
    </dict>
</plist>
```

The dictionary under the `StartCaldendarInterval` is where we configure our schedule - this isn't quite as flexible as `cron` unless I missed something, but gives the functionality we're after. There's another key `StartInterval` that can be used to run after a specific number of seconds, but won't run anyway if the window is missed.

## Setup the script
Add this (tweaked to your requirements) in the location you put here: `<string>/path/to/script/com.drtbz.brewupd</string>`
```shell
#!/bin/bash
# it makes life easy if you setup a linked copy of the plist in this repo.
logger -s "brew updates starting"
{ brew update && brew upgrade ;} >> /Users/tom/cronlogs/brewupd.log 2>&1
logger -s "brew updates complete"
sleep 10
```
dont forget to `chmod +x` this script or it all falls apart

## launchctl
Now it's just a case of loading the `.plist` 
```shell
launchctl load /path/to/com.drtbz.brewupd.plist
```

there is also a `launchctl unload` that does what it says on the tin.

We can also setup debugging to console on the next run. use the simple name `com.drtbz.brewupd` and it will give you the full name e.g `gui/501/com.drtbz.brewupd`
`sudo launchctl debug gui/501/com.drtbz.brewupd --stdout --stderr`

## Next Steps
The next post will cover a slightly more complex schedule to cleanup the logs ~ every 60 days or so

## References
[Apple Dev Docs - Scheduling Timed Jobs](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/ScheduledJobs.html)
