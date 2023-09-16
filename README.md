# Advanced_Physical_Design_Using_OpenLane

# Day1

<details>
  <summary>Introduction</summary>
With the introduction of open-source technology for chip creation, many RTL designs and EDA Tools were made available for free. The [SKY130 PDK] fills the gap in a whole Open source chip development.(https://skywater-pdk.readthedocs.io/en/latest/rules.html) from Skywater Technologies and Google. There were a number of EDA Tools with distinct functions throughout the design cycle. The design flow was not clear, and the Skywater pdk was only compatible with industrial equipment.  These problems were addressed by [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane), which offered a fully automated and tidy RTL to GDSII flow. OpenLane is not a product; rather, it is a flow made up of a number of EDA tools, automation scripts, and Skywater-pdks that have been optimized for use with open-source EDA tools.    
</details>

<details>
 <summary> Overall Design Flow</summary>
An RTL design is created for a design specification using HDLs like Verilog or VHDL, or it can be created using high-level synthesis tools like SystemC, MATLAB HDL Coder, Bluespec, etc. 
The process of converting the RTL Netlist into a manufactured IC then starts, and is known as the Physical Design Flow.
Floor planning, which entails placing preplaced cells, power planning, etc., comes first in the physical design process. The placement of logical synthesis comes next. So that the clock's skew is at a minimal or under the necessary threshold, we now perform CTS (Clock Tree Synthesis). Following CTS, all of the assembled components are routed. A process known as "Static Timing Analysis" is used between each and every step in the physical design flow, from logic synthesis through routing, to analyze the design at each stage and confirm that it is actually right.  Magic is an open source application to view the layouts for every stage. You can extract a tiny netlist, run a SPICE simulation, and compare the results with the post-layout Simulation using ngspice.

Physical Design begins with Floor planning - placing the preplaced cells, power planning etc., secondly Placement 

![1](https://github.com/SahilSira/Advanced_Physical_Design_Using_OpenLane/assets/140998855/46f473f3-1222-408b-b748-e8a6a0297680)


  
</details>
<details>
  <summary>OpenLane Flow</summary>

### 1.  Synthesis 
The RTL Level Design is then synthesized using a Logic Synthesizer. We use Yosys which is an Open Source Logic Synthesizer. The RTL Netlist is then  converted into a synthesised netlist where there are details about the standard cells and its implementations. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts  into  a  RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants 

### 1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area etc.,

### 1.2 Deign Exploration Utility 
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing.

### 1.3 Design For Test - DFT Insertion
This is an optional step carried out by Fault. It is used to test the design. 

###  2. Floor Planning and Power Planning
This is done by OpenROAD flow. The macros and IPs are placed in the core before proceding further. This is called as pre-placement. Floor planning is done separately for the macros and it is called macro floor planning. They are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present. Then to prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin. 

When several blocks tap power from a single source, there is a problem of Voltage Droop at the Vdd and Ground Bounce at the Vss which can again push the logic out of the required noise margin into the undefined state. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source. 

### 3. Placement
There are two types of placement.  The other required logic is placed optimally.
Placement is of two steps
- Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
- Detialed Placement - After Global placement is done minimal alterations are done to correct the issues

### 4. Clock Tree Synthesis 
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.

### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges
- OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
- OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode
OpenLane allows the user to chose either of the above approaches

### 6. Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

 - Global Routing - This is done inside the OpenROAD flow (FastRoute)
 - Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the **Logic Equivalence Check** is performed by Yosys, since OpenROAD does some optimisations the circuit.  

### 7. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

### 8. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.  

### 9. Sign-off Steps
- Design Rule Check (DRC) is performed by Magic
- Layout Versus Schematic (LVS) is performed by Netgen

### 10. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file 

## OpenLane Installation and Environment Setup

Prior to the installation of the OpenLane install the dependencies and packages using the command shown below :</br>
``` 
sudo apt-get update
sudo apt-get upgrade
sudo apt install -y build-essential python3 python3-venv python3-pip make git
```

**Steps to install OpenLane, PDKs and Tools**</br>
```
cd $HOME
git clone https://github.com/The-OpenROAD-Project/OpenLane --recurse-submodules 
cd OpenLane
make
make test
cd /C:/mavi/OpenLane/designs/ci
cp -r * ../
```

### Steps to run synthesis in OpenLane:


```
cd ~/OpenLane
make mount
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```

OpenLane invokes the following

- `Yosys` - RTL Synthesis and maps to yosys generic cells
- `abc` - Technology mapping with the Skywater130 PDK. Here `sky130_fd_sc_hd` Skywater Foundry produced High density standard cells are used.
- `OpenSTA` - This does the Static Timing Analysis on the netlist generated after synthesis and generated the timing reports 

View the synthesis statistics

![2](https://github.com/SahilSira/Advanced_Physical_Design_Using_OpenLane/assets/140998855/325d79fd-3070-4afb-9371-8da3f09fab3f)


### Key concepts

#### Flops ratio 

- The flop ratio is defined as the ratio of the number of flops to the total number of cells
- Here flop ratio is **1596/10104 = 0.1579** (i.e: 15.8%) [From the synthesis statistics]


</details>


</details>
