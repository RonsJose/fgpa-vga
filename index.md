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

<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/40a9cc45-b419-419a-b5da-4058bc2c2a5d" />


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

<img width="800" height="374" alt="image" src="https://github.com/user-attachments/assets/91c314d9-39d3-41e3-a52f-e920c87834e1" />

You can see at the start that the reset is high for 20 ns this is to make sure all the counters and registers initialize properly without starting with unknown values. While reset is high the vid_on flag is low, when reset goes low the vid_on flag goes high and remains active for all 640 pixels and you can see the red, geen, and blue rgb values go through all 8 colours in the stripes. Then when the beam goes past 640 pixels into the horizontal blanking area vid_on goes low, and you can see hsync go low after it has reached the limit and go back to high and the counter for which row it is increase from 0 to 1. Vsync doesnt go low because it only resets after each frame and 10 us is not enough time for that.

> [NOTE]
> 
> Even though the rgb colour values are not set to 0 in the picture of the waveform when the vid_on flag goes low, using 2-1 multiplexers we force the rgb values to 0 regardless of what colourStripes says although it will still keep counting through the colours even in the blanking period which is what we see in the waveform.
 
### **Synthesis**

I'll describe it in more detail when going over my designs synthesis but the simple explaination for synthesis and implementation is that it translates the verilog RTL design into a gate level netlist specific for your FPGA board. Implementation takes the netlist from the synthesis and decides where and how to wire the gates on the board.

### **Demonstration**

<img width="800" height="600" alt="20261125_142358" src="https://github.com/user-attachments/assets/620ed9c7-6f9e-44ca-bda3-b5898c860ec5" />
 You can see all the 8 colour stripes on the monitor that are being output from the basys board, and the reset sets the screen to black.


## **My VGA Design Edit**
My final design is to have like a car moving across a road left to right and then when it reaches the right side it comes out the left side and continues forever. I don't think it is immensly complex and seems quite doable. I already have code that might work, I haven't had a chance to test it on the board yet. But before that I planned to make a villager face pixel art so for now I'll do this on that since I know it works. 

### **Code Adaptation**
At first I messed around with the colour stripes code and changed the rows and coloumns to see what that would do. And I got the following two images as a result:
<div style="display: flex; gap: 1px;">
    <img width="399" height="400" alt="image" src="https://github.com/user-attachments/assets/804a77d1-2898-4ff1-a899-252abd8fe3f0" style="width: 399px;"/>
    <img width="399" height="400" alt="image" src="https://github.com/user-attachments/assets/4b986d8e-bf50-4fdb-8fbd-30021ed08284" style="width: 399px;"/>
</div>


Then over the mid term, I tried to find a way to get the output of my verilog code displayed, and I came across the verilogVGA by Benjamin Otto on VS code. It's not the same as Vivado, and you have the change the template code a decent bit to get it to work but it does show you the display. At first I thought it only did static images but I found out recently that it does have support for moving images. But using the extension I figured out how to manipulate the pixels further by making some flags.

<p align="center">
    <img width="400" height="110" alt="image" src="https://github.com/user-attachments/assets/14d19b8d-ae9e-4c7e-b757-a05fa3aabbe1" />
</p>
<p align="center">
    <img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/fbce4d3b-6f97-49ea-9b44-55eeedf4d93a" />
</p>
For the villager face, I got an of the villager face on minecraft and drew out a 640 x 480 box and mapped each pixel to a box on the vga screen with what colour I was going to make that area.

For the villager face from Minecraft, I used a picture that had each pixel of the villager clearly visable and divided it into coloumns and rows. Then I sketched out an 8x8 square and mapped it to the dimenions of a VGA display 640x480 and decided what color each pixel was going to be.
<p align="center">
    <img width="800" height="525" alt="image" src="https://github.com/user-attachments/assets/eda8c8d6-4182-4260-9b31-c0508af52b55" />
</p>




### **Simulation**
I simulated my own design by commenting out the color stripes module and replacing it with my module for the car scene in the testbench. I also changed the params to a smaller range so that it is easier to see the changes through the waveform.

```verilog
parameter HDISP = 6;
parameter VDISP = 2;
parameter HFP = 1;
parameter VFP = 1;
parameter HPW = 1;
parameter VPW = 1;
parameter HLIM = 10;
parameter VLIM = 6;
parameter COUNTER_WIDTH = 3;
parameter COUNT_FROM = 0;
parameter COUNT_TO = 3;
parameter COUNT_RESET = 3;
parameter T = 10;

//ColourStripes #(.COUNTER_WIDTH(COUNTER_WIDTH), .COUNT_TO(COUNT_TO),
//              .COUNT_FROM(COUNT_FROM), .COUNT_RESET(COUNT_RESET)) i_colour_stripes
//   (.clk(clk), .rst(rst), .row(row), .col(col), .red(red), .green(green), .blue(blue));

vga_car_scene #() i_vga_car_scene
    (.clk(clk), .rst(rst), .vsync(vsync), .row(row), .col(col), .red(red), .green(green), .blue(blue)
    );
```

