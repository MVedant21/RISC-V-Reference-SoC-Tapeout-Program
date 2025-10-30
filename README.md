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

# WEEK 1
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


## Labs on Incomplete If

### Incomplete If-1 (incomp_if.v)

Code
```
module incomp_if (input i0, input i1, input i2, output reg y);
  always @ (*)
  begin
    if (i0)
      y <= i1;
  end
endmodule
```

The image below shows the simulation.

![Alt text](5.4.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Latch is inferred, as else condition is not mentioned. The logic is independent of `i2`. The previous value of `i1`, before `i0 = 0` is latched to `y`.

The image below shows the output of synthesis.

![Alt text](5.5.jpg)


### Incomplete If-2 (incomp_if2.v)

Code
```
module incomp_if2 (input i0, input i1, input i2, input i3, output reg y);
  always @ (*)
  begin
    if (i0)
      y <= i1;
    else if (i2)
      y <= i3;
  end
endmodule
```

The image below shows the simulation.

![Alt text](5.6.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_if2.v
synth -top incomp_if2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Latch is inferred, as else condition is not mentioned. The latch is enabled using `i0||i2`. NOR gate is used instead of OR for better optimization. The input to the latch is some combinational logic including `i0`,`i1`,`i3` with the help of a  mux.

The image below shows the output of synthesis.

![Alt text](5.7.jpg)



## Labs on Incomplete overlapping Case

### Incomplete case (incomp_case.v)

Code
```
module incomp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
  
  always @ (*) begin
    case (sel)
      2'b00: y = i0;
      2'b01: y = i1;
    endcase
  end

endmodule
```

The image below shows the simulation.

![Alt text](5.8.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Latch is inferred for both the cases 2 and 3. The enable of the latch is `sel[1]` as its common for both cases. The input to the latch is the combinational logic between `i0` and `i1`. `i2` is not required at all. Mux is used to choose between `i0` and `i1`. 

The image below shows the synthesis output.

![Alt text](5.9.jpg)


### Complete case (comp_case.v)

Code
```
module incomp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
  
  always @ (*) begin
    case (sel)
      2'b00: y = i0;
      2'b01: y = i1;
    endcase
  end

endmodule
```

The image below shows the simulation.

![Alt text](5.10.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog comp_case.v
synth -top comp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Latch is not inferred here as all cases are considered.
The image below shows the synthesis output.

![Alt text](5.11.jpg)



### Partial case (partial_case_assign.v)

Code
```
module partial_case_assign (input i0, input i1, input i2, input [1:0] sel, output reg y, output reg x);

  always @ (*) begin
    case (sel)
      2'b00: begin
        y = i0;
        x = i2;
      end
      2'b01: y = i1;
      default: begin
        x = i1;
        y = i2;
      end
    endcase
  end

endmodule

```

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog partial_case_assign.v
synth -top partial_case_assign
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Latch is inferred here as one case does not assign value to `y`.  Mux is used to choose between i0 and i1 where `sel = sel1 + sel0'`
The image below shows the synthesis output.

![Alt text](5.12.jpg)


### Bad case (bad_case.v)

Code
```
module bad_case (input i0, input i1, input i2, input i3, input [1:0] sel, output reg y);

always @(*)
begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;
    endcase
end

endmodule
```

For the case `2'b1?`, the tool gets confused and latches the output to `1'b1` until this case if finished. This leads to ambiguity.
The image below shows the simulation.

![Alt text](5.13.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog bad_case.v
synth -top bad_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog bad_case_net.v
show
```

The image below shows the synthesis output. No latches will be inferred as all cases are covered, although overlap of cases is there.

![Alt text](5.14.jpg)

Command to run simulation for GLS.

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v bad_case_net.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```

For the case `2'b1?`, the output follows `i3`. Conclusion is that the cases should be mutually exclusive to avoid ambuguity in the post synthesis and pre synthesis simulation.
Image below shows post synthesis simulation.

![Alt text](5.15.jpg)


## For loop and For-Generate

### For loop 

Code
```
always @(*) begin
	MUX_OUT = 1'b0; 
	integer i; 
	for (i = 0; i < 32; i = i + 1) begin
		if (SELECT == i) begin
			MUX_OUT = DATA_IN[i];
		end
	end
end
```

Used inside `always` block for multiple evaluations. Like in the above code its used to produce a `32:1 mux` using blocking statements to ensure appropriate flow. Similarly, it can be used for other designs like demux, etc.


### For-Generate loop

Code
```
generate
	genvar i; 
	for (i = 0; i < N; i = i + 1) begin : bitwise_and_instance
		
		single_AND u_and (.a (A[i]), .b (B[i]), .y (Y_OUT[i]));
	end
endgenerate
```

Always used outside the `always` block. Its used to instantiate or replicate hardware. As in the above example its instantiating `N and` gates with inputs from `bus A and B`.
Same can be used for other purposed like Ripple Carry Adder, etc.



## Labs on For Loop

### Mux generation (mux_generate.v)

Code
```
module mux_generate (input i0, input i1, input i2, input i3, input [1:0] sel, output reg y);
    wire [3:0] i_int;
    assign i_int = {i3, i2, i1, i0}; 
    integer k; 
    
    always @ (*) begin
        y = 1'b0; 
        for (k = 0; k < 4; k = k + 1) begin
			if (k == sel) 
				y = i_int[k];
        end
    end
endmodule
```

The image below shows the simulation.

![Alt text](5.16.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mux_generate.v
synth -top mux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog mux_generate_net.v
show
```

As expected we get the `4:1 mux` generated along with a latch to store the output of the mux in the variable `y`.
The image below shows the synthesis output.

![Alt text](5.17.jpg)

Command to run simulation for GLS.

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v mux_generate_net.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd
```

The image below shows the post synthesis simulation. The simulation matches the pre-synthesis simulation.

![Alt text](5.18.jpg)


### Demux generation 1(demux_case.v)

Code
```
module demux_case (output o0, output o1, output o2, output o3, output o4, output o5, output o6, output o7, input [2:0] sel, input i);
    reg [7:0] y_int; 
    assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int; 
    
    always @ (*) begin
        y_int = 8'b00000000; 
        case (sel)
            3'b000 : y_int[0] = i; 
            3'b001 : y_int[1] = i;
            3'b010 : y_int[2] = i;
            3'b011 : y_int[3] = i;
            3'b100 : y_int[4] = i;
            3'b101 : y_int[5] = i;
            3'b110 : y_int[6] = i;
            3'b111 : y_int[7] = i;
        endcase
    end
    
endmodule
```

The image below shows the simulation.

![Alt text](5.19.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog demux_case.v
synth -top demux_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog demux_case_net.v
show
```

As expected we get the `1:8` generated along with a latch to store the output of the mux in the variable `y`.
The image below shows the synthesis output.

![Alt text](5.20.jpg)

Command to run simulation for GLS.

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v demux_case_net.v tb_demux_case.v
./a.out
gtkwave tb_demux_case.vcd
```

The image below shows the post synthesis simulation. The simulation matches the pre-synthesis simulation.

![Alt text](5.21.jpg)



### Demux generation 2(demux_generate.v)

Code
```
module demux_generate (output o0, output o1, output o2, output o3, output o4, output o5, output o6, output o7, input [2:0] sel, input i);
    reg [7:0] y_int; 
    assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int; 
    integer k; 
    
    always @ (*) begin
        y_int = 8'b0; 
        for (k = 0; k < 8; k = k + 1) begin
            if (k == sel) 
                y_int[k] = i;
        end
    end
    
endmodule
```

The image below shows the simulation.

![Alt text](5.22.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog demux_generate.v
synth -top demux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog demux_generate_net.v
show
```

As expected we get the `1:8` generated along with a latch to store the output of the mux in the variable `y`.
The image below shows the synthesis output.

![Alt text](5.23.jpg)

Command to run simulation for GLS.

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v demux_generate_net.v tb_demux_generate.v
./a.out
gtkwave tb_demux_generate.vcd
```

The image below shows the post synthesis simulation. The simulation matches the pre-synthesis simulation.

![Alt text](5.24.jpg)

The output of both demux scenarios match with each other, proving that using for-loop is an easier method of coding the same logic for greate N.


## Labs on For Loop

### Ripple Carry Adder(rca.v)

Code
```
module fa (input a, input b, input c, output co, output sum);
    assign {co, sum} = a + b + c; 
endmodule


module rca (input [7:0] num1, input [7:0] num2, output [8:0] sum);
    wire [7:0] int_sum;
    wire [7:0] int_co; 
    genvar i; 

    fa u_fa_0 (.a (num1[0]),.b (num2[0]),.c (1'b0),.co (int_co[0]),.sum (int_sum[0]));

    generate
        for (i = 1; i < 8; i = i + 1) begin : fa_u_fa_i 
            fa u_fa_1 (.a (num1[i]),.b (num2[i]),.c (int_co[i-1]),.co (int_co[i]),.sum (int_sum[i]));
        end
    endgenerate

    assign sum[7:0] = int_sum;
    assign sum[8] = int_co[7]; 

endmodule
```

Command to run simulation.
```
iverilog fa.v rca.v rca_tb.v
./a.out
gtkwave rca_tb.vcd
```

The image below shows the simulation.

![Alt text](5.25.jpg)

Commands to run synthesis.
```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog fa.v rca.v
synth -top rca
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog rca_net.v
show rca
```

The image below shows the synthesis output.

![Alt text](5.26.jpg)

Command to run simulation for GLS.

```
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v rca_net.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

The image below shows the post synthesis simulation. The simulation matches the pre-synthesis simulation.

![Alt text](5.27.jpg)

</details>



# WEEK 2

<details>

<summary><b> BabySoC Fundamentals & Functional Modelling </b></summary>

## Understanding System-on-Chip (SoC)

A System-on-Chip (SoC) is an integrated circuit that integrates almost all components of a computer or electronic system into a single chip. It functions as a complete system, contrasting with traditional designs that use separate chips for the Central Processing Unit (CPU), memory, and peripherals. SoCs are the foundation of modern, compact, and power-efficient electronics, such as smartphones, smartwatches, and IoT devices. They are valued for their space savings (compactness), energy efficiency (due to reduced distance for data transfer), and high performance.

### Components of a Typical SoC

A typical SoC is a complex integration of several functional blocks, connected via an internal communication fabric:

- CPU (Central Processing Unit): The brain of the SoC. It executes software instructions, performs calculations, and manages data processing. Modern SoCs often incorporate multiple CPU cores.
- Memory Subsystem: Includes various types of memory: RAM (Random Access Memory) for volatile, fast data storage during operation, and ROM/Flash for non-volatile storage (operating system, firmware), when the system is OFF.
- Peripherals/I/O Ports: Specialized hardware blocks that interface the SoC with the external world and other internal functions. Examples include: GPU (Graphics Processing Unit), DSP (Digital Signal Processor), Communication Modules (Wi-Fi, Bluetooth), Input/Output interfaces (USB, I2C), and custom blocks like the PLL (Phase-Locked Loop) and DAC (Digital-to-Analog Converter) in BabySoC.
- Interconnect Fabric: An on-chip network (often a bus or a Network-on-Chip, NoC) that provides the communication paths for the CPU, memory, and all peripherals to exchange data and control signals efficiently.


## BabySoC: A Simplified Model for Learning SoC

BabySoC (VSDBabySoC) is a simple, open-source teaching chip designed to make learning about complex Systems-on-Chip (SoCs) easier. It uses the RVMYTH RISC-V processor but leaves out the confusing parts of commercial chips, keeping only the basic, essential concepts.

### Why BabySoC is a Simplified Model

- Focused Components: It integrates a small, manageable set of essential components: the RVMYTH CPU, a Phase-Locked Loop (PLL) for precise clock generation, and a 10-bit Digital-to-Analog Converter (DAC) for analog interfacing. This limited scope allows us to focus on the interaction between a processor, timing mechanism, and an analog IP without getting overwhelmed around complex designs.
- Open-Source and Documented: Its foundation on the open-source RISC-V architecture and its highly documented design make its internals transparent, ideal for deep study and experimentation.
- Core Interface Demonstration: BabySoC clearly demonstrates a crucial design element: digital-to-analog interfacing. The system uses the RVMYTH CPU to process digital values, which are then fed to the DAC for conversion into an analog output (e.g., for audio/video), providing a tangible example of a mixed-signal design.
- Technology Exposure: The project is based on the Sky130 technology, giving us hands-on exposure to a real, open-source industrial fabrication process.


## Role of Functional Modeling in SoC Design Flow

Functional modeling is the critical first step in the SoC design flow, occurring before the Register-Transfer Level (RTL) and Physical Design stages.

### Functional Modeling

Focuses on Behavior (What it does). The output is a calidated C/C++ or high-level HDL model. It's crucial because it eliminates architectural errors early, before committing to hardware structure.

### RTL Design (Register-Transfer Level)

Focuses on Structure (How it's built). The output is detailed Verilog/VHDL code. It's crucial because it translates the function into a gate-level ready hardware description.

### Physical Design

Focuses on Layout (Where it goes). The output is the GDSII file (for fabrication). It's crucial because it deals with timing, power, and physical constraints of the actual chip.


In conclusion, the BabySoC platform is a great teaching tool because it connects theory to practice. It's a simple, open-source system that covers all the main SoC concepts, like how the CPU works, how to manage the clock, and how to convert digital signals to analog. This makes it an ideal way to see how functional modeling is crucial for building a successful final chip.

</details>






<details>

<summary><b>  Labs (Hands-on Functional Modelling) </b></summary>

## VSDBabySoC Modeling

This is the top-level module that integrates the rvmyth, pll, and dac modules.

```
  - Inputs:
     - reset: Resets the core processor.
     - VCO_IN, ENb_CP, ENb_VCO, REF: PLL control signals.
     - VREFH: DAC reference voltage.
  - Outputs:
     - OUT: Analog output from DAC.
     - Connections:
     - RV_TO_DAC - A 10-bit bus that connects the RISC-V core output to the DAC input.
     - CLK - The clock signal generated by the PLL.
```

### Installation

First we need to install some important packages,
```
$ sudo apt install make python python3 python3-pip git iverilog gtkwave docker.io
$ sudo chmod 666 /var/run/docker.sock
$ cd ~
$ pip3 install pyyaml click sandpiper-saas
```

Now we can clone this repository in an arbitrary directory.(Make sure that the sandpiper-saas and docker.io files are in the same path as the github repo.).
```
git clone https://github.com/manili/VSDBabySoC.git
```


### Pre-Synthesis Simulation

Run the following command to perform a pre-synthesis simulation.
```
iverilog -o output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM \
    -I src/include -I src/module \
    src/module/testbench.v src/module/vsdbabysoc.v
cd output/pre_synth_sim
./pre_synth_sim.out

(OR)

cd VSDBabySoC
make pre_synth_sim
```

The result of the simulation (i.e.`pre_synth_sim.vcd`) will be stored in the `output/pre_synth_sim` directory.

Command to see the simulation using GTKwave.
```
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```

Two most important signals are `CLK` and `OUT`. The `CLK` signal is provided by the `PLL` and the `OUT` is the output of the `DAC model`. The image below shows the final result of the modeling process.

![Alt text](w2.4.jpg)

In this picture we can see the following signals:

**CLK**: This is the input `CLK` signal of the `RVMYTH` core. This signal comes from the `PLL`, originally.
**reset**: This is the `input reset` signal of the `RVMYTH` core. This signal comes from an external source, originally.
**OUT**: This is the output `OUT` signal of the `VSDBabySoC` module. This signal comes from the `DAC` (due to simulation restrictions it behaves like a digital signal which is incorrect), originally.
**RV_TO_DAC[9:0]**: This is the `10-bit output [9:0] OUT` port of the `RVMYTH` core. This port comes from the `RVMYTH register #17`, originally.
**OUT**: This is a real datatype wire which can simulate analog values. It is the output wire real `OUT` signal of the `DAC` module. This signal comes from the DAC, originally.

**PLEASE NOTE** that the sythesis process does not support real variables, so we must use the simple wire datatype for the `\vsdbabysoc.OUT` instead. The iverilog simulator always behaves wire as a digital signal. As a result we can not see the analog output via `\vsdbabysoc.OUT` port and we need to use `\dac.OUT` (which is a real datatype) instead.

The image below shows the command line output.

![Alt text](w2.3.jpg)


### Synthesis

To perform the synthesis process do the following.
```
cd ~/VSDBabySoC
make synth
```

We get the result in the `output/synth/vsdbabysoc.synth.v` file.


### Post-Synthesis Simulation(GLS)

Run the following command to perform a pre-synthesis simulation.
```
iverilog -o output/post_synth_sim/post_synth_sim.out -DPRE_SYNTH_SIM \
    -I src/include -I src/module \
    src/module/testbench.v src/module/vsdbabysoc.v
cd output/post_synth_sim
./post_synth_sim.out

(OR)

cd VSDBabySoC
make post_synth_sim
```

The result of the simulation (i.e.`post_synth_sim.vcd`) will be stored in the `output/post_synth_sim` directory.

Command to see the simulation using GTKwave.
```
gtkwave output/post_synth_sim/post_synth_sim.vcd
```

Two most important signals are `CLK` and `OUT`. The `CLK` signal is provided by the `PLL` and the `OUT` is the output of the `DAC model`. The image below shows the final result of the modeling process.

![Alt text](w2.2.jpg)

In this picture we can see the following signals:

- **\core.CLK**: This is the input `CLK` signal of the `RVMYTH` core. This signal comes from the `PLL`, originally.
- **reset**: This is the `input reset` signal of the `RVMYTH` core. This signal comes from an external source, originally.
- **OUT**: This is the output `OUT` signal of the `VSDBabySoC` module. This signal comes from the `DAC` (due to simulation restrictions it behaves like a digital signal which is incorrect), originally.
- **\core.OUT[9:0]**: This is the `10-bit output [9:0] OUT` port of the `RVMYTH` core. This port comes from the `RVMYTH register #17`, originally.
- **OUT**: This is a real datatype wire which can simulate analog values. It is the output wire real `OUT` signal of the `DAC` module. This signal comes from the DAC, originally.

**PLEASE NOTE** that the sythesis process does not support real variables, so we must use the simple wire datatype for the `\vsdbabysoc.OUT` instead. The iverilog simulator always behaves wire as a digital signal. As a result we can not see the analog output via `\vsdbabysoc.OUT` port and we need to use `\dac.OUT` (which is a real datatype) instead.

The image below shows the Synthesis output.

![Alt text](w2.1.jpg)




## Rvmyth Modeling (RISC-V Core)

The rvmyth module is a simple RISC-V based processor. It outputs a 10-bit digital signal (OUT) to be converted by the DAC.
```
  Inputs:
     - CLK: Clock signal generated by the PLL.
     - reset: Initializes or resets the processor.
  Outputs:
     - OUT: A 10-bit digital signal representing processed data to be sent to the DAC.
```

### Installation

Clone this repository in an arbitrary directory.
```
git clone https://github.com/manili/VSDBabySoC.git
```

### Pre-Synthesis Simulation

Run the following commands to perform a pre-synthesis simulation.
```
cd rvmyth
iverilog mythcore_test.v tb_mythcore_test.v
./a.out
gtkwave tb_mythcore_test.vcd
```

The image below shows the simulation output.

![Alt text](w2.5.jpg)

We can see from the simulation that the 10-bit digital processed data is coming as output (`out`) in synchronization with the `clock` signal.





## AVSDPLL Modeling (PLL Module)

The dac module converts the 10-bit digital signal from the rvmyth core to an analog output.
```
  Inputs:
     - VCO_IN, ENb_CP, ENb_VCO, REF: Control and reference signals for PLL operation.
  Output:
     - CLK: A stable clock signal for synchronizing the core and other modules.
```




## AVSDDAC Modeling (DAC Module)

The dac module converts the 10-bit digital signal from the rvmyth core to an analog output.
```
  Inputs:
     - D: A 10-bit digital input from the processor.
     - VREFH: Reference voltage for the DAC.
  Output:
     - OUT: Analog output signal.
```

### Installation

Clone this repository in an arbitrary directory.
```
git clone https://github.com/vsdip/rvmyth_avsddac_interface.git
```

### Pre-Synthesis Simulation

Run the following commands to perform a pre-synthesis simulation.
```
cd rvmyth_avsddac_interface/iverilog/Pre-synthesis
iverilog avsddac.v avsddac_tb_test.v
./a.out
gtkwave avsddac_tb_test.vcd
```

The image below shows the simulation output.

![Alt text](w2.6.jpg)

In this picture we can see the following signals:

- **D[9:0]**: DAC 10-bit digital input.
- **EN**: Enable is high for the block to be active.
- **OUT**: Corresponding analog values to the input D, as the output.
- **VREFH**: Reference voltage high = 3.7V .
- **VREL**: Reference voltage low = 0V .

Now integrate both rvymth and DAC using a Top level module and test it to verify the correctness of the integration.

Run the following commands to perform a pre-synthesis simulation.
```
iverilog rvmyth_avsddac.v rvmyth_avsddac_TB.v
./a.out
gtkwave rvmyth_avsddac.vcd
```

The image below shows the simulation output.

![Alt text](w2.7.jpg)

In this picture we can see the following signals:

- **out[9:0]**: rvymth 10-bit digital output.
- **D[9:0]**: DAC 10-bit digital input.
- **Out**: DAC analog output.


</details>







# WEEK 3

<details>

<summary><b> Post-Synthesis GLS & STA Fundamentals </b></summary>

## Post-Synthesis GLS

### Key-aspects of GLS for BabySoC

1) Verification with Timing Information:
- GLS is performed using Standard Delay Format (SDF) files to ensure timing correctness.
- This checks if the SoC behaves as expected under real-world timing constraints.
  
2) Design Validation Post-Synthesis:
- Confirms that the design's logical behavior remains correct after mapping it to the gate-level representation.
- Ensures that the design is free from issues like metastability or glitches.

3) Simulation Tools:
- Icarus Verilog(iVerilog) is used for compiling and running the gate-level netlist.
- Yosys is used to generate the netlist from the given RTL codes and libraries.
- Waveforms are analyzed using GTKWave.


Below mentioned are the generalized steps for running the GLS from start.

Load the Top-Level Design and Supporting Modules.
```
yosys
read_verilog /home/vedant/VSDBabySoC/src/module/vsdbabysoc.v
read_verilog -I /home/vedant/VSDBabySoC/src/include /home/vedant/VSDBabySoC/src/module/rvmyth.v
read_verilog -I /home/vedant/VSDBabySoC/src/include /home/vedant/VSDBabySoC/src/module/clk_gate.v
```

Load the Liberty Files for Synthesis.
```
read_liberty -lib /home/vedant/VSDBabySoC/src/lib/avsdpll.lib
read_liberty -lib /home/vedant/VSDBabySoC/src/lib/avsddac.lib
read_liberty -lib /home/vedant/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Run Synthesis Targeting vsdbabysoc.
```
synth -top vsdbabysoc
```

Map D Flip-Flops to Standard Cells.
```
dfflibmap -liberty /home/vedant/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Perform Optimization and Technology Mapping.
```
opt
abc -liberty /home/vedant/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```

Perform Final Clean-Up and Renaming.
```
flatten
setundef -zero
clean -purge
rename -enumerate
```

Check stat and write the Synthesized Netlist.
```
stat
write_verilog -noattr /home/vedant/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v
```

Run the following iverilog command to compile the testbench.
```
iverilog -o /home/vedant/VSDBabySoC/output/post_synth_sim/post_synth_sim.out -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 -I /home/vedant/VSDBabySoC/src/include -I /home/vedant/VSDBabySoC/src/module /home/vedant/VSDBabySoC/src/module/testbench.v
```

Run the Simulation and view using GTKWave.
```
cd output/post_synth_sim/
./post_synth_sim.out
gtkwave post_synth_sim.vcd
```



As I could not find the `rvymth.v` code in the repositories shared in all the pdfs till now, I used the make file to run the GLS and generate the simulation for the gtkwave Post-synthesis.

The commands used to do this are given below along with pre-synthesis and post-synthesis outputs.

Clone the repository
```
git clone https://github.com/manili/VSDBabySoC.git
```

Command to run the simulation(Pre-Synthesis)
```
cd VSDBabySoC
make pre_synth_sim

gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```

The image below shows the output of the Pre-synthesis simulation.

![Alt text](w2.4.jpg)


Command to run synthesis(GLS)
```
cd VSDBabySoC
make synth
```

The image below shows the command-line output post Synthesis.

![Alt text](w2.1.jpg)

Command to run the simulation(Post-Synthesis)
```
cd VSDBabySoC
make post_synth_sim

gtkwave output/post_synth_sim/post_synth_sim.vcd
```

The image below shows the output of the Post-synthesis simulation.
![Alt text](w2.2.jpg)


We can clearly see from the Pre-Synthesis and Post-Synthesis simulation outputs that are attached `GLS = Functional outputs`.
The waveforms show the same set of variables simulated with matching outputs.



## Fundamentals of STA (Static Timing Analysis) 


### Timing Analysis Summary

Static Timing Analysis (STA) ensures reliable data transfer across sequential elements by verifying timing paths with respect to the clock.  
Each **timing path** is analyzed to confirm that **setup** and **hold** requirements are met for all signals.


### Path-Based Analysis

Each path in the circuit is defined as:

- **Start Point:** Flip-Flop (FF) **clock pin** or **input port**  
- **End Point:** Flip-Flop (FF) **data (D) pin** or **output port**

Logic gates are represented as **nodes** forming a **Directed Acyclic Graph (DAG)**.  
A **source** and **sink** node are added to compute:  

- **AAT (Actual Arrival Time)**  
- **RAT (Required Arrival Time)**  

**Slack = RAT − AAT**  
Positive slack → timing met 
Negative slack → violation 


### Setup and Hold Checks

| Type | Purpose | Delay Type | Slack Term | Condition |
|------|----------|-------------|-------------|------------|
| **Setup Analysis** | Ensures data arrives **before** next clock edge | **Max delay** | **Setup Slack (Max Slack)** | Data must settle before capture |
| **Hold Analysis** | Ensures data remains **stable after** same clock edge | **Min delay** | **Hold Slack (Min Slack)** | Data must not change too early |

**Path types analyzed:**
- `reg → reg`
- `in → reg`
- `reg → out`
- `in → out`
- Clock gating  
- Recovery/Removal  
- Data–Data  
- Latch (Time-Borrow / Time-Given)


### Clock Definitions and Analysis

Accurate **clock definition** is vital for STA. Clock parameters directly influence setup/hold margins:

- **Clock Skew:** Difference in clock arrival times at start and end points  
- **Pulse Width:** Ensures proper clock duty cycle  
- **Clk-to-Q Delay:** Time for FF output to respond to a clock edge  

All these affect the **effective data arrival** and **required times** used for slack computation.


### Slew and Load Analysis

| Category | Parameters Checked | Purpose |
|-----------|--------------------|----------|
| **Slew / Transition** | Data (min/max), Clock (min/max) | Ensures valid rise/fall times |
| **Load Analysis** | Fanout (min/max), Capacitance (min/max) | Determines delay due to loading |


### Final Timing Computation Flow

1. Define clocks and identify start/end points  
2. Convert logic into nodes → form **DAG**  
3. Calculate **AAT** (data arrival) and **RAT** (required time)  
4. Compute **Setup and Hold Slack**  
5. Verify no negative slack exists (timing closure achieved)


### Conclusion 

STA validates that signals launched from **FF clock/input** reach the **FF D/output** on time, meeting both **setup (max)** and **hold (min)** constraints, guided by clock behavior, path delays, and load characteristics.



## Generating Timing Graphs with OpenSTA

OpenSTA installation was done using the below repository.
```
https://github.com/The-OpenROAD-Project/OpenSTA
```


The tcl file used to generate the result is mentioned below.
Code:
```
set list_of_lib_files(1) "sky130_fd_sc_hd__tt_025C_1v80.lib"
set list_of_lib_files(2) "sky130_fd_sc_hd__ff_100C_1v65.lib"
set list_of_lib_files(3) "sky130_fd_sc_hd__ff_100C_1v95.lib"
set list_of_lib_files(4) "sky130_fd_sc_hd__ff_n40C_1v56.lib"
set list_of_lib_files(5) "sky130_fd_sc_hd__ff_n40C_1v65.lib"
set list_of_lib_files(6) "sky130_fd_sc_hd__ff_n40C_1v76.lib"
set list_of_lib_files(7) "sky130_fd_sc_hd__ss_100C_1v40.lib"
set list_of_lib_files(8) "sky130_fd_sc_hd__ss_100C_1v60.lib"
set list_of_lib_files(9) "sky130_fd_sc_hd__ss_n40C_1v28.lib"
set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
read_liberty ./timing_libs/$list_of_lib_files($i)
read_verilog vsdbabysoc.synth.v
link_design vsdbabysoc
current_design
read_sdc vsdbabysoc_synthesis.sdc
check_setup -verbose
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} > ./sta_output/min_max_$list_of_lib_files($i).txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_max_slack.txt
report_worst_slack -max -digits {4} >> ./sta_output/sta_worst_max_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_min_slack.txt
report_worst_slack -min -digits {4} >> ./sta_output/sta_worst_min_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_tns.txt
report_tns -digits {4} >> ./sta_output/sta_tns.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_wns.txt
report_wns -digits {4} >> ./sta_output/sta_wns.txt
}
```

### Timing Analysis Results

This repository contains the timing analysis results for various PVT (Process, Voltage, Temperature) corners. The analysis focuses on key timing metrics: Worst Setup Slack, Worst Hold Slack, Worst Negative Slack (WNS), and Total Negative Slack (TNS).

### Results Table

The following table summarizes the timing analysis for each PVT corner:

| PVT_CORNER   | Worst Setup Slack | Worst Hold Slack | WNS       | TNS           |
| :----------- | :---------------- | :--------------- | :-------- | :------------ |
| tt_025C_1v80 | 2.2603            | 0.3096           | 0         | 0             |
| ff_100C_1v65 | 4.1853            | 0.2491           | 0         | 0             |
| ff_100C_1v95 | 5.5202            | 0.1960           | 0         | 0             |
| ff_n40C_1v56 | 1.8047            | 0.2915           | 0         | 0             |
| ff_n40C_1v65 | 3.1788            | 0.2551           | 0         | 0             |
| ff_n40C_1v76 | 4.2413            | 0.2243           | 0         | 0             |
| ss_100C_1v40 | -11.2888          | 0.9053           | -11.2888  | -9245.0244    |
| ss_100C_1v60 | -4.8042           | 0.6420           | -4.8042   | -3378.2246    |
| ss_n40C_1v28 | -55.7561          | 1.8296           | -55.7561  | -46170.3242   |
| ss_n40C_1v35 | -35.1855          | 1.3475           | -35.1855  | -28713.4316   |
| ss_n40C_1v40 | -27.0853          | 1.1249           | -27.0853  | -21725.4824   |
| ss_n40C_1v44 | -22.7070          | 0.9909           | -22.7070  | -17801.5625   |
| ss_n40C_1v76 | -5.2654           | 0.5038           | -5.2654   | -3208.7738    |

### Interpretation of Metrics

*   **PVT_CORNER**: Represents the Process, Voltage, and Temperature conditions under which the timing analysis was performed.
    *   `tt`: Typical Process
    *   `ff`: Fast Process
    *   `ss`: Slow Process
    *   `025C`, `100C`, `n40C`: Temperature in Celsius (e.g., 25°C, 100°C, -40°C)
    *   `1v80`, `1v65`, `1v95`, etc.: Voltage (e.g., 1.80V, 1.65V, 1.95V)
*   **Worst Setup Slack**: The smallest positive or most negative setup slack found in the design. A negative value indicates a setup timing violation.
*   **Worst Hold Slack**: The smallest positive or most negative hold slack found in the design. A negative value indicates a hold timing violation.
*   **WNS (Worst Negative Slack)**: The single worst (most negative) slack value across all timing paths in the design. A negative WNS indicates a timing violation.
*   **TNS (Total Negative Slack)**: The sum of all negative slack values in the design. A negative TNS indicates the total magnitude of timing violations.

### Observations

*   The `tt` and `ff` corners generally show positive setup and hold slack, indicating no timing violations under these conditions.
*   The `ss` (slow process) corners, particularly at lower voltages (`1v28`, `1v35`, `1v40`, `1v44`), exhibit significant negative WNS and TNS, indicating severe timing violations. This is expected as the slow process corner represents the worst-case scenario for performance.
*   As voltage increases in the `ss_n40C` corner (e.g., from `1v28` to `1v76`), the negative slack values improve (become less negative), suggesting better timing performance at higher voltages.


Below are the generated graphs.

![Alt text](w2.8.jpg)

![Alt text](w2.9.jpg)


</details>








# WEEK 4
<details>
<summary><b> Day1 - Introduction to Circuit Design and SPICE Simulations </b></summary>

## Why do we need SPICE simulations?

SPICE simulations verify circuit behavior and performance before implementation. In circuit design, logic gates like NAND, NOR, and NOT are built using NMOS and PMOS transistors.

SPICE simulations let us test and fine-tune circuits virtually before building them. For example, if we provide an input waveform to a circuit, SPICE generates the output waveform, showing how the circuit behaves.

These waveforms help us measure things like delay. By tweaking the design parameters, such as the width (W) and length (L) of transistors, we can control the current flow, which in turn affects the delay. This makes it easier to optimize the circuit for better performance.

For example, in an inverter:

- PMOS (pull-up) and NMOS (pull-down) drains connect to the output with a capacitive load.
- Both gates connect to the input, PMOS source to VDD, and NMOS source to GND.

SPICE ensures the circuit meets functionality, timing, and power requirements.

![Alt text](w4.1.jpg)


## LABS

### SPICE Lab with Sky130 Models

To use SPICE with Sky130 technology, you have to clone the below GitHub repository containing Sky130 models and circuits for simulation.
```
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
```

Download NGSpice using the below command.
```
sudo apt install ngspice
```

#### Important Files in the Repo:

`/sky130CircuitDesignWorkshop/design/sky130_fd_pr/cells/nfet_01v8/sky130_fd_pr__nfet_01v8__tt.pm3.spice`
This file contains the SPICE model for the NFET (N-channel MOSFET) in the Sky130 process at typical (tt) conditions.

`/sky130CircuitDesignWorkshop/design/sky130_fd_pr/cells/nfet_01v8/sky130_fd_pr__nfet_01v8__tt.corner.spice`
This file provides the corner model for the NFET, used for simulating different process variations.

`/sky130CircuitDesignWorkshop/design/sky130_fd_pr/models/sky130.lib.pm3.spice`
This library file contains all the SPICE models for components in the Sky130 process node.


### Ids vs Vds over constant Vgs plot

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
 XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
 R1 n1 in 55
 Vdd vdd 0 1.8V
 Vin in 0 1.8V

*simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control

 run
 display
setplot dc1
.endc
.end
```

Use the below commands.
```
ngspice day1_nfet_idvds_L2_W5.spice
plot -vdd#branch
```

The below images show the terminal output along with plots.

![Alt text](w4.2.jpg)

![Alt text](w4.3.jpg)

![Alt text](w4.4.jpg)


</details>




<details>
<summary><b> Day2 - SPICE simulation for lower nodes and velocity Saturation effect </b></summary>

## SPICE simulation for lower nodes

![Alt text](w4.9.jpg)

![Alt text](w4.10.jpg)

For **long-channel devices**, drain current shows a **quadratic dependence** on gate voltage.

For **short-channel devices**, it is **quadratic at low gate voltage** but becomes **linear at higher voltages** due to velocity saturation. 

![Alt text](w4.11.jpg)

At **lower electric fields**, carrier velocity increases **linearly** with the electric field.

At **higher electric fields**, velocity saturates and becomes **constant** due to **velocity saturation**.

![Alt text](w4.12.jpg)

![Alt text](w4.13.jpg)

![Alt text](w4.14.jpg)

Velocity Saturation Drain Current Model:
- Technology Parameter: Saturation voltage (Vdsat) – the voltage at which carrier velocity saturates, independent of Vgs or Vds.
- Vgt is minimum → FET is in saturation region (both long and short channel).
- Vds is minimum → FET is in linear region (both long and short channel).
- Vdsat is minimum → FET velocity saturates (only for short channel).

![Alt text](w4.15.jpg)

![Alt text](w4.16.jpg)


### OBSERVATIONS

- Observation 1: For short-channel devices, at higher electric fields, the device enters velocity saturation, causing Ids to remain constant as Vds increases. Here, Ids becomes a linear function of Vgs, unlike the quadratic dependence in long-channel devices.
- Observation 2: Velocity saturation causes the device to saturate earlier.


## CMOS Voltage Transfer Characteristics (VTC):

Vout is high when Vin is low, and Vout is low when Vin is high.

The transition region shows a steep drop where both NMOS and PMOS conduct, defining the switching threshold.

In MOS devices, `Rp` (PMOS) and `Rn` (NMOS) act as non-linear resistors, where their resistance is a function of drain current `Ids`, influenced by gate voltage `Vgs` and drain voltage `Vds`.


### PMOS/NMOS Drain Current vs. Drain Voltage:

Linear Region (Vds < Vdsat):
- Drain current (Ids) increases linearly with Vds.
- Device behaves like a resistor controlled by Vgs.

Saturation Region (Vds ≥ Vdsat):
- Drain current (Ids) becomes constant and independent of Vds.
- For NMOS, Ids depends on Vgs - Vth.
- For PMOS, Ids depends on Vth - Vgs.


1) Step1. Convert PMOS gate-source voltage to Vin.
2) Step2. Convert PMOS and NMOS drain-source voltages to Vout.
3) Step3. Relate the drain-source voltages of both devices to the output voltage ``Vout` by considering their respective operating regions (linear or saturation).
4) Step4. Merge the PMOS and NMOS load curves by combining their drain current (Ids) characteristics with respect to Vout.



## LABS

### Id vs Vds PLOT

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control

run
display
setplot dc1
.endc
.end
```

Use the below commands.
```
ngspice day2_nfet_idvds_L015_W039.spice
plot -vdd#branch
```

The below images show the terminal output along with plots.

![Alt text](w4.5.jpg)

![Alt text](w4.6.jpg)



### Id vs Vgs PLOT

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.1 

.control

run
display
setplot dc1
.endc
.end
```

Use the below commands.
```
ngspice day2_nfet_idvgs_L015_W039.spice
plot -vdd#branch
```

The below images show the terminal output along with plots.

![Alt text](w4.7.jpg)

![Alt text](w4.8.jpg)


</details>







<details>
<summary><b> Day3 - SPICE dynamic simulation and CMOS switching threshold </b></summary>

## Voltage Transfer Characteristics: SPICE Simulations

For a CMOS inverter, the SPICE deck includes:
- Component Connectivity: Define connections between components like PMOS, NMOS, Vdd, Vss, and Vin/Vout.
- Component Values: Specify values for components such as threshold voltages (Vth), transistor sizes, and supply voltages.
- Identify Nodes: Identify all electrical nodes like Vin, Vout, source, drain, and bulk for both transistors.
- Name Nodes: Assign names to each node, ensuring clear identification during simulations (e.g., Vout for the output node).

![Alt text](w4.23.jpg)

### SPICE simulation for CMOS inverter

SPICE Netlist for CMOS Inverter A SPICE netlist for a CMOS inverter includes transistor models, power supply connections, input voltage source, transistor specifications (PMOS and NMOS), and simulation commands (e.g., `.tran` for transient analysis).

### Static Behavior Evaluation: CMOS Inverter Robustness and Switching Threshold (Vm)

The **switching threshold (Vm)** is the point where `Vin = Vout`, and both transistors are in saturation (since `Vds = Vgs`). At **Vm**, maximum power is drawn due to large current, and it can be graphically found at the intersection of the VTC with the Vin = Vout line. The analytical expression for Vm is obtained by equating the drain currents of PMOS and NMOS (`IDSn = IDSp`).

![Alt text](w4.24.jpg)

![Alt text](w4.25.jpg)


In the **velocity-saturated** case, the switching threshold (Vm) is the point where both NMOS and PMOS transistors are in saturation, and the drain currents are equal. This occurs when the VDS of both devices is less than the saturation voltage, i.e., `VDSAT < (Vm − VT)`. The threshold voltage Vm can be derived by equating the drain currents of both transistors, with the device widths and lengths (W/L ratios) playing a key role in determining the point where both transistors conduct equally.

![Alt text](w4.26.jpg)

![Alt text](w4.27.jpg)

![Alt text](w4.28.jpg)


### Velocity Saturation and Switching Threshold (Vm) Analysis

- Case 1: Velocity Saturation Occurs
	- Happens in short-channel devices or high supply voltage.
	- Switching threshold Vm occurs when currents of PMOS and NMOS transistors are equal.
	- The ratio r (PMOS to NMOS strength) influences Vm.
	- For Vm ≈ VDD/2, r ≈ 1.
	- Vm can be adjusted by changing the PMOS or NMOS width:
		- Increase PMOS width → shift Vm upwards.
		- Increase NMOS width → shift Vm downwards.

- Case 2: Velocity Saturation Does Not Occur
	- Applies to long-channel devices or low supply voltage.
	- The switching threshold Vm is still affected by r but with a simpler formula.
	- If r ≈ 1, Vm is near VDD/2.

- PMOS Width Effect on VTC
	- Increasing PMOS width shifts Vm upwards.
	- Increasing NMOS width shifts Vm downwards.
	- Vm is relatively stable with small variations in transistor ratios.


![Alt text](w4.29.jpg)


### Applications of CMOS inverter in clock network and STA

![Alt text](w4.30.jpg)

![Alt text](w4.31.jpg)




## LABS

### Voltage transfer characteristics PLOT

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc
.end
```

Use the below commands.
```
ngspice day3_inverter_vtc_Wp084_Wn036.spice
plot out vs in
```

The below images show the terminal output along with plots.

![Alt text](w4.17.jpg)

![Alt text](w4.18.jpg)

![Alt text](w4.19.jpg)


**Switching Threshold (VM):**

The input voltage (Vin) at which the output voltage (Vout) equals the input voltage.

- It is the intersection point on the Voltage Transfer Curve (VTC).
- Also referred to as the midpoint voltage of the CMOS inverter.


### CMOS Transient Analysis

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands
.tran 1n 10n
.control
run
.endc
.end
```

Use the below commands.
```
ngspice day3_inverter_tran_Wp084_Wn036.spice
plot out vs time in
```

The below images show the terminal output along with plots.

![Alt text](w4.20.jpg)

![Alt text](w4.21.jpg)

![Alt text](w4.22.jpg)

**Propagation Delay:**

The time difference (measured at 50% of input-output transition) between the input signal change and the corresponding output signal switch in a logic gate (e.g., inverter).

</details>









<details>
<summary><b> Day4 - Static Behavior Evaluation: CMOS Inverter Robustness and Noise Margin </b></summary>

## CMOS Inverter Robustness and Noise Margin

- Introduction to Noise Margin:
	- Noise margin is the maximum noise voltage a CMOS circuit can tolerate without logic errors.
	- If noise amplitude is below the noise margin, it is attenuated through the logic gate, ensuring error-free signal transmission.

- Function of Noise Margin:
	- Ensures signals with small noise added to logic 1 remain as logic 1 and those with noise on logic 0 remain as logic 0.

- Application:
	- Consider two back-to-back CMOS inverters.
	- The voltage transfer curve (VTC) determines noise margins:


### Noise Margin: VIL and VIH Points

- Definition:
	- `VIL` and `VIH` are operational points where the slope of the voltage transfer curve (VTC) is `-1.`
	- These points correspond to the gain of the inverter being equal to -1.
 
- Logic Recognition:
	- Input voltage `0 to VIL` → Recognized as `Logic 0.`
	- Input voltage `VIH to VDD` → Recognized as `Logic 1.`


### Noise Margins in CMOS Logic Gates

1) Conditions for Proper Operation:

	- The following conditions must hold for a logic gate to function correctly in the presence of noise:
	```
	VOL_MAX < VIL_MAX  
	VOH_MIN > VIH_MIN  
	```

2) Behavior in Different Input Ranges:

	- For Vin ≤ VIL:
	The inverter gain magnitude is less than unity, resulting in minimal output change for a given input change.
	
	- For Vin ≥ VIH:
	The gain magnitude is also less than unity, leading to minimal output change.
	
	- For VIL < Vin < VIH:
	The gain magnitude exceeds unity, causing significant output changes. This range is called the undefined region because noise signals pushing Vin into this range may introduce errors.

3) Noise Margin Equations:

	- Low-Level Noise Margin (NML):
	```
	NML = VIL_MAX - VOL_MAX
	```
	
	- High-Level Noise Margin (NMH):
	```
	NMH = VOH_MIN - VIH_MIN
	```
	
	- Overall Noise Margin (NM):
	```
	NM = Min(NML, NMH)  
	```

The above noise margins define the tolerance of a circuit to noise without compromising its logical operation.


![Alt text](w4.32.jpg)

![Alt text](w4.33.jpg)

![Alt text](w4.34.jpg)

![Alt text](w4.35.jpg)

Noise Margin Robustness against variations in Device Ratio.

![Alt text](w4.36.jpg)




## LABS

### Static behavior evaluation: CMOS inverter robustness, Noise margin

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15

Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc
.end
```

Use the below commands.
```
ngspice day4_inv_noisemargin_wp1_wn036.spice
plot out vs in
```

The below images show the terminal output along with plots.

![Alt text](w4.37.jpg)

![Alt text](w4.38.jpg)

![Alt text](w4.39.jpg)



</details>








<details>
<summary><b> Day5 - CMOS power supply and device variation robustness evaluation </b></summary>


## Static Behavior Evaluation: CMOS Inverter Robustness - Power Supply Variation

Overview:
- Power supply scaling affects the static behavior of the CMOS inverter, including the switching threshold, noise margins, and overall robustness.

Smart SPICE Simulation:
- Simulate the CMOS inverter across different supply voltage values (VDD).
- Analyze how the voltage transfer characteristics (VTC) and critical parameters (like VM and noise margins) shift with varying power supply.

Key Observations:
- As VDD decreases, the switching threshold (VM) may shift closer to the center of the supply range, but noise margins reduce.
- Lower power supply reduces overall noise immunity and can impact performance.
- At higher VDD, both static power dissipation and noise margins increase.

![Alt text](w4.42.jpg)


### Simulation Observations: Power Supply Variations

Robust Operation at Lower VDD:
- The CMOS inverter operates effectively even at 0.8V, which is below half the original supply voltage (1.8V) and close to the transistor threshold voltages.
  
Switching Threshold Proportionality:
- With a fixed transistor ratio (r), the switching threshold (VM) is approximately proportional to VDD.

Gain in Transition Region:
- The inverter gain in the transition region increases as the supply voltage decreases.

Transition Region Width:
- The width of the transition region reduces when the supply voltage is scaled down from the original VDD.


![Alt text](w4.45.jpg)

![Alt text](w4.46.jpg)


### Limitations of Operating at Low Supply Voltages

Performance Degradation:
- Lowering the supply voltage reduces energy dissipation but significantly increases gate transition times, negatively affecting performance.

Parameter Sensitivity:
- At low supply voltages, the DC characteristics become highly sensitive to variations in device parameters, such as transistor threshold voltages.

Reduced Signal Swing:
- Scaling VDD reduces signal swing, lowering internal noise (e.g., crosstalk) but increasing sensitivity to external noise sources, which do not scale proportionally. 


### CMOS Inverter Robustness to Device Variations
Design vs. Real Operating Conditions:
- Gates are designed for nominal conditions, but actual operating temperatures and device parameters can vary widely after fabrication.

DC Characteristics Stability:
- The DC characteristics of the CMOS inverter are relatively insensitive to variations in device parameters, allowing the gate to remain functional across a broad range of operating conditions.

Sources of Variation:
- One common source of variation is the etching process during fabrication, which can affect device parameters.


![Alt text](w4.47.jpg)

![Alt text](w4.48.jpg)

![Alt text](w4.49.jpg)


### Sources of variation: Oxide Thickness

![Alt text](w4.50.jpg)

![Alt text](w4.51.jpg)



## LABS

### Smart SPICE simulation for power supply variations

Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15

Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

.control
let powersupply = 1.8
alter Vdd = powersupply
    let voltagesupplyvariation = 0
    dowhile voltagesupplyvariation < 6
        dc Vin 0 1.8 0.01
        let powersupply = powersupply - 0.2
        alter Vdd = powersupply
        let voltagesupplyvariation = voltagesupplyvariation + 1
    end

plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in xlabel "input voltage(V)" ylabel "output voltage(V)" title "Inverter dc characteristics as a function of supply voltage"

.endc
.end
```

Use the below commands to get the output.
```
ngspice day5_inv_supplyvariation_Wp1_Wn036.spice
```

The below images show the terminal output along with plots.

![Alt text](w4.40.jpg)

![Alt text](w4.41.jpg)



### CMOS Inverter Robustness: Extreme Device Width Variation
- Robustness to Width Variation:

	The CMOS inverter remains functional even under extreme variations in device width (W) due to its static behavior properties.

- Effect on Switching Threshold:
  
	Large deviations in device width primarily affect the switching threshold ( V_M ) but do not drastically impact the inverter's ability to operate as a logic gate.

- Impact on Noise Margins:

	Variations in width cause asymmetry in the noise margins, with one margin (high or low) reducing while the other increases.

- Key Insight:

	The CMOS inverter exhibits robust static behavior, tolerating extreme width variations without functional failure


![Alt text](w4.42.jpg)


Code.
```
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42 l=0.15

Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc
.end
```


Use the below commands to get the output.
```
ngspice day5_inv_device_variation_Wp1_Wn036.spice
plot out vs in
```

The below images show the terminal output along with plots.

![Alt text](w4.43.jpg)

![Alt text](w4.44.jpg)

</details>








# WEEK 5
<details>
<summary><b> Day1 - VSD Hardware Design Program </b></summary>

## OpenROAD installation guide

1. Clone the OpenROAD Repository

```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts
```

![Alt text](w5.1.jpg)

2. Run the Setup Script

```
sudo ./setup.sh
```

![Alt text](w5.2.jpg)

3. Build OpenROAD

```
./build_openroad.sh --local
```

![Alt text](w5.3.jpg)

4. Verify Installation

```
source ./env.sh
yosys -help  
openroad -help
```

![Alt text](w5.4.jpg)

![Alt text](w5.5.jpg)

5. Run the OpenROAD Flow

```
cd flow
make
```

![Alt text](w5.6.jpg)

6. Launch the graphical user interface (GUI) to visualize the final layout

```
 make gui_final
```

![Alt text](w5.7.jpg)


### ORFS Directory Structure and File formats

OpenROAD-flow-scripts/

```
├── OpenROAD-flow-scripts             
│   ├── docker           -> It has Docker based installation, run scripts and all saved here
│   ├── docs             -> Documentation for OpenROAD or its flow scripts.  
│   ├── flow             -> Files related to run RTL to GDS flow  
|   ├── jenkins          -> It contains the regression test designed for each build update
│   ├── tools            -> It contains all the required tools to run RTL to GDS flow
│   ├── etc              -> Has the dependency installer script and other things
│   ├── setup_env.sh     -> Its the source file to source all our OpenROAD rules to run the RTL to GDS flow
```

Inside the flow/ Directory

```
├── flow           
│   ├── design           -> It has built-in examples from RTL to GDS flow across different technology nodes
│   ├── makefile         -> The automated flow runs through makefile setup
│   ├── platform         -> It has different technology note libraries, lef files, GDS etc 
|   ├── tutorials        
│   ├── util            
│   ├── scripts                 
```

![Alt text](w5.8.jpg)

</details>





<details>
<summary><b> Day2 - Floorplan and Placement of VSDBabySoC in OpenROAD </b></summary>

### RTL2GDS Flow for VSDBabySoC: Initial Steps

1) Create Directories:
	- Inside `OpenROAD-flow-scripts/flow/designs/sky130hd/`, create a folder named vsdbabysoc.
	- Create another folder named vsdbabysoc in `OpenROAD-flow-scripts/flow/designs/src/` and place all Verilog files here.

