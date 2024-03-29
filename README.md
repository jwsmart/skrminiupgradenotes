# Ender 3 SKR Mini E3 v3 upgrade notes

My notes on upgrading my Ender 3's to the SKR Mini E3 v3
Mine is configured to used the stock microswitch for homing Z, and the BLTouch for bed leveling.

## Todo

The hard stuff?

* gross z-offset calibration.
* E-Step calibration.
* fine z-offset calibration.

## Old motherboard removal

Not much to say here, other than ***take pictures of all the connections***.
Lots of hot glue was used on the stock Creality board.  I used a small knife and fine needle-nose pliers to remove it, and plenty of force.

* Take pictures of all the connections so you can see the polarity.
* If you want to save yourself a little time, check your current value for E-Steps.  You can use `M503` to output the current settings.  Write them down or save that output, it might be useful later.  I had only changed my E-Steps, so none of the other items were interesting.

## Re-terminating the hotend fan

The hotend fan on my Ender 3 was bare wire, because it connected to screw terminals on the original Creality mainboard.
The correct connector is a 2-pin JST-XH plug.  I bought these: https://smile.amazon.com/gp/product/B01MCZE2HM

The positive wire (which was indeed red on my Ender 3) for the fan goes in pin 0 on the JST-XH.  

![Close-up photo of SKR Mini E3 V3 FAN1 connector with noting JST-XH plug pin 1](Fan1-Connector-Detail.png)

## BLtouch install

Not much to say here other than you don't have to get super fussy with the wire routing if you don't want to.

## New mainboard installation

Again nothing much here, the most notable things are the firmware defaults for the fans.

![Photo of SKR Mini E3 V3 with Fan connectors labeled](Fan-Connectors.png)

* FAN0 on the Mini is the part cooling fan by default.  My connector had blue and yellow wires.
* FAN2 on the Mini is the enclosure fan.  Red and black wires.
* FAN1 on the Mini is the part cooling fan.  Red and block wires.  See photo above.
* It does not matter which connector is used for the Z stepper.  
* Direct link to the SKR Mini E3 V3 connector diagram: <https://github.com/bigtreetech/BIGTREETECH-SKR-mini-E3/blob/master/hardware/BTT%20SKR%20MINI%20E3%20V3.0/Hardware/BTT%20E3%20SKR%20MINI%20V3.0_Top.pdf>

## Test steppers and enclosure fan

My SRK Mini E3 V3 shipped with a Marlin 2.0 bugfix firmware that is configured to *only* enable the enclosure fan when the steppers have been running, as the main source of heat for the mainboard is the stepper drivers.

Once you're ready to test the printer, power it up, and ***keep one hand hovering over the power switch***.

* Check that the enclosure fan is not running.  (It shouldn't be if you just powered on the printer.)
* Home the axes, making sure that nothing bad happens.  I was shocked at how quiet this was.
* Once homing is done, move the X, Y and Z axes via the menus, and ensure they move in the expected directions.
* Check that the enclosure fan is now running at 100% after moving the axes.
* If you disable the steppers and leave the printer alone for a bit, the enclosure fan will turn off.  I love this feature.

## Test the heated bed

* Set the bed temp to something that is warmer than ambient, and not hot enough to be dangerous, and verify that it does indeed heat and hold a temperature.

## Test the Hotend and Hotend Fan

* Set the hotend to 51 degrees c.  Watch as the hotend heats up, when the hotend hits 50, the hotend fan should come on at 100%.
* Set the hotend to 49 (or off), and watch the hotend fan turn off when the temperature falls below 50.

## Test the part cooling fan

Connect the mainboard to a computer (or Octoprint) you can use to send gcode commands.

* Use `M106` to set the part cooling fan to full speed.
* Use `M107` to stop it.

## Re-flash the firmware with BLTouch Support

* Direct link to firmware with bltouch support: https://github.com/bigtreetech/BIGTREETECH-SKR-mini-E3/blob/master/firmware/V3.0/Marlin/firmware-ender3-bltouch.bin
* Put this file in the root of an SD card, and rename it to `firmware.bin`.
* Power off the printer
* Inset the SD card
* Power on the printer and watch it flash (it's quick!)

## Zero the Z-probe offset to prevent bed scraping

In order to make sure your print head doesn't make a nice scrape along your printing surface, zero out the z-probe offset.  The default may be reasonable, so write it down if you want it for reference.

* It's the 2nd item under `Configuration`.

## Rough Bed level

Use your favorite method for 1st layer calibration to get your 1st layer close.  If you're reading this, you probably already have a favorite way to do this already - on the off chance that you don't, give CHEP's method a try: <https://www.youtube.com/watch?v=_EfWVUJjBdA>

This will get your bed level close enough to successfully calibrate your E-Steps.

## E-step calibration

My general process for e-step calibration is to use calipers on a single-wall cube.  It's close enough. This Teaching Tech video is pretty good, too: <https://www.youtube.com/watch?v=rp3r921DBGI>

If you just want to do the quick & dirty caliper method, make a box in your favorite slicer:
![making a cube in prusaslicer](add-box.png)

I like to make mine 20x20x10 to make it print real fast:
![20x20x10 box](box-dimensions.png)

Set to 0 Solid top layers and 1 perimeter:
![Layers and Perimeters setting for box](box-layers.png)

Lastly, set to 0% infill.  Should print real fast, and still give useful results.
![Print time for the e-step calibration box.](box-printtime.png)

## Z-Probe offset calibration

There are a *ton* of 1st layer calibration g-codes and samples.

CHEP has one here: <https://www.chepclub.com/bed-level.html>
TeachingTech has an entire site to help with calibration here: <https://teachingtechyt.github.io/calibration.html#firstlayer>

The version of Marlin that ships with the SRK Mini I got did *not* have the z-probe wizard, so I had to manually adjust the z-probe offset.

### Rough z-probe offset without Marlin Z-Probe Wizard

For a rough z-probe offset, do what's recommended in the Marlin docs for M851: <https://marlinfw.org/docs/gcode/M851.html>.

1. Home the Z Axis.
2. Raise Z and deploy the probe.
3. Move Z down slowly until the probe triggers.
4. Take the current Z value and negate it. (for example, 5.2 becomes -5.2)
5. Set the Z-Offset via the menus, or with `M851 Z`

### Fine Tuning the Z-Probe offset

My favorite way to do this is to set up 9 boxes in my favorite slicer, 20 x 20 x 0.2, at a .2 layer height.

## Change Slicer settings

### Start GCODE

Insert a `G29` into your start GCODE, after homeing (`G28`).
