# Week6 Tasks
This week we will go through OpenLane that comprises the RTL2GDSII flow for a design `picorv32a`. Each day's task description is mentioned that will help you navigate clearly. The `logs` for each day are also present 

## Day1 : 
Get a thorough understanding of the basics of what a package is, it's contents,and a deeper dive into the RTL2GDSII flow using open source tools. You will also understand what do you mean by Instruction Set Algorithm (ISA), how a sofware runs on the hardware and carrying out sysnthesis of picorv32a. We also calculate the flop ratio and flop percentage of the design. 

## Day2 :
You will get to see the floorplan of the picorv32a design using OpenLane flow and generate necessary outputs. We will calculate the area in microns from the floorplan def and visualize the floorplan using the opensource tools `magic`. The placement process for the picorv32a design is done and necessary outputs are generated. We will visualize the congestion aware placement of standard cells using magic again.

## Day3 :
You will go through SPICE simulation of a inverter cell whose SPICE deck is obtained from the layout of an inverter on the opensource tool `magic`. We open a custom inverter design in magic and extract the post-layout SPICE deck for SPICE simulation. The document also goes through simple DRC violations that are fixed by updating the `sky130A.tech` file.

## Day4 :
This document will give you a description of verifying the conditions for the layout of the inverter previously dicussed. It discusses on gnerating a `.lef` file from a custom saved inverter layout file and wokring on it. We insert the extracted .lef file into `/picorv32a/src/` directory and run a new OpenLane flow. Also includes detailed STA analysis of the `picorv32a` with inclusion of the custom `vsdinv` cell.The document shows working on reducing the slack by modifying the cells that have large slack and creating a new verilog netlist to run a new OpenLane flow. It works around errors finding alternate solutions and running CTS using TritonCTS for posrt-openroad CTS and also compare values for slack values by removing a buffer type and not removing it.

## Day5 :
In this document we will carry out the routing stage of PnR. It describes how how to incorporate the Power Network for the design then perform routing and visualise the final routed output on magic. It also talks about post-routing timing analysis to verify if the deisign meets timing requirements by utilizing design obtained from extracting parasitics.