# openHAB Sunrise Alarm
[OpenHAB](https://openhab.org) ruleset to simulate sunrise with smart light bulbs depending on Android (!) phone's alarm clock time. Automatically schedules `Timer`s and `WakeUp` commands on  [Tasmota](https://tasmota.github.io/)-flashed smart light bulbs whenever alarm clock settings are changed.

Settings in openHAB sitemap

<img src="https://github.com/nicolaus-hee/openhab-sunrise-alarm/blob/main/openhab-app-screenshot.png" alt="openHAB Android app screenshot" width="300">

Result in Tasmota web UI (set by openHAB rule)

<img src="https://github.com/nicolaus-hee/openhab-sunrise-alarm/blob/main/tasmota-timers.png" alt="Tasmota timers" width="500">

## Features
- [x] Auto detect next alarm time from Android phone incl. week day
- [x] Disable sunrise when alarms disabled
- [x] Custom sunrise duration
- [x] Auto switch-off x minutes after alarm
- [x] Complete sunrise x minutes before alarm

## Installation
_TL;DR: Copy `sunrise.rules` to your rules, `sunrise.items` to your items and include the desired items in your sitemap. Adapt MQTT messages in rules to your smart light item names. Enable [alarm time sync](https://www.openhab.org/docs/apps/android.html#send-device-information-to-openhab) in your openHAB Android app to the `AlarmClockTime` item._

### Set up items

Add sunrise configuration items to your items configuration file.

```
DateTime AlarmClockTime "Next alarm [%s]"
Switch Sunrise_Enabled "Sunrise enabled"
Number Sunrise_Minutes_Duration "Sunrise duration [%d min]"
Number Sunrise_Minutes_Before_Alarm "Full sun x min before alarm [%d min]"
Number Sunrise_Minutes_Before_Switch_Off "Keep sun on for x min after alarm [%d min]"
````

Obviously, you also need switchable lights, like so:

``` 
Switch shelly_bulb_1_power "Bulb 1" { channel="mqtt:topic:broker:269970ecf7:shelly_bulb_1_power" }
Dimmer shelly_bulb_1_dimmer "Bulb 1 Dimmer" { channel="mqtt:topic:broker:269970ecf7:shelly_bulb_1_dimmer" }
Color shelly_bulb_1_hsbcolor "Bulb 1 Color" { channel="mqtt:topic:broker:269970ecf7:shelly_bulb_1_hsbcolor" }
Dimmer shelly_bulb_1_white "Bulb 1 White" { channel="mqtt:topic:broker:269970ecf7:shelly_bulb_1_white" }

Switch shelly_bulb_2_power "Bulb 2" { channel="mqtt:topic:broker:998aecdec5:shelly_bulb_2_power" }
Dimmer shelly_bulb_2_dimmer "Bulb 2 Dimmer" { channel="mqtt:topic:broker:998aecdec5:shelly_bulb_2_dimmer" }
Color shelly_bulb_2_hsbcolor "Bulb 2 Color" { channel="mqtt:topic:broker:998aecdec5:shelly_bulb_2_hsbcolor" }
Dimmer shelly_bulb_2_white "Bulb 2 White" { channel="mqtt:topic:broker:998aecdec5:shelly_bulb_2_white" } 
```

### Set up sitemap

Add items to your sitemap file, like so:

```
Text label="Sunrise Alarm" {
   Switch item=Sunrise_Enabled
   Default item=AlarmClockTime
   Setpoint item=Sunrise_Minutes_Duration minValue=0 maxValue=60
   Setpoint item=Sunrise_Minutes_Before_Alarm minValue=0 maxValue=60
   Setpoint item=Sunrise_Minutes_Before_Switch_Off minValue=0 maxValue=60
}
```

Set initial value for the 'minutes' items or else the rules will fail.

### Sync alarm time from Android phone to openHAB

Set up your openHAB app in such way it will send alarm clock timers to your openHAB installation, choose the `AlarmClockTime` item. Refer to [app manual for details](https://www.openhab.org/docs/apps/android.html#send-device-information-to-openhab).

### Set up rules

Add rules from `sunrise.rules` to your rules folder or file. Make sure to adapt the MQTT messages in the rules (all lines starting wtih `mqttActions.publishMQTT`) to your smart light items.

### Set up Tasmota bulbs
In the web UI console, enter the following commands for each light:

```
rule1 on Clock#Timer=1 do Wakeup 100 endon
rule1 1
rule2 on Clock#Timer=2 do Backlog HSBColor 0,0,100; Power 0 endon
rule2 1
```

This defines the `WakeUp` sequence to be launched when `timer1` is triggered and the light's switch off when `timer2` is reached.

## To do
- [ ] Notifications
- [ ] Custom colors
- [ ] Absence detection
- [ ] Don't fire after (actual) sunrise
