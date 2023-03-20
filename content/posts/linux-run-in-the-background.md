---
title: "Linux Run in the Background"
date: "2023-03-03T14:28:09+07:00"
tags: ["tech", "linux"]
draft: false
comment: true
---

{{< quote >}}
**Environment:** Ubuntu 22.04, but it should work in other Linux distributions.

**Source**: https://www.sobyte.net/post/2023-03/linux-services-in-the-background/
{{</ quote >}}

You run some commands, it may take a long time to execute, and you don't want to wait for it. So what should you do in that scenario?

{{< mermaid >}}
graph TD
A(You don't want to wait <br>for a running command) --> B{Do you want to see output?}
A --> C{Do you want the command to stay <br>running after the session ends?}
C --> |Yes| E(screen)
B --> |No| D(Ctrl-Z or &)
B --> |Yes| F{Do you have requirements <br>for log cutting, running users?}
F --> |No| G(nohup)
F --> |Yes| H(supervisor or systemd)
{{</ mermaid >}}

## Ctrl-Z or &

For example, you have to excute `ping` command:

```shell
ping -c 20 8.8.8.8
```

You want to send it to background, you can do the following:

```shell
$ ping -c 20 8.8.8.8 &
[1] 467176
# Or hit ctrl-z, the effect of the & symbol is similar to ctrl-z
$ ping -c 20 8.8.8.8 # Press ctrl+z to hang
^Z[1]  + 467680 suspended
# View the hung background processes
$ job -l
[1]  + 467680 suspended
# Bring the background task process back up and see that process is executing again
# fg %<number>
$ fg %1
```

## screen

If you want the program to continue running after the session ends, consider using `screen`.

```shell

$ sudo apt install screen

# man screen
$ man screen

SCREEN(1)                                                                   General Commands Manual                                                                   SCREEN(1)

NAME
       screen - screen manager with VT100/ANSI terminal emulation

SYNOPSIS
       screen [ -options ] [ cmd [ args ] ]
       screen -r [[pid.]tty[.host]]
       screen -r sessionowner/[[pid.]tty[.host]]

...
# Create a ping session
$ screen -R <session-name>


# Exit the session, recreate another one and use the previous command
# you can see that the command is still running
$ screen -R <session-name>
```

## nohup

`screen` is more suitable for executing some scripts than `nohup`, while `nohup` is suitable for running a long-time tasks.

```shell
$ nohup command > command.log 2>&1 &

# There are 3 types of input and output defined in Linux:
# 0: standard input
# 1: standard output
# 2: error output
#
# 2>&1: the error output 2 is redirected to the standard output &1
# combined with > to be output to the file log.
```

## supervisor

`nohup` is still very good for performing simple background tasks, but if you have requirements for output log cutting, running users,... you need to consider daemon type programs such as `supervisor` and `systemd`.

`supervisor` is a lightweight process control system, used to run programs for long period of time, often used for web background programs, proxy programs, etc.

```shell
$ sudo apt install supervisor
$ sudo systemctl enable supervisor
$ sudo systemctl start supervisor

# Create supervisor file
$ sudo cat <<EOF >/etc/supervisor/conf.d/sync.conf
[program:syncthing]
directory=/home/apps/syncthing/syncthing-linux-amd64-v1.19.0
command=/home/apps/syncthing/syncthing-linux-amd64-v1.19.0/syncthing
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/var/log/supervisor/%(program_name)s-stdout.log
stdout_logfile_maxbytes=5MB
stdout_capture_maxbytes=5MB
stdout_logfile_backups=5
stderr_logfile=/var/log/supervisor/%(program_name)s-stderr.log
stderr_logfile_maxbytes=5MB
stderr_capture_maxbytes=5MB
stderr_logfile_backups=5
user = apps
environment = HOME="/home/apps/syncthing/syncthing-linux-amd64-v1.19.0"
EOF

# Make the newly created service effective
$ sudo supervisorctl update
# update: means update the list of services and re-run the new/modified services
# status: check the running status of all services
# start/stop/restart: start/stop/restart the service
```

## systemd

`systemd` is the system suite that comes with the most distributions to replace `SysV init`, providing reliable parallelism during the boot process and centralized management of processes, daemons, services and mount points. It is a relatively heavyweight system, and unless there is a clear need to use it, it is generally not considered a way to run small applications (or commands) in the background.

`systemd` common user unit configuration directory:

```shell
$HOME/.config/systemd/user
/usr/lib/systemd/user
$HOME/.local/share/systemd/user
/etc/systemd/user
```

```shell
# Create file
$ cat <<EOF >$HOME/.config/systemd/user/sync.service
[Unit]
Description=Sync Files
Documentation=https://syncthing.net
After=network.target

[Service]
Type=simple
Environment=HOME="/home/apps/syncthing/syncthing-linux-amd64-v1.19.0"
ExecStart=/home/apps/syncthing/syncthing-linux-amd64-v1.19.0/syncthing
# Logging requires systemd version > 240
StandardOutput=append:/home/apps/syncthing/syncthing-linux-amd64-v1.19.0/logs/standard-output.log
StandardError=append:/home/apps/syncthing/syncthing-linux-amd64-v1.19.0/logs/standard-error.log

[Install]
WantedBy=multi-user.target
EOF

$ systemctl start sync --user
```
