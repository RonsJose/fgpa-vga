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
Create a new RTL project and add the source, contraints, and simulation files that are given. Select the xc7a35tcpg236-1 board and finish the setup. 

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

> [NOTE]
> 
> Even though the rgb colour values are not set to 0 in the picture of the waveform when the vid_on flag goes low, using 2-1 multiplexers we force the rgb values to 0 regardless of what colourStripes says although it will still keep counting through the colours even in the blanking period which is what we see in the waveform.
 
### **Synthesis**

I'll describe it in more detail when going over my designs synthesis but the simple explaination for synthesis and implementation is that it translates the verilog RTL design into a gate level netlist specific for your FPGA board. Implementation takes the netlist from the synthesis and decides where and how to wire the gates on the board.

### **Demonstration**

I forget to take a picture, but I'll take one of the demo one later. Basicly you can see all the 8 colour stripes on the monitor that are being output from the basys board, and the reset sets the screen to black.


## **My VGA Design Edit**
My final design is to have like a car moving across a road left to right and then when it reaches the right side it comes out the left side and continues forever. I don't think it is immensly complex and seems quite doable. I already have code that might work, I haven't had a chance to test it on the board yet. But before that I planned to make a villager face pixel art so for now I'll do this on that since I know it works. 

### **Code Adaptation**
At first I messed around with the colour stripes code and changed the rows and coloumns to see what that would do. And I got the following two images as a result:
<img width="2211" height="1659" alt="image" src="https://github.com/user-attachments/assets/804a77d1-2898-4ff1-a899-252abd8fe3f0" />
<img width="2195" height="1646" alt="image" src="https://github.com/user-attachments/assets/4b986d8e-bf50-4fdb-8fbd-30021ed08284" />
Then over the mid term, I tried to find a way to get the output of my verilog code displayed, and I came across the verilogVGA by Benjamin Otto on VS code. It's not the same as Vivado, and you have the change the template code a decent bit to get it to work but it does show you the display. At first I thought it only did static images but I found out recently that it does have support for moving images. But using the extension I figured out how to manipulate the pixels further by making some flags.
<img width="433" height="110" alt="image" src="https://github.com/user-attachments/assets/14d19b8d-ae9e-4c7e-b757-a05fa3aabbe1" />
<img width="1000" height="828" alt="image" src="https://github.com/user-attachments/assets/fbce4d3b-6f97-49ea-9b44-55eeedf4d93a" />
For the villager face, I got an of the villager face on minecraft and drew out a 640 x 480 box and mapped each pixel to a box on the vga screen with what colour I was going to make that area..
<img width="2853" height="2140" alt="image" src="https://github.com/user-attachments/assets/eda8c8d6-4182-4260-9b31-c0508af52b55" />



### **Simulation**
<img width="830" height="325" alt="image" src="https://github.com/user-attachments/assets/201bb42b-272e-40c0-9f7f-57e7a7a3ff3f" />

Show how you simulated your own design. Are there any things to note? Demonstrate your understanding. Add a screenshot. Guideline: 1-2 short paragraphs.
### **Synthesis**

#### **RTL analysis**
<img width="1832" height="713" alt="Screenshot 2026-11-25 011849" src="https://github.com/user-attachments/assets/a92c6f2d-fdb8-4f6e-aff0-44faa5322433" />

Describe the synthesis & implementation outputs for your design, are there any differences to that of the original design? Guideline 1-2 short paragraphs.

### **Demonstration**
If you get your own design working on the Basys3 board, take a picture! Guideline: 1-2 sentences.
<img width="2060" height="1545" alt="image" src="https://github.com/user-attachments/assets/af78a474-fcfe-41ec-9b1c-69644cf76697" />

The code for the villager face screen:
````verilog
module VillagerFace #(
    parameter COUNTER_WIDTH = 32, 
    parameter COUNT_FROM = 0,
    parameter COUNT_TO = 32'b1 << 26,
    parameter COUNT_RESET = 32'b1 << 27)
    (input clk, rst,
     input [10:0] row, col, 
     output [3:0] red, green, blue);

reg [3:0] red_reg, green_reg, blue_reg, red_next, green_next, blue_next;

