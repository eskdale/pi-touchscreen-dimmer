# pi-touchscreen-dimmer
Leaving the backlight on the Official Raspberry Pi touchscreen can quickly wear it out and waste power.
If you have a use that requires the pi to be on all the time, but does not require the
display on all the time, then turning off the backlight or dimming it while not in use can dramatically
increase the life of the backlight and save power.

This work is based on the original by Timothy Hollabaugh https://github.com/timothyhollabaugh/pi-touchscreen-timeout.git and the a lot of changes kindly done by Dougie Lawson https://github.com/DougieLawson/backlight_dimmer.git This readme is an edited version of Dougie's readme (originally by Timothy) with applicable changes

pi-touchscreen-dimmer will transparently dim or turn off the display backlight after there
has been no input for a specifed timeout, independent of anything using the display
at the moment. It will then turn the touchscreen back on when input is received. The
timeout period and the level it dims to is set by a command-line argument.  Dimming to a level 0 will mean the backlight is off.

**Note:** This does not stop the event from getting to whatever is running on the
display. Whatever is running will still receive an event, even if the display
is off.

The program will use a linux event device like `/dev/input/event0` to receive events
from the touchscreen, keyboard, mouse, etc., and `/sys/class/backlight/10-0045/brightness`
to adjust the backlight brightness. The event device is a command-line parameter without the
/dev/input/ path specification.

# Installation

Clone the repository and change directories:
```
git clone https://github.com/eskdale/pi-touchscreen-dimmer.git
cd pi-touchscreen-dimmer
```
Assuming you don't have other input devices
Run it  - This will give a 10 second timeout with a minumum brightness of 15 reset by event0.
```
sudo ./timeout 10 15 event0
```

or Build and run it!
```
make
sudo ./timeout 10 15 event0
```

If you have multiple devices you can use the lsinput command to find which they are.  If you haven't installed it already

```
sudo apt install input-utils
lsinput

```

Multiple devices may be specified.
Look for `name: generic ft5x06 (00)` with bustype: `BUS_I2C` in the list.

**Note:** It must be run as root or with `sudo` to be able to access the backlight, unless you run the following
```
sudo su -c 'echo SUBSYSTEM==\"backlight\", RUN+=\"/bin/chmod 0666 /sys/class/backlight/%k/brightness /sys/class/backlight/%k/bl_power\" > /etc/udev/rules.d/99-backlight.rules'
sudo reboot
```

Use the following to change the brightness manually
```
echo 120 > /sys/class/backlight/rpi_backlight/brightness
```
or just manually change the value of the number in that file.

It can be run at startup, for example by putting a line in
`/etc/rc.local`  copy the files `timout` and `run-dimmer.sh` to the /etc folder

Add the line below to the `/etc/rc.local` This needs you to have lsinput installed as above.  The run-dimmer.sh will determine the event number for the Touchscreen so even if you have other devices it will use the touchscreen event.  You may wish to edit this file to modify the delay and the minimum brightness and also if you wish add events for keyboard or mice to also reset the timeout.

```
/etc/run-dimmer.sh
```

### Conflict with console blanker

When running this program without X Windows running, such as when running a Kivy
program in the console at startup, you may run into a conflict with a console
blanker.  In such an instance, the backlight will be turned on, but with the
console blanked, it seems like the backlight has not come on.

In this case, follow one of these methods for disabling the console blanker:
   * Raspbian Jessie :
     Add the following line to /etc/rc.local (on the line before the final exit 0)
     and reboot:
```
  sh -c "TERM=linux setterm -blank 0 >/dev/tty0"
```

   Even though /dev/tty0 is used, this should propagate across all terminals.

   * Raspbian Wheezy :
     Edit /etc/kbd/config and change the values for the variable shown below,
     then reboot:
```
  BLANK_TIME=0
```