2) Copy Folders:
	- From your VSDBabySoC folder, copy the following folders into sky130hd/vsdbabysoc:
		- gds: Contains `avsddac.gds`, `avsdpll.gds`.
		- include: Contains `sandpiper.vh`, `sandpiper_gen.vh`, `sp_default.vh`, `sp_verilog.vh`.
		- lef: Contains `avsddac.lef`, `avsdpll.lef`.
		- lib: Contains `avsddac.lib`, `avsdpll.lib`.

3) Copy Constraint and Configuration Files:
	- Copy `vsdbabysoc_synthesis.sdc` into `sky130hd/vsdbabysoc`.
	- Copy `macro.cfg` and `pin_order.cfg` into `sky130hd/vsdbabysoc`.

4) Create Config File:
	- Create a `config.mk` file in `sky130hd/vsdbabysoc` with the required configuration details.


`config.mk`

This script sets up environment variables and configurations for the design and synthesis of a System-on-Chip (SoC) using the OpenROAD flow. The design is based on the "vsdbabysoc" and targets the "sky130hd" platform.

</details>









# WEEK 6
<details>
<summary><b> Day1 - Inception of Open soure EDA, OpenLANE, Sky130 PDK </b></summary>

## Chip Design Overview (Arduino Example)

### Package & Structure
- **Package:** `7 mm × 7 mm`
- **Wire Bonds:** Connect output terminals of the die (chip) to the package pins.
- **Pads:** Metal terminals on the chip through which **signals (s/g)** are sent or received.
- **Core of Chip:** Contains the **logical part** of the circuit (digital logic).
- **Foundry IPs:** Pre-designed components provided by the foundry:
  - `SRAM` – Static Random Access Memory  
  - `DAC` – Digital-to-Analog Converter  
  - `ADC` – Analog-to-Digital Converter  
