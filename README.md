# RISC-V-Reference-SoC-Tapeout-Program

<details>
<summary><b> Day 0 - Tools Installation</b></summary>

Understanding the flow of the tapeout program.  

The 4 major outputs i.e. the output of the "c"-code, the verilog code, the SoC output and the output of the tapeout chip should be the same. Basically the functionality is being checked at 4 major stages of the asic flow ensuring that the final product is in terms with the design application.  

## Yosys
```
$ sudo apt-get update
$ git clone https://github.com/YosysHQ/yosys.git
$ cd yosys
$ sudo apt install make (If make is not installed please install it)
$ sudo apt-get install build-essential clang bison flex \
 libreadline-dev gawk tcl-dev libffi-dev git \
 graphviz xdot pkg-config python3 libboost-system-dev \
 libboost-python-dev libboost-filesystem-dev zlib1g-dev
$ make config-gcc
$ make
$ sudo make install 
```
![Alt text](b.jpg)


## Iverilog
```
sudo apt-get update
sudo apt-get install iverilog 
```
![Alt text](c.jpg)


## GTKWave
```
sudo apt-get update
sudo apt install gtkwave 
```

![Alt text](d.jpg)
![Alt text](e.jpg)



</details>

<details>
<summary><b> Day 1 - Introduction to Verilog RTL Design and Synthesis</b></summary>

## Introduction to open-source simulator Iverilog

RTL design is simulated to check for its adherence wrt to the spec. To simulate we use Iverilog.

We use a testbench to instantiate the values for the Verilog code variables which is given as input to check for both the verilog code simulation as well as for the netlist.

Folder structure of the git clone:

- `lib` - contains sky130 standard cell library
- `my_lib/verilog_models` - contains all the standard cells verilog model
- `verilog_files` - contains the lab experiments source files

Command to run the design and testbench

```
iverilog good_mux.v tb_good_mux.v
```

Output of iverilog is vcd file which is given as input to gtkwave. A a.out file is created, executing which the iverilog dumps the vcd file.

## Introduction to GTKWave

gtkwave is used to display the waveforms, giving the vcd file as the input.

Command to view the vcd file in gtkwave

```
./a.out
gtkwave tb_good_mux.vcd
```
The image below shows the waveform generated.
![Alt text](1.a.jpg)


## Introduction to Yosys

Yosys is a synthesizer which converts the RTL code to gate-level netlist. The verilog code along with the lib file are the inputs given to it, which then generates the gate-level netlist as the output.

## Using Yosys Sky130 PDKs and verilog codes

The images below show the hierarchy of the commands used to generate the netlist. It starts with syntax checking and analysing the verilog code and mapping it to general gates. Then we map the boolean logic to standard cells from the .lib file.

Be in the verilog_codes directory and follow the below commands

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux.v
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![Alt text](1.b.jpg)
![Alt text](1.c.jpg)
![Alt text](1.e.jpg)
![Alt text](1.d.jpg)

The below image shows the generated netlist as the output of the synthesis procedure and to do that follow the below code.

```
write_verilog <module_name>
!vim <module_name>
```

![Alt text](1.g.jpg)

</details>





<details>
<summary><b> Day 2 - Timing libs, Hierarchical vs Flat Synthesis and Efficient Flop Coding Styles</b></summary>

## Introduction to timing .lib

Libraries are defined on the basis of PVT contraints (P-process, V-voltage, T-temperature).

The below image shows the PVT constraints:
- tt stands for typical in the .lib name
- 025C stands for temperature of 25 C in the .lib name
- 1v80 stands for voltage of 1.8V in the .lib name

