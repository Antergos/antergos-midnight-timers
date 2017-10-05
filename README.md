# Antergos Midnight Timers
Avoid system slow down at midnight

### Problem:
On machines with middle-range HDDs and modest CPUs, there is a noticeable system slowdown every day at midnight that can extend for a minute or more, depending on the computer capabilities.

This is caused by a group of 4 (four) timers that are all set to fire up at midnight on a normal Antergos installation:
```
$ systemctl list-timers
NEXT                         LEFT         LAST                         PASSED       UNIT                         
Wed 2017-09-27 01:25:48 WAT  2min 8s left n/a                          n/a          systemd-tmpfiles-clean.timer
Thu 2017-09-28 00:00:00 WAT  22h left     Wed 2017-09-27 00:00:38 WAT  1h 23min ago logrotate.timer              
Thu 2017-09-28 00:00:00 WAT  22h left     Wed 2017-09-27 00:00:38 WAT  1h 23min ago man-db.timer                 
Thu 2017-09-28 00:00:00 WAT  22h left     Wed 2017-09-27 00:00:38 WAT  1h 23min ago shadow.timer                 
Thu 2017-09-28 00:00:00 WAT  22h left     Wed 2017-09-27 00:00:38 WAT  1h 23min ago updatedb.timer 
```
These timer units all carry the following common configuration:
```
[Timer]
OnCalendar=daily
AccuracySec=12h
```
This setup is a good idea on on many server types, on which timer elapse coalescence is a desirable trait. But is not adequate on desktop computers such as those that are the main target of an Antergos installation. They will all tend to run at the same time along with any other timer units within the `AccuracySec` time window, necessarily causing a system slowdown on older computers.
 
### Solution

I propose that Cnchi adds a local configuration file to each of these 4 timers, to `/etc/systemd/system` during setup.

The local configuration file for each of the timers would be the equivalent of running the command `# systemctl edit --full <timer-unit-file>` and would make no change to the original configuration except for adding the following directive to the `[Timer]` section:

```
[Timer]
OnCalendar=daily
AccuracySec=12h
RandomizedDelaySec=5m
```

This directive introduces a small random elapse window of 5 minutes, allowing the service executed by the timers to all start at a different time and thus eliminating the system slowdown caused by an increase in computer activity.

This is the result of introducing such directive. Notice the scheduling difference from the above list:
```
$ systemctl list-timers
NEXT                         LEFT     LAST                         PASSED       UNIT
Thu 2017-09-28 23:43:24 WAT  23h left Wed 2017-09-27 23:43:24 WAT  26min ago    systemd-tmpfiles-clean.timer
Fri 2017-09-29 00:00:11 WAT  23h left Thu 2017-09-28 00:04:24 WAT  5min ago     pkgfile-update.timer
Fri 2017-09-29 00:00:22 WAT  23h left Thu 2017-09-28 00:02:58 WAT  6min ago     updatedb.timer
Fri 2017-09-29 00:01:44 WAT  23h left Thu 2017-09-28 00:00:24 WAT  9min ago     man-db.timer
Fri 2017-09-29 00:03:38 WAT  23h left Thu 2017-09-28 00:05:04 WAT  4min 47s ago logrotate.timer
Fri 2017-09-29 00:04:02 WAT  23h left Thu 2017-09-28 00:04:24 WAT  5min ago     shadow.timer
```

### Rationale

* This is a much proper organization for timer events on a typical desktop computer that is used for work and entertainment.

* The use of the systemd local configuration path means that Antergos won't have to make any changes to the current packages that own these timers and thus doesn't have to bring them to the Antergos repo.

* Not using the other method of creating drop-in replacements (through `/etc/systemd/system/<timer-unit>.d/override.conf` files) means that users are still given room for further configuration. 
 
### Additional Notes:

This issue was discussed in more detail at the Antergos forums: https://forum.antergos.com/topic/7979/reduce-systemd-timers-coalescence-in-antergos-default-setup
