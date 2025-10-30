In this document we will carry out the routing stage of PnR. It describes how how to incorporate the `Power Network` for the design then perform routing and visualise the final routed output on magic. It also talks about post-routing timing analysis to verify if the deisign meets timing requirements by utilizing design obtained from extracting parasitics.

# Power Distribution Network 
To create a power distribution network, lets start from scratch by creating a new OpenLane flow. Follow the commands given below

```bash 
cd Desktop/work/tools/openlane_working_dir/openlane
docker
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1
run_synthesis
init_floorplan
place_io
tap_decap_or
run_placement
unset ::env(LIB_CTS) # Not required unless you get an error
run_cts
gen_pdn 
```
- Commands to generate PDN
![docker](images/Screenshot%20from%202025-10-30%2009-07-46.png)
![Commands 2](images/Screenshot%20from%202025-10-30%2009-08-00.png)
![Commands 3](images/Screenshot%20from%202025-10-30%2009-08-11.png)
![Commands 4](images/Screenshot%20from%202025-10-30%2009-08-40.png)
![Commands 5](images/Screenshot%20from%202025-10-30%2009-09-39.png)
![Commands 6](images/Screenshot%20from%202025-10-30%2009-10-50.png)
![Commands 7](images/Screenshot%20from%202025-10-30%2009-12-50.png)

To load the PDN on magic follow these commands
```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-10-03-34/tmp/floorplan/
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read 14-pdn.def &
```

- Generated Power Distribution Network
![PDN](images/Screenshot%20from%202025-10-30%2009-14-59.png)

- Zoomed in to view power and ground rails
![PWR and Gnd](images/Screenshot%20from%202025-10-30%2009-15-38.png)
![STD cells](images/Screenshot%20from%202025-10-30%2009-16-00.png)

# Routing 
After creating the power distribution network, our design is ready for routing. Follow the command given below 
```bash
run_routing
```
![Routing Run](images/Screenshot%20from%202025-10-30%2009-26-57.png)
![Routing Violations](images/Screenshot%20from%202025-10-30%2010-16-15.png)
![Routing Success](images/Screenshot%20from%202025-10-30%2010-15-55.png)

Once the routing is completed we can view it in magic by following these commands
```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/30-10-03-34/results/routing/
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &
```

- Routed Design
![Routed magic output](images/Screenshot%20from%202025-10-30%2010-20-24.png)

- Zommed in routed design to view metal contacts
![Metal Contacts](images/Screenshot%20from%202025-10-30%2010-21-23.png)

- Fast route guide 
![Fast route guide](images/Screenshot%20from%202025-10-30%2010-04-27.png)

In the updated version of OpenLane, extracting parsitics is included in the routing process so no need to run separate commands for it. 

- `.spef` File generation after routing
![spef file](images/Screenshot%20from%202025-10-30%2010-15-34.png)

# Post-route Timing Analysis
Once the routing of the `picorv32a` design is completed, we perform a timing analysis again in the OpenROad environment. To perform timing analysis carry out these steps

```bash 
openroad
read_lef /openLANE_flow/designs/picorv32a/runs/30-10_03-34/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/30-10_03-34/results/routing/picorv32a.def
write_db pico_route.db
read_db pico_route.db
read_verilog /openLANE_flow/designs/picorv32a/runs/30-10_03-34/results/synthesis/picorv32a.synthesis_preroute.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]
read_spef /openLANE_flow/designs/picorv32a/runs/30-10_03-34/results/routing/picorv32a.spef # Read SPEF
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
exit 
```

- Running OpenRoad
![Running Openroad](images/Screenshot%20from%202025-10-30%2010-28-22.png)

- Post-route Hold Slack
![hold slack](images/Screenshot%20from%202025-10-30%2010-28-33.png)

- Post-route Setup Slack
![Setup slack](images/Screenshot%20from%202025-10-30%2010-28-43.png)