// Next state & output logic state  machine 
always@* begin
   if((row >= 11'd0 && row <11'd180) && (col >= 11'd0 && col <11'd80)) begin
      red_next   <= 4'b1111;
      green_next <= 4'b1100;
      blue_next  <= 4'b1010;
   end
   else  if((row >= 11'd0 && row <11'd180) && (col>=11'd80 && col<11'd160)) begin
      red_next   <= 4'b1111;
      green_next <= 4'b1101;
      blue_next  <= 4'b1011;;
   end
   else if((row >= 11'd0 && row < 11'd60) && (col>=11'd160 && col<11'd240)) begin
      red_next   <= 4'b1111;
      green_next <= 4'b1100;
      blue_next  <= 4'b1010;
   end
   else if((row >= 11'd60 && row < 11'd180) && (col>=11'd160 && col<11'd240)) begin
    red_next   <= 4'b1111;
    green_next <= 4'b1101;
    blue_next  <= 4'b1011;
   end
     else if((row >= 11'd0 && row < 11'd180) && (col>=11'd240 && col<11'd480)) begin
    red_next   <= 4'b1111;
    green_next <= 4'b1101;
    blue_next  <= 4'b1011;
   end
   else if((row >= 11'd0 && row < 11'd180) && (col>=11'd480 && col<11'd560)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end
   else if((row >= 11'd0 && row < 11'd240) && (col>=11'd560 && col<11'd640)) begin
red_next   <= 4'b1111;
green_next <= 4'b1101;
blue_next  <= 4'b1011;
   end
   else if((row >= 11'd180 && row < 11'd240) && (col>=11'd0 && col<11'd80)) begin
   red_next   <= 4'b1111;
green_next <= 4'b1101;
blue_next  <= 4'b1011;
   end
   else if((row >= 11'd180 && row < 11'd240) && (col>=11'd80 && col<11'd560)) begin
red_next   <= 4'b0110;
green_next <= 4'b0100;
blue_next  <= 4'b0010;
   end 
else if((row >= 11'd240 && row < 11'd300) && (col>=11'd0 && col<11'd80)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end 
   else if((row >= 11'd240 && row < 11'd300) && (col>=11'd80 && col<11'd160)) begin
red_next   <= 4'b1111;
green_next <= 4'b1111;
blue_next  <= 4'b1111;
   end 
   else if((row >= 11'd240 && row < 11'd300) && (col>=11'd160 && col<11'd240)) begin
red_next   <= 4'b0101;
green_next <= 4'b1100;
blue_next  <= 4'b0111;
   end 
   else if((row >= 11'd240 && row < 11'd300) && (col>=11'd240 && col<11'd400)) begin
red_next   <= 4'b1111;
green_next <= 4'b1101;
blue_next  <= 4'b1011;
   end 
   else if((row >= 11'd240 && row < 11'd300) && (col>=11'd400 && col<11'd480)) begin
red_next   <= 4'b0101;
green_next <= 4'b1100;
blue_next  <= 4'b0111;
   end
   else if((row >= 11'd240 && row < 11'd300) && (col>=11'd480 && col<11'd560)) begin
red_next   <= 4'b1111;
green_next <= 4'b1111;
blue_next  <= 4'b1111;
   end 
 else if((row >= 11'd240 && row < 11'd300) && (col>=11'd560 && col<11'd640)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end 
 else if((row >= 11'd300 && row < 11'd480) && (col>=11'd0 && col<11'd80)) begin
red_next   <= 4'b1011;
green_next <= 4'b1000;
blue_next  <= 4'b0101;
   end 
  else if((row >= 11'd300 && row < 11'd480) && (col>=11'd80 && col<11'd160)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end 
  else if((row >= 11'd300 && row < 11'd360) && (col>=11'd160 && col<11'd240)) begin
red_next   <= 4'b1111;
green_next <= 4'b1101;
blue_next  <= 4'b1011;
   end 
  else if((row >= 11'd360 && row < 11'd420) && (col>=11'd160 && col<11'd240)) begin
red_next   <= 4'b0110;
green_next <= 4'b0100;
blue_next  <= 4'b0011;
   end 
  else if((row >= 11'd420 && row < 11'd480) && (col>=11'd160 && col<11'd240)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end 
  else if((row >= 11'd300 && row < 11'd480) && (col>=11'd240 && col<11'd400)) begin
red_next   <= 4'b1011;
green_next <= 4'b1000;
blue_next  <= 4'b0101;
   end 
  else if((row >= 11'd300 && row < 11'd360) && (col>=11'd400 && col<11'd480)) begin
red_next   <= 4'b1111;
green_next <= 4'b1101;
blue_next  <= 4'b1011;
   end 
  else if((row >= 11'd360 && row < 11'd420) && (col>=11'd400 && col<11'd480)) begin
red_next   <= 4'b0110;
green_next <= 4'b0100;
blue_next  <= 4'b0011;
   end 
  else if((row >= 11'd420 && row < 11'd480) && (col>=11'd400 && col<11'd480)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end 
   else if((row >= 11'd300 && row < 11'd480) && (col>=11'd480 && col<11'd560)) begin
red_next   <= 4'b1111;
green_next <= 4'b1100;
blue_next  <= 4'b1010;
   end 
   else if((row >= 11'd300 && row < 11'd480) && (col>=11'd560 && col<11'd640)) begin
red_next   <= 4'b1011;
green_next <= 4'b1000;
blue_next  <= 4'b0101;
   end 
   else begin
      red_next   <= 4'b0000;
      green_next <= 4'b0000;
      blue_next  <= 4'b0000;
   end
end

// Register RGB outputs    
always@(posedge clk, posedge rst) begin
    if(rst == 1'b1) begin
        red_reg   <= 4'b0000;
        green_reg <= 4'b0000;
        blue_reg  <= 4'b0000;
    end
    else begin
        red_reg   <= red_next;
        green_reg <= green_next;
        blue_reg  <= blue_next;
    end
end

// Assign output wires
assign red = red_reg;
assign green = green_reg;
assign blue = blue_reg;

endmodule
````
Code for moving block:
```verilog
`timescale 1ns / 1ps

module vga_car_scene 
    (input clk, rst, vsync,
     input [10:0] col, row,
     output [3:0] red, green, blue 
     );

    reg [3:0] red_reg, green_reg, blue_reg, red_next, green_next, blue_next;

    localparam CAR_W = 10'd70;
    localparam CAR_H = 10'd35;
    localparam CAR_Y = 10'd370;

    reg [9:0] car_x = 10'd0;
    reg vsync_prev;

    always @(posedge clk) begin
        if (rst) begin
            car_x <= 10'd0;
            vsync_prev <= 1'b1;
        end else begin
            vsync_prev <= vsync;
            if (vsync_prev && !vsync) begin 
                if (car_x >= 10'd640)
                    car_x <= 10'd0;
                else
                    car_x <= car_x + 10'd3; 
            end
        end
    end

    
    
    wire [10:0] rel_x = (col >= car_x) ? (col - car_x) : (col + 11'd640 - car_x);
    
    wire is_car = (rel_x < CAR_W) && (row >= CAR_Y) && (row < CAR_Y + CAR_H);

   
    wire is_road = (row >= 10'd320);

    
    always @(*) begin
        if (is_car) begin
                red_next   <= 4'b1111;
                green_next <= 4'b0000;
                blue_next  <= 4'b0000; 
        end
        else if (is_road) begin
                red_next   <= 4'b0100;
                green_next <= 4'b0100;
                blue_next  <= 4'b0100;
        end
        else begin
                red_next   <= 4'b0101;
                green_next <= 4'b1010;
                blue_next  <= 4'b1111;
        end
    end

always@(posedge clk, posedge rst) begin
    if(rst == 1'b1) begin
        red_reg   <= 4'b0000;
        green_reg <= 4'b0000;
        blue_reg  <= 4'b0000;
    end
    else begin
        red_reg   <= red_next;
        green_reg <= green_next;
        blue_reg  <= blue_next;
    end
end

assign red = red_reg;
assign green = green_reg;
assign blue = blue_reg;

endmodule
```


## References
[AMD simulation tutorial](https://docs.amd.com/v/u/2018.2-English/ug937-vivado-design-suite-simulation-tutorial)

[Video on how to do simulation in Vivado](https://www.youtube.com/watch?v=sHVuhg0ttZQ)

[FPGA design flow](https://medium.com/@ahe24mobile/the-fpga-design-flow-explained-from-rtl-to-bitstream-290a3b983e4e)

[Synthesis](https://www.synopsys.com/glossary/what-is-synthesis.html)

[Minecraft Villager face](https://minecraftfaces.com/villager-face/) 

[Verilod code extenstion on vs code](https://marketplace.visualstudio.com/items?itemName=otjamin.verilogvga)
