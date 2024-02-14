# GOR-1000k---CPU-Development
Designed a MIPS architecture CPU using Verilog that can handle arithmetic and logic functions.

Here is my code that was created in Verilog along with the boolean Maps that creates the equations, logic gate diagrams.


/* CS 354 - Digital Design
   4-bit CPU (behavioral implementation)
   
   Jahmaro Gordon 5/3/2023
*/
module cpu (Instruction, WriteData, CLK);
   input [8:0] Instruction;
   input CLK;
   output [3:0] WriteData;
   wire [8:0] IR;
   
   wire [3:0] A,B,Result;
   wire [2:0] ALUctl;
   instr_reg instr(Instruction,IR,CLK);
   control ctl (IR[8:6],Sel,ALUctl);
   quad2x1mux mux(IR[5:2],Result,Sel,WriteData);
   regfile regs(IR[5:4],IR[3:2],IR[1:0],WriteData,A,B,CLK);
   ALU alu (ALUctl, A, B, Result);
endmodule

module D_flip_flop(D,CLK,Q);
   input D,CLK; 
   output Q; 
   wire CLK1, Y;
   not  not1 (CLK1,CLK);
   D_latch D1(D,CLK, Y),
           D2(Y,CLK1,Q);
endmodule 

module D_latch(D,C,Q);
   input D,C; 
   output Q;
   wire x,y,D1,Q1; 
   nand nand1 (x,D, C), 
        nand2 (y,D1,C), 
        nand3 (Q,x,Q1),
        nand4 (Q1,y,Q); 
   not  not1  (D1,D);
endmodule 

module instr_reg (Instruction,IR,CLK);
   input [8:0] Instruction;
   input CLK;
// Replace the following with 9 instances of D flip-flops
//   output reg [8:0] IR;
//   always @(negedge CLK)
//      IR = Instruction;
output [8:0]IR;
//9 D flip-flops
D_flip_flop D0(Instruction[0],CLK,IR[0]);
D_flip_flop D1(Instruction[1],CLK,IR[1]);
D_flip_flop D2(Instruction[2],CLK,IR[2]);
D_flip_flop D3(Instruction[3],CLK,IR[3]);
D_flip_flop D4(Instruction[4],CLK,IR[4]);
D_flip_flop D5(Instruction[5],CLK,IR[5]);
D_flip_flop D6(Instruction[6],CLK,IR[6]);
D_flip_flop D7(Instruction[7],CLK,IR[7]);
D_flip_flop D8(Instruction[8],CLK,IR[8]);
endmodule

module control (OP,Sel,ALUctl);
  input [2:0] OP;
//Replace the following with gate-level code
//   output reg Sel;
//   output reg [2:0] ALUctl;
//   always @(OP) case (OP)
//     3'b000: {Sel,ALUctl} = 4'b1010; // ADD
//     3'b001: {Sel,ALUctl} = 4'b1110; // SUB
//     3'b010: {Sel,ALUctl} = 4'b1000; // AND
//     3'b011: {Sel,ALUctl} = 4'b1001; // OR
//     3'b100: {Sel,ALUctl} = 4'b1111; // SLT
//     3'b101: {Sel,ALUctl} = 4'b0000; // LI

//   endcase
//   endmodule
  
 output [2:0] ALUctl;
 output Sel;
 
 wire [2:0] OPI;  //OpInvert
 assign OPI = ~OP;
// SEL
// not (Nop0,OP[0]);     //op[0] '
// not (Nop1,OP[1]);     //op[1]'
// not (Nop2,OP[2]);     //op[2] '
or (Sel,OPI[2],OP[1],OPI[0]); //sel =  op[2] ' + op[1] + op[0] '

//ALUCTL [2]
and (alv0,OP[2],OPI[1],OPI[0]);  // ALUctl[2] =  OP[2]* OP[1]' * OP[0]
and (alv1,OPI[2],OPI[1]);
or (ALUctl[2],alv0,alv1); //  ALUctl[2] =  OP[2]* OP[1]' * OP[0] '  +  OP[2]' *  OP[1] '  


//ALUCTL [1]
and a1(Inv1,OPI[0],OPI[1]); //op[1]' & op[0]'    jdoodle.v:100: error: Unable to bind wire/reg/memory `Nop['sd1]' in `test_cpu.cpu1.ctl'
and a2(Inv2,OPI[2],OPI[1]);  // op[2]' & op[1]'