- **Macros:** Reusable, larger functional blocks (e.g., memory blocks, PLLs).


## RISC-V ISA (Instruction Set Architecture)

### Software → Hardware Flow
```text
C Program 
   ↓
Compiler → Assembly Language Program 
   ↓
Assembler → Machine Language Program (binary instructions)
   ↓
Binary Instructions (bits) → executed by hardware
   ↓
Chip Layout (Physical Representation)
```

## Process Design Kit (PDK)

A **Process Design Kit (PDK)** is a collection of files that **model a semiconductor fabrication process** for Electronic Design Automation (EDA) tools. It is essential for designing integrated circuits (ICs) that are manufacturable.

| Component | Description |
| :--- | :--- |
| **Process Design Rules** | Rules governing the physical layout of the IC. Includes files for: |
| | - **DRC** (Design Rule Check) |
| | - **LVS** (Layout Versus Schematic) |
| | - **PEX** (Parasitic Extraction) |
| **Device Models** | Mathematical models (e.g., SPICE models) describing the electrical behavior of transistors and other devices at different operating conditions. |
| **Digital Standard Libraries** | Characterized data (timing, power, noise) for the basic logic gates (**standard cells**), allowing EDA tools to synthesize and optimize the design. |
| **I/O Libraries** | Characterized data for input/output buffer cells used to interface the core logic with the outside world. |


