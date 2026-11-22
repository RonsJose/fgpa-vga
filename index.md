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

<img width="1918" height="1031" alt="image" src="https://github.com/user-attachments/assets/40a9cc45-b419-419a-b5da-4058bc2c2a5d" />


### **Template Code**

A VGA interface works by instructing each individual pixel going left to right per line on a 640x480 display. It uses horizontal syncs to reset the electron beam back to the right hand side and the vertical sync to reset the current frame and go back to the top left position to get ready for the next frame.

VGASync tracks the horizontal and veritcal sync and sets the vid_on flag, to track the position of the beam. VGAColourStripes splits the 640 horizontal pixels into 8 sections assigning 80 pixels to each colour. VGATop wires everything together and routes the RBG outputs so that it does not create noise when the beam is being reset by using the vid_on flag.

### **Simulation**
Simulation is used to test the hardware logic in software before testing in the real world, its a way to imitate a real world system to test scenarios and predict outcomes and understand how it would work safely and affordably. In vivado to simulate the VGA driver you need a simulation source since both colourStripes and VGASync need a clock and reset to work, in the demo we used TestBench it has both a clock and reset. 

```verilog
    // TB Clock
    always begin

        clk = 1'b0;
    
        #(T/2);
    
        clk = 1'b1;
   
        #(T/2);

    end


    // TB Reset

    initial begin

        rst = 1'b1;
    
        repeat(2) begin
    
        @(negedge clk);
        
    end
    
      rst = 1'b0;
      
    end
 ```
Then we run a behavorial simulation which executes the verilog RTL logic over a set amount of time that you can choose, and lets you see the waveform without ever having to use a physical board. I ran the behavioral simulation for 10us with a clock period of 10 ns and it provided me with the waveform below:

<img width="1057" height="374" alt="image" src="https://github.com/user-attachments/assets/91c314d9-39d3-41e3-a52f-e920c87834e1" />

You can see at the start that the reset is high for 20 ns this is to make sure all the counters and registers initialize properly without starting with unknown values. While reset is high the vid_on flag is low, when reset goes low the vid_on flag goes high and remains active for all 640 pixels and you can see the red, geen, and blue rgb values go through all 8 colours in the stripes. Then when the beam goes past 640 pixels into the horizontal blanking area vid_on goes low, and you can see hsync go low after it has reached the limit and go back to high and the counter for which row it is increase from 0 to 1. Vsync doesnt go low because it only resets after each frame and 10 us is not enough time for that.
 
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
