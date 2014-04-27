05. Power Management
---------------------------

acpid and powertop setup
============================

Install `acpid` and `powertop`:

    pacman -S acpid powertop

Create `/usr/lib/systemd/system/powertop.service` and put this in the file:

```
[Unit]
Description=Powertop tunings
[Service]
Type=oneshot
RemainAfterExit=no
ExecStart=/usr/bin/powertop --auto-tune
#"powertop --auto-tune" still needs a terminal for some reason.
#Possibly a bug?
Environment="TERM=xterm"

[Install]
WantedBy=multi-user.target
```

Enable the services for acpid and powertop.

    systemctl enable acpid
    systemctl enable powertop


Backlight control setup
=========================

To get the Macbook's brightness control keys working:

Create `/etc/acpi/handlers/bl`:

```
#!/bin/sh
bl_dev=/sys/class/backlight/gmux_backlight
step=100

case $1 in
  -) echo $(($(< $bl_dev/brightness) - $step)) >$bl_dev/brightness;;
  +) echo $(($(< $bl_dev/brightness) + $step)) >$bl_dev/brightness;;
esac
```

Make it executable:

    chmod +x /etc/acpi/handlers/bl

Then bind the relevant keys to acpi events:

In `/etc/acpi/events/bl_d`:

```
event=video/brightnessdown
action=/etc/acpi/handlers/bl -
```

In `/etc/acpi/events/bl_u`:

```
event=video/brightnessup
action=/etc/acpi/handlers/bl +
```

When you start acpid (`systemctl start acpid`) or reboot the computer, the backlight should now be controllable with the keyboard.

Suspend/resume on lid close/open
=================================

In `/etc/acpi/handler.sh`, look for `button/lid)`:

```
...
button/lid)
    case "$3" in
...
```

Modify this block so that it looks like this:

```
    button/lid)
        case "$3" in
            close)
		/usr/bin/systemctl suspend
                logger 'LID closed'
                ;;
            open)
		/usr/bin/systemctl resume
                logger 'LID opened'
                ;;
            *)
```

Basically, you need to add `/usr/bin/systemctl {suspend,resume}` to `close/open)`.