## RTL-to-GDSII Design Flow (Physical Design)

The RTL-to-GDSII flow converts the high-level design description (RTL) into a final, manufacturable layout format (GDSII).

### 1. Synthesis
* **Goal:** Convert the Register-Transfer Level (RTL) Hardware Description Language (HDL) code (e.g., VHDL, Verilog) into a **gate-level netlist**.
* **Process:** Maps the RTL logic to the specific primitive logic gates found in the **digital standard libraries** (from the PDK).

### 2. Floorplanning
* **Goal:** Define the chip's physical structure.
* **Process:** Determines **chip dimensions**, fixes major **pin locations**, and defines the legal areas for standard cells (**row definitions**).

### 3. Power Planning
* **Goal:** Create a robust distribution network for power ($V_{DD}$) and ground ($V_{SS}$).
* **Process:** Adds **power pads**, builds wide **power rings** around the core, and implements a grid of **power straps** across the core to ensure stable voltage supply to all cells.

### 4. Placement
* **Goal:** Position the standard cells onto the chip's core area.
* **Process:** Places the cells onto the predefined floorplan **rows**, aligning them with the legal placement **sites** (involves both global and detailed optimization).

### 5. Clock Tree Synthesis (CTS)
* **Goal:** Create a balanced, low-skew clock distribution network.
* **Process:** Builds a dedicated network (a 'tree') to deliver the clock signal simultaneously to all sequential elements (e.g., flip-flops) with minimal timing differences (skew).

