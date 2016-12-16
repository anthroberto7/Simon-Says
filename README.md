# Simon-Says

Description of the Problem.
  The problem Simon-Says accomplishes is the creation of a matching game using an FPGA, a keyboard, and a desktop monitor. The game will take user input, and save the end of the sequence. After both players have at least one input the game can be told to check the sequence ends for a match. If they do match, then a check will appear and the level will increase by one. If they fail to make a match the current level remains.
Relevant background information to understand the problem:
The systems used in this design are Verilog hardware language, a vga output, and a PS2 port for the keyboard. These can be finicky systems to manipulate, especially the last two. A vga uses two important counters to display images onto the screen. These are the vertical and horizontal syncs. These two counters will tell how long the vga needs to collect data on each clock cycle in order to have the full are of image the designer wants displayed. For example my vertical sync module goes for 545 pixels, and my horizontal sync goes for 799. Every time the horizontal sync maxes, the vertical sync adds a pixel, and the horizontal counter begins again. This is similar to a typewriter filling a page. Every time a horizontal lines pixels are collected, it drops a line and begins the process over until the page is filled.
Another important system to understand is the PS2 port. The PS2 port uses its own clock as well as the base clock. The module to receive input from the PS2 keyboard involves a state machine that prepares, accepts, and checks the data inputs. Because keyboards can be very tricky, and due to their wiring design they often sent to send bad signals. The keyboard begins in a state that is ready to receive data, then once data is sent, the module checks 3 pieces of information. They keyboard sends a bit to say data is coming. It sends the key bits, then it will send an end bit. Checking for this sequence allows the keyboard module to ensure the data it is receiving is complete.
Design Description
The beginning of the top level module is pretty strait forward. This is just calling all of the inputs, outputs, wires, and registers that will be being used. The first chunk of actual code the user comes to is the keyboard implementation. The actual data module for the keyboard is at the bottom of the page. This should not be edited. The keyboard module is called forward and implemented in the top level module here. Then for the data to be correctly received and used I had to create a new rxdata1 register to use in the always block. The case statement below is showing the hexadecimal code for the key presses that will be used as inputs. Player twos was commented out because it was a priority to complete the project and I could not trouble shoot player twos inputs not saving properly to its register. The comments to the right of the case state first the color that they activate, followed by the keyboard key.
The next few commented sections initialize the vga and begin its counters for the vertical and horizontal sync. These can be edited to be longer or shorter if the user wishes to change the display size. Also, be aware that the vga runs on a 25MHz clock. The instantiate modules are for objects that are drawn below. The image display will display these when called upon. This is how the players change, the color boxes appear, as well as the checkmark for match.
Next few comments sections involve system response to a match, and detecting the user input and saving it in the player register. The game will check for a match on the positive edge of the state one key. I chose this because it allowed for easy control and debugging. The states were never implemented, as this remains only a two player game and has no ai system versus human. Then the user input is detected. Since I had shut off the user two input, so was their color storing register for the purpose of the demo. It had been working then for unknown reasons quit. But, this is also where the inputs of the user are stored in LED form showing the binary color input, and in the 7-segment displays so that trouble shooting is easy, and the registers can be viewed at all times. This is the end of the top level module.
Below is the VGA output module. This module is where the buttons were built and their “bright” responses to user input are created. This section was well commented for readers to help understand how it was built. Then the case statements below are how to draw with this VGA module. The pixels you turn on in the area that is assigned when “L” is on. This is the same for player numbers, level numbers, and the checkmark. Next shows the colors selected for the buttons, and the boundary box of the game area. The bright responses were tricky to play with, but you’ll see that it always needs to be on. This is because when a colour button is pressed it displays the bright.  If the colour button is not pressed it will display the faded button on screen.
The module to receive input from the PS2 keyboard involves a state machine that prepares, accepts, and checks the data inputs. Because keyboards can be very tricky, and due to their wiring design they often sent to send bad signals. The keyboard begins in a state that is ready to receive data, then once data is sent, the module checks 3 pieces of information. They keyboard sends a bit to say data is coming. It sends the key bits, then it will send an end bit. Checking for this sequence allows the keyboard module to ensure the data it is receiving is complete.
This project completed its task to be a Simon_Says game at a very basic level. What is essentially a matching game is the skeleton of something that could become much more complex. This document describes each module and how they work at a level that should make sense for a designer that understands Quartus II software and Verilog code. Many complicated paths were taken during this project, such as implementation of keyboards, visual displays, and the meshing of the two.

Video of Project
https://www.youtube.com/watch?v=b-0q34-UIvc


Aknoloedgments 
http://www.instructables.com/id/Make-your-own-Simon-Says-Game/
Quite a bit of code from this was used. There were edits to work in the keyboard, and the game structer was
refinished by Anthony Roberto
The VGA module came from here, as well as the VGA implimentation from the top level heirarchy.

#Andrew Quintana
//KeyBoard			
module keyboard (
  input clk,
  input PS2_DATA,
  input PS2_CLOCK,
  output reg [7:0] rxdata,
  output reg datafetched
); 

parameter idle    = 2'b01;
parameter receive = 2'b10;
parameter ready   = 2'b11;

reg [7:0] previousKey; 
reg [1:0]  state=idle;
reg [15:0] rxtimeout=16'b0000000000000000;
reg [10:0] rxregister=11'b11111111111;
reg [1:0]  datasr=2'b11;
reg [1:0]  clksr=2'b11;



reg rxactive;
reg dataready;
  
always @(posedge clk ) 
begin 
  rxtimeout<=rxtimeout+1;
  datasr <= {datasr[0],PS2_DATA};
  clksr  <= {clksr[0],PS2_CLOCK};


  if(clksr==2'b10)
    rxregister<= {datasr[1],rxregister[10:1]};


  case (state) 
    idle: 
    begin
      rxregister <=11'b11111111111;
      rxactive   <=0;
      dataready  <=0;
		datafetched <=0;
      rxtimeout  <=16'b0000000000000000;
      if(datasr[1]==0 && clksr[1]==1)
      begin
        state<=receive;
        rxactive<=1;
      end   
    end
    
    receive:
    begin
      if(rxtimeout==50000)
        state<=idle;
      else if(rxregister[0]==0)
      begin
        dataready<=1;
        rxdata<=rxregister[8:1];
        state<=ready;
        datafetched<=1;
      end
    end
    
    ready: 
    begin
      if(datafetched==1)
      begin
        state     <=idle;
        dataready <=0;
        rxactive  <=0;
		  datafetched <=0;
      end  
    end  
  endcase
end 
endmodule
