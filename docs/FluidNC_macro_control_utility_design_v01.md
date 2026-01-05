For your last question: I think having me place the probe tip in the hole at the exact point I want to probe would be best.  The Z axis would never move.  Let's do the full html utility.  Another problem with the fluidnc webui is that the area for macros is tiny and truncates the names making it even more difficult to use. Lets set of specifications for the html utility before starting.

1. Tabs for each function (unless you have a better suggestion): CNC Control, Cylinder Probe, Hole Probe, Two Point Side Probe, T-Slot Height Map, Settings.

2. FluidNC connection info at the top not taking up too much space.
   
   All UI features to be as compact as possible within reason to make it easy to see the entire tab without scrolling if possible.

##### 

##### CNC Control:

Buttons for:

1. Home XYZ

2. Set each axis WCS zero.  Click the button and that axis will be set to zero. 

3. Go to each axis wcs zero individually

4. Go to XY wcs zero

5. Get and display current fluidnc MCS and WCS coordinates and display.  

6. Turn mist on/off with ability to enter the gcode for doing that.

7. Turn vacuum on/off with ability to enter the gcode for doing that.

8. Turn IoT PDU on/off with ability to enter the gcode for doing that.

9. Anything else you think may be useful.

##### Cylinder and Hole probe:

- Ability to have quick pick presets for common probe settings. Not sure what would work best here.  Maybe dynamic number of presets with + and - buttons to add or remove a preset and a save button.  Would have a nick name for each preset too.

- Ability to enter custom ad-hoc sizes for one-off probe operations.

- Button to set each X and Y and both axis zero's after probe has run.  Do not set zero automatically.

- Feature button to probe-again to perform another probe job.  The purpose would be to compare results from multiple probing operations.  The results of each operation will be displayed on the page in a text box(s) with the difference and average calculated between each result.  A secondary purpose for this feature that may need it's own area or tab is that I want to test the 3d probe by performing a probe op and noting results, turning the spindle with the probe still in it 90 degrees and probing again and noting the results and then 90 degrees again until I've turned the probe 360 degrees.  I have trued the probe needle but I'm not sure it's not deformed on the inside from past crashes and causing the probe to read as if the needle is off center even though it's physically true on the outside.

##### Two Point Side Probe:

The purpose will be to align work pieces using the 3d probe to probe the side of the work piece in multiple places to determine if the work piece is straight with the axis.  Example: getting setup to machine a piece of flat bar stock 6mm high and 250mm long placed parallel to the Y axis.  I would get it close to position with a square and then use this utility and the probe side wall near the front of the cnc and towards the back.  Then compare the results and adjust and probe again to get 0 difference.  The workflow would be to set the probe tip at the starting point which would be with the tip Z about mid ways down the 6mm side wall near the front of the cnc and about 5mm to the side.  Set zero and/or note the X and Y MCS coordinates.  Then click the probe button and the results would be recorded on the web page.  Then move the Y axis to the rear of the work piece at the same Z height and probe again.  The results would be displayed on the web page and any difference calculated.  Assuming I would need to adjust the work piece at least slightly, this next part would be important.  If I have the workpiece clamped down lightly, then I should be able to just move the rear of the piece +X or -X the difference in the probe results.  Then probe again at front and back.  So it would be useful to be able to set the two probe positions in the UI so I could easily repeat the probing process until I get the work piece aligned exactly.  Workflow would be: 1. I position the probe tip about 5mm to one side of the workpiece.  2. Click a button in the UI to mark the first point and also select which direction the probe should move to get to the work piece.  It could be on the left or right.  3. Click probe and the results would be recorded and displayed.  4. Manually jog to the far end of the work piece and click a button in the UI to indicate the second probe point. 4. Click probe and the results of both operations would be recorded and comparred.  The UI would indicate how much I need to move the far end and in which direction.  I will always try to keep the first probe point from moving and only adjust the far end but sometimes they both move a little.  5. I would adjust the work piece and click probe again while the probe is still at the second probe point. 6. Repeat until the work piece is in the correct postion.  7. Click a button in the UI to repeat the probe at both the first and second points again automatically without me manually jogging.  This is to recheck in case the front of the piece moved.

I would need to be able to perform the two point probe operation for parts aligned on either the X or the Y axis and with the probe starting at either side of the part.

##### T-slot height map:

Same as already in the html file.

##### Settings:

I need the utility to be configurable but also save settings.  However, I need the settings saved embedded in the single html file.  Not to cache.  My idea is to have a settings tab that would have a large text box or similar type area.  The settings would be in a format like json and saved inside that text box.  The utility would read the settings on startup and or clicking a refresh button.  Settings from all other tabs and global settings would be stored there.  Then selecting save-as from the browser menu and saving the html file would save the settings as well.  The format doesn't have to be json and can be anything you think would work best.  It can be plain text.  Doesn't matter about that.  Just a way to embed and save settings in the single html file.  I can save-as with a new file name for versioning and etc.  What do you think?
