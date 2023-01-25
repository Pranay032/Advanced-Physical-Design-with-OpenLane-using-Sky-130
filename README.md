# Advanced Physical Design with OpenLane using Sky130
![09](https://user-images.githubusercontent.com/118599201/214614256-6f311a0a-0da4-4844-997d-4ab054307024.png)

# Physical Design with OpenLane using SKY130 PDK

This project is done in the course " https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/" by VLSI System Design Corporation. In this project a complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom desgined standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified

# Table of Contents
## Introduction
With the advent of open-source technologies for Chip development, there were several RTL designs, EDA Tools which were open-sourced. The missing piece in a complete Open source chip development was filled by the SKY130 PDK from Skywater Technologies and Google. There were several EDA Tools, which played specfic roles in the design cycle. There was not a clean design flow and Skywater pdk was compatible with only the industrty tools. OpenLane addressed these issues in providing a completely automated and clean RTL to GDSII flow. OpenLane is not a tool, but a flow which consists of several EDA tools, automation scripts and Skywater-pdks tuned to work specifically with the open-source EDA tools.
