---
draft: False
title: launchd agent for local log cleanup
description: Cleanup of launchd "cronlogs"

date: 2022-04-20T19:00:00+00:00
tags: 
    - launchd
    - mac-config
    - automation
categories:
    - mac-config
    - automation
---
Following on from [launchd agents for brew upgrade](https://drtbz.com/posts/20220419-launchd-cronjob/) this will cover setting up a cleanup job for all the logs we might now generate from scheduled tasks

## Setup the scheduled "agent"

Just like last time, here's the `com.drtbz.cronlogcleaner.plist` file added to `~/Library/LaunchAgent`

As you'll see - this uses an array of dictionaries for `StartCalendarInterval` to set it to run on the first day of odd numbered months

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.drtbz.cronlogcleaner</string>
        <key>Program</key>
        <string>/path/to/com.drtbz.cronlogcleaner</string>
        <key>EnvironmentVariables</key>
        <dict>
            <key>PATH</key>
            <string>/bin:/usr/bin:/usr/local/bin:/opt/homebrew/bin/</string>
        </dict>
        <key>StandardOutPath</key>
        <string>/path/to/cronlogs/cronlogcleaner.log</string>
        <key>StandardErrorPath</key>
        <string>/path/to/cronlogs/cronlogcleaner.log</string>
        <key>WorkingDirectory</key>
        <string>/path/to/com.drtbz.cronlogcleaner/folder/</string>
        <key>StartCalendarInterval</key>
        <array>
            <dict>
                <key>Month</key>
                <integer>1</integer>
                <key>Day</key>
                <integer>1</integer>
            </dict>
            <dict>
                <key>Month</key>
                <integer>3</integer>
                <key>Day</key>
                <integer>1</integer>
            </dict>
            <dict>
                <key>Month</key>
                <integer>5</integer>
                <key>Day</key>
                <integer>1</integer>
            </dict>
            <dict>
                <key>Month</key>
                <integer>7</integer>
                <key>Day</key>
                <integer>1</integer>
            </dict>
            <dict>
                <key>Month</key>
                <integer>9</integer>
                <key>Day</key>
                <integer>1</integer>
            </dict>
            <dict>
                <key>Month</key>
                <integer>11</integer>
                <key>Day</key>
                <integer>1</integer>
            </dict>
        </array>
    </dict>
</plist>
```

## Setup the script
Add this (tweaked to your requirements) in the location you put here: `<string>/path/to/script/com.drtbz.cronlogcleaner</string>`

It will quite simply remove all files it finds in the directory, except for it's own log file. It will output the name of all files it deletes when it runs.

```shell
#!/bin/bash
# brew updater script for LaunchD job: /Users/tom/Library/LaunchAgents/cronlogcleaner.plist
# there should be a linked copy of the plist in this repo.
# enable extended globbing for the rm command
shopt -s extglob
logger -s "Clean up of /User/tom/cronlogs starting"
{ cd /path/to/cronlogs && rm -v !("cronlogcleaner.log") ;} >> /path/to/cronlogs/cronlogcleaner.log 2>&1
logger -s "Clean up of /User/tom/cronlogs complete"
sleep 10
```
dont forget to `chmod +x` this script or it all falls apart

## launchctl
Now it's just a case of loading the `.plist` 
```shell
launchctl load /path/to/com.drtbz.cronlogcleaner.plist
```

## References
[Apple Dev Docs - Scheduling Timed Jobs](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/ScheduledJobs.html)