### 6. Routing
* **Goal:** Implement the necessary interconnects (wires) between all standard cells.
* **Process:** Uses the available **metal layers** (defined by the PDK) to form all signal and power connections, respecting the routing grid and design rules (involves global and detailed routing phases).

### 7. Sign-Off
* **Goal:** Verify the final layout against all design constraints to ensure manufacturability and performance.
* **Key Checks:**
    * **DRC** (Design Rule Check): Checks the physical layout against the geometric rules of the fab process.
    * **LVS** (Layout Versus Schematic): Verifies that the physical layout accurately matches the original gate-level netlist.
    * **STA** (Static Timing Analysis): Confirms that the design meets all required timing constraints (setup and hold times) under various conditions.


### **Final Output:**
The successful completion of the sign-off process results in the **GDSII** file—the final layout data used by the fabrication plant (fab).

Below image shows the OpenLANE ASIC flow

![Alt text](w6.1.jpg)


## Open Source tools
Logic Equivalence Check - yosys

Physical Implementation - OpenROAD

Design for Testability(DFT) - FAULT

Static Timing Analysis(STA) - OpenSTA

Design Rule Check(DRC) - Magic

Layour vs Schematic(LVS) - Magic and Netgen

</details>




<details>
<summary><b> Day2 - Good floorplan vs bad floorplan and introduction to library cells </b></summary>