```
!vim ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

![Alt text](2.a.jpg)

'-cell' marks the start of the cell. It consists of different characteristics of the cell as mentioned below:
- Area
- Power associated with pin
- Width
- Delay
- Input capacitance
- Transition

Same cell(same logic functionality) will have different types, having different characteristics in terms of area and other parameters.

## Hierarchical vs Flat Synthesis

### Hierarchical Synthesis

The image below shows the report of synthesising the multiple_modules.v. The code has both the sub-modules instantiated.

![Alt text](2.e.jpg)

We can see both the sub-modules- the And gate and the Or gate have been instantiated differently. Rather than seeing AND or OR gate, we see sub_modules when we run the command 'show' as shown in the screenshot. Basically, the hierarchy is preserved. This is an example of Hierarchical Synthesis.

![Alt text](2.b.jpg)

If we look into the sub_module2 in synthesized netlist 'multiple_modules_hier.v', we see that rather than OR gate, the inputs a & b, pass through the inverter and then NAND gate. It is because in CMOS, stacking PMOS, which happens in 'OR' gate is bad as PMOS has lower mobility than NMOS, which is stacked in NAND gate, and always have to be wider to get some meaningful output. One can also say that the charing and dishcharging is faster in a NANd gate compared to NOR or other gates. The next step is to check .lib file for the answer.


### Flat Synthesis

The design can be flattened by using the command `flatten`.

The image below shows the code along with the generated netlist and the logical diagram output. Here one can see that the submodules aren not instantiated. Rather the gates have been instantiated in the logical diagram along with the module names. This proves that flattening has broken down the hierarchy.

![Alt text](2.c.jpg)


### Sub-module Level Synthesis

RTL (Register Transfer Level) designs are often modular, with various functional blocks or sub-modules. Sub-module level synthesis allows each of these sub-modules to be synthesized independently.

Sub-module level synthesis is necessary for the following reasons:-
- Optimization and Area Reduction: By synthesizing sub-modules separately, the synthesis tool can optimize each one individually. It performs logic optimization, technology mapping, and area minimization for each sub-module. This leads to more efficient use of resources and reduced overall chip area.
- Reusability: When we have multiple instances of the same module, synthesizing one will save resources and time.
- Parallel Processing: To divide and conquer i.e. it is more efficient to synthesise each module concurrently when the design is massive. It helps reduce the TAT.

The commands to run sub-module synthesis:

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top sub_module1
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The image below shows the synthesis of the sub-module1. 

![Alt text](2.d.jpg)



## Various Flop Coding Styles and Optimization

### How to prevent glitches in the circuit? How do flip-flops help here?

Glitches can occur in digital circuits due to various reasons such as signal delays, noise, or timing issues. Flops prevent glitches during the operation in the following ways:

- Synchronization: Flops are edge-triggered devices, meaning they respond only to transitions of the input signal (e.g., rising edge, falling edge). This synchronization ensures that the output changes only at specific points, reducing the likelihood of glitches caused by transient signal variations.
- Timing Control: Flops are typically controlled by a clock signal, ensuring that all circuit operations occur synchronously. This eliminates timing issues that could lead to glitches due to data arriving at different times.


### Different types of FLops:

The type of flop changes on the basis of the set-reset signals and their usage.

The image below shows the codes of different type of flops.

![Alt text](2.f.jpg)

The image below shows DFF with asynchronous reset HDL simulation in Iverilog and waveform display in GTKwave. Irrespective of the clock and d, as soon as async_reset=1, q=0.

![Alt text](2.g.jpg)

The image below shows DFF with asynchronous set HDL simulation in Iverilog and waveform display in GTKwave. Irrespective of the clock and d, as soon as async_set=1, q=1.

![Alt text](2.h.jpg)

### Synthesis of Flops

Below are the commands to synthesize DFF with asynchronous reset.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![Alt text](2.i.jpg)

On synthesizing DFF with synchronous reset we get NOR gate with inverted d as shown in the image below. However, on evaluating the boolean expression, we reach the same logic realization. 
The flow of commands remains the same. Just have to change the name of the file accordingly.

![Alt text](2.k.jpg)

![Alt text](2.j.jpg)


## Synthesizing mult2 (multiply by 2)

To implement `y[3:0] = 2*a[2:0]`, we append a `1'b0` to the `a[2:0]` i.e, `y[3:0] = {a[2:0],0}`. This is also equal to left shift the input bits by 1. This can be realized by just wiring. So we expect no hardware which is also seen in the screenshot below, analysis after synthesis and show. The command 'abc' is not required for mapping when there are no cells.

