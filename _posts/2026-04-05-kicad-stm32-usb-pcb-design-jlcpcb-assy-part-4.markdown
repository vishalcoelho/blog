---
layout: posts
title:  "First STM32 PCB Design in KiCad and JLCPCB Assembly: Part IV - Manufacturing and Assembly"
date:   2026-04-05 13:52:00 -0500
categories: kicad pcb-design
hidden: false
---
<!-- excerpts go here, right after the front matter -->
These are my notes from following [Phil's Lab Tutorial](https://www.youtube.com/watch?v=C7-8nUU6e3E) on designing your first PCB in KiCad 9.0 and using JLCPCB as the assembly house.

---

# Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
- [Placing the Order Number](#placing-the-order-number)
- [Tooling for Assembly](#tooling-for-assembly)
- [Exporting the Assembly Files](#exporting-the-assembly-files)
- [Placing Component Centroids](#placing-component-centroids)
- [Creating a Bill of Materials (BOM)](#creating-a-bill-of-materials-bom)
- [Ordering](#ordering)
  - [Upload Gerber](#upload-gerber)
  - [PCB Assembly](#pcb-assembly)
  - [BOM and Position Files](#bom-and-position-files)
- [Conclusion](#conclusion)


---

# Introduction

In Part 3 we wired up our board, poured our ground and power planes and, annotated parts of the board on the top silkscreen to aid the user in identifying commonly used components. In this tutorial we will learn to finalize our design for manufacturing and assembly.

# Placing the Order Number

- JLCPcb typically adds an order number to the board
- the PCB designer is exptected to add a placeholder in the top silkscreen that the manufacturing house replaces with the order number:
  1. Choose the Front Silkscreen -> Text
  2. Type `JLCJLCJLCJLC`
  3. Reduce the size
  4. place text under the microcontroller

        ![Placing the JLCPCB order number placeholder](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_number.png)

> :memo: Note that we are intentionally obscuring the order number under the microcontroller

# Tooling for Assembly

Since we are getting JLPCB to assemble our board we need to add `tooling holes`

> :memo: Tooling holes are essential for securing and aligning printed circuit boards (PCBs) during the manufacturing and assembly processes. They help ensure accurate drilling, stencil printing, and component placement, which are critical for the overall quality and functionality of the PCB

JLCPCB needs 3-4 holes 0.576mm in radius spread around the board


1. Select the `Edge Cuts` layer
2. Select the `Circle` tool
3. Draw a circle near each of the mounting holes, 0.576mm in radius

    ![Add Tooling Holes](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_tooling_holes.png)

    ![Final Tooling Holes](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_tooling_holes_3d.png)

# Exporting the Assembly Files

1. Create two new folders in your project folder: `gerber` and `assembly`

    ![export folders](/assets/images/2025_10_12_post_kicad_pcb_design/export_folders.png)

2. `File` -> `Plot`
   1. Uncheck `Generate Gerber job file`
   2. Click `Generate Drill Files` -> `Generate` (default settings OK) -> `Close`
   3. Set the `Output Directory` to the `gerber` folder you created previously

   > :memo: Ignore the pop-up about relative paths being used. Click Yes/OK

   4. `Plot`
3. Zip up the gerber files
    ![Zip gerbers](/assets/images/2025_10_12_post_kicad_pcb_design/zip_gerber.png)

# Placing Component Centroids

> :memo: Pick and Place machines need to know the center of all the  components in order to place them correctly.

1. `File` -> `Fabrication Outputs` -> `Component Placements`

    ![Footprint position file](/assets/images/2025_10_12_post_kicad_pcb_design/footprint_position.png)

2. Select the `assemlby` folder as the output directory
3. Choose the `CSV` format
4. Check the option to `Generate a single file with both front and back position`
5. We need to hand edit the generated file to match JLCPCBs format
    ![Generated footprint position file](/assets/images/2025_10_12_post_kicad_pcb_design/footprint_positions_file_rename.png)

    > :memo: I have renamed the file from the default generated.

    1. Delete the following columns: `Val`, `Package`
    2. Switch columns: `Rot` and `Side`
    3. Rename columns:
       1. `Ref` -> `Designator`
       2. `PosX` -> `Mid X`
       3. `PosY` -> `Mid Y`
       4. `Side` -> `Layer`
       5. `Rot`-> `Rotation`
        ![Edit line](/assets/images/2025_10_12_post_kicad_pcb_design/footprint_positions_file_edit.png)


# Creating a Bill of Materials (BOM)

1. Go back to the Schematic Editor
2. `Edit Symbols Field`  -> `+` -> Create a `LCSC Part #` column
    ![Create a JLCPCB Part # column](/assets/images/2025_10_12_post_kicad_pcb_design/create_lcsc_part_col.png)
3. Go to [JLCPCB Parts Center](https://jlcpcb.com/parts)
   1. Search for a part, say *Capacitor 0603 2.2uF*
   2. Check the `basic` filter and apply
    ![Finding parts on JLCPCB](/assets/images/2025_10_12_post_kicad_pcb_design/finding_parts_on_jlcpcb.png)

   3. Copy the JLCPCB part # from the website to the symbols table against the appropriate component
   ![Completed part order list](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_completed_part_number_list.png)

   4. Hit `Apply, Save Schematic & Continue` -> `Ok`
   5. Select `Generate Bill of Materials...`
      1. Go to the `Edit` tab and rearrange the columns to only show
            `Value`, `Reference`, `Footprint`, `JLCPCB Part #`
         in that order
      2. Set the output file to `assembly/<filename>.csv`
      3. `Export`
        ![Export BOM](/assets/images/2025_10_12_post_kicad_pcb_design/export_bom.png)
      4.  As before, we need to hand edit this file to match JLCPCBs nomenclature. Change
        ```"Reference","Value","Footprint","JLCPCB Part #","Qty"```
        to
        ```Comment,Designator,Footprint,JLCPCB Part #```

# Ordering

## Upload Gerber

1. On the JLCPCB website, click `Order Now`
2. Start by uploading the gerber zip file you created
    ![JLCPCB Order 1](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_1.png)

    >:memo: I chose the cheapest shipping option and lead free solder.

3. Since we are doing a differential pair trace for the USB line we must select the `JLC04161H-7268` stackup for our board

    > :memo: A PCB stackup refers to the arrangement of copper and dielectric layers that make up a PCB.
    ![PCB Stackup](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_2.png)

4. Change `Mark on PCB` to `Order Number(Specify Position)`
   ![Order Number](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_3.png)

## PCB Assembly

![PCB Assembly](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_4.png)

Check the following
1. Tooling holes: `Added by Customer`
2. Parts Selection: `By Customer`
3. Rest of the settings are default
4. `Next` takes you to a review screen
    ![Review](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_5.png)

## BOM and Position Files

1. Upload the BOM and Position file -> `Process BOM & CPL` files
    ![BOM](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_6.png)
    :warning: If you see an error, its likely an issue with an entry in one of the files
    ![Error](/assets/images/2025_10_12_post_kicad_pcb_design/bom_doesnt_match_cpl_error.png)
    :star: I had to delete the entry for the mounting holes as they had no JLCPCB part #

2. Review the parts list and cost before accepting. Resolve any issues like *shortfalls* or  before proceeding.
    ![BOM Review](/assets/images/2025_10_12_post_kicad_pcb_design/jlcpcb_order_7.png)

3. :grey_question:OPTIONAL:grey_question: We see that a lot of the parts are rendered with their correct orientation. Now the starting indicators on the silkscreen should help JLCPCB place these parts correcty, but you can also fix it by editing the position file with the right rotation and reuploading
    ![Bad Rotation](/assets/images/2025_10_12_post_kicad_pcb_design/bad_rotation.png)

    For example, U1 (the microcontroller) needs to be rotated 270 degrees CCW, use the rotation tools at the top
    to correct the orientation of each part

    > :memo: The molex connectors seem to be off-center, switch to 2D -> drag the part into position -> switch to 3D.

    ![Rotation Rectified](/assets/images/2025_10_12_post_kicad_pcb_design/rotation_rectified.png)

4. When satisfied, move to the `Quote and order` phase and checkout.

# Conclusion

We learned to setup our board for manufacturing and assembly: exported Gerbers, Bill of Materials and Componenet position files, and create a new quote on JLCPCB.