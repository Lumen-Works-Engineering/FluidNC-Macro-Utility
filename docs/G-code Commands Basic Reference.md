# G-code Commands Basic Reference

## 1. Modal vs. Non-Modal Commands

G-code commands fall into two "memory" categories. Understanding this distinction prevents crashes where the machine "remembers" a speed or mode you thought was finished.

## **Modal Commands (The "Sticky" Ones)**

Once you send a modal command, the machine remembers it and applies it to every subsequent line until you explicitly change it.

- **Examples:** `G00` (Rapid), `G01` (Feed), `F` (Feed Rate), `S` (Spindle Speed), `G90/G91` (Absolute/Incremental).

- **Why it matters:** You don't need to type `G01` on every line.
  
  - *Line 1:* `G01 X10 F500` (Turn on Linear Move mode at 500mm/min)
  
  - *Line 2:* `X20` (Machine knows: "Still in G01, still at F500. I'll move to X20.")
  
  - *Line 3:* `X30` (Machine knows: "Still in G01...")

## **Non-Modal Commands (The "One-Shot" Ones)**

These commands execute *only* on the line they are written. They have no memory.

- **Examples:** `G53` (Machine Coordinates), `G04` (Dwell/Pause), `G28` (Return to Home), `G10` (Set Offsets).

- **Why it matters:** If you write `G53 G0 X0` to park the machine, the very next line reverts back to your previous work coordinate system (G54). It does *not* stay in Machine Coordinates.

## 2. The "M" Codes (Miscellaneous)

M-codes control the physical switch-like functions of your machine. In FluidNC, these are mapped to specific GPIO pins in your `config.yaml`.

| Code    | Standard Function     | FluidNC Specifics                                                                                                                                    |
| ------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **M3**  | **Spindle ON (CW)**   | Turns on the spindle or fires the laser. If you have a Laser mode configured, M3 allows the power to scale with speed (preventing burns in corners). |
| **M4**  | **Spindle ON (CCW)**  | Historically "Counter-Clockwise." In Laser mode, this is often the "Dynamic Power" mode, while M3 is "Constant Power."                               |
| **M5**  | **Spindle/Laser OFF** | Immediately cuts power to the spindle/laser pin. **Safety:** Always put an M5 at the end of your file.                                               |
| **M7**  | **Mist Coolant ON**   | Activates the pin defined as `mist_pin` in your config.                                                                                              |
| **M8**  | **Flood Coolant ON**  | Activates the pin defined as `flood_pin` in your config (often used for air assist solenoids on lasers).                                             |
| **M9**  | **Coolant OFF**       | Turns off *both* Mist (M7) and Flood (M8).                                                                                                           |
| **M0**  | **Program Pause**     | Stops movement and waits for you to press "Cycle Start" (or the resume button). Useful for manual tool changes.                                      |
| **M30** | **Program End**       | Stops the spindle, turns off coolant, and "rewinds" the program.                                                                                     |

## 3. Canned Cycles (Drilling Loops)

*Note: FluidNC has basic support for these, but they are powerful time-savers.*

A "Canned Cycle" is a macro. Instead of writing 3 lines of code to drill a hole (Move down, Move up, Move to next spot), you write one line, and the machine handles the repetitive motion.

- **G80 (Cancel Cycle):** **Important.** You must run this after you are done drilling. If you don't, the machine will try to drill a hole at every subsequent move command.

- **G81 (Simple Drill):** Moves to XY, plunges to Z-depth, then retracts.
  
  - `G81 X10 Y10 Z-5 R2 F100`
  
  - *Translation:* Go to X10 Y10. Rapid down to 2mm (R). Feed down to -5mm (Z). Rapid back up to 2mm.

- **G83 (Peck Drill):** Essential for deep holes in metal. It drills a little, pulls out to clear chips, goes back in, and repeats.
  
  - `G83 X10 Y10 Z-10 R2 Q2 F100`
  
  - *The "Q" Value:* This is the "peck depth." It will drill 2mm, retract, drill another 2mm, retract, etc., until it hits Z-10.

**Pro Tip for FluidNC:** While FluidNC supports these G-codes, many hobby CAM processors (like Fusion 360's default GRBL post-processor) will manually "unroll" these cycles into long lists of G0/G1 commands. If you hand-write code, use G81/G83. If you use software, let it handle the complexity.
