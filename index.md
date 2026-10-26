---
layout: home
title: FPGA VGA Driver Project
tags: fpga vga verilog
categories: demo
---

Hello, in this blog I'm going to be devloping and adapting an FPGA VGA driver to display graphics on a
640x480 display

## **Template VGA Design**
### **Project Set-Up**
Create a new RTL project and add the source, contraints, and simulation files. Select the xc7a35tcpg236-1 board and finish the setup. 

Next we need to add a clock by going into IP Catalog > Foundational elements > clock and reset > Clocking wizard. And we need to set this to 25 MHz.

<img src="https://raw.githubusercontent.com/melgineer/fpga-vga-verilog/main/docs/assets/images/VGAPrjSum.png">

### **Template Code**

> [!NOTE]
> A VGA interface works by instructing each individual pixel going left to right per line on a 640x480 display. It uses horizontal syncs to reset the electron beam back to the right hand side and the vertical sync to reset the current frame and go back to the top left position to get ready for the next frame.

VGASync tracks the horizontal and veritcal sync and sets the vid_on flag, to track the position of the beam. VGAColourStripes splits the 640 horizontal pixels into 8 sections assigning 80 pixels to each colour. VGATop wires everything together and routes the RBG outputs so that it does not create noise when the beam is being reset.

### **Simulation**
Explain the simulation process. Reference any important details, include a well-selected screenshot of the simulation. Guideline: 1/2 short paragraphs.
### **Synthesis**
Describe the synthesis and implementation processes. Consider including 1/2 useful screenshot(s). Guideline: 1/2 short paragraphs.
### **Demonstration**
Perhaps add a picture of your demo. Guideline: 1/2 sentences.

## **My VGA Design Edit**
Introduce your own design idea. Consider how complex/achievabble this might be or otherwise. Reference any research you do online (use hyperlinks).
### **Code Adaptation**
Briefly show how you changed the template code to display a different image. Demonstrate your understanding. Guideline: 1-2 short paragraphs.
### **Simulation**
Show how you simulated your own design. Are there any things to note? Demonstrate your understanding. Add a screenshot. Guideline: 1-2 short paragraphs.
### **Synthesis**
Describe the synthesis & implementation outputs for your design, are there any differences to that of the original design? Guideline 1-2 short paragraphs.
### **Demonstration**
If you get your own design working on the Basys3 board, take a picture! Guideline: 1-2 sentences.

## **More Markdown Basics**
This is a paragraph. Add an empty line to start a new paragraph.

Font can be emphasised as *Italic* or **Bold**.

Code can be highlighted by using `backticks`.

Hyperlinks look like this: [GitHub Help](https://help.github.com/).

A bullet list can be rendered as follows:
- vectors
- algorithms
- iterators

Images can be added by uploading them to the repository in a /docs/assets/images folder, and then rendering using HTML via githubusercontent.com as shown in the example below.

<img src="https://raw.githubusercontent.com/melgineer/fpga-vga-verilog/main/docs/assets/images/VGAPrjSrcs.png">