![Alt text](2.l.jpg)

## Synthesizing mult9 (multiply by 9)

`y=9*a` can be considered `8*a+1*a` To implement `y[5:0] = 9*a[2:0]`, we append 000 to a[2:0] and then add a i.e, `y[5:0] = {a[2:0],000} + a[2:0]`. This can be realized just by wiring. So we expect no hardware which is also seen in the screenshot below, analysis after synthesis and show. The command 'abc' is not required for mapping when there are no cells.


![Alt text](2.m.jpg)

The multiply by 2 and multiply by 9 are special cases of synthesis, which post synthesis donot use any registers.
</details>






<details>
<summary><b> Day 3 - Combinational and Sequential Optimizations</b></summary>

## Introduction to Optimizations

### Combinational Logic Optimization

It means squeezing the logic to get the most optimized design in terms of area and power. the most commonly used techniques are:
- Constant propagation using direct optimization
- Boolean logic optimization using K-map(<5 variables) and Quine McKlusky(>5 variables)

The image below is an example of constant propogation.

![Alt text](3.0.jpg)

The image below is an example of boolean logic optimization.

![Alt text](3.1.jpg)


### Sequential Logic Optimization

The technqiues used are:

1) Basic
- Sequential constant propagation
2) Advanced 
- Static optimization
- Retiming
- Sequential logic cloning (floorplan aware synthesis)

An example of sequential constant propagation is of DFF with asynchronous reset where D input is grounded. Here one can just conclude `y = 1`. 

To note, the same technique cannot be applied to DFF with the asynchronous set because while `Q=1` when `Set=1`, but `Q=0` at `Set=0` at the next CLK pulse. Q is dependent not only on Set but also on the clock edge.

Retiming is a technique to improve the performance of the circuit. Here one can switch the logical implementation circuit between FFs to next/prior set of FFs in order to increase the performance of the circuit.


## Combinational Logic Optimizations

Command used for optimization:
```
opt_clean -purge
```

### Optimization of opt_check.v

Code
```
module opt_check (input a , input b , output y);
        assign y = a?b:0;
endmodule
```

For opt_check.v the assignment `y = a?b:0` reduces to `y = ab`. 

The logic implementation after synthesis for opt_check.v is shown below, showing only AND gate.

![Alt text](3.2.jpg)


### Optimization of opt_check2.v

Code
```
module opt_check2 (input a , input b , output y);
        assign y = a?1:b;
endmodule
```

For opt_check2.v the assignment `y = a?1:b` reduces to `y = a+b`. 

The logic implementation after synthesis for opt_check2.v is shown below, showing only OR gate.

![Alt text](3.3.jpg)


### Optimization of opt_check3.v

Code
```
module opt_check3 (input a , input b, input c , output y);
	       assign y = a?(c?b:0):0;
endmodule
```

For opt_check3.v the assignment `y = a?(c?b:0):0` reduces to `y = a+b`. 

The logic implementation after synthesis for opt_check3.v is shown below, showing 3 input AND gate.

![Alt text](3.4.jpg)


### Optimization of opt_check4.v

Code
```
module opt_check3 (input a , input b, input c , output y);
	       assign y = a?(b?c:(c?a:0)):(!c);
endmodule
```

For opt_check4.v the assignment `y = a?(b?c:(c?a:0)):(!c)` reduces to `y = a xnor b`. 

The logic implementation after synthesis for opt_check4.v is shown below, showing 3 input AND gate.

![Alt text](3.5.jpg)


### Optimization of multiple_module_opt.v

Code
```
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule

module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));

assign y = c | (b & n1); 
endmodule
```

For multiple_module_opt.v the boolean logic reduces to `y = c | (a & b)`. 