Just like in the demo reset is high at the start to let the registers and counters initialize without random values. You can see when vsync goes low indicating that a frame has ended and it's going onto the next one that the car_x variable (the horizontal position of the car) increases each time. The colors don't change because of the limited size of the parameters to show the changes in vsync and car_x in the waveform. Because of the limited parameters for the testbench, vid_on is low during most of the waveform. This is because there is only 2 rows that are active, the others are for vertical blanking which is why vid_on is only high for two rows and low for the rest.
<p align="center">
    <img width="800" height="330" alt="image" src="https://github.com/user-attachments/assets/fc287b7f-ef3a-4e6b-878d-9a3ee683cb9b" />
</p>


### **RTL analysis**
RTL analysis basically checks for any kind of errors through your verilog code syntax, compliation and design rules. It then translates the verilog code into a netlist of gates and creates a viewable schematic. This is not FGPA tecnology mapping, it's not for the board your using specifically it is just for the structural and logical verification of your design.

<img width="800" height="427" alt="image" src="https://github.com/user-attachments/assets/b5208e28-ecbb-4491-8ee6-c12518689fa8" />


### **Synthesis**
The synthesis takes the netlist from the RTL analysis and creates the FPGA netlist specific for the basys3. The outputs of my design are the same as the original although the process is different. The output for both are the rgb values, red, green & blue. 

<p align="center">
Original design synthesis : My design synthesis
</p>
<div style="display: flex; gap: 1px;">
    <img width="399" height="400" alt="image" src="https://github.com/user-attachments/assets/d0a47f2c-abab-4d3c-a498-926c72e0d82e" style="width: 399px;"/>
    <img width="399" height="400" alt="image" src="https://github.com/user-attachments/assets/a1127fd3-9dcc-4746-9aac-a367c4eb7f17" style="width: 399px;"/>
</div>

<p align="center">
Original design implementation : My design implementation
</p>
<div style="display: flex; gap: 1px;">
    <img width="399" height="400" alt="image" src="https://github.com/user-attachments/assets/a7a51607-ad15-4fa0-92fc-5dd971065f29" style="width: 399px;"/>
    <img width="399" height="400" alt="image" src="https://github.com/user-attachments/assets/3101bd68-f804-45bf-90c4-d42680027b64" style="width: 399px;"/>
</div>



### **Demonstration**
If you get your own design working on the Basys3 board, take a picture! Guideline: 1-2 sentences.
<p float="left">
<img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/af78a474-fcfe-41ec-9b1c-69644cf76697" />
<img width="800" height="500" alt="20261125_141857" src="https://github.com/user-attachments/assets/36bf0c9e-30c8-4715-bfbf-1ba7da62c2e7" />
<img width="800" height="500" alt="20261125_141900" src="https://github.com/user-attachments/assets/852747b6-2e23-4ffa-a8a3-5afb8a0db4a1" />
<img width="800" height="500" alt="20261125_141902" src="https://github.com/user-attachments/assets/4680a954-e006-4840-8dcb-91a5b710fb64" />
</p>


### **Code**

This part of the code is mostly the same as the ColourStripes module, the key differences being I removed the parameters that were used for ColourCycle, added vsync as an input.
I made local parameters to include the dimensions of the car, I made the variable that stores the horizontal location and set it to 0 to make sure it starts on the left side. I also made the variable that stores the previous state of vsync to detect changes.

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
```
On positive edges of the clock cycle, this code check if reset has been set and if it has reset the position of the car back to 0(the very left side), if not save the current verison as the previous verison of vsync this does falling edge detection. The if statement checks to see if vysnc has changed, it does this by anding the previous vysnc with the opposite of the current one if it is different there is a negative clock edge, this is done to check if a new frame has begun. If there is a new frame we check the position of the car horizontally and if it is greather than the range of the vga resolution (640) then set the position of the car back to 0 which will bring it back to the left side. Otherwise move the car forward by 3 pixels each time to make the car move forward.

```verilog
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
```
The first line is basicly an if statement condensed into one line using a ternany operator to check the position of where the car started compared to where the current pixel is. This is a very important line that makes it so that when the right side of the car hits the border the entire card doesnt get set to the start only the pixels that go beyond the 640 mark, thus a warping effect.

The second line checks to see if the current pixel being drawn is a pixel belonging to the car.

The third row just says everything above(visually below) this point is the road, unless it is part of the car as stated in the line above.

```verilog
    wire [10:0] rel_x = (col >= car_x) ? (col - car_x) : (col + 11'd640 - car_x);
    
    wire is_car = (rel_x < CAR_W) && (row >= CAR_Y) && (row < CAR_Y + CAR_H);

   
    wire is_road = (row >= 10'd320);
```
This section just assigns the rbg colour values from the sections defined eariler.

```verilog
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
```
This registers the rbg outputs and then assigns them to the output wire, you could this another way but this is cleaner code.
```verilog
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

[Verilog code extenstion on vs code](https://marketplace.visualstudio.com/items?itemName=otjamin.verilogvga)

[Repo of a design that I looked at](https://github.com/FPGADude/Digital-Design/blob/main/FPGA%20Projects/VGA%20Projects/VGA%20Bouncing%20Square/top.v)

[Video explaining that design ^ ](https://www.youtube.com/watch?v=PDgg1SN8g6s&t=1s)