# Floorplanning and Powerplanning

## Utilization factor and aspect ratio
Consider the below circuit as the example.

![Alt text](w6.2.jpg)

A **core** is the section in the chip where the fundamental logic of the design is placed

A **die**, which consists of the core, is small semiconductor material specimen on which the fundamental circuit is fabricated.

**Utilization factor**  = area occupied by netlist / total area of the core.

If utilization factor  = 1, it means 100% utilization is achieved.

**Aspect ratio** = Height/width

As per the below placement of the circuitry, the aspect ratio = 1 and the utilization factor = 1.

![Alt text](w6.3.jpg)

As per the below placement of the circuitry, the aspect ratio = 0.5 and the utilization factor = 0.5.

![Alt text](w6.4.jpg)


## Preplaced cells
There are certain combinational circuits which consists of huge amounts of gates and implementing them in the actual netlist might not be feasible. These kind of combinational circuits are taken out of the main circuitry and implemented separately.

Consider the below example where the combinational circuit is split into 2 different blocks and then later generalised a Black Box. This black box needs to be implemented once on the chip and then can be reused multiple times.

![Alt text](w6.5.jpg)

![Alt text](w6.6.jpg)

These black boxes are now seprarated and considered as two separate IP's or modules.

Similarly there are other IP's also available like Memory, Comparator, Mux, etc.