or (ALUctl[1],Inv1,Inv2); // op[1]' & op[0]' + op[2]' & op[1]'

// ALUCTL [0]
// and (a0, OP[2],OPI[0]);
// and (a1, OP[1],OPI[0]);
// or (ALUctl[0] ,a0,a1);  // op[1] + op[1] & op[0]

and (a0, OP[2],OPI[1],OPI[0]);
and (a1, OPI[2],OP[1],OP[0]);
or (ALUctl[0] ,a0,a1);  // op[1] + op[1] & op[0]
endmodule



module mux2x1(x,y,z,Out);

input x,y,z;
output Out;
wire i,j,k;
and (i,y,z);  //and gate with wires y,z coming in and turns into wire signal i
not (j,z);     //z'
and (k,j,x);  // xz'
or (Out,i,k);  // Out = yz + xz'
endmodule

module quad2x1mux (I0,I1,Sel,Out);
  input [3:0] I0,I1;
  input Sel;
  output [3:0] Out;
// Replace the following with gate-level code
 // assign Out = (Sel)? I1: I0;

 mux2x1 mu0(I0[0],I1[0],Sel,Out[0]);
 mux2x1 mu1(I0[1],I1[1],Sel,Out[1]);
 mux2x1 mu2(I0[2],I1[2],Sel,Out[2]);
 mux2x1 mu3(I0[3],I1[3],Sel,Out[3]);

endmodule

module regfile (ReadReg1,ReadReg2,WriteReg,WriteData,ReadData1,ReadData2,CLK);
  input [1:0] ReadReg1,ReadReg2,WriteReg;
  input [3:0] WriteData;
  input CLK;
  output [3:0] ReadData1,ReadData2;
  reg [3:0] Regs[0:3]; 
  assign ReadData1 = Regs[ReadReg1];
  assign ReadData2 = Regs[ReadReg2];
  initial Regs[0] = 0;
  always @(negedge CLK)
     Regs[WriteReg] = WriteData;
endmodule

module ALU (ALUctl, A, B, ALUOut);
  input [2:0] ALUctl;
  input [3:0] A,B;
  output reg [3:0] ALUOut;
  output Zero,Overflow;
  always @(ALUctl, A, B) //re-evaluate if these change
  case (ALUctl)
    3'b000: ALUOut <= A & B;
    3'b001: ALUOut <= A | B;
    3'b010: ALUOut <= A + B;
    3'b110: ALUOut <= A - B;
    3'b111: ALUOut <= A < B ? 1:0;
  endcase
endmodule



module test_cpu;
   reg [8:0] Instruction;
   reg CLK;
   wire [3:0] WriteData;
   cpu cpu1 (Instruction, WriteData, CLK);
   initial
   begin
     $display("\nCLK Instruction WriteData\n-------------------------"); 
     $monitor("%b   %b   %d (%b)", CLK,Instruction,WriteData,WriteData);
     #1 Instruction = 9'b101_0111_01; // li $1, 7 # $1 = 7  ,  $t1 = 7
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction
     #1 Instruction = 9'b101_0101_10; // li $2, 5 # $2 = 5 
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction
     #1 Instruction = 9'b001_01_10_11; // sub $3, $1, $2 # $3 = 2 , sub $t3, $t1, $t2
        CLK=1;  
     #1 CLK=0; // Negative edge - execute instruction
     
        //   Add more instructions
      #1 Instruction = 9'b101_1010_11; // li $t3 , 10 # $t3 = 10 (4b 1010)
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    
     
     #1 Instruction = 9'b011_10_11_10; // or $t2, $t2, $t3 # $t2 = 15 (4b 1111) 
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    
     #1 Instruction = 9'b010_10_11_11; //  and $t3, $t2, $t3   # $t3 = 10 (4 b1010)
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    
     #1 Instruction = 9'b100_11_10_10; // slt $t2, $t3, $t2   # $t2 = 1
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    
     #1 Instruction = 9'b000_11_10_10; // add $t2, $t3, $t2   # $t2 = 11 (4 b1011= -5)
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    
     #1 Instruction = 9'b000_01_10_11; // add $t3, $t1, $t2   # $t3 = 2 (7+(-5)=2)
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    
     #1 Instruction = 9'b100_10_11_01; // slt $t1, $t2, $t3      # $t1 = 0 (behavioral ALU) or 1 (gate-level ALU)
        CLK=1;
     #1 CLK=0; // Negative edge - execute instruction    

//   ...
   end
endmodule
