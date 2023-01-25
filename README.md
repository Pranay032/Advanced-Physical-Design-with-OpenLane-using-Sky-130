# Advanced Physical Design with OpenLane using Sky130
![09](https://user-images.githubusercontent.com/118599201/214614256-6f311a0a-0da4-4844-997d-4ab054307024.png)

# Physical Design with OpenLane using SKY130 PDK

This project is done in the course " https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/" by VLSI System Design Corporation. In this project a complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom desgined standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified

# Table of Contents
## Overview
OpenLANE is an opensource tool or flow used for opensource tape-outs. The OpenLANE flow comprises a variety of tools such as Yosys, ABC, OpenSTA, Fault, OpenROAD app, Netgen and Magic which are used to harden chips and macros, i.e. generate final GDSII from the design RTL. The primary goal of OpenLANE is to produce clean GDSII with no human intervention. OpenLANE has been tuned to function for the Google-Skywater130 Opensource Process Design Kit.
## Introduction
With the advent of open-source technologies for Chip development, there were several RTL designs, EDA Tools which were open-sourced. The missing piece in a complete Open source chip development was filled by the SKY130 PDK from Skywater Technologies and Google. There were several EDA Tools, which played specfic roles in the design cycle. There was not a clean design flow and Skywater pdk was compatible with only the industrty tools. OpenLane addressed these issues in providing a completely automated and clean RTL to GDSII flow. OpenLane is not a tool, but a flow which consists of several EDA tools, automation scripts and Skywater-pdks tuned to work specifically with the open-source EDA tools.
## Inception of Opensource EDA
### How to talk to computers?
The RISC-V Instruction Set Architecture (ISA) is a language used to talk to computers whose hardware is based on RISC-V core. If a user wishes to run a certain application software on a computer, its corresponding C/C++/Java program must be converted into instructions by the compliler. The ouput of the compiler is hardware dependent. These instructions go as inputs to the assembler which outputs binary language that the hardware logic in the chip layout can make sense of. According to the bits received, the digital logic consisting of gates performs the function required by the user of the application software.

### SoC Design & OpenLANE
### Components of opensource digital ASIC design
The design of digital Application Specific Integrated Circuit (ASIC) requires three enablers or elements - Resistor Transistor Logic Intellectual Property (RTL IPs), Electronic Design Automation (EDA) Tools and Process Design Kit (PDK) data.
Open Source Digital ASIC Design requires three open-source components:

1.RTL Designs = github.com, librecores.org, opencores.org
2.EDA Tools = OpenROAD, OpenLANE,QFlow
3.PDK = Google + Skywater 130nm Production PDK

The ASIC flow objective is to convert RTL design to GDSII format used for final layout. The flow is essentially a software also known as automated PnR (Place & route).

# OpenLane Flow
![11](https://user-images.githubusercontent.com/118599201/214637522-dffe65d5-540e-482f-a416-a7b4e58d75f0.png)
![openlane_flow](https://user-images.githubusercontent.com/118599201/214636946-686b4cf3-3b6c-4f64-ad76-8fc00f71bee0.png)

### 1. Synthesis
The RTL Level Design is then synthesized using a Logic Synthesizer. We use Yosys which is an Open Source Logic Synthesizer. The RTL Netlist is then converted into a synthesised netlist where there are details about the standard cells and its implementations. Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts into a RTL Netlist. abc does the tehnology mapping to the required skywater-pdk variants

#### 1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area et.,

#### 1.2 Deign Exploration Utility
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing

#### 1.3 Design For Test - DFT Insertion
This is an optional step carried out by Fault. It is used to test the design

### 2. Floor Planning and Power Planning
This is done by OpenROAD flow. The macros and IPs are placed in the core before proceding further. This is called as pre-placement. Floor planning is done separately for the macros and it is called macro floor planning. They are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present. Then to prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin.

When several blocks tap power from a single source, there is a problem of Voltage Droop at the Vdd and Ground Bounce at the Vss which can again push the logic out of the required noise margin into the undefined state. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source.

### 3. Placement
There are two types of placement. The other required logic is placed optimally. Placement is of two steps

Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
Detialed Placement - After Global placement is done minimal alterations are done to correct the issues
### 4. Clock Tree Synthesis
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow. This is done by TritonCTS.

### 5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges

OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode OpenLane allows the user to chose either of the above approaches
### 6. Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

Global Routing - This is done inside the OpenROAD flow (FastRoute)
Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing. Before performing this step the Logic Equivalence Check is performed by Yosys, since OpenROAD does some optimisations the circuit.
### 7. RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

### 8. STA
At this stage again OpenSTA is used to perform the Static Timing Analysis.

### 9. Sign-off Steps
Design Rule Check (DRC) is performed by Magic
Layout Versus Schematic (LVS) is performed by Netgen
### 10. GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file.
