# Logitech keyboard wake and FN swap udev rules

Udev rules for Linux based OSs to allow use of the functions keys without the FN key on Logitech keyboards and allow HIDs to wake PC from sleep.

## Getting Started

Download the single udev rules file to your PC.

### Prerequisites

Logitech HID++2.0 device and recent udev.  No binaries required, the text udev rules do it alone.

### Installing

Simply copy the udev rules file to your `/etc/udev/rules.d/` directory (_as root_).  To activate: `udevadm control --reload-rules` (_as root_), then unplug and plug back in your Logitech Unifying receiver or device.

The udev rules are configured for the Logitech (046d) Unifying receiver (c52b) and Logitech K400 Plus keyboard connected as device #1.
If you have different device IDs, names, or they are connected in a different order, or require a different HID++ call, you will need to manually edit the udev rules file.  See below…

## How?

### HID wake PC from sleep
First, you might need to ensure that your BIOS settings allow USB devices to wake your PC from sleep.

Ensure that the `idVendor` and `idProduct` fields in the rules file match your USB device IDs.  To get your Logitech USB device ID, you can simply run `lsusb`.

The udev rule can also be dertermined by running: `sleep 5; udevadm monitor` (_as root_); unplugging USB device, waiting 5 seconds, then plugging the USB device back in.  The first **add _/devices/…_ (usb)** event that shows up should be your USB device.  The `udevadm info -a /sys/devices/…` command will then show the **ATTR**ibutes you can use to uniquely identify that specific USB device.  Finally the `ATTR{power/wakeup}="enabled"` assignment will enable the USB device to wake the PC from sleep.  To see if any USB devices are configured to do so: `grep . /sys/bus/usb/devices/*/power/wakeup`.

### FN swap / inversion for function keys

The udev rule gets triggered everytime the specified keyboard is powered on (_because my keyboard's FN mode resets when powered off_).  The udev rule sends a 7 byte HID++2.0 protocol message to the raw HID device to invert FN functionality for the functions keys.  Note that bytes in the udev rule are in **octal** for the `printf` command, with decimal/hex values below, the 7 bytes are:

|Short msg code|Dev.Index|Feature|FuncID + SoftID|Parameters|
|:-:|:-:|:-:|:-:|:-:|
|\20|\1|\11|\24|\0\0\0
|16 _(0x10)_|1|9 _(FN swap)_|1 (4bitsHigh) + 4 (4bitsLow)|0, 0, 0|

 Device index 1 to 6 refers to wireless unifying devices, `0xff` (\377) would refer to a directly connected device.  So, if you have multiple Logitech wireless devices connected to one Unifying receiver, and the keyboard you want FN swap to apply to is not the first one, you will need to change the device index byte.  _(You may be able to get the device index using: `grep HID_PHYS /sys/devices/…/uevent | tail -c2` with the device name obtained below)._  If your device's FN swap feature number is different, you might need to change that byte.

The udev rule was determined by running: `udevadm monitor` (_as root_); turning off the wireless device, waiting a few seconds, then turning the device back on.  This should list the **change _/devices/…_ (power_supply)** events that relate to that specific device.  The `udevadm info -a /sys/devices/…` command will then show the **ATTR**ibutes you can use to uniquely identify that specific device (_.e.g. could use ATTR{serial_number}=="404d-xx-xx-xx-xx" instead of model_name to target specific keyboard_).  Finally the RUN command executes a very simple inline shell script to write bytes to the related hidraw device.  (with wireless unifying devices this still turns out to be the parent unifying receiver).

## See also

If you want a software utility to get more information about, or pair and unpair, your Logitech HIDs, consider the command line version of:

[Solaar](https://github.com/pwr-Solaar/Solaar) = _Linux device manager for a wide range of Logitech devices_.  (**CAUTION**: may allow unprivileged access to your device's firmware when installing the GUI!)

For security reasons also consider updating your Logitech Unifying receiver firmware using the latest [fwupd](https://fwupd.org/).

`evtest` (_as root_) can be used to show keyboard key bindings (`apt install evtest`).

### References
[Solaar's Logitech HID++ documentation](https://github.com/pwr-Solaar/Solaar/tree/master/docs/logitech)
