OpenXC Retro Gauge
=========================

The Retro Gauge is a mechanical vehicle gauge that displays data received over a
serial communication line. One possible use of the gauge is to display vehicle
data from the [OpenXC Platform][openxc].

The design intent was to keep the gauge
as open and accessible as possible so that the community could contribute and
benefit from the design.

For more details on the project, visit the [project page on the OpenXC
website](http://openxcplatform.com/projects/retro-gauge.html).

For an instructional lab on how to build the gauge, see the [Retro Gauge
Lab](http://retro-gauge-lab.openxcplatform.com/).

## Retro Gauge Android Driver App

The Gauge Driver is the Android side of the Retro Gauge. It supplies the gauge
with a rounded two digit data value, percent (derived from previously set
bounds), and color data.

### Dependencies

This Android app uses the OpenXC library. The
[documentation](http://openxcplatform.com/android/api-guide.html) provides
instructions for how to use the library, but it is automatically included in
the GaugeDriver dependencies. Check the app's `build.gradle` file. 

The Gauge Driver also requires the [FTDriver by
ksksue](https://github.com/openxc/FTDriver). This dependecy is also 
automatically included in the `build.gradle` file and is made available by an 
Android package manager called `Jitpack`. 

### Connections

The Gauge Driver connects to the [OpenXC Vehicle Interface](http://openxcplatform.com/vehicle-interface/hardware.html#obtaining-a-vehicle-interface)
via Bluetooth to get access to vehicle data. The Gauge Driver connects to the 
Retro Gauge hardware via USB controlled by FTDriver, a serial connection library.

### Data Format

The Retro Gauge can receive two types of data packet.

* Color data is delivered with three digits within angle brackets. `<###>`
  Valid values are between 0 and 259.
* Value data is delivered with two digits each of value and percentage. The
  numbers are separated by a comma and enclosed in parentheses. `(vv,pp)`

### Operation

After startup, the user has the following options:

* Start Updates/Stop Updates – This switch toggles the updates to the Retro
  Gauge. When off, no information is sent to the gauge.
* Exit - Exits the application. Note that if the user navigates away from this
  activity via the home or back buttons, updates will continue in the
  background. The Exit button ceases all USB communication with the gauge.
* Data Source Selection - The large buttons labeled "Vehicle Speed", "Miles per
  Gallon", and "Steering Wheel Angle" select those data sources for the
  application. When a data source is selected, a hard coded color pallet is
  set. Hard coded bounds are also set for computing a percentage. All this
  happens in the button handlers.
* Color control slider - The user can use this slider to manually change the
  color of the gauge.
* Map Color To Value - If this is checked, the app ignores the slider and
  instead sets the color by the percentage value. Each data source uses a
  different color scheme. Vehicle Speed data sets colors ranging from green at
  zero to red at 80mph. Miles Per Gallon data sweeps through red at zero to
  green at 50 miles per gallon and above. Steering Wheel data sweeps through
  the entire spectrum.
* Manual Data Entry - This is a debugging tool. Text entered in the text box is
  sent to the Retro Gauge when the Send button is tapped.

### Under the Hood

**FTDriver**

In the Gauge Driver, the serial connection to the Retro Gauge is set up in 
the `OnCreate` function only if the app is not returning
from running in the background. Vendor and product IDs do not need to be set,
they are hard coded in the FTDriver code to connect to the FTDI chip.

In preparation for a lack of permission, an Intent `mPermissionIntent` is
created to receive the intent `ACTION_USB_PERMISSION`. `mBroadcastReceiver` is
registered to handle this incoming intent. Once all this is in place, the
FTDriver object `mSerialPort` is created, its permission intent is set to
`mPermissionIntent`, and the app attempts to start the port. If the user has not
yet granted permission, the start fails.

Down in the definition of `mBroadcastReceiver`, we handle the user's granting or
denial of permission to use the port.

Once we've successfully started the port, we start updates with 
`onTimerToggle()`.

**Timer**

The duration of the timer is hard coded as `mTimerPeriod`.

The timer created is `mReceiveTimer`. It is created as `null` and when it is
stopped with `onTimerToggle`, it is set to `null` again. It can therefore be 
used to determine whether or not it's currently running. If it's `null`, it's 
not running.

The timer task, `ReceiveTimerMethod`, polls incoming data,
converts/scales/rounds the data as needed, calculates a percentage on hard coded
parameters, and sends the value and percentage to the virtual serial port.

If the user has selected that the color of the gauge should be controlled by the
data, the color is calculated and sent to the gauge in this method as well.

**Data Sources**

The Gauge Driver is set up to use three data sources: `vehicle_speed`,
`odometer`, and `steering_wheel_angle`. When the user selects a data source,
`mDataUsed` is set. `ReceiveTimerMethod` determines which data and parameters to
use via the `mUsedData` variable. `mGaugeMin` and `mGaugeRange` are also set.
These as used for the computation of a percentage. All this happens in the
button handler for each data set.

## Resources

This repository is one of 4 repositories listed below:

**Mechanical Enclosure Design** - The
[retro-gauge-enclosure](http://github.com/openxc-retro-gauge/retro-gauge-enclosure)
repository contains .STL and STEP files for the 3D printable gauge housing.

**Hardware** - The
[retro-gauge-hardware](http://github.com/openxc-retro-gauge/retro-gauge-hardware)
repository contains Eagle schematics and PCB layouts for the gauge's embedded
hardware.

**Firmware** - The
[retro-gauge-firmware](http://github.com/openxc-retro-gauge/retro-gauge-firmware)
repository contains Arduino source code to run on the Retro Gauge.

**Android** - The
[retro-gauge-android](http://github.com/openxc-retro-gauge/retro-gauge-android)
repository contains an Android application to control the gauge with vehicle
data from the [OpenXC Platform][openxc].

## License

The firmware source code is available under the BSD open source license.

[openxc]: http://openxcplatform.com
