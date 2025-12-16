# **Bambu Lab P1S Plotter Setup Guide (Falu Modular System)**

This repository contains a safe, incremental workflow for turning a **Bambu Lab P1S** into a pen plotter using the **Falu Modular System**.  

**⚠️ CRITICAL WARNING \- READ BEFORE PROCEEDING ⚠️**  

**YOU ARE PROCEEDING AT YOUR OWN RISK.**  
This process involves modifying printer "G-code", disabling safety sensors, and moving the machine in non-standard ways.

* **Risk of Hardware Damage:** If you fail to remove the pen before the print finishes, the printer **WILL** crash the toolhead into the bed, potentially bending the nozzle, destroying the build plate, or damaging the Z-axis motors.  
* **Risk of Collision:** If you install the mount before the printer has homed, the toolhead **WILL** crash into the front door or frame.  
* **Supervision Required:** NEVER walk away from the printer while in plotter mode. Always stay within reach of the power switch or "Stop" button.

**The author of this guide accepts no responsibility for damage to your printer.**

---

## **Quick Reference (For Experienced Users)**

If you've done this before, here's the condensed checklist:

**Profiles Setup:**
1. **Filament:** Flow `0.01`, Temps `180°C`/`25°C`, Fans off, Max volumetric `22 mm³/s`
2. **Printer:** Z-Hop `Normal`/`3.0mm`, Retraction `0.01mm`, Z-Offset `17.0mm`, Excluded area `0x0, 258x0, 258x55, 48x55, 48x258, 0x258`, Custom G-code with pauses
3. **Process:** Layer `0.10mm`, `Arachne`, Min wall `50%`, Walls `1`, Shells `0`, Infill `5-100%` `Rectilinear Aligned`, Speed `300-400mm/s`, Support/Brim/Skirt off
4. Activate all three profiles before slicing

**File Prep:**
- Import SVG → Unlock uniform scale → Set Z to `0.1mm` → Place in safe zone → Slice

**Test Sequence:**
- Phase 2: Start empty → Pause → Attach base only → Resume → Remove at end
- Phase 3: Start empty → Pause → Attach full assembly (no pen) → Resume → Check clearances → Remove at end
- Phase 4: Start empty → Pause → Attach assembly + pen (tip high) → Resume → Live adjust pen depth during print → Remove at end

**For detailed explanations, see the full guide below.**

---

## **Prerequisites: Printed Parts & Tools**