The logic implementation after synthesis for multiple_module_opt.v is shown below.

![Alt text](3.6.jpg)


### Optimization of multiple_module_opt2.v

Code
```
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;

sub_module1 U1 (.a(a) , .b(1'b0) , .y(n1));
sub_module1 U2 (.a(b), .b(c) , .y(n2));
sub_module1 U3 (.a(n2), .b(d) , .y(n3));
sub_module1 U4 (.a(n3), .b(n1) , .y(y));

endmodule
```

For multiple_module_opt.v the boolean logic reduces to `y = 1'b0`. 

The logic implementation after synthesis for multiple_module_opt.v is shown below.

![Alt text](3.7.jpg)


## Sequential Logic Optimizations

### Optimizing dff_const1.v

Code
```
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b0;
	else
		q <= 1'b1;
end

endmodule
```

For dff_const1.v, `q=0` as long as `reset=1`. However, when `reset=0` `q` doesn't immediately becomes `1` rather at the next rising edge of the clk as shown below. So the optimization cannot be applied. 

The image below shows the gtkwave output and the code to run the gtkwave is the same as before.

![Alt text](3.8.jpg)

Below are the commmands to run synthesis.

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const1.v
synth -top dff_const1
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The logic implementation after synthesis for dff_const1.v is shown below.

![Alt text](3.9.jpg)


### Optimizing dff_const2.v

Code
```
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end

endmodule
```

For dff_const2.v, `q=1` as long as `reset=1` and `q=1` even `if reset=0`. So the optimization is applied.

Below are the commmands to run synthesis.

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const2.v
synth -top dff_const2
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The logic implementation after synthesis for dff_const2.v is shown below.

![Alt text](3.10.jpg)



### Optimizing dff_const3.v

Code
```
module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```

For dff_const3.v, there are two flops. `q1=0` as long as `reset=1`. However, when `reset=0` `q1` doesn't immediately become `1`, rather at the next rising edge of the clk with some propagation delay as shown below. `q=1` as long as `reset=1`, acting as set rather than reset. However, when `reset=0`, `q` samples `q1` as `0` as there are some propagation delay for q1as shown below. At the next clk edge `q` samples `q1` as `1`. So the optimization cannot be applied.

The image below shows the gtkwave output and the code to run the gtkwave is the same as before.

![Alt text](3.11.jpg)

Below are the commmands to run synthesis.

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const3.v
synth -top dff_const3
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The logic implementation after synthesis for dff_const3.v is shown below.

![Alt text](3.12.jpg)



## Sequential Optimzations for Unused Outputs

### Optimization of Case1: 3-bit Up Counter with q[0] used (counter_opt.v)

Example of a counter where bits at the position of [2] and [1] are unused.

Code
```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```
The commands to run synthesis remain the same as done for the DFF modules.

We see only one flop after the synthesis and is also seen in synthesis report after `synth -top counter_opt.v`.

![Alt text](3.13.jpg)


### Optimization of Case2: 3-bit Up Counter (counter_opt2.v)

Example of a counter where all bits are used.

Code
```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = (count[2:0] == 3'b100);

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```
The commands to run synthesis remain the same as done for the DFF modules.

We see three flop after the synthesis and is also seen in synthesis report after `synth -top counter_opt.v`.

![Alt text](3.14.jpg)

![Alt text](3.15.jpg)

</details>







<details>
<summary><b> Day 4 - GLS, Blocking vs Non-blocking and Synthesis-Simulation Mismatch</b></summary>

## GLS, Synthesis-Simulation Mismatch, and Blocking/Non-blocking Statements

### Why is Gate Level Simulation (GLS) necessary?

- Verify the correctness of the design after synthesis
- Ensure the timing of the design is met which is done with delay annotation (timing aware)

So, essentially we are simulating the verilog file and the netlist file to ensure that the functionality is preserved. GTKwave is used to simulate the waveforms for both.


### Synthesis Simulation Mismatches
It happens because of the following reasons:
- Missing sensitivity list
- Blocking vs non-blocking assignments
- Non-standard verilog coding