- The arrangement of these IPs in a chip is referred to as Floorplanning.
- These IP blocks have used-defined locations and hence are placed in a chip before placement and routing is done. Hence, they are called as pre-placed cells.
- The remaining the logical cells are placed onto the core during P&R.

Once the preplaced cells are assigned location during Floorplanning, they cannot be moved in the later stages.


## De-coupling capacitors

It acts as a local, high-speed energy reservoir. When a logic gate switches state (e.g., from 0 to 1), it draws a large, instantaneous surge of current. The inductance of the main power lines cannot supply this current fast enough, leading to a temporary voltage drop, known as `I.R` drop or ground bounce.

The decap quickly discharges to supply this instantaneous current, locally stabilizing the $V_{DD}$ rail and preventing the power supply voltage from dropping significantly.

By preventing the power supply ($V_{DD}$) from dropping during current surges, decaps ensure that the gates can maintain their proper output voltage levels ($V_{OH}$ and $V_{OL}$), which are fundamental to the noise margin calculation. A drop in $V_{DD}$ causes $V_{OH}$ to decrease, directly reducing the $NM_H$. 

Similarly, ground bounce causes $V_{OL}$ to rise, reducing $NM_L$.By stabilizing the supply voltage, decaps minimize the noise injected into the power rails, thereby ensuring the logic gates operate with the intended, larger noise margins, preventing functional failures and improving overall circuit reliability.

