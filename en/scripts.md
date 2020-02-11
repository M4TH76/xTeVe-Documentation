## Linux Systemd

Create a [systemd](https://en.wikipedia.org/wiki/Systemd) startup script. The script also works in FreBSD Jails.  

1. Create a new file (xteve.service) in:
```
/etc/systemd/system/
```
2. Open the file with an text editor and copy the following content:
```
[Unit]
Description=xTeVe Service
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/path_to_executable/xteve 
ExecReload=/usr/bin/killall xteve
ExecStop=/usr/bin/killall xteve
KillMode=process
Restart=always
RestartSec=15

User=running_user
Group=running_group
[Install]
WantedBy=multi-user.target
```
**The path to the xteve binary, user and the group must be adjusted**

3. Save file and execute the following commands as root:
```
sudo systemctl enable xteve
sudo systemctl start xteve
```

## FreeBSD Service
Create a [rc.d](https://www.freebsd.org/doc/en_US.ISO8859-1/articles/rc-scripting/index.html) startup script.  

1. Create a new file (/etc/rc.d/) in:
2. Open the file with an text editor and copy the following content:
```
#!/bin/sh
# PROVIDE: xTeVe
# REQUIRE: DAEMON

. /etc/rc.subr
name=xteve

user="running_user"
path="/path_to_executable/xteve"


config_path="/home/${user}"
rcvar=xteve_enable

start_cmd="start_cmd"
stop_cmd="stop_cmd"

start_cmd()
{
  export HOME="${config_path}"

  echo "starting $name..."
  su -m "$user" -c "$path" &
}

stop_cmd()
{
  pids=$(pgrep "$name")
  pkill -9 "$name"
  echo "stop $name PID: $pids"
}

xteve_enable=${xteve_enable-"YES"}
pidfile=${xteve_pidfile-"/var/run/${name}.pid"}
run_rc_command "$@"
```
## FreeNAS rc.d file
```
#!/bin/sh

# $FreeBSD$

# PROVIDE: xTeVe
# REQUIRE: DAEMON

. /etc/rc.subr
name=xteve
user="plex"
path="/Plex/xteve/xteve"
config_path="/Plex"
rcvar=xteve_enable


command=/usr/sbin/daemon
command_args="-f -u ${user} ${path}"
pidfile=/var/run/xteve.pid



start_precmd=start_precmd
stop_postcmd=stop_postcmd


start_precmd()
{
  export HOME="${config_path}"
}

stop_postcmd()
{
  pids=$(pgrep "$name")
  pkill -9 "$name"
  echo "stop $name PID: $pids"
}

run_rc_command "$1"
```

**The path to the xteve binary and user must be adjusted.**

3. Save file and execute the following commands:
```
chmod +x /etc/rc.d/xteve
sysrc xteve_enable="YES"
```


Manual start of xTeVe (execute as root):
```
service xteve start
```

Manually quit xTeVe (execute as root):
```
service xteve stop
```