(1) Missing sensitivity list

Consider 2 cases where one is trying to implement a mux. The inputs are `i0` and `i1`. In case one the sensitivity list contains `sel`, whereas the other contains `*`. For case-1 always block is evaluated only when `sel` is changing. So output `y` is not evaluated when `sel` is not changing although `i0` and `i1` are changing. Rather it acts like a latch as the output is latched onto the input `sel` changes. The case 2 represents the correct design coding for mux as its sensitive to sel and both inputs. In this case always is evaluated for any signal changes.


(2) Blocking vs Non-blocking Assignments

Blocking Statements
- Represented by `=`.
- Executes the statements in the order it is written inside always block.
- So the first statement is evaluated before the second statement.

Non-Blocking Statements
- Represented by `<=`.
- Executes all the RHS when always block is entered and assigns to LHS.
- Parallel execution.


Ex1:

The left side of the code below gives us the correct execution. While the right side can lead to serious issues as `d` is assigned to `q` directly. So choosing non-blocking statements is best practice.

```
module code_blocking (input clk, input reset,	 module code_blocking (input clk, input reset,	
                      input d,										  input d,					
                      output reg q);							      output reg q);			
  reg q0;											reg q0;										
  always @(posedge clk, posedge reset) begin		always @(posedge clk, posedge reset) begin	
    if (reset) begin								  if (reset) begin							
      q0 = 1'b0;										 q0 = 1'b0;								
      q  = 1'b0;										 q  = 1'b0;								
    end												  end											
    else begin										  else begin									
      q  = q0;   										 q0 = d;
      q0 = d;											 q  = q0;									
    end											      end
  end												 end
endmodule										 endmodule
```


Ex2:

Blocking Statements Leading to Synthesis Simulation Mismatch.

In the code shown below, `y` gets the old `q0` value. This will mimic delay or flop. But when you synthesize, there will be no flop. If the order is changed (right side code), latest value of `q0` is assigned to `y`.

When synthesized, both will lead to the same circuit. However, simulation will result in different behavior. For the left side of the code, `y` gets the old `q0` value and for the right side of the code, `y` gets the latest `q0` value leading to a synthesis simulation mismatch.

This issue is resolved by using non-blocking statements.

```
module code (input a, b, c,		 	module code (input a, b, c,					
             output reg y);						 output reg y);

  reg q0;							  reg q0;

  always @(*) begin					  always @(*) begin
    y  = q0 & c;   						q0 = a | b;
    q0 = a | b;   						y  = q0 & c;
  end								  end

endmodule						    endmodule

```


## Labs on GLS and Synthesis-Simulation Mismatch

### Ternary operator MUX (ternary_operator_mux.v)

Code
```
module ternary_operator_mux (input i0 , input i1 , input sel , output y);
	assign y = sel?i1:i0;
endmodule
```

Command to run the simulation using gtkwave remains the same, just change the verilog file names.

HDL Simulation waveform of ternary_operator_mux.v is shown in the image below.

![Alt text](4.4.jpg)

The commands to run the synthesis for ternary_operator_mux.v

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog ternary_operator_mux_net.v
```

![Alt text](4.5.jpg)

The commands to do GLS for ternary_operator_mux.v

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

The GLS output is shown below.

![Alt text](4.6.jpg)


### Bad MUX (bad_mux.v)

The `always` block is executed only at `sel` signal. It works like a flop rather than mux. The Verilog code of bad_mux.v

Code
```
module bad_mux (input i0 , input i1 , input sel , output reg y);
always @ (sel)
begin
	if(sel)
		y <= i1;
	else 
		y <= i0;
end
endmodule
```

Command to run the simulation using gtkwave remains the same, just change the verilog file names.

HDL Simulation waveform of bad_mux.v is shown in the image below.

![Alt text](4.7.jpg)

The commands to run the synthesis for bad_mux.v.

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog bad_mux_net.v
```

