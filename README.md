# RTL Design & Synthesis Workshop

Internship screening covering Verilog RTL design, logic synthesis, and flop coding styles using EDA tools.

---

## Module 1 – Introduction to Verilog RTL Design & Synthesis

This section introduces RTL coding, explaining how it translates abstract logic into hardware. We also learned how to simulate designs with iverilog, view waveforms in GTKWave, and perform synthesis using Yosys.

### Step 1 – Workspace setup

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop/
ls 
```
### Step 2 – Simulate with iverilog


```bash
ls
iverilog good_mux.v tb_goodmux.v
./a.out
```
![iverilog simulation](result_images/task1.png)

### Step 3 – View waveforms in GTKWave
Testbenches (or `.tb` files) allow us to monitor the functionality and timing behavior of our module. In this case, we examined a simple 2x1 multiplexer.

```bash
gtkwave tb_goodmux.vcd
```

![GTKWave waveform](result_images/task2.png)

### Step 4 – Synthesise with Yosys
Synthesis maps abstract hardware logic to standard cells by generating a netlist.
```bash
yosys
> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
> read_verilog good_mux.v
> synth -top good_mux
```

![Yosys initialisation](result_images/yosysinitialisation.png)
![Yosys 2](result_images/yosys2.png)

### Step 5 – Map to standard cells 
Yosys allows us to examine netlists and leaf cells via the following commands:

```bash
> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
> show
> write_verilog -noattr good_mux_netlist.v
```

![abc1](result_images/abc1.png)
![show1](result_images/show1.png)
![netlist1](result_images/netlist1.png)



---

## Module 2 – Timing Libs, Hierarchical vs Flat Synthesis & Flop Coding Styles

### Step 6 – Exploring libraries

Opening the Sky130 `.lib` file lets us study cell attributes such as timing arcs, drive strength, and power. We also used a `multiple_module.v` file to examine the two primary types of synthesis.

```bash
kate ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Step 7 – Hierarchical vs Flat Synthesis

**Hierarchical** – The hierarchy of the submodules is maintained, meaning the schematic hides the internal details of the module:

```bash
yosys
> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
> read_verilog multiple_module.v
> synth -top multiple_modules
> show
```

![Hierarchy visible](result_images/hierarchy_visible.png)

**Flat** – All hierarchy is dissolved, revealing the internal functionality of the modules when the `show` command is executed:

```bash
> flatten
> show
```

![Flattened netlist](result_images/flatten1.png)
![Flattened schematic](result_images/flatten2.png)

**Sub-module synthesis** – Primarily useful for large or repeated blocks:

```bash
> synth -top sub_module1
```

![Sub-module](result_images/submodule1.png)

### Step 8 – Flop Coding Styles

Here, we simulated, synthesized, and examined different D flip-flop variants. Running `dfflibmap` before ABC ensures the correct flip-flop cells are selected. We primarily focused on flip-flops with asynchronous and synchronous resets: 

```bash
> dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
> show
```
Asynchronous Reset
```
module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule

```

![Async reset](result_images/asyncres.png)
![Async reset1](result_images/asyncres1.png)

Asynchronous Set
```


module dff_async_set ( input clk ,  input async_set , input d , output reg q );
always @ (posedge clk , posedge async_set)
begin
	if(async_set)
		q <= 1'b1;
	else	
		q <= d;
end
endmodule

```

![Async set](result_images/async_set.png)
![Async set1](result_images/async_set_1.png)

```

module dff_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk )
begin
	if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule

```

![Sync reset](result_images/syncres.png)
![Sync reset1](result_images/syncres1.png)
### Step 9 – Special optimisations (mult2, mult8)
Multiplying by 2 is essentially a left shift operation. Yosys accomplishes this optimization simply by rewiring the inputs to the outputs:

```bash
> synth -top mult2
> abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib   # reports 0 cells
> show
```

![mult2](result_images/abc_mult2.png)
![mult8](result_images/abc_mult8.png)

---

## Tools Used

- [iverilog](http://iverilog.icarus.com/) – Verilog simulation
- [GTKWave](http://gtkwave.sourceforge.net/) – Waveform viewer
- [Yosys](https://yosyshq.net/yosys/) – Logic synthesis
- [Sky130 PDK](https://github.com/google/skywater-pdk) – Standard cell library