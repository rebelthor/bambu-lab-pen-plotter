# **Bambu Lab P1S Plotter Setup Guide (Falu Modular System)**

This repository contains a safe, incremental workflow for turning a **Bambu Lab P1S** into a pen plotter using the **Falu Modular System**.  

**⚠️ CRITICAL WARNING \- READ BEFORE PROCEEDING ⚠️**  

**YOU ARE PROCEEDING AT YOUR OWN RISK.**  
This process involves modifying printer "G-code", disabling safety sensors, and moving the machine in non-standard ways.

* **Risk of Hardware Damage:** If you fail to remove the pen before the print finishes, the printer **WILL** crash the toolhead into the bed, potentially bending the nozzle, destroying the build plate, or damaging the Z-axis motors.  
* **Risk of Collision:** If you install the mount before the printer has homed, the toolhead **WILL** crash into the front door or frame.  
* **Supervision Required:** NEVER walk away from the printer while in plotter mode. Always stay within reach of the power switch or "Stop" button.

**The author of this guide accepts no responsibility for damage to your printer.**

## **Prerequisites: Printed Parts & Tools**

Before starting the testing phases, ensure you have these files printed from the [Falu Modular System on MakerWorld](https://www.google.com/search?q=https://makerworld.com/en/models/2029113-modular-system-for-a1-p1-x1-series):

1. Head-Mount-V1-Series X and Series P.stl (Base clip)  
2. Spring Module D9 5mm.stl (Compliance mechanism)  
3. Guide Stabilo D 8 55mm.stl (Pen adapter)

**You also need:**

* Stabilo Point 88 Fineliner (or similar pen).  
* Painter's tape and Paper.

## **Phase 1: Software Setup (Do This First)**

We must configure the slicer before touching the printer to ensure it doesn't heat up or try to extrude plastic.

### **1\. Create the "Plotter" Filament Profile**

*It is critical to create a specific filament profile to prevent the printer from trying to push plastic.*

1. Open **Orca Slicer**.  
2. Edit a **Generic PLA** profile.  
3. **Save As:** Plotter Filament.

**Settings:**

* **Flow Ratio:** 0.01 (Prevents extrusion motor from pushing filament).  
* **Nozzle Temperature:**  
  * **Initial Layer:** 180°C  
  * **Other Layers:** 180°C  
  * *(Why 180? It prevents "Cold Extrusion" errors in firmware, but is cool enough to prevent oozing).*  
* **Bed Temperature:** 25°C (Room temp).  
* **Cooling Tab:**  
  * **Part Cooling Fan:** 0% (Min) / 0% (Max).  
  * **Auxiliary Fan:** 0%.  
  * **Chamber Fan:** 0%.  
* **Advanced (Optional but Recommended):**  
  * **Standby Temperature:** 0°C.  
  * **Vitrification Temperature (Soften Temp):** 0°C.

### **2\. Create the "Plotter" Printer Profile**

1. Edit your default "Bambu Lab P1S 0.4 nozzle" profile.  
2. **Save As:** P1S Plotter (Stabilo).

#### **A. Extruder Settings**

* **Extruder Tab:**  
  * **Z-Hop when retracting:** Enabled (Checked).  
  * **Z-Hop Type:** Normal (**Crucial**: 90° vertical lift).  
  * **Z-Hop Height:** 3.0 mm (Prevents drag lines).  
  * **Retraction Length:** 0.01 mm (To unlock Z-hop).  
  * **Max volumetric speed:** 22 mm³/s.

#### **B. Set Z-Offset (Safety Gap)**

* **Extruder/Print Options Tab:**  
  * **Z-Offset:** 17.0 mm.  
  * *This "lies" to the printer, keeping the nozzle 17mm above the bed so the pen (which hangs lower) touches the paper instead.*

#### **C. Set Excluded Bed Area (Safety Zones)**

*Because the pen module occupies physical space, we must tell the slicer to avoid certain areas to prevent the toolhead from crashing into the frame.*

* Go to the **General** (or **Basic Information**) tab.  
* Find the **Excluded bed area** field.  
* **Copy and Paste** this exact coordinate string into the box:  
  0x0, 258x0, 258x55, 48x55, 48x258, 0x258

* *Result:* The slicer will now display a greyed-out zone on the bed where you cannot place objects.

#### **D. Modify Machine G-code (The Logic)**

Go to the **Machine G-code** tab.  
**Start G-code:**

* Replace the **entire block** with the following code. This ensures the printer initializes safely and pauses for you to attach the pen.
```
  ;===== machine: P1S \====================  
  ;===== date: 20240919 \==================  
  ;===== start printer sound \================  
  M17  
  M400 S1  
  M1006 S1  
  M1006 A0 B10 L100 C37 D10 M60 E37 F10 N60  
  M1006 A0 B10 L100 C41 D10 M60 E41 F10 N60  
  M1006 A0 B10 L100 C44 D10 M60 E44 F10 N60  
  M1006 A0 B10 L100 C0 D10 M60 E0 F10 N60  
  M1006 A46 B10 L100 C43 D10 M70 E39 F10 N100  
  M1006 A0 B10 L100 C0 D10 M60 E0 F10 N100  
  M1006 A43 B10 L100 C0 D10 M60 E39 F10 N100  
  M1006 A0 B10 L100 C0 D10 M60 E0 F10 N100  
  M1006 A41 B10 L100 C0 D10 M100 E41 F10 N100  
  M1006 A44 B10 L100 C0 D10 M100 E44 F10 N100  
  M1006 A49 B10 L100 C0 D10 M100 E49 F10 N100  
  M1006 A0 B10 L100 C0 D10 M100 E0 F10 N100  
  M1006 A48 B10 L100 C44 D10 M60 E39 F10 N100  
  M1006 A0 B10 L100 C0 D10 M60 E0 F10 N100  
  M1006 A44 B10 L100 C0 D10 M90 E39 F10 N100  
  M1006 A0 B10 L100 C0 D10 M60 E0 F10 N100  
  M1006 A46 B10 L100 C43 D10 M60 E39 F10 N100  
  M1006 W

  ;===== reset machine status \=================  
  M290 X40 Y40 Z2.6666666  
  G91  
  M17 Z0.4 ; lower the z-motor current  
  G380 S2 Z30 F300 ; G380 is same as G38; lower the hotbed , to prevent the nozzle is below the hotbed  
  G380 S2 Z-25 F300 ;  
  G1 Z5 F300;  
  G90  
  M17 X1.2 Y1.2 Z0.75 ; reset motor current to default  
  M960 S5 P1 ; turn on logo lamp  
  G90  
  M220 S100 ;Reset Feedrate  
  M221 S100 ;Reset Flowrate  
  M73.2   R1.0 ;Reset left time magnitude  
  M1002 set\_gcode\_claim\_speed\_level : 5  
  M221 X0 Y0 Z0 ; turn off soft endstop to prevent protential logic problem  
  G29.1 Z{+0.0} ; clear z-trim value first  
  M204 S10000 ; init ACC set to 10m/s^2

  ;===== wipe nozzle \===============================  
  M1002 gcode\_claim\_action : 14  
  M975 S1  
  M106 S255  
  G1 X65 Y230 F18000  
  G1 Y264 F6000  
  M109 S{nozzle\_temperature\_initial\_layer\[initial\_no\_support\_extruder\]-20}  
  G1 X100 F18000 ; first wipe mouth

  G0 X135 Y253 F20000  ; move to exposed steel surface edge  
  G28 Z P0 T300; home z with low precision,permit 300deg temperature  
  G29.2 S0 ; turn off ABL  
  G0 Z5 F20000

  G1 X60 Y265  
  G92 E0  
  G1 E-0.5 F300 ; retrack more  
  G1 X100 F5000; second wipe mouth  
  G1 X70 F15000  
  G1 X100 F5000  
  G1 X70 F15000  
  G1 X100 F5000  
  G1 X70 F15000  
  G1 X100 F5000  
  G1 X70 F15000  
  G1 X90 F5000  
  G0 X128 Y261 Z-1.5 F20000  ; move to exposed steel surface and stop the nozzle  
  M104 S140 ; set temp down to heatbed acceptable  
  M106 S255 ; turn on fan (G28 has turn off fan)

  M221 S; push soft endstop status  
  M221 Z0 ;turn off Z axis endstop  
  G0 Z0.5 F20000  
  G0 X125 Y259.5 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 Z0.5 F20000  
  G0 X125 Y262.5  
  G0 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 Z0.5 F20000  
  G0 X125 Y260.0  
  G0 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 Z0.5 F20000  
  G0 X125 Y262.0  
  G0 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 Z0.5 F20000  
  G0 X125 Y260.5  
  G0 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 Z0.5 F20000  
  G0 X125 Y261.5  
  G0 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 Z0.5 F20000  
  G0 X125 Y261.0  
  G0 Z-1.01  
  G0 X131 F211  
  G0 X124  
  G0 X128  
  G2 I0.5 J0 F300  
  G2 I0.5 J0 F300  
  G2 I0.5 J0 F300  
  G2 I0.5 J0 F300

  M109 S140 ; wait nozzle temp down to heatbed acceptable  
  G2 I0.5 J0 F3000  
  G2 I0.5 J0 F3000  
  G2 I0.5 J0 F3000  
  G2 I0.5 J0 F3000

  M221 R; pop softend status  
  G1 Z10 F1200  
  M400  
  G1 Z10  
  G1 F30000  
  G1 X128 Y128  
  G29.2 S1 ; turn on ABL  
  ;G28 ; home again after hard wipe mouth  
  M106 S0 ; turn off fan , too noisy  
  ;===== wipe nozzle end \================================

  ;===== bed leveling \==================================  
  M1002 judge\_flag g29\_before\_print\_flag  
  M622 J1

     M1002 gcode\_claim\_action : 1  
    G29 A X{first\_layer\_print\_min\[0\]} Y{first\_layer\_print\_min\[1\]} I{first\_layer\_print\_size\[0\]} J{first\_layer\_print\_size\[1\]}  
    M400  
    M500 ; save cali data

  M623  
  ;===== bed leveling end \================================  
  ;===== mech mode fast check============================  
  G1 X128 Y128 Z10 F20000  
  M400 P200  
  M970.3 Q1 A7 B30 C80  H15 K0  
  M974 Q1 S2 P0

  G1 X128 Y128 Z10 F20000  
  M400 P200  
  M970.3 Q0 A7 B30 C90 Q0 H15 K0  
  M974 Q0 S2 P0

  M975 S1  
  G1 F30000  
  G1 X230 Y15  
  G28 X ; re-home XY  
  ;===== mech mode fast check============================

  {if scan\_first\_layer}  
  ;start heatbed  scan====================================  
  M976 S2 P1  
  G90  
  G1 X128 Y128 F20000  
  M976 S3 P2  ;register void printing detection  
  {endif}

  ;========turn off light and wait extrude temperature \=============  
  M1002 gcode\_claim\_action : 0  
  M973 S4 ; turn off scanner  
  M400 ; wait all motion done before implement the emprical L parameters  
  ;M900 L500.0 ; Empirical parameters  
  M109 S\[nozzle\_temperature\_initial\_layer\]  
  M960 S1 P0 ; turn off laser  
  M960 S2 P0 ; turn off laser  
  M106 S0 ; turn off fan  
  M106 P2 S0 ; turn off big fan  
  M106 P3 S0 ; turn off chamber fan

  M975 S1 ; turn on mech mode supression  
  G90  
  M83  
  T1000

  M400 U1 ; PAUSE FOR PEN ATTACHMENT
```
**End G-code:**

* **Add to the very TOP** of the End G-code box:
```
  ; \--- SAFE PLOTTER FINISH \---  
  G91 ; Relative positioning  
  G1 Z20 F3000 ; Lift UP 20mm immediately  
  G90 ; Absolute positioning  
  M400 U1 ; PAUSE: REMOVE PEN NOW  
  ; \---------------------------
```
### **3\. Create the "Plotter" Process Profile**

1. Edit a "0.20mm Standard" profile.  
2. **Save As:** Plotter Process.

**A. Quality Settings:**

* **Layer Height:** 0.10 mm.  
* **First Layer Height:** 0.10 mm.  
* **Wall Generator:** Arachne.  
* **Min Wall Width:** 50% (or 0.2mm).  
* **Precision \> Slice gap closing radius:** 0.001 mm.  
* **Precision \> Resolution:** 0.012.  
* **Precision \> XY Hole Compensation:** \-0.075.  
* **Precision \> XY Contour Compensation:** \+0.075 (Makes lines bolder).  
* **Ironing:** None (Uncheck all).

**B. Strength Settings:**

* **Walls:** 1\.  
* **Top/Bottom Shells:** 0\.  
* **Sparse Infill Density:** 0% (unless filling a solid shape).  
* **Sparse Infill Pattern:** Rectilinear (or Rectilinear Aligned).

**C. Speed Settings:**

* **Outer Wall:** 300 \- 400 mm/s (For Stabilo).  
* **Inner Wall:** 300 \- 400 mm/s.

**D. Other Essential Settings:**

* **Support \> Enable Support:** Unchecked (Disabled).  
* **Others \> Brim Type:** No brim (Disabled).  
* **Others \> Skirt Loops:** 0 (Disabled).  
* *Note: If these are on, the pen will draw off the paper or create unwanted lines.*

## **Phase 2: Test the Base Mount (Dry Run)**

**Goal:** Ensure the printer moves correctly and pauses without crashing, with minimal hardware attached.

1. **Prepare File:** Slice a simple SVG (like a small square) using your new profiles.  
   * **IMPORTANT:** Select the object, click **Scale**, unlock the "Uniform Scale" box, and set **Z (Size)** to **0.1 mm**. This ensures exactly 1 layer is generated.  
   * **Placement:**  
     * You will see a greyed-out zone on the bed (from the Excluded Bed Area setting).  
     * **Place your object in the valid (non-grey) area.**  
     * This placement correlates to the safe physical area on your paper.  
2. **⚠️ START PRINT EMPTY ⚠️:** Ensure the toolhead is completely **empty** (NO mounts, NO pen).  
3. **Wait for Start Pause:**  
   * The printer will Home and Level.  
   * It will then **PAUSE**.  
4. **MANUAL ACTION:**  
   * Use the printer screen controls to **move the toolhead forward (Y-axis)** until you can reach it comfortably.  
5. **INSTALL BASE:**  
   * Clip **only** the Head-Mount-V1 onto the toolhead.  
6. **Resume:**  
   * Press Resume on the screen.  
   * Printer should "ghost draw" the square in the air.  
7. **End Pause:**  
   * Printer will lift up and **PAUSE**.  
   * Use screen controls to **move toolhead forward**.  
   * **⚠️ REMOVE THE BASE ⚠️**  
   * Press Resume to finish.

## **Phase 3: Test Full Assembly (No Pen)**

**Goal:** Ensure the full plastic assembly fits and clears the frame.

1. **Assemble:** Assemble the Spring Module and Guide Stabilo onto the Base Mount *separately* (in your hands).  
2. **⚠️ START PRINT EMPTY ⚠️:** Ensure toolhead is **empty**.  
3. **Wait for Start Pause:**  
   * Printer Homes \-\> Pauses.  
   * Move head forward via screen.  
4. **INSTALL ASSEMBLY:**  
   * Snap the full plastic assembly (Base \+ Spring \+ Guide) onto the toolhead.  
5. **Resume:**  
   * Watch closely to ensure the protruding plastic parts do not hit the door, the auxiliary fan (left side), or the poop chute (back).  
6. **End Pause:**  
   * Printer lifts \-\> Pauses.  
   * Move head forward.  
   * **⚠️ REMOVE ASSEMBLY ⚠️**  
   * Press Resume.

## **Phase 4: Pen Calibration & Live Tuning**

**Goal:** Dial in the exact pen height safely by iterating. Do NOT try to guess the height while the printer is paused at the start.

1. **Prepare Paper:** Tape a sheet of paper to the bed.  
   * *Margins:* 29mm from Right, 41mm from Back.  
2. **⚠️ START PRINT EMPTY ⚠️:** Start the print with the toolhead **empty**.  
3. **START PAUSE (Attach):**  
   * The printer initializes and pauses.  
   * **ACTION 1:** Move toolhead forward via screen.  
   * **ACTION 2:** Attach the **Full Pen Assembly**.  
   * **ACTION 3:** **CRITICAL:** Insert the pen so the tip is **barely protruding** (just below the plastic guide). We want it to be *too high* at first.  
   * **ACTION 4:** Press **RESUME**.  
4. **THE ITERATIVE LOOP:**  
   * The printer will move to the paper and start "drawing" (likely floating in the air).  
   * **STEP A: Manual Pause:** Press **PAUSE** on the printer screen.  
   * **STEP B: Adjust:** Move head forward if needed. Push the pen down slightly (1-2mm).  
   * **STEP C: Resume:** Press Play.  
   * **STEP D: Observe:**  
     * If still floating \-\> Repeat Steps A-C.  
     * If drawing \-\> Watch the Z-Hop (travel moves).  
       * *Success:* Draws lines, lifts cleanly when moving.  
       * *Failure:* Draws lines, but drags a line across the paper when moving. (Pen is too low).  
5. **END PAUSE (Safety Step):**  
   * The drawing finishes. The printer lifts up and **PAUSES**.  
   * **ACTION 1:** Use the printer screen to **move the head forward**.  
   * **ACTION 2:** ⚠️ **REMOVE THE PEN ASSEMBLY. ⚠️**  
   * *Warning:* If you do not remove the assembly now, the printer may lower itself at the very end of the script and crush the pen against the bed.  
   * **ACTION 3:** Press **RESUME** to finish.

### **Fine Tuning**

* **Too much pressure?** (Pen tip smashing): Increase **Z-Offset** in slicer (e.g., 17.0 \-\> 17.5) OR lift the pen slightly higher in the holder.  
* **Too little pressure?** (Skipping lines): Decrease **Z-Offset** in slicer (e.g., 17.0 \-\> 16.5) OR push pen deeper manually at the start pause.

## **Troubleshooting**

| Problem | Solution |
| :---- | :---- |
| **Slicer generates 50+ layers** | Unlock scale in slicer and set object Z-height to 0.1mm (matching your layer height). |
| **Printer pauses then immediately resumes** | Your G-code is missing U1. Ensure it says M400 U1. |
| **Pen drags during travel** | Z-Hop is not working. Check "Z-Hop Type" is **Normal** (not Slope/Spiral). |
| **Lines ignored by slicer** | Enable **Arachne** wall generator and set Min Wall Width to **0.2mm**. |