The synthesis report shows it is still inferring the mux but not the flop.

![Alt text](4.8.jpg)

The commands to do GLS for bad_mux.v

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_mux_net.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

The GLS output is shown below. This shows correct functionality which is different from HDL simulation, leading to synthesis simulation mismatch.

![Alt text](4.9.jpg)




## Labs on Synthesis-Simulation Mismatch for Blocking Statements

### Blocking Caveat (blocking_caveat.v)

Code
```
module blocking_caveat (input a , input b , input  c, output reg d); 
reg x;
always @ (*)
begin
	d = x & c;
	x = a | b;
end
endmodule
```

Command to run the simulation using gtkwave remains the same, just change the verilog file names.

HDL Simulation waveform of blocking_caveat.v is shown in the screenshot below. `d` takes the old value of `x` causing incorrect functionality.

![Alt text](4.1.jpg)

Below are the commands to run the synthesis for blocking_caveat.v.

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog blocking_caveat_net.v
```

The synthesis report and logic synthesis is shown below.

![Alt text](4.2.jpg)

The commands to do GLS for blocking_caveat.v

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v blocking_caveat_net.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

The GLS output is shown below. In this case, `d` takes the current value of `x` causing incorrect functionality.The waveform shows correct functionality which is different from HDL simulation, leading to synthesis simulation mismatch.

![Alt text](4.3.jpg)

</details>








<details>
<summary><b> Day 5 - Optimization in Synthesis</b></summary>

## IF-ELSE statements

If all the cases have been mentioned using a `if-else` statement, it generates a priority encoder or a combination of muxes. It is used to create priority logic.

Code
```
if <cond 1>
	c1
else if <cond 2>
	c2
else
	c3
```

The image below shows the hardware that will be generated.

![Alt text](5.1.jpg)


### Caveat in IF-ELSE statement

If one has just used `if`, `else-if` in the constrait without else statement, latch will be inferred.

Code
```
if <cond 1>
	c1
else if <cond 2>
	c2
```

The image below shows the hardware that will be inferred.

![Alt text](5.2.jpg)



There might be exceptions to this like in case of designing a counter. If you dont mention the else statement, there will be a latch inferred, but would be logically correct as in that case when `en = 0`, the present value of `cout` will be latched/stored.

Code 
```
reg [2:0] count;
always @(posedge clk, posedge reset) begin
	if (reset)
		count <= 3'b000;
	else if (en)
		count <= count + 1;
end
```

So, essentially one must think of the hardware implementation before using the if-else statements and ensure that latches arent inferred unless necessary.


## CASE statements

If all cases have been covered in the case statement, then it results in a mux with number of cases as inputs and log2(#cases) as the select reg bit-size.

Code
```
case (sel) begin  #(sel is 2 bit reg)
2'b00 : x = a;
2'b01 : x = b;
2'b10 : x = c;
default : x = d;
end
```

The image below shows the hardware that will be generated.

![Alt text](5.3.jpg)


### Caveats in CASE statement

1) If no default is used when all cases are not mentioned, latch is inferred.

Code
```
case (sel) begin  #(sel is 2 bit reg)
2'b00 : x = a;
2'b01 : x = b;
2'b10 : x = c;  #(Here case 4 is not mentioned, hence latch will be inferred)
end
```

2) Partial assignment in case even after mentioning defaut case, also results in latch being inferred.

Code
```
case (sel) begin  #(sel is 2 bit reg)  #(Here 2 muxes will be inferred for reg x and y.)
2'b00 : x = a
		y = b;
2'b01 : x = b;  #(Here y is not assigned any value, hence latch will be inferred)
default: x = c
	     y = b;
end
```

3) Should not have overlapping case statements.

Code
```
case (sel) begin  #(sel is 2 bit reg)
2'b00 : x = a;
2'b01 : x = b;
2'b10 : x = c;
2'b1? : x = d;  #(Here case 10 and 1? both will get executed one after another and value will always be d at the end if sel = 10)
```


</details>