Before starting the testing phases, ensure you have these files printed from the [Falu Modular System on MakerWorld](https://makerworld.com/en/models/2029113-modular-system-for-a1-p1-x1-series):

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
2. In the top toolbar, click the **Filament** dropdown.
3. Select a **Generic PLA** profile, then click the **Edit** icon (pencil).
4. **Modify the following settings** (listed below).
5. When done, click **Save** and choose **Save As** → name it **"Plotter Filament"**.

**Settings to Change:**

* **Flow Ratio:** `0.01`
    - *Why needed:* The plotter uses a pen instead of extruding plastic. This near-zero value prevents the extruder motor from attempting to push filament while still keeping the extrusion system "active" in firmware.

    - *Effect:* Values above `0.01` would cause the extruder motor to grind filament unnecessarily. A value of `0` might trigger firmware safety checks or disable features that depend on active extrusion.

* **Nozzle Temperature:**
  * **Initial Layer:** `180°C`
  * **Other Layers:** `180°C`
    - *Why needed:* Bambu Lab firmware has a "cold extrusion prevention" safety feature that blocks movement if the nozzle is below a minimum temperature (typically 170-180°C). Setting `180°C` satisfies this requirement.

    - *Effect:* This temperature is high enough to prevent firmware errors but low enough that no plastic will ooze or cause thermal issues. Lower temperatures would trigger "cold extrusion" errors and halt the print.

* **Bed Temperature:** `25°C`
    - *Why needed:* Room temperature prevents any unnecessary heating of the bed, saving energy and avoiding thermal expansion.

    - *Effect:* Higher temperatures would waste power and could warp paper. The bed doesn't need heating since we're drawing, not printing.

* **Cooling Tab:**
  * **Part Cooling Fan:** `0%` (Min) / `0%` (Max)
  * **Auxiliary Fan:** `0%`
  * **Chamber Fan:** `0%`
    - *Why needed:* Fans create vibration and air movement that can disturb the pen tip's contact with paper, causing inconsistent line quality or paper movement.

    - *Effect:* Disabling all fans ensures stable, vibration-free operation. Running fans would introduce micro-vibrations that show up as wavy or inconsistent lines.

* **Max volumetric speed:** `22 mm³/s`
    - *Why needed:* This setting limits how fast the firmware thinks material is being extruded. Since we're using flow ratio `0.01`, this effectively caps movement speeds.

    - *Effect:* This value works with the flow ratio to prevent firmware from limiting speeds. Higher values have minimal impact since flow is near-zero.

### **2\. Create the "Plotter" Printer Profile**

1. In the top toolbar, click the **Printer** dropdown.
2. Select your **"Bambu Lab P1S 0.4 nozzle"** profile, then click the **Edit** icon (pencil).
3. **Modify the following settings** (listed below in sections A-D).
4. When done, click **Save** and choose **Save As** → name it **"P1S Plotter (Stabilo)"**.

#### **A. Extruder Settings**

* **Extruder Tab:**
  * **Z-Hop Type:** `Normal` (90° vertical lift)
      - *Why needed:* "Normal" Z-hop lifts the toolhead straight up vertically before travel moves. This is critical because the pen must lift completely clear of the paper to avoid dragging ink between drawing segments.

      - *Effect:* Without Z-hop, the pen drags across the paper during travel moves, creating unwanted lines. The "Slope" or "Spiral" hop types would cause the pen to drag during the lifting motion itself.

  * **Z-Hop Height:** `3.0 mm`
      - *Why needed:* The pen tip must clear the paper surface plus any slight paper curl or tape thickness. 3mm provides reliable clearance.

      - *Effect:* Too low (< 2mm) risks the pen still touching paper during travel. Too high (> 5mm) wastes time and adds unnecessary Z-axis movement.

  * **Retraction Length:** `0.01 mm`
      - *Why needed:* Bambu Lab firmware requires a non-zero retraction value to enable Z-hop. Since we're not extruding plastic, the actual retraction distance is irrelevant, but it must be `> 0` to unlock the Z-hop feature.

      - *Effect:* A value of `0` would disable Z-hop entirely. The `0.01mm` value is the minimum needed to activate the feature without causing the extruder motor to work.

#### **B. Set Z-Offset (Initial Starting Point)**

* **Extruder/Print Options Tab:**
  * **Z-Offset:** `17.0 mm` (starting value - you will fine-tune this later)
      - *Why needed:* The pen tip extends approximately 17mm below where the nozzle normally sits. By adding this offset, we tell the printer to keep the "nozzle" (actually the pen holder) 17mm higher than it thinks, which places the actual pen tip at the correct drawing height.

      - *Effect:* This offset "tricks" the firmware into positioning the pen correctly. Without it, the toolhead would try to position the nozzle at paper level, driving the pen tip 17mm into the bed and damaging both the pen and build plate.

      - *Calibration Note:* `17.0mm` is a safe starting point. You will refine this value during Phase 4 (Pen Calibration) using either live adjustments OR by changing this setting and re-slicing. See **"Z-Offset Calibration Guide"** section below for the full process.

#### **C. Set Excluded Bed Area (Safety Zones)**

*Why needed: The pen module extends beyond the normal nozzle footprint. When the toolhead moves to certain edge positions, this module can collide with the printer's frame, front door, or auxiliary fan housing. The excluded area coordinates create a "no-go zone" that the slicer respects.*

* Go to the **General** (or **Basic Information**) tab.
* Find the **Excluded bed area** field.
* **Copy and Paste** this exact coordinate string into the box:
  `0x0, 258x0, 258x55, 48x55, 48x258, 0x258`

* *Effect: The slicer will display a greyed-out zone on the bed where objects cannot be placed. This prevents you from accidentally positioning drawings in areas where the pen module would crash into the printer structure.*
*   - *What the coordinates mean:* This creates an L-shaped exclusion zone in the bottom-left corner of the bed, ensuring the module stays within safe physical bounds.

**Visual representation of the safe zone (top-down view):**
```
    BACK (Y=258mm)
    ╔══╦═════════════════════════════╗
    ║XX║                             ║
    ║XX║                             ║
    ║XX║                             ║
    ║XX║     SAFE DRAWING ZONE       ║ RIGHT
    ║XX║     (Place objects here)    ║ (X=258mm)
    ║XX║                             ║
    ║XX║                             ║
    ╠══╣                             ║
    ║XXXXXXXXX EXCLUDED ZONE XXXXXXXX║
    ╚════════════════════════════════╝
    FRONT (Y=0mm)
    LEFT (X=0mm)
```
The greyed-out L-shaped zone (marked with X) is the excluded area where the pen module could collide with the printer frame.

#### **D. Modify Machine G-code (The Logic)**

*Why needed: The start and end G-code sequences control exactly what the printer does before and after drawing. The custom code adds critical safety pauses that give you time to manually attach and remove the pen module, preventing crashes.*

Go to the **Machine G-code** tab.
**Start G-code:**

* Replace the **entire block** with the following code.
* *What this does:* The modified start sequence runs through the normal printer initialization (homing, bed leveling, heating), then forces the bed down 50mm and parks the toolhead at the front-center position. The final `M400 U1` command triggers a pause, during which you can safely attach the pen module without risk of collision.*
```
;===== machine: P1S ====================
;===== date: 20240919 ==================
;===== start printer sound ================
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

;===== reset machine status =================
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
M1002 set_gcode_claim_speed_level : 5
M221 X0 Y0 Z0 ; turn off soft endstop to prevent protential logic problem
G29.1 Z{+0.0} ; clear z-trim value first
M204 S10000 ; init ACC set to 10m/s^2

;===== wipe nozzle ===============================
M1002 gcode_claim_action : 14
M975 S1
M106 S255
G1 X65 Y230 F18000
G1 Y264 F6000
M109 S{nozzle_temperature_initial_layer[initial_no_support_extruder]-20}
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
;===== wipe nozzle end ================================

;===== bed leveling ==================================
M1002 judge_flag g29_before_print_flag
M622 J1

   M1002 gcode_claim_action : 1
  G29 A X{first_layer_print_min[0]} Y{first_layer_print_min[1]} I{first_layer_print_size[0]} J{first_layer_print_size[1]}
  M400
  M500 ; save cali data

M623
;===== bed leveling end ================================
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

{if scan_first_layer}
;start heatbed  scan====================================
M976 S2 P1
G90
G1 X128 Y128 F20000
M976 S3 P2  ;register void printing detection
{endif}

;========turn off light and wait extrude temperature =============
M1002 gcode_claim_action : 0
M973 S4 ; turn off scanner
M400 ; wait all motion done before implement the emprical L parameters
;M900 L500.0 ; Empirical parameters
M109 S[nozzle_temperature_initial_layer]
M960 S1 P0 ; turn off laser
M960 S2 P0 ; turn off laser
M106 S0 ; turn off fan
M106 P2 S0 ; turn off big fan
M106 P3 S0 ; turn off chamber fan

M975 S1 ; turn on mech mode supression
G90 ; Ensure absolute coordinates
G1 Z50 F3000 ; *** FORCE BED DOWN ***
G1 X128 Y50 F6000 ; *** PARK HEAD FRONT-CENTER ***
M83
T1000

M400 U1 ; PAUSE FOR PEN ATTACHMENT
```
**End G-code:**

* **Add to the very TOP** of the End G-code box:
* *What this does:* Immediately when drawing completes, the bed lifts 20mm to create clearance, then pauses. This CRITICAL pause gives you time to remove the pen module before the printer executes its normal end sequence (which might lower the bed back down or move in ways that would crash the module).*
```
  ; \--- SAFE PLOTTER FINISH \---  
  G91 ; Relative positioning  
  G1 Z20 F3000 ; Lift UP 20mm immediately  
  G90 ; Absolute positioning  
  M400 U1 ; PAUSE: REMOVE PEN NOW  
  ; \---------------------------
```
### **3\. Create the "Plotter" Process Profile**

1. In the top toolbar, click the **Process** dropdown.
2. Select a **"0.20mm Standard"** profile, then click the **Edit** icon (pencil).
3. **Modify the following settings** (listed below in sections A-D).
4. When done, click **Save** and choose **Save As** → name it **"Plotter Process"**.

**A. Quality Settings:**

* **Layer Height:** `0.10 mm`
    - *Why needed:* Since we're drawing in 2D (not building up layers), the layer height determines how the slicer interprets the model. `0.10mm` ensures the model is treated as a single drawing layer.

    - *Effect:* Must match the Z-height of your imported SVG/model. If your model is `0.1mm` tall, this setting ensures exactly 1 layer is generated.

* **First Layer Height:** `0.10 mm`
    - *Why needed:* Must match layer height for single-layer drawings.

    - *Effect:* Different values would cause the slicer to generate multiple layers or fail to slice properly.

* **Wall Generator:** `Arachne`
    - *Why needed:* Arachne is a variable-width wall generator that handles thin features and fine details better than the classic generator. It can trace lines thinner than the nozzle diameter.

    - *Effect:* The classic generator ignores features narrower than the nozzle width. Arachne will attempt to draw them, making it essential for detailed line art.

* **Min Wall Width:** `50%` (or `0.2mm`)
    - *Why needed:* This tells Arachne the minimum line width to attempt drawing. At `50%` of a `0.4mm` nozzle (`0.2mm`), it will trace even very fine SVG lines.

    - *Effect:* Higher values skip thin lines. Lower values might generate too many thin paths that overlap poorly.

* **Precision > Slice gap closing radius:** `0.001 mm`
    - *Why needed:* This controls how aggressively the slicer "heals" tiny gaps in geometry. A very small value (`0.001mm`) prevents the slicer from merging separate lines that should stay distinct.

    - *Effect:* Larger values cause nearby lines to merge together, losing fine detail in drawings.

* **Precision > Resolution:** `0.012`
    - *Why needed:* Controls how precisely the slicer follows curves. Lower values create smoother curves with more points.

    - *Effect:* Higher values (e.g., `0.05`) would make curves look faceted or polygonal. Too low (e.g., `0.001`) generates excessive points and slows both slicing and plotting.

* **Precision > XY Hole Compensation:** `-0.075`
    - *Why needed:* Makes internal features (holes, interior shapes) slightly smaller. This compensates for pen width spreading.

    - *Effect:* Without compensation, drawn circles appear slightly larger than designed. This shrinks them to match intent.

* **Precision > XY Contour Compensation:** `+0.075`
    - *Why needed:* Makes outer contours slightly larger to compensate for pen width. Results in bolder, more visible outer lines.

    - *Effect:* Positive compensation ensures perimeter lines aren't drawn too thin. Adjust based on pen thickness.

* **Ironing:** `None` (Uncheck all)
    - *Why needed:* Ironing is a 3D printing feature that makes top surfaces smooth by doing extra passes. It's irrelevant for 2D plotting.

    - *Effect:* Leaving it enabled would cause the pen to make redundant passes over already-drawn areas.

**B. Strength Settings:**

* **Wall loops:** `1`
    - *Why needed:* We only want to trace the outline/perimeter of shapes once. Additional walls would create duplicate overlapping lines.

    - *Effect:* Setting to `2+` would draw multiple concentric outlines, wasting ink and making lines appear thicker/darker.

* **Top shell layers:** `0`
* **Bottom shell layers:** `0`
    - *Why needed:* Top and bottom shells are solid infill layers for 3D prints. For 2D drawing, we only want the perimeter.

    - *Effect:* Non-zero values would cause the slicer to try filling in solid areas, which may or may not be desired depending on your drawing.

* **Sparse Infill Density:** `5-100%` (`5%` for light hatching, `100%` for solid fill)
    - *Why needed:* Controls how much the interior of closed shapes is filled. Start with `5%` for slight hatching, increase if you want more coverage. `100%` creates fully solid-filled areas but is time-consuming.

    - *Effect:* `5%` creates light cross-hatching. Higher values (`20-50%`) add more density. `100%` fills completely solid. `0%` draws only outlines with no interior fill.

* **Sparse Infill Pattern:** `Rectilinear Aligned` (default, looks more natural)
    - *Why needed:* Rectilinear Aligned creates clean parallel lines aligned with the object's orientation, producing a more natural appearance for filled shapes.

    - *Effect:* Aligned lines follow the object's geometry naturally. Regular Rectilinear works but may appear less organic. Other patterns (honeycomb, gyroid) create complex infill that may not be aesthetically desired for drawings.

**C. Speed Settings:**

* **Outer Wall:** `300 - 400 mm/s` (For Stabilo)
    - *Why needed:* The Stabilo pen tip can handle high speeds without skipping or lifting off the paper. Fast speeds dramatically reduce drawing time.

    - *Effect:* Lower speeds (< `200mm/s`) work but take much longer. Higher speeds (> `450mm/s`) may cause the pen to skip or the spring mechanism to lose contact with paper. Different pen types need different speeds (e.g., Posca requires `30-40mm/s`).

* **Inner Wall:** `300 - 400 mm/s`
    - *Why needed:* Should match outer wall speed for consistent line quality throughout the drawing.

    - *Effect:* Mismatched speeds would create visible differences between different parts of the drawing.

**D. Other Essential Settings:**

* **Support > Enable Support:** `Unchecked` (Disabled)
    - *Why needed:* Support structures are for 3D prints with overhangs. They have no meaning for 2D drawings.

    - *Effect:* If enabled, the slicer might generate random support geometry that would be drawn as unwanted lines on your paper.

* **Others > Brim Type:** `No brim` (Disabled)
    - *Why needed:* A brim is an extra outline around prints to improve bed adhesion. For plotting, it would draw unwanted border lines around your artwork.

    - *Effect:* Enabled brims waste paper space and add lines you didn't intend in your design.

* **Others > Skirt Loops:** `0` (Disabled)
    - *Why needed:* A skirt is a test outline drawn before the actual print starts. For plotting, this would draw test lines on your paper.

    - *Effect:* Non-zero values make the pen draw loops around your design before starting the actual drawing, wasting paper and ink.

### **4\. Activate Your Plotter Profiles**

**CRITICAL:** Before slicing any files, you must activate all three custom profiles:

1. In the top toolbar, click the **Filament** dropdown → select **"Plotter Filament"**.
2. Click the **Printer** dropdown → select **"P1S Plotter (Stabilo)"**.
3. Click the **Process** dropdown → select **"Plotter Process"**.

You should now see all three profile names displayed in the toolbar. These will remain active until you change them.

### **5\. Import and Prepare an SVG File**

Before testing, you need to prepare a simple drawing file:

1. **Create or find a simple SVG:**
   - Create a basic shape (square, circle) in any vector program, OR
   - Download a simple SVG from the web
   - Start small (30-50mm size) for testing

2. **Import SVG into Orca Slicer:**
   - Drag and drop the `.svg` file into the 3D viewport, OR
   - Click **File → Import → Import 3D Model** and select your SVG
   - The SVG will appear as a flat 3D object on the build plate

3. **CRITICAL: Set Z-Height to Match Layer Height:**
   - Select the imported object (click on it)
   - In the right panel, click the **Scale** icon
   - **UNCHECK** the "Uniform Scaling" lock icon
   - Manually set **Z (Size)** to **`0.1 mm`** (matching your Layer Height setting)
   - This ensures exactly 1 layer is generated instead of 50+ layers

4. **Position the object:**
   - You will see a **greyed-out L-shaped exclusion zone** on the bed (from the Excluded Bed Area setting)
   - **Drag your object into the valid (non-grey) area** - this is the safe zone where the pen module won't collide with the printer frame
   - Position it away from edges: at least 30mm from right edge, 45mm from back edge

5. **Slice the file:**
   - Click **Slice Plate** button (top right)
   - Orca Slicer will generate the drawing path
   - Verify in the preview that you see exactly 1 layer, not multiple layers
   - **Do NOT send to printer yet** - you will do this in the testing phases below

## **Phase 2: Test the Base Mount Only (Verify Pauses Work)**

**Goal:** Verify that the printer pauses correctly and the base mount clips on securely, without risking collision from protruding parts.

1. **Send your prepared file to the printer** (from step 5 above) using network send or SD card/USB.
2. **⚠️ START PRINT EMPTY ⚠️:** Start the print job with the toolhead completely **empty** (NO mounts, NO pen).
3. **Wait for Start Pause:**
   - The printer will Home and Level.
   - It will then **PAUSE**.
4. **MANUAL ACTION:**
   - Use the printer screen controls to **move the toolhead forward (Y-axis)** until you can reach it comfortably.
5. **INSTALL BASE:**
   - Clip **only** the Head-Mount-V1 onto the toolhead.
6. **Resume:**
   - Press Resume on the screen.
   - Printer should "ghost draw" the square in the air.
7. **End Pause:**
   - Printer will lift up and **PAUSE**.
   - Use screen controls to **move toolhead forward**.
   - **⚠️ REMOVE THE BASE ⚠️**
   - Press Resume to finish.

## **Phase 3: Test Full Assembly Without Pen (Verify Clearance)**

**Goal:** Ensure the spring module and pen guide don't collide with the printer frame, door, or fans during movement. This tests the excluded bed area and physical clearances.

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

## **Z-Offset Calibration Guide**

The Z-offset value (`17.0mm` starting point) determines pen pressure on paper. You have **two methods** to calibrate:

### **Method 1: Live Adjustment During Print (Recommended for First Time)**
This is the iterative process described in Phase 4 above:
1. Start with pen tip barely protruding from holder (too high initially)
2. Begin print, observe if pen touches paper
3. Pause → push pen down 1-2mm → resume
4. Repeat until pen draws cleanly AND lifts clear during travels
5. Note the final pen position in the holder for future prints

**Pros:** Immediate feedback, no re-slicing needed
**Cons:** Requires manual adjustment each print

### **Method 2: Adjust Z-Offset Setting in Slicer (For Repeatable Results)**
Once you understand the correct pen depth:
1. Edit your **Printer Profile** → **Extruder/Print Options Tab**
2. Adjust **Z-Offset** value:
   - **Pen pressing too hard?** Increase offset (e.g., `17.0` → `17.5mm`)
   - **Pen too light/skipping?** Decrease offset (e.g., `17.0` → `16.5mm`)
3. Save the profile
4. Re-slice your file
5. Print with pen installed at consistent depth in holder

**Pros:** Repeatable, no manual adjustment needed each print
**Cons:** Requires re-slicing after each change

### **Symptoms Guide**
- **Lines too faint or missing:** Pen not touching paper → decrease Z-offset OR push pen deeper
- **Pen pressing hard, ink bleeding:** Too much pressure → increase Z-offset OR pull pen higher
- **Pen drags during travels:** Z-hop not lifting high enough → check "Z-Hop Type" is `Normal`, increase "Z-Hop Height"

## **Troubleshooting**

| Problem | Solution |
| :---- | :---- |
| **Slicer generates 50+ layers** | Unlock scale in slicer and set object Z-height to `0.1mm` (matching your layer height). |
| **Printer pauses then immediately resumes** | Your G-code is missing `U1`. Ensure it says `M400 U1`. |
| **Pen drags during travel** | Z-Hop is not working. Check "Z-Hop Type" is **`Normal`** (not Slope/Spiral). |
| **Lines ignored by slicer** | Enable **`Arachne`** wall generator and set Min Wall Width to **`0.2mm`**. |