The image below shows how the decaps are placed around the preplaced cells.

![Alt text](w6.7.jpg)


## Power Planning

In the image below we see that there is only one power supply which is connected to the four macros. 

![Alt text](w6.10.jpg)

The issue in the above case is that each macro is connected to only one single power source. 

For example when we have multiple bit changing from HIGH -> LOW, i.e. the capacitors are discharging. We have a single ground path where all this charge will be drained. This creates a ground bump leading to indefinite voltage levels hampering the logic. 

Similarly, when multiple bits change from LOW -> HIGH, i.e. the capacitors are getting charged. Since, we have a single $V_{DD}$ path from where all these capacitors will be charged, it leads to indefinite voltage levels hampering the logic. 

Hence, we need to design and place the power lines like in the image below to overcome this issue.

![Alt text](w6.8.jpg)

The image below shows the power grids are designed to ensure that there are immediate nearby power connections for the logic circuitry.

![Alt text](w6.9.jpg)


## Pin placement and logical cell placement blockage

Input and Output pins are placed in the area between the core and the die. 

The clk pins will be bigger in size compared to the other pins are they are responsible for driving the circuitry. Hence, to increase the speed, we increase the area and reduce the resistance.

Post the pin placement, we block the area so that nothing is placed near the pins. This is called as the Logical cell placement blockage. Post this, the Floorplan is ready for the Placement and Routing stage as shown in the image below.

![Alt text](w6.11.jpg)



# Placement and Routing

## Netlist binding and initial place design

The image below shows that we have the netlist, the floorplan and physical view of the logical gates as the input for the Placement stage.

![Alt text](w6.12.jpg)

Once we have the floorplan, our next step is to use the libraries and match the requirements of the design. These libraries contain the delay information of different types of the same logic gate. Each logic gate will have different sizes in the libraries. The EDA tool chooses the appropriate size of the standard cell to meet the design constraints.

Once these cells are chosen, the netlist is to be placed over the given floorplan. 


## Optimizing the placement

Once we have placed the netlist on the floorplan, before sending it Routing stage, we have to check for the capacitance of the interconnects, the intergrity of the signal and other things to ensure that placement is optimal and doesnt lead to any kind of failures.

To deal with signal integrity we insert repeaters, so that the signal doesnt fade off between the logical circuitry and the functionality remains valid.

Once the buffers and the blocks are placed over the floorplan, we check the timing constraints, frequency constraints, area constraints and other specifications to ensure that the placement is optimal. If not the placement is changed in accordance with the specifications.

The image below shows how the placement can be done for the given netlist and floorplan.

![Alt text](w6.13.jpg)


# Cell design and characterization flows

## Cell Design Flow

Inputs -> Design Steps -> Outputs

1. Inputs

Process Design Kits(PDKs) : The foundry provides this PDK file which consists of DRC and LVS rules, SPICE models, library and user-defined specs.
- **DRC and LVS rules** : Contains design rules such as constraints on poly width, poly to actice spacing, etc.
- **SPICE models** : Threshold voltage equations, Linear region current equation, Saturation region current equation.
- **Library and user-defined specs** : Cell height(Distance between the power and ground rail), Supply voltage, Metal layers, Pin location, Drawn gate length

2. Design Steps
- **Circuit Design** : Designing the circuit on the basis of the function given and in accordance with the constraints mentioned in the PDK file. 
- **Layout Design** : Generate Euler's path from the circuit designed and get the stick diagram for the circuit. You also get the information of the capacitance and resistances of each and every metal layer and contacts.
- **Characterization** : Step where we get our timing, noise and power information.

3. Outputs
- The output of the Circuit design is a Circuit Description Language(CDL) file.
- The output of the Layout design is GDSII file, LEF file, extracted SPICE netlist(.cir).
- The output of the Characterization step is timing, noise and power .libs and functionality of the circuit.


## Characterization Flow

The following are the eight steps involved in characterization:
- Read the model files.
- Read the extracted SPICE netlist.
- Recognize the behaviour of the logic gates used.
- Read the sub-circuits of the extracted SPICE netlist.
- Attach the necessary power sources.
- Apply the stimulus.
- Provide the necessary output capacitances.
- Provide the necessary simulation commands.


## Timings Characterization

1. Timing threshold definitions.
- slew_low_rise_thr
- slew_high_rise_thr
- slew_low_fall_thr
- slew_high_fall_thr
- in_rise_thr
- in_fall_thr
- out_rise_thr
- out_fall_thr

The above eight steps are together filed and given as input to the GUNA software. This software generates timing, noise and power .libs

2. Propogation Delay

Propogation delay = time(out_*_thr) - time(in_*_thr)

The image below show few examples of calculating Propogation Delay through simulations.

![Alt text](w6.14.jpg)

![Alt text](w6.15.jpg)

3. Transition Time

Transition time = time(slew_high_fall_thr) - time(slew_low_fall_thr) or transition time = time(slew_high_rise_thr) - time(slew_low_rise_thr).

The image below shows an example calculating Transition time through simulation.

![Alt text](w6.16.jpg)

</details>


<details>
<summary><b> Day3 - Design library cell using Magic Layout and ngspice characterization </b></summary>

## CMOS fabrication process
1. Selecting a Substrate.
2. Create active region for transistors.
	- We need to create insulation between the pockets i.e. the active transistors so that they donot interfere in each other's functionality.
 	- So, for that we first grow a silicon dioxide layer on our selected substrate.
  	- Deposit a silicon nitride layer on silicon dioxide layer.
   	- Then, deposit a layer of photoresist on the silicon nitride layer.
   	- UV light is passed on the top of the layout. This leads to the removal of photoresist in the non-protected areas.
   	- Remove the mask which was used to protect certain area. The silicon nitride will be etched in the part where mask was not used.
   	- Remove the photoresist layer as silicon nitride itself will act as a protection layer to grow the oxides on the other areas.
   	- The layour is now placed in oxidation furnace. Field oxide is grown. This process is called **LOCOS** (local oxidation of silicon).
   	- Etch out all the silicon nitride left.
  
The images below shows the initial and final diagrams of this step.

![Alt text](w6.17.jpg)

![Alt text](w6.18.jpg)

3. N-well and P-well formation.
	- We now use another type of mask.. We repeat the process of UV light and remove the mask.
 	- Then we use Boron(p-type material) to create the P-well. This process is called **Ion Implantation**.
  	- Repeat the above two steps using a different mask and create the N-well with the help of Phosphorous(n-type material).
   	- Take complete structure to driving furnace so that the wells are well diffused.

The images below shows the initial and final diagrams of this step.

![Alt text](w6.19.jpg)

![Alt text](w6.20.jpg)

4. Formation of Gate
	- Repeat steps done in 3, to form the small N and P wells.
 	- The original oxide layer is stripped using dilute hydrofluoric solution and new oxide layer is re-grown to give high quality oxide.
  	- Add a polysilicon layer over the oxide layer and add n-type ion implants for low gate resistance.
   	- Add another mask to create the gates and remove the polysilicon layer except where the mask was.

The images below show the initial and final stage.

![Alt text](w6.21.jpg)

![Alt text](w6.22.jpg)

5. Lightly doped drain(LDD) formation.
	- Add different masks again to create N-implant in P-well and P-implant in N-well.
 	- Add a thick silicon dioxide layer and perform **Plasma anistropic etching**.

![Alt text](w6.23.jpg)


</details>
