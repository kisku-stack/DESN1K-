# STREAM 1 TEAM E - MORPHING AIRPLANE WING 

# Operation Chameleon: Morphing Wing Control

Arduino firmware plus a browser control panel (Web Serial) for a 3D-printed compliant morphing wing.

## Repo layout
- `firmware/morphing_wing/morphing_wing.ino`: flash this to the Arduino (Uno/Nano)
- `web/index.html`: open in **Chrome or Edge** (Web Serial is not in Firefox/Safari). Works in simulation without hardware.

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

## Tune before testing
1. **Hinge limits**: `minA`/`maxA` in the `.ino`, and the Limits boxes in the UI. Start narrow and widen slowly.
2. **Airfoil geometry**: the `CFG` block at the top of the script in `index.html` (thickness, hinge position, deflection at max servo angle, AoA range). Measure these from the CAD.a

## Behaviour
- Pot moved or button pressed -> **analogue** takes over. Clicking Digital in the UI hands control back.
- In analogue the button flips the left servo (opposite mode, as in the original sketch).
- LED: cyan = roll, green = morph, magenta = independent, amber = analogue, flashing red = compromised.
- **Disco**: hold the button for 2 s, or click the logo in the UI 5 times.
- **Compromised** button (bottom-right corner of UI): locks digital control and forces analogue, for the pitch.

## Serial protocol (115200)
PC to Arduino: `D,L,R` `SRC,0|1` `SM,0|1` `M,1..3` `AOA,deg` `LIM,min,max` `DISCO,0|1` `ALERT,0|1`
Arduino to PC: `T,pot,curL,curR,analogue,inverted,disco` every 50 ms