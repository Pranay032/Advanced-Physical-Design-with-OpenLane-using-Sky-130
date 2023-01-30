# Advance Physical Design Using OpenLANE/Sky130
![09](https://user-images.githubusercontent.com/118599201/215371874-c8572480-a194-4bc7-8041-005f4f900d49.png)

This project is done in the course ["Advanced Physical Design using OpenLANE/Sky130"](https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/) by VLSI System Design Corporation. In this project a complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom designed standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified.


## Table of Contents
1. [Overview](#overview)
2. [Inception of Opensource EDA](#inception-of-opensource-eda)
   - [How to Talk to computers?](#how-to-talk-to-computers)
   - [SOC Design & OpenLANE](#soc-design--openlane)
     - [Components of opensource digital ASIC design](#components-of-opensource-digital-asic-design)
     - [Simplified RTL2GDS Flow](#simplified-rtl2gds-flow)
     - [OpenLANE ASIC Flow](#openlane-asic-flow)
   - [Opensource EDA tools](#opensource-eda-tools)
     - [OpenLANE design stages](#openlane-design-stages)
     - [OpenLANE Files](#openlane-files)
     - [Invoking OpenLANE](#invoking-openlane)
     - [Design Preparation Step](#design-preparation-step)
     - [Review of files & Synthesis step](#design-preparation-step#review-of-files-&-synthesis-step)
3. [Floorplanning & Placement and library cells](#floorplanning-&-placement-and-library-cells)
   - [Floorplanning considerations](#floorplanning-considerations)
     - [Utilization Factor & Aspect Ratio](#utilization-factor--aspect-ratio)
     - [Pre-placed cells](#pre-placed-cells)
     - [Decoupling capacitors](#decoupling-capacitors)
     - [Power Planning](#power-planning)
     - [Pin Placement](#pin-placement)
     - [Floorplan run on OpenLANE & view in Magic](#floorplan-run-on-openlane--view-in-magic)
   - [Placement](#placement)
     - [Placement Optimization](#placement-optimization)
     - [Placement run on OpenLANE & view in Magic](#placement-run-on-openlane--view-in-magic)
   - [Standard Cell Design Flow](#standard-cell-design-flow)
   - [Standard Cell Characterization Flow](#standard-cell-characterization-flow)
   - [Timing Parameter Definitions](#timing-parameter-definitions)
4. [Design library cell](#design-library-cell)
   - [SPICE Deck creation & Simulation](#spice-deck-creation--simulation)
   - [CMOS inverter Switching Threshold Vm](#cmos-inverter-switching-threshold-vm)
   - [16 Mask CMOS Fabrication](#16-mask-cmos-fabrication)
   - [Inverter Standard cell Layout & SPICE extraction](#inverter-standard-cell-layout--spice-extraction)
   - [Inverter Standard cell characterization](#inverter-standard-cell-characterization)
   - [Magic Features & DRC rules](#magic-features--drc-rules)
5. [Timing Analysis & CTS](#timing-analysis--cts)
   - [Create port definition](#create-port-definition)
   - [Standard Cell LEF generation](#standard-cell-lef-generation)
   - [Integrating custom cell in OpenLANE](#integrating-custom-cell-in-openlane)
   - [Post-synthesis timing analysis](#post-synthesis-timing-analysis)
   - [Clock Tree Synthesis](#clock-tree-synthesis)
6. [Final steps in RTL2GDS](#final-steps-in-rtl2gds)
   - [Power Distribution Network generation](#power-distribution-network-generation)
   - [Routing](#routing)
   - [GDSII](#gdsii)
   - [LEF](#lef) 
7. [Acknowledgements](#acknowledgements)



## Overview
OpenLANE is an opensource tool or flow used for opensource tape-outs. The OpenLANE flow comprises a variety of tools such as Yosys, ABC, OpenSTA, Fault, OpenROAD app, Netgen and Magic which are used to harden chips and macros, i.e. generate final GDSII from the design RTL. The primary goal of OpenLANE is to produce clean GDSII with no human intervention. OpenLANE has been tuned to function for the Google-Skywater130 Opensource Process Design Kit.

## Inception of Opensource EDA

### How to talk to computers?

The RISC-V Instruction Set Architecture (ISA) is a language used to talk to computers whose hardware is based on RISC-V core. If a user wishes to run a certain application software on a computer, its corresponding C/C++/Java program must be converted into instructions by the compliler. The ouput of the compiler is hardware dependent. These instructions go as inputs to the assembler which outputs binary language that the hardware logic in the chip layout can make sense of. According to the bits received, the digital logic consisting of gates performs the function required by the user of the application software.

### SoC Design & OpenLANE

#### Components of opensource digital ASIC design
The design of digital Application Specific Integrated Circuit (ASIC) requires three enablers or elements - Resistor Transistor Logic Intellectual Property (RTL IPs), Electronic Design Automation (EDA) Tools and Process Design Kit (PDK) data.


- Opensource RTL Designs: github, librecores, opencores
- Opensource EDA tools: QFlow, OpenROAD, OpenLANE
- Opensource PDK data: Google Skywater130 PDK

The ASIC flow objective is to convert RTL design to GDSII format used for final layout. The flow is essentially a software also known as automated PnR (Place & route).

#### Simplified RTL2GDS Flow

![RTL2GDS flow](https://user-images.githubusercontent.com/86701156/124006238-a139c780-d9f7-11eb-8da9-6069b055fbe0.PNG)


- Synthesis: RTL Converted to gate level netlist using standard cell libraries (SCL)
- Floor & Power Planning: Planning of silicon area to ensure robust power distribution
- Placement: Placing cells on floorplan rows aligned with sites
  - Global Placement: for optimal position of cells
  - Detailed Placement: for legal positions
- Routing: Valid patterns for wires
- Signoff: Physical (DRC, LVS) and Timing verifications (STA)

#### OpenLANE ASIC Flow


![OpenLaneflow](https://user-images.githubusercontent.com/83152452/131135115-46148ff1-9489-48f6-a334-6702c25def59.png)

From conception to product, the ASIC design flow is an iterative process that is not static for every design. The details of the flow may change depending on ECO’s, IP requirements, DFT insertion, and SDC constraints, however the base concepts still remain. The flow can be broken down into 11 steps:

1. Architectural Design – A system engineer will provide the VLSI engineer with specifications for the system that are determined through physical constraints. 
   The VLSI engineer will be required to design a circuit that meets these constraints at a microarchitecture modeling level.

2. RTL Design/Behavioral Modeling – RTL design and behavioral modeling are performed with a hardware description language (HDL). 
   EDA tools will use the HDL to perform mapping of higher-level components to the transistor level needed for physical implementation. 
   HDL modeling is normally performed using either Verilog or VHDL. One of two design methods may be employed while creating the HDL of a microarchitecture:
   
    a. RTL Design – Stands for Register Transfer Level. It provides an abstraction of the digital   circuit using:
   
   - i. 	 Combinational logic
   - ii. 	 Registers
   - iii.  Modules (IP’s or Soft Macros)
 
    b. 	Behavioral Modeling – Allows the microarchitecture modeling to be performed with behavior-based modeling in HDL. This method bridges the gap between C and HDL allowing HDL design to be performed

3. RTL Verification - Behavioral verification of design

4. DFT Insertion - Design-for-Test Circuit Insertion

5. Logic Synthesis – Logic synthesis uses the RTL netlist to perform HDL technology mapping. The synthesis process is normally performed in two major steps:

     - GTECH Mapping – Consists of mapping the HDL netlist to generic gates what are used to perform logical optimization based on AIGERs and other topologies created 
       from the generic mapped netlist.
       
     - Technology Mapping – Consists of mapping the post-optimized GTECH netlist to standard cells described in the PDK
  
6. Standard Cells – Standard cells are fixed height and a multiple of unit size width. This width is an integer multiple of the SITE size or the PR boundary. Each standard cell comes with SPICE, HDL, liberty, layout (detailed and abstract) files used by different tools at different stages in the RTL2GDS flow.

7. Post-Synthesis STA Analysis: Performs setup analysis on different path groups.

8. Floorplanning – Goal is to plan the silicon area and create a robust power distribution network (PDN) to power each of the individual components of the synthesized netlist. In addition, macro placement and blockages must be defined before placement occurs to ensure a legalized GDS file. In power planning we create the ring which is connected to the pads which brings power around the edges of the chip. We also include power straps to bring power to the middle of the chip using higher metal layers which reduces IR drop and electro-migration problem.

9. Placement – Place the standard cells on the floorplane rows, aligned with sites defined in the technology lef file. Placement is done in two steps: Global and Detailed. In Global placement tries to find optimal position for all cells but they may be overlapping and not aligned to rows, detailed placement takes the global placement and legalizes all of the placements trying to adhere to what the global placement wants.

10. CTS – Clock tree synteshsis is used to create the clock distribution network that is used to deliver the clock to all sequential elements. The main goal is to create a network with minimal skew across the chip. H-trees are a common network topology that is used to achieve this goal.

11. Routing – Implements the interconnect system between standard cells using the remaining available metal layers after CTS and PDN generation. The routing is performed on routing grids to ensure minimal DRC errors.

The Skywater 130nm PDK uses 6 metal layers to perform CTS, PDN generation, and interconnect routing.

### Opensource EDA tools

OpenLANE utilises a variety of opensource tools in the execution of the ASIC flow:
Task | Tool/s
------------ | -------------
RTL Synthesis & Technology Mapping | [yosys](https://github.com/YosysHQ/yosys), abc
Floorplan & PDN | init_fp, ioPlacer, pdn and tapcell
Placement | RePLace, Resizer, OpenPhySyn & OpenDP
Static Timing Analysis | [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
Clock Tree Synthesis | [TritonCTS](https://github.com/The-OpenROAD-Project/OpenLane)
Routing | FastRoute and [TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute) 
SPEF Extraction | [SPEF-Extractor](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
DRC Checks, GDSII Streaming out | [Magic](https://github.com/RTimothyEdwards/magic), [Klayout](https://github.com/KLayout/klayout)
LVS check | [Netgen](https://github.com/RTimothyEdwards/netgen)
Circuit validity checker | [CVC](https://github.com/d-m-bailey/cvc)

#### OpenLANE design stages

1. Synthesis
	- `yosys` - Performs RTL synthesis
	- `abc` - Performs technology mapping
	- `OpenSTA` - Performs static timing analysis on the resulting netlist to generate timing reports
2. Floorplan and PDN
	- `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
	- `ioplacer` - Places the macro input and output ports
	- `pdn` - Generates the power distribution network
	- `tapcell` - Inserts welltap and decap cells in the floorplan
3. Placement
	- `RePLace` - Performs global placement
	- `Resizer` - Performs optional optimizations on the design
	- `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. CTS
	- `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. Routing
	- `FastRoute` - Performs global routing to generate a guide file for the detailed router
	- `CU-GR` - Another option for performing global routing.
	- `TritonRoute` - Performs detailed routing
	- `SPEF-Extractor` - Performs SPEF extraction
6. GDSII Generation
	- `Magic` - Streams out the final GDSII layout file from the routed def
	- `Klayout` - Streams out the final GDSII layout file from the routed def as a back-up
7. Checks
	- `Magic` - Performs DRC Checks & Antenna Checks
	- `Klayout` - Performs DRC Checks
	- `Netgen` - Performs LVS Checks
	- `CVC` - Performs Circuit Validity Checks

#### OpenLANE Files

The openLANE file structure looks something like this:

- skywater-pdk: contains PDK files provided by foundry
- open_pdks: contains scripts to setup pdks for opensource tools 
- sky130A: contains sky130 pdk files

#### Invoking OpenLANE and Design Preparation 

Openlane can be invoked using docker command followed by opening an interactive session.
flow.tcl is a script that specifies details for openLANE flow.

```
docker
./flow.tcl -interactive
package require openlane 0.9
```

```
prep -design picorv32a
```

![02](https://user-images.githubusercontent.com/118599201/215371579-806c6f9f-cfb0-48ac-9f6a-ee08f00a61da.png)

![03](https://user-images.githubusercontent.com/118599201/215372096-f00d1686-c15f-4a32-897a-3781108cc101.png)

#### Review of files & Synthesis step
* A "runs" folder is generated within the picorv32a folder.
* The merged file is created during the merging operation in the pircorv32a design preparation (it merges lef and techlef files)
* Next, we run the synthesis of picorv32a design in the openlane interactive terminal:

`run_synthesis`
![04](https://user-images.githubusercontent.com/118599201/215372329-bdf02203-fab1-4ed1-b3f1-adce443ea312.png)
![05](https://user-images.githubusercontent.com/118599201/215372333-3cf21bf3-a3d7-418e-87aa-62e8aa48f280.png)
![06](https://user-images.githubusercontent.com/118599201/215372341-64097333-ac8c-40e8-8efa-a7027a206e8b.png)

* The yosys and ABC tools are utilised to convert RTL to gate level netlist.
* Calcuation of Flop Ratio:
```
Flop ratio = Number of D Flip flops 
             ______________________
             Total Number of cells
```

```
dfxtp_4 = 1613,
Number of cells = 14876,
Flop ratio = 1623/18036 = 0.1084 = 10.84%
```
![flip flops](https://user-images.githubusercontent.com/118599201/215404277-57752317-40bc-4957-9280-b918ce53a8aa.png)
* We may check the success of the synthesis step by checking the synthesis folder for the synthesized netlist file (.v file)
* The synthesis statistics report can be accessed within the reports directory. It is usually the last yosys file since files are listed chronologically by date of modification.


## Floorplanning & Placement and library cells

### Floorplanning considerations

#### Utilization Factor & Aspect Ratio  

Two parameters are of importance when it comes to floorplanning namely, Utilisation Factor and Aspect Ratio. They are defined as follows:

```
Utilisation Factor =  Area occupied by netlist
                     __________________________
                        Total area of core
```

```
Aspect Ratio =  Height
               ________
                Width
```
                                  
A Utilisation Factor of 1 signifies 100% utilisation leaving no space for extra cells such as buffer. However, practically, the Utilisation Factor is 0.5-0.6. Likewise, an Aspect ratio of 1 implies that the chip is square shaped. Any value other than 1 implies rectanglular chip.

#### Pre-placed cells

Once the Utilisation Factor and Aspect Ratio has been decided, the locations of pre-placed cells need to be defined. Pre-placed cells are IPs comprising large combinational logic which once placed maintain a fixed position. Since they are placed before placement and routing, the are known as pre-placed cells.

#### Decoupling capacitors

Pre-placed cells must then be surrounded with decoupling capacitors (decaps). The resistances and capacitances associated with long wire lengths can cause the power supply voltage to drop significantly before reaching the logic circuits. This can lead to the signal value entering into the undefined region, outside the noise margin range. Decaps are huge capacitors charged to power supply voltage and placed close the logic circuit. Their role is to decouple the circuit from power supply by supplying the necessary amount of current to the circuit. They pervent crosstalk and enable local communication.

#### Power Planning

Each block on the chip, however, cannot have its own decap unlike the pre-placed macros. Therefore, a good power planning ensures that each block has its own VDD and VSS pads connected to the horizontal and vertical power and GND lines which form a power mesh.

#### Pin Placement

The netlist defines connectivity between logic gates. The place between the core and die is utilised for placing pins. The connectivity information coded in either VHDL or Verilog is used to determine the position of I/O pads of various pins. Then, logical placement blocking of pre-placed macros is performed so as to differentiate that area from that of the pin area.

#### Floorplan run on OpenLANE & view in Magic

* Importance files in increasing priority order:

1. ```floorplan.tcl``` - System default envrionment variables
2. ```conifg.tcl```
3. ```sky130A_sky130_fd_sc_hd_config.tcl```

* Floorplan envrionment variables or switches:

1. ```FP_CORE_UTIL``` - floorplan core utilisation
2. ```FP_ASPECT_RATIO``` - floorplan aspect ratio
3. ```FP_CORE_MARGIN``` - Core to die margin area
4. ```FP_IO_MODE``` - defines pin configurations (1 = equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - vertical metal layer
6. ```FP_CORE_HMETAL``` - horizontal metal layer

***Note: Usually, vertical metal layer and horizontal metal layer values will be 1 more than that specified in the files***
 
 To run the picorv32a floorplan in openLANE:
 ```
 run_floorplan
 
 ```

![20](https://user-images.githubusercontent.com/118599201/215374784-678d96d0-0838-428e-987c-7126d9bf8713.png)
 

Post the floorplan run, a .def file will have been created within the ```results/floorplan``` directory. 
We may review floorplan files by checking the ```floorplan.tcl```. 
The system defaults will have been overriden by switches set in ```conifg.tcl``` and further overriden by switches set in ```sky130A_sky130_fd_sc_hd_config.tcl```.
 
To view the floorplan, Magic is invoked after moving to the ```results/floorplan``` directory:

```
magic -T /home/lyagapranayreddy32/Desktop/work/toos/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.min.lef def read picorv32a.def &
```
![15](https://user-images.githubusercontent.com/118599201/215374793-9ee20d1a-70c9-484b-ab88-573e83092eaa.png)

 
* One can zoom into Magic layout by selecting an area with left and right mouse click followed by pressing "z" key. 
* Various components can be identified by using the ```what``` command in tkcon window after making a selection on the component.
* Zooming in also provides a view of decaps present in picorv32a chip.
* The standard cell can be found at the bottom left corner.

![16](https://user-images.githubusercontent.com/118599201/215375152-04a95e42-bf1f-42c8-8772-14f625137694.png)
![17](https://user-images.githubusercontent.com/118599201/215375156-6e303636-4c3a-4127-9636-247b5ffcd53b.png)

### Placement 

#### Placement Optimization

The next step in the OpenLANE ASIC flow is placement. The synthesized netlist is to be placed on the floorplan. Placement is perfomed in 2 stages:

1. Global Placement: It finds optimal position for all cells which may not be legal and cells may overlap. Optimization is done through reduction of half parameter wire length.
2. Detailed Placement: It alters the position of cells post global placement so as to legalise them.

Legalisation of cells is important from timing point of view. 

#### Placement run on OpenLANE & view in Magic

Command:

```
run_placement
```
![21](https://user-images.githubusercontent.com/118599201/215375366-efd3f209-f135-413e-9580-3043abcfe5dc.png)


The objective of placement is the convergence of overflow value. If overflow value progressively reduces during the placement run it implies that the design will converge and placement will be successful. Post placement, the design can be viewed on magic within ```results/placement``` directory:

```
magic -T /home/lyagapranayreddy32/Desktop/work/toos/openlane_working_dir/openlane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &

```
![22](https://user-images.githubusercontent.com/118599201/215375427-cf1de7f7-3c87-4bf9-90fd-3502b68ea7ad.png)

Zoomed-in views of the standard cell placement:

![23](https://user-images.githubusercontent.com/118599201/215375468-c30bf808-cd51-44b2-8011-acb35327657a.png)

***Note: Power distribution network generation is usually a part of the floorplan step. However, in the openLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN***


### Standard Cell Design Flow

Standard cell design flow involves the following:

1. Inputs: PDKs, DRC & LVS rules, SPICE models, libraries, user-defined specifications. 
2. Design steps: Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

### Standard Cell Characterization Flow

A typical standard cell characterization flow includes the following steps:

1. Read in the models and tech files
2. Read extracted spice netlist
3. Recognise behaviour of the cell
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide necessary output capacitance loads
8. Provide necessary simulation commands

The opensource software called GUNA can be used for characterization. Steps 1-8 are fed into the GUNA software which generates timing, noise and power models.

### Timing Parameter Definitions

Timing defintion | Value
------------ | -------------
slew_low_rise_thr  | 20% value
slew_high_rise_thr |  80% value
slew_low_fall_thr | 20% value
slew_high_fall_thr | 80% value
in_rise_thr | 50% value
in_fall_thr | 50% value
out_rise_thr | 50% value
out_fall_thr | 50% value

```
rise delay =  time(out_fall_thr) - time(in_rise_thr)

Fall transition time: time(slew_high_fall_thr) - time(slew_low_fall_thr)

Rise transition time: time(slew_high_rise_thr) - time(slew_low_rise_thr)
```

A poor choice of threshold points leads to neative delay value. Therefore a correct choice of thresholds is very important.

## Design library cell

OpenLANE allows users to make changes to environment variables on the fly. For instance, if we wish to change the pin placement from equidistant to some other style of placement we may do the following in the openLANE flow:

```
set ::env(FP_IO_MODE) 2
```

### SPICE Deck creation & Simulation

A SPICE deck includes information about the following:

1. Model description
2. Netlist description
3. Component connectivity
4. Component values
5. Capacitance load
6. Nodes
7. Simulation type and parameters
8. Libraries included

### CMOS inverter Switching Threshold Vm

Thw sitching threshold of a CMOS inverter is the point on the transfer characteristic where Vin equals Vout (=Vm). At this point both PMOS and NMOS are in ON state which gives rise to a leakage current

### 16 Mask CMOS Fabrication

The 16-mask CMOS process consists of the following steps:

1. Selection of subtrate: Secting the body/substrate material.
2. Creating active region for transistors: Isolation between active region pockets by SiO2 and Si3N4 deposition followed by photolithography and etching.
3. N-well and P-well formation: Ion implanation by Boron for P-well and by Phosphorous for N-well formation.
4. Formation of gate terminal: NMOS and PMOS gates formed by photolithography techniques.
5. LDD (lightly doped drain) formation: LDD formed to prevent hot electron effect.
6. Source & drain formation: Screen oxide added to avoid channelling during implants followed by Aresenic implantation and annealing.
7. Local interconnect formation: Removal of screen oxide by HF etching. Deposition of Ti for low resistant contacts.
8. Higher level metal formation: CMP for planarization followed by TiN and Tungsten deposition. Top SiN layer for chip protection.

### Inverter Standard cell Layout & SPICE extraction

The Magic layout of a CMOS inverter will be used so as to intergate the inverter with the picorv32a design. To do this, inverter magic file is sourced from [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign) by cloning it within the ```openlane_working_dir/openlane``` directory as follows:

```
git clone https://github.com/nickson-jose/vsdstdcelldesign
```

This creates a vsdstdcelldesign named folder in the openlane directory.

To invoke magic to view the ```sky130_inv.mag``` file, the sky130A.tech file must be included in the command along with its path. To ease up the complexity of this command, the tech file can be copied from the magic folder to the vsdstdcelldesign folder.

The sky130_inv.mag file can then be invoked in Magic very easily:

```
magic -T sky130A.tech sky130_inv.mag &
```
![25](https://user-images.githubusercontent.com/118599201/215378582-aef47deb-a094-45c2-8a57-baa93c1586a9.png)
![24](https://user-images.githubusercontent.com/118599201/215378589-ddd45134-85ec-405b-a3b9-ab9e2bf02e20.png)
In Sky130 the first layer is called the local interconnect layer or Locali.

To verify whether the layout is that of CMOS inverter, verification of P-diffusiona nd N-diffusion regions with Polysilicon can be observed.

Other verification steps are to check drain and source connections. The drains of both PMOS and NMOS must be connected to output port (here, Y) and the sources of both must be connected to power supply VDD (here, VPWR).

**LEF or library exchange format:**
A format that tells us about cell boundaries, VDD and GND lines. It contains no info about the logic of circuit and is also used to protect the IP.

**SPICE extraction:**
Within the Magic environment, following commands are used in tkcon to achieve .mag to .spice extraction:

```
extract all
ext2spice cthresh 0 rethresh 0
ext2spice
```

![d3-02](https://user-images.githubusercontent.com/118599201/215378734-516d672c-e9ea-4330-8869-1e586e4b5293.png)

This generates the ```sky130_in.spice``` file. This SPICE deck is edited to include ```pshort.lib``` and ```nshort.lib``` which are the PMOS and NMOS libraries respectively. In addition, the minimum grid size of inverter is measured from the magic layout and incorporated into the deck as: ```.option scale=0.01u```. The model names in the MOSFET definitions are changed to ```pshort.model.0``` and ```nshort.model.0``` respectively for PMOS and NMOS. 

Finally voltage sources and simulation commands are defined as follows:

```
VDD VPWR 0 3.3V
VSS VGND 0 0
Va A VGND PUSLE(0V 3.3V 0 0.1ns 0.1 ns 2ns 4ns)
.tran 1n 20n
.control
run 
.endc
.end
```

The final sky130_inv.spice file is modified to:

![d3](https://user-images.githubusercontent.com/118599201/215378892-767b6529-a37c-426c-a566-9c9b4c79def5.png)
![d3-1](https://user-images.githubusercontent.com/118599201/215378923-3810dd2a-2c24-4ab2-826c-3da900cd36df.png)
For simulation, ngspice is invoked in the terminal:

```
ngspice sky130_inv.spice
```

The output "y" is to be plotted with "time" and swept over the input "a":

```
plot y vs time a
```

![d3001](https://user-images.githubusercontent.com/118599201/215379430-82b46e1e-373c-4b69-a86b-7c70ebd4500a.png)

The waveform obtained is as shown:

![d310](https://user-images.githubusercontent.com/118599201/215379437-6864e044-ada5-4e5c-8c3b-b74c63e1bef7.png)


The spikes in the output at switching points is due to low capacitance loads. This can be taken care of by editing the spice deck to increase the load capacitance value.

### Inverter Standard cell characterization

Four timing parameters are used to characterize the inverter standard cell:

1. Rise transition: Time taken for the output to rise from 20% of max value to 80% of max value
2. Fall transition- Time taken for the output to fall from 80% of max value to 20% of max value
3. Cell rise delay = time(50% output rise) - time(50% input fall)
4. Cell fall delay  = time(50% output fall) - time(50% input rise)

The above timing parameters can be computed by noting down various values from the ngspice waveform.

![WhatsApp Image 2023-01-30 at 8 19 42 AM](https://user-images.githubusercontent.com/118599201/215378207-3b55b94a-e28e-4e3b-bbd9-5268c8270c12.jpeg)
![WhatsApp Image 2023-01-30 at 8 19 43 AM](https://user-images.githubusercontent.com/118599201/215378216-6ce0fbc5-a7a5-4cc0-878f-2f65f007c476.jpeg)



## Timing Analysis & CTS

A requirement for ports as specified in ```tracks.info``` is that they should be at intersection of horizontal and vertical tracks. The CMOS Inverter ports A and Y are on li1 layer. It needs to be ensured that they're on the intersection of horizontal and vertical tracks. We access the tracks.info file for the pitch and direction information:

![day-401](https://user-images.githubusercontent.com/118599201/215430622-4c2dd65d-e4df-4d8c-a037-73cc1876b560.png)

To ensure that ports lie on the intersection point, the grid spacing in Magic (tkcon) must be changed to the li1 X and li1 Y values. Convergence of grid and tracks can be achieved using the following command:

```
grid 0.46um 0.34um 0.23um 0.17um
```


![402](https://user-images.githubusercontent.com/118599201/215430913-bd740e90-e1ea-424d-b8a5-962b30b00ed5.png)

### Create port definition

Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. 

The easiest way to define a port is through Magic Layout window and following are the steps:

- In Magic Layout window, first source the .mag file for the design (here inverter). Then **Edit >> Text** which opens up a dialogue box.

![port A](https://user-images.githubusercontent.com/118599201/215499855-6ad821b2-e71b-49d7-914b-91a7504f7eb6.png)

- For each layer (to be turned into port), make a box on that particular layer and input a label name along with a sticky label of the layer name with which the port needs to be associated. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:
![port Y](https://user-images.githubusercontent.com/118599201/215500251-4c47e5a4-67a1-4c44-ab9c-2a244b50ad00.png)

In the above two figures, port A (input port) and port Y (output port) are taken from locali (local interconnect) layer. Also, the number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

- For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1 (Notice the sticky label).
![port vpwr](https://user-images.githubusercontent.com/118599201/215500229-3a1bca64-b3bc-4d69-a16e-44dc03195287.png)

![port vgnd](https://user-images.githubusercontent.com/118599201/215500221-0eea6206-ce1f-4832-ae4a-465bd9330331.png)


### Standard Cell LEF generation

Before the CMOS Inverter standard cell LEF is extracted, the purpose of ports must be defined:

Select port A in magic:
```
port class input
port use signal
```
Select Y area
```
port class output
port class signal
```
Select VPWR area
```
port class inout
port use power
```

Select VGND area
```
port class inout
port use ground
```
![port direct](https://user-images.githubusercontent.com/118599201/215500275-c3cd769d-5a13-47df-877f-9fa21295f3b9.png)
LEF extraction can be carried out in tkcon as follows:

```
lef write
```

![lef](https://user-images.githubusercontent.com/118599201/215501182-c871e87a-8c00-48a2-89f2-41d861fb4a09.png)

This generates ```sky130_vsdinv.lef``` file.


### Integrating custom cell in OpenLANE

In order to include the new standard cell in the synthesis, copy the sky130_vsdinv.lef file to the ```designs/picorv32a/src``` directory  
Since abc maps the standard cell to a library abc there must be a library that defines the CMOS inverter. The ```sky130_fd_sc_hd_typical.lib``` file from ```vsdstdcelldesign/libs``` directory needs to be copied to the ```designs/picorv32a/src``` directory (Note: the slow and fast library files may also be copied).

Next, ```config.tcl``` must be modified:
```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

```modified config.tcl file:```

```

# User config
set ::env(DESIGN_NAME) "picorv32a"

# Change if needed
set ::env(VERILOG_FILES) "./designs/picorv32a/src/picorv32a.v"
set ::env(SDC_FILES) "./designs/picorv32a/src/picorv32a.sdc"


# turn off clock
set ::env(CLOCK_PERIOD) "5.000"
set ::env(CLOCK_PORT) "clk"

set ::env(CLOCK_MET) $::env(CLOCK_PORT) 


set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib "
set ::env(LIB_MIN) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_MAX) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib "
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]

set filename $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
#set filename $::env(DESIGN_DIR)/$::env(PDK)_$::env(STD_CELL_LIBRARY)_config.tcl
if { [file exists $filename] == 1 } {
      source $filename
}
```

In order to integrate the standard cell in the OpenLANE flow, invoke openLANE as usual and carry out following steps:

```
prep -design picorv32a -tag my_run1 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
![merge](https://user-images.githubusercontent.com/118599201/215540877-96c8060e-5ef9-412a-a9c4-88a731710a6a.png)
![syyynth](https://user-images.githubusercontent.com/118599201/215535637-2e8e9ce4-2b1a-4a98-955c-ca294b447e78.png)
![synth-04](https://user-images.githubusercontent.com/118599201/215535703-1f469e91-3d55-4b5b-97ff-da9b362eaad2.png)
Next floorplan is run, followed by placement:
To run the floorplan for the design using a custom standard cell, the standard command "run_floorplan" produces an error. To resolve this issue, we will need to take a longer route and use the internal commands to have greater control over the process.

The first step in this process is to run the "floorplan" command using the following command:
```
init_floorplan
```
![init](https://user-images.githubusercontent.com/118599201/215540445-ffef8df8-a424-4430-b951-0c00164c86e9.png)
Then, the second step is to run the IO placer using the following command.
```
place_io
```
![ioplace](https://user-images.githubusercontent.com/118599201/215540660-56fc54a0-960c-4953-9e86-b235623152f1.png)
The goal of global placement is to generate a legal placement of blocks that meets the design constraints and sets the foundation for the next step of the design flow.
```
global_placement_or
```
![global](https://user-images.githubusercontent.com/118599201/215546247-f3ddf303-ca48-4f89-8d22-b2f157189942.png)
In detailed placement, the blocks or components are placed in a more specific location and orientation on the chip, taking into account the interconnections between the blocks, routing channels, and the overall area utilization of the chip. The output of the detailed placement step is a detailed layout of the circuit, which serves as the basis for the next step in the design flow.
```
detailed_placement
```
![detailed](https://user-images.githubusercontent.com/118599201/215547349-7b4fcabd-3b73-49ce-a78e-8c19efe92a79.png)
To prevent irregularities in the power rails, we place tap and decap cells using the following command:
```
tap_decap_or
```
![tap](https://user-images.githubusercontent.com/118599201/215541582-a54d264a-b2f8-444e-ba62-d6b826418ad3.png)
We re-run the detailed placement after placing the tap and decap cells. This is done to adjust specific components so that they don't violate any rules, taking into account the wire lengths and capacitances estimated in the previous detailed placement run. The command used is the same:
```
detailed_placement
```
![tap next](https://user-images.githubusercontent.com/118599201/215541443-58daaed9-2fed-4377-bba3-9cce130dbe6c.png)
To generate the Power Distribution Network (PDN), we use the following command:
```
gen_pdn
```
![get pdn](https://user-images.githubusercontent.com/118599201/215546211-83150a5b-a6c7-46be-acda-0517e1f9acfa.png)

To check the layout invoke magic from the ```results/placement``` directory:

```
magic -T /home/devipriya/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```
![placement file](https://user-images.githubusercontent.com/118599201/215543571-32286f7b-1dbf-4e53-b8f1-2d4cd093526d.png)
![placement after get](https://user-images.githubusercontent.com/118599201/215543556-cdc7880b-11e5-47c9-9ead-6c555aa50d6f.png)
Since the custom standard cell has been plugged into the openLANE flow, it would be visible in the layout.

![vsdinv ](https://user-images.githubusercontent.com/118599201/215543487-1a6833b5-9d34-4f31-a090-8c32fc2092ab.png)



### Clock Tree Synthesis

The purpose of building a clock tree is enable the clock input to reach every element and to ensure a zero clock skew. H-tree is a common methodology followed in CTS.
Before attempting a CTS run in TritonCTS tool, if the slack was attempted to be reduced in previous run, the netlist may have gotten modified by cell replacement techniques. Therefore, the verilog file needs to be modified using the ```write_verilog``` command. Then, the synthesis, floorplan and placement is run again. To run CTS use the below command:

```
run_cts
```
![cts](https://user-images.githubusercontent.com/118599201/215548960-8589306b-a75b-44b0-a8ca-d72604ded922.jpeg)
The CTS run adds clock buffers in therefore buffer delays come into picture and our analysis from here on deals with real clocks. Setup and hold time slacks may now be analysed in the post-CTS STA anlysis in OpenROAD within the openLANE flow:


## Final steps in RTL2GDS


### Routing 
Routing is connecting components in an IC to form a functional circuit. It finds the optimal path for interconnections between components, considering factors like wire length, congestion, and routing layers, to meet design constraints. Routing is a complex and crucial step, impacting the final performance, power consumption, and area utilization of the chip.

After the completion of the power distribution network generation, we finally run the routing throughout the design with the following command.


```
run_routing
```

![rout1](https://user-images.githubusercontent.com/118599201/215551442-40e99faa-b651-4b93-bd46-e75ebf0840e5.png)

## GDSII

GDS Stands for Graphic Design Standard. This is the file that is sent to the foundry and is called as "tape-out". 


In openLane use the command `run_magic`

The GDSII file is generated in the `results/magic` directory.

No DRC errors are found.
![final_1](https://user-images.githubusercontent.com/118599201/215562488-e8567721-f7de-4513-91a5-4525df171863.png)
The layout pictures are shown below:

![mag](https://user-images.githubusercontent.com/118599201/215552115-57ea6716-5251-496e-855e-71ad6babb592.png)

![maggi](https://user-images.githubusercontent.com/118599201/215552162-a6bb7869-4c7d-4164-9b7b-eaf424655d7b.png)



![layout](https://user-images.githubusercontent.com/118599201/215552209-6d7a0f33-7417-4593-ba00-78711f67b31b.png)

## Acknowledgements 

- [The OpenROAD Project](https://github.com/The-OpenROAD-Project/OpenLane)
- [Nickson Jose](https://github.com/nickson-jose/vsdstdcelldesign)
