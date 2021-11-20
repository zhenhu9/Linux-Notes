## CentOS 基础命令 III

Unit files can live in several different places. /usr/lib/systemd/system is the main place where packages deposit their unit files during installation; on some systems, the path is /lib/systemd/system instead. The contents of this directory are considered stock, so you shouldn’t modify them. Your local unit files and customizations can go in /etc/systemd/system. There’s also a unit directory in /run/systemd/system that’s a scratch area for transient units. 

systemd maintains a telescopic view of all these directories, so they’re pretty much equivalent. If there’s any conflict, the files in /etc have the highest priority.

#### Commonly used systemctl subcommands

|Subcommand                   | Function|
| ----                        | ---------------- |
|list-unit-files [ pattern ]  |Shows installed units; optionally matching pattern|
|enable unit                  |Enables unit to activate at boot|
|disable unit                 |Prevents unit from activating at boot|
|isolate target               |Changes operating mode to target|
|start unit                   |Activates unit immediately|
|stop unit                    |Deactivates unit immediately|
|restart unit                 |Restarts (or starts, if not running) unit immediately|
|status unit                  |Shows unit’s status and recent log entries|
|kill pattern                 |Sends a signal to units matching pattern|
|reboot                       |Reboots the computer|
|daemon-reload                |Reloads unit files and systemd configuration|


#### Unit file statuses

|State     | Meaning|
| ----     | ---------------- |
|bad       |Some kind of problem within systemd; usually a bad unit file|
|disabled  |Present, but not configured to start autonomously|
|enabled   |Installed and runnable; will start autonomously|
|indirect  |Disabled, but has peers in Also clauses that may be enabled|
|linked    |Unit file available through a symlink|
|masked    |Banished from the systemd world from a logical perspective|
|static    |Depended upon by another unit; has no install requirements|



#### Mapping between init run levels and systemd targets

|Run level    | Target             | Description|
|0            |poweroff.target     | System halt|
|emergency    |emergency.target    |Bare-bones shell for system recovery|
|1, s, single |rescue.target       |Single-user mode|
|2            |multi-user.target a |Multiuser mode (command line)|
|3            |multi-user.target a |Multiuser mode with networking|
|4            |multi-user.target a |Not normally used by init|
|5            |graphical.target    |Multiuser mode with networking and GUI|
|6            |reboot.target       |System reboot|

a. By default, multi-user.target maps to runlevel3.target, multiuser mode with networking.

systemctl get-default	# To see the target the system boots into by default.

systemctl set-default	# To set the target the system boots into by default.







