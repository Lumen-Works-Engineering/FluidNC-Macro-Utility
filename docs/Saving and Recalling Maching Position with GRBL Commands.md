Saving and Recalling CNC Machine Position

Excellent workflow! Here's exactly how to set that up:

## Complete Workflow

**Initial Setup (do this once per fixture/workpiece setup):**

1. **Home the machine:**
   
   text
   
   `$H`

2. **Jog to your exact start position** (L-fixture corner where work begins)

3. **Save this position using G28.1:**
   
   text
   
   `G28.1`
   
   This stores the current machine coordinate position permanently in EEPROM[[discuss.inventables](https://discuss.inventables.com/t/learning-about-g28/12205)]​[[youtube](https://www.youtube.com/watch?v=7cX9xqsL6V8)]​

4. **Write down the machine coordinates** displayed (X, Y, Z values in MCS) as a backup reference

5. **Set your work zero at this location:**
   
   text
   
   `G10 L20 P1 X0 Y0 Z0`

## Returning After Restart/Reset

When you need to return to that exact position after removing the workpiece or restarting:

1. **Home the machine again:**
   
   text
   
   `$H`

2. **Return to your saved position:**
   
   text
   
   `G28`
   
   This moves directly to the position you saved with G28.1[[youtube](https://www.youtube.com/watch?v=7cX9xqsL6V8)]​[[discuss.inventables](https://discuss.inventables.com/t/learning-about-g28/12205)]​

3. **Clamp your workpiece** back against the L-fixture, and you're ready to run your job

## Why This Works Perfectly

- **G28.1 stores the position in machine coordinates** based on homing switches, so it's completely repeatable[[discuss.inventables](https://discuss.inventables.com/t/learning-about-g28/12205)]​[[youtube](https://www.youtube.com/watch?v=7cX9xqsL6V8)]​

- **The position persists through power-offs, resets, and E-stops** - it's saved in EEPROM[[youtube](https://www.youtube.com/watch?v=7cX9xqsL6V8)]​

- **G28 returns you to that exact MCS location** regardless of work coordinate offsets[[discuss.inventables](https://discuss.inventables.com/t/learning-about-g28/12205)]​[[youtube](https://www.youtube.com/watch?v=7cX9xqsL6V8)]​

- **Your L-fixture ensures** the workpiece is mechanically positioned the same way each time

## Optional: Use G30 for a Second Position

You can also save a second reference position (like a tool change location) using `G30.1` and return to it with `G30`. This gives you two stored positions in your FluidNC setup.[[youtube](https://www.youtube.com/watch?v=7cX9xqsL6V8)]​[[discuss.inventables](https://discuss.inventables.com/t/learning-about-g28/12205)]​

---

The GRBL command to move the carriage to a specific position using Machine Coordinate System (MCS) after homing is **G53**.[youtube+1](https://www.youtube.com/watch?v=GZiK9ez2Hys)

## How G53 Works

G53 allows you to command movement to absolute machine coordinates relative to your homing switch positions. The command is non-modal, meaning it only applies to the single line where it's issued.[youtube+1](https://www.youtube.com/watch?v=61FNml8M3i0)

**Syntax example:**

text

`G53 G0 X-50 Y-50 Z-10`

This would move to X=-50mm, Y=-50mm, Z=-10mm in machine coordinate space.[[youtube](https://www.youtube.com/watch?v=GZiK9ez2Hys)]​

## Understanding MCS After Homing

When you complete a homing cycle with `$H`, the MCS is set to 0,0,0 at the homing switch positions. The machine coordinates remain fixed relative to these switches and shouldn't be changed.[[github](https://github.com/grbl/grbl/issues/613)]​

## Alternative Position Commands

For your CNC router workflow, you might also find these useful:

- **G28**: Returns to a pre-set position that you define with `G28.1` (commonly used for safe/start locations after homing)[[github](https://github.com/grbl/grbl/issues/613)]​[[youtube](https://www.youtube.com/watch?v=GZiK9ez2Hys)]​

- **G30**: Similar to G28 but sets an alternate predefined position[[youtube](https://www.youtube.com/watch?v=GZiK9ez2Hys)]​

- **G10**: Used to set Work Coordinate System (WCS) offsets relative to the MCS[[github](https://github.com/grbl/grbl/issues/613)]​

Since you're running FluidNC on your Genmitsu 4040 Pro Max, the G53 command should work identically as it follows standard GRBL conventions for machine coordinate movements.

---

Perfect! Here's the exact workflow for your Genmitsu 4040 Pro Max running FluidNC:

## Command Sequence

1. **Home the machine:**
   
   text
   
   `$H`

2. **Move to your desired position using machine coordinates:**
   
   text
   
   `G53 G0 X-100 Y-50 Z-10`
   
   (Replace with your actual desired MCS coordinates)

3. **Set zero for that axis at the current position:**
   
   text
   
   `G10 L20 P1 X0 Y0 Z0`

## Understanding G10 L20

The `G10 L20 P1` command sets the work coordinate system (G54) offset so that the **current position becomes** whatever coordinates you specify.[github+1](https://github.com/gnea/grbl/issues/225)

- **L20**: Sets offset based on current position as reference[[millrightcnc](https://millrightcnc.com/2018/07/gcodeintro/)]​

- **P1**: Specifies G54 work coordinate system (P2=G55, P3=G56, etc.)[github+1](https://github.com/gnea/grbl/issues/225)

- **X0 Y0 Z0**: Makes current location become 0,0,0 in work coordinates[[millrightcnc](https://millrightcnc.com/2018/07/gcodeintro/)]​

## Alternative: Setting Individual Axes

If you only want to zero specific axes (like just Z for tool height), you can specify only those:

text

`G10 L20 P1 Z0`

This zeros only the Z-axis while leaving X and Y unchanged.[[millrightcnc](https://millrightcnc.com/2018/07/gcodeintro/)]​

## G92 vs G10 L20

You might see references to `G92 X0 Y0 Z0` which also sets zero at the current position, but `G10 L20` is preferred because it permanently stores the offset in the work coordinate system, whereas G92 creates a temporary offset that can be lost.[reddit+2](https://www.reddit.com/r/CNC/comments/13mmss7/how_do_i_set_home_position_in_grbl/)[[youtube](https://www.youtube.com/watch?v=61FNml8M3i0)]​
