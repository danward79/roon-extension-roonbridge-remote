# Remote Control RoonBridge using Linux Keycode & system events

**Credit:** This extension is a modification/fork of the work done by [naepflin](https://github.com/naepflin) in the original [roon-extension-linuxkeyboardremote](https://github.com/naepflin/roon-extension-linuxkeyboardremote).

## Introduction

This extension monitors Linux keycode events and activates Roon commands based on a configured event.

The extension is intended to be installed on a bridge device and appears in the Roon extensions section of the settings as the hostname of the device. This is intentional as it allows for several independent devices to run the extension for there own control.

## Requirements

The extension has been tested with:
- Node.js v12.14.1
- NPM 6.13.4
- Raspbian GNU/Linux 10 (buster)

Git also needs to be installed.

## How the extension works

The extension uses linux keycode system events to trigger the Roon api. These events are detailed in [linux event codes](https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h). 

| Keycode Event | Event Code | Roon API       |
| ------------- |:----------:|----------------|
| KEY_PLAY      | 207        | play           |
| KEY_PAUSE     | 119        | pause          |
| KEY_PLAYPAUSE | 164        | playpause      |
| KEY_STOP      | 128        | stop           |
| KEY_NEXT      | 407        | next           |   
| KEY_PREVIOUS  | 412        | previous       |
| KEY_STOP      | 128        | stop           |
| KEY_VOLUMEDOWN| 114        | volume down    |
| KEY_VOLUMEUP  | 115        | volume up      |
| KEY_HOMEPAGE  | 172        | convenience tba|
| KEY_POWER     | 116        | convenience tba|

The keycodes are hard-coded and could be changed in the app.js, however for this use case this is probably unnecessary, as the intent is to map IR or other inputs to trigger the internal events elsewhere, this can be achieved using [ir-keytable](https://manpages.debian.org/testing/ir-keytable/ir-keytable.1.en.html). 

The configuration of ir-keytable is discussed below.

## Extension Installation

To install this extension, it is necessary to clone this repository, then install the service that runs the extension automatically. 

After the extension is successfully started, the extension should be enabled and selected on the Roon Core.

***Note:*** The installation presumes that the extension is installed in the **pi** user's home folder.

Use the following steps (*Some steps will require **sudo***): 
```bash
git clone https://github.com/danward79/roon-extension-roonbridge-remote.git
cd roon-extension-roonbridge-remote
npm install

cp roonbridge-remote.service /etc/systemd/system/
systemctl daemon-reload
systemctl enable roonbridge-remote
systemctl start roonbridge-remote
```

You can verify that the service is running using:
`systemctl status roonbridge-remote.service `

There should be no errors, see below:
```bash
roonbridge-remote.service - RoonBridge-Remote
   Loaded: loaded (/etc/systemd/system/roonbridge-remote.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2020-01-22 03:30:05 AEDT; 7h ago
 Main PID: 533 (node)
    Tasks: 11 (limit: 2319)
   Memory: 34.1M
   CGroup: /system.slice/roonbridge-remote.service
           └─533 /usr/bin/node /home/pi/roonbridge/extensions/roon-extension-linuxkeyboardremote/.

Jan 22 03:30:05 RoonBridge77dd1bc4 systemd[1]: Started RoonBridge-Remote.
```

The extension is run as a systemd service, so will automatically start with the host system.

## Extension Configuration

Open the Roon Controller and enable the extension. Then, pick the zone and device path (e.g. "/dev/input/event0") in the extension settings.

## Infrared Control Config

ir-keytable is used to manage and map incomming IR commands (scancodes) to internal OS events (keycodes) and requires a functioning IR receiver, ir-keytable will also need to be installed and run as a service to allow seamless functionality.

### IR Receiver Install

You can use a [Vishay TSOP38238](https://www.vishay.com/product?docid=82491), connected to the Raspberry Pi as such:

| IR Receiver Pin | GPIO Pin  | Physical GPIO Pin# |
|:---------------:|:---------:|:------------------:|
| 1(Out)          | GPIO 26   | 37                 |
| 2(Gnd)          | Ground    | 39                 |
| 3(VS)           | 3V3 power | 1 or 17            |

See below header pinout (courtesy of [raspberrypi.org](https://www.raspberrypi.org)):
![alt text](GPIO-Pinout-Diagram-2.png "Raspberry Pi Header pinout")

After physically interfacing the IR receiver, the GPIO needs to be enbled in /boot/config.txt (*nano will require **sudo***):
```bash
nano /boot/config.txt
```

Uncomment or add: `dtoverlay=gpio-ir,gpio_pin=26`
then reboot: `sudo reboot`

### Test IR Receiver

In order to test the receiver, install ir-keytable.

Use the following steps (*Some steps will require **sudo***):
```bash
apt-get install ir-keytable
```

Then run `ir-keytable -t`, if the receiver is installed correctly when buttons on the remote are pressed, there should be events similar to this:

```
pi@RoonBridge77dd1bc4:~ $ ir-keytable -t
Testing events. Please, press CTRL-C to abort.
30484.360061: lirc protocol(necx): scancode = 0x2d2d3b
30484.360100: event type EV_MSC(0x04): scancode = 0x2d2d3b
30484.360100: event type EV_SYN(0x00).
30484.420054: lirc protocol(necx): scancode = 0x2d2d3b repeat
30484.420088: event type EV_MSC(0x04): scancode = 0x2d2d3b
30484.420088: event type EV_SYN(0x00).
```

If this is not the case ***Do Not Proceed!***

### Map IR Remote to events

This section deals with mapping the remote control scancodes to the keycodes used by the extension... and is incomplete.

## License

Apache 2.0
