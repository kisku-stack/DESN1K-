# STREAM 1 TEAM E - MORPHING AIRPLANE WING

# Operation Chameleon: Morphing Wing Control

Arduino firmware plus a browser control panel (Web Serial) for a 3D-printed compliant morphing wing (NACA 0013, 150 mm chord). Two servos bend the trailing edge, a third servo sets angle of attack, and an RGB LED in the stand shows the current mode.

## Repo layout
- `firmware/morphing_wing/morphing_wing.ino`: flash this to the Arduino (Uno/Nano)
- `docs/index.html`: the whole control panel in one file. Open it in **Chrome or Edge** (Web Serial is not in Firefox/Safari). With no Arduino connected it runs as a simulation.

## Viewing the website
- **Locally:** double-click `docs/index.html`.
- **GitHub Pages:** repo Settings > Pages > Deploy from a branch > `main` and `/docs` > Save. It goes live at `https://<username>.github.io/<repo>/` after about a minute. The Arduino still connects over USB to whichever laptop opens the page.

## Wiring
| Part | Pin |
|---|---|
| Left servo | A4 |
| Right servo | A2 |
| AoA servo | D9 |
| Potentiometer wiper | A3 |
| Button (to GND) | D2 (INPUT_PULLUP) |
| RGB LED R / G / B | D3 / D5 / D6, each through ~220 ohm |

Set `COMMON_ANODE` in the sketch if your LED is common-anode. Power the three servos from an external 5 V supply (shared GND) rather than the Arduino 5 V pin.

## Calibration and tuning
**Firmware (`morphing_wing.ino`)**
- `zeroL`, `zeroR`, `trimR`: the servo angles where each wing is flat (currently 95, 50 and +14 trim on the right).
- `spanL`, `spanR`: how far each servo may travel either side of zero (currently 30). The website's Limits boxes set this too: span = (max - min) / 2, so the default 0 to 60 gives +/-30.
- `ROLL_DIFF`: roll difference between the two wings in analogue mode (currently 10 degrees).
- `AOA_CENTER` and `AOA_LIM`: the AoA servo angle for 0 degrees (90) and its limit (+/-15).
- Start with narrow limits and widen slowly so the hinge is never forced.

**Website (`CFG` block at the top of the script in `index.html`)**
- `chord`, `span`, `thick`, `pivot`, `hinge`: measured from the CAD (150 mm, 90 mm, NACA 0013, hole at 30% chord, hinge at 50 mm).
- `maxDefl`: trailing-edge deflection at either servo limit. This is still a guess, so measure it on the real wing.
- `rollDiff`: keep equal to `ROLL_DIFF` in the sketch.
- `neutral`: servo angle where the wing is flat (null = middle of the limits).
- `dir`: set to -1 if a higher servo angle moves the trailing edge up.
- `skin`: how smoothly the left and right angles blend across the span in the 3D views.

## Control panel
- **Exhibit A, live wing profile:** the morphed airfoil with pivot and hinge marked, plus a roll front view. The readout gives the closest NACA 4-digit shape, max camber and position, thickness, trailing-edge deflection and chord tilt. It is an approximation, since a bent compliant section is not exactly a NACA curve.
- **Exhibit B, controls:**
  - Source toggle (digital or analogue) and a smoothing option. With smoothing on, the wing slides to the set angle and the red marker (ghost) on each slider shows where it really is.
  - Angle of attack slider, -15 to +15 degrees (UI only).
  - Mode 1: roll + morph. The roll slider is limited to a 10 degree difference between the wings, and a second slider morphs both wings together.
  - Mode 2: vertical morph (top = trailing edge up, bottom = down).
  - Mode 3: left and right wings independently.
  - Only the active mode is usable; the others grey out.
- **Exhibit C, flow simulation:** a CFD picture of air flowing past the current wing shape. Views: speed, vorticity, wind streaks, 3D, and 3D (GPU, a real 3D simulation using WebGPU, which needs a recent Chrome or Edge). Tunnel speed is an input from 0 to 25 m/s. The flow restarts about a second after the wing stops changing. It shows the real Reynolds number for your wing and speed next to the simulation's own much lower one, and a red arrow for the net force. Cl and Cd are only shown above 8 m/s. This is a qualitative picture (low Reynolds number, simplified), not a source of results to quote.

## Behaviour
- **Digital:** the page drives the wing from the sliders.
- **Analogue:** the potentiometer morphs both wings together, with the middle position as flat. The button toggles roll, which holds the wings 10 degrees apart while the pot still morphs them. Entering analogue, and each button toggle, sends both wings to zero first before the pot takes over.
- Touching the pot more than a small amount, or pressing the button, switches to analogue. Clicking Digital in the UI hands control back. If no Arduino is connected, the page shows a pretend pot and roll button so you can demo it.
- LED: cyan = roll, green = morph, magenta = independent, amber = analogue, flashing red = compromised.
- **Disco:** hold the button for 2 s, or click the logo in the UI 5 times. The wings sweep out of phase and the LED cycles colours.
- **Compromised:** the small "!" button in the bottom-right corner of the UI shows a red takeover screen, locks digital control, and forces analogue. Use it for the pitch, then carry on with the pot.

## Serial protocol (115200 baud)
PC to Arduino: `D,L,R` `SRC,0|1` `SM,0|1` `M,1..3` `AOA,deg` `LIM,min,max` `DISCO,0|1` `ALERT,0|1`
Arduino to PC: `T,pot,L,R,analogue,roll,disco` every 50 ms. L and R are hinge angles on the same scale as the `D` command, so the page draws the real wing position.

## Troubleshooting
- **"Linked, but no data":** the wrong sketch is on the board. Upload `morphing_wing.ino` at 115200 baud.
- **No Connect option or an error:** use Chrome or Edge.
- **Servos buzz or the Arduino resets:** power the servos from the external supply, not the Arduino 5 V pin.
- **3D (GPU) view skipped:** the browser or graphics card doesn't support WebGPU; use the other views.