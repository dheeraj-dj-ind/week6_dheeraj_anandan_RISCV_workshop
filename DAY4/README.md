This document will give you a description of verifying the conditions for the layout of the inverter previously dicussed. It discusses on gnerating a `.lef` file from a custom saved inverter layout file and wokring on it. We insert the extracted `.lef` file into `/picorv32a/src/` directory and run a new `OpenLane` flow. Also includes detailed STA analysis of the `picorv32a` with inclusion of the custom `vsdinv` cell.The document shows working on reducing the slack by modifying the cells that have large slack and creating a new verilog netlist to run a new OpenLane flow. It works around errors finding alternate solutions and running CTS using TritonCTS for posrt-openroad CTS and also compare values for slack values by removing a buffer type and not removing it. 

# Veryfying DRC Rule 
To verify the DRC rule for heigh and width of a cell we have to modying the grid size. To modify it we take a view at the `tracks.info` file for lenght and pitch values. 

![Track.info](images/tracks%20info.png)

In order to activate the grid, you simlpy select the cell and press 'g'.

From there we take the `li1` x and y values and insert into the grid function in the `tkon` window. The image below shows how the grid looks before updating the grid

![grid](images/changing%20grid%20values,%20before.png)

After updating the grid values, the grid looks like the below image

![grid after](images/changing%20grid%20values,%20after.png)

## Now to verify the rules for size of the cell, we count the number of boxes that is spans. 
- For width, the value should me an odd multiple of the cell length. 

![Heigth](images/verifying%20width%20of%20cell.png)

It can be seen that the width is spanning 3 boxes, which confirms that is follows the rule. 

- For height, it should be an even multiple of the cell length 

![Width](images/verifying%20width%20of%20cell.png)

It can be seen that the height is spanning 8 boxes, which confirms the rule. 

# Creating the Custom Inverter Cell 

In the `sky130_inv.mag` file, save it at `sky130_vsdinv.mag` by entering the below command in the tkon window

![Extracting custom file](images/changing%20name%20of%20file.png)

Next, close the `sky130_inv.mag` file and open `sky130_vsdinv.mag` by entering the command in the terminal

```bash 
magic -T sky130A.tech sky130_vsdinv.mag
```

Now generate the `.lef` file from the opened magic inverter layout

![Extract lef](images/creating%20lef%20file.png)

The extracted `sky130_vsdinv.lef` file is shown below 

![Lef file](images/lef%20file.png)

Now we copy the `.lef` file created into the `/picorv32a/src` directory along with the library files. 

```bash 
cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```
![copied files](images/copying%20lef%20file%20to%20src%20dir.png)
![Coppied files 2](images/copying%20library%20files%20to%20src%20dir.png)

# Editing TCL file 
We edit the `config.tcl` inside the `/picorv32a/` folder by adding these lines 

```bash 
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```
# Running the OpenLane flow
```bash 
cd Desktop/work/tools/openlane_working_dir/openlane
docker
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
![Running OpenLane](images/overwriting%20old%20openlane%20output.png)
![Syn Success](images/syn%20success.png)

- Chip area before synthesis stratagies
![Area](images/chip%20area.png)

# Improving Timing 

To improve timing run the below commands
```bash 
prep -design picorv32a -tag 28-10_03-59 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
echo $::env(SYNTH_STRATEGY)
set ::env(SYNTH_STRATEGY) "DELAY 3"
echo $::env(SYNTH_BUFFERING)
echo $::env(SYNTH_SIZING)
set ::env(SYNTH_SIZING) 1
echo $::env(SYNTH_DRIVING_CELL)
run_synthesis
```
## Included Macro in merge.lef file
- Merged Lef file
![Merged Lef](images/checking%20merged%20lef%20for%20vsdinv.png)

- Running commands to reduce slack
![Running again](images/running%20again%20for%20syn%20commands.png)
![Run Synth Commands](images/syn%20commands.png)

```bash 
run_floorplan
```
- Running floorplan
![Synth Cmmand Succes](images/running%20floorplan.png)

```bash
run_placement
```
- Running placement
![Running placement](images/placment%20success.png)

- Chip area after synthesis stratagies
![area](images/area%20after%20syn%20commands.png)

## Loading placement on Magic
```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

- Finding the custom `sky130_vsdinv` cell
![vsdinv placement](images/verifying%20for%20vsdinv.png)


Type the following command after selecting the custom vsdinv cell in tkon window and you will get a visualization of the metal connections.
```bash
expand
```
![vsdinv](images/expanded%20vsdinv.png)

# Post-synthesis Time Analysis

The design after using the `SYNTH_STRATEGIES` gave `tns = 0` and `wns = 0` which gives a completley optimistic design. In this section we are going to try and reduce slack using OpenSTA by modifying cells. For this, entering into a new OpenLane flow is preferable. Run the same commands as before. 

```bash 
cd Desktop/work/tools/openlane_working_dir/openlane
docker
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
```
Now create a `pre_sta.conf` file containing the following 

![pre_sta](images/pre_sta%20conf.png)

Next create a `my_base.sdc` file in `/openlane/designs/picorv32a/src/` directory, based on the file `/openlane/scripts/base.sdc`
![my_base](images/my_base%20sdc.png)

## Running the pre_sta.conf file

Run the STA configure file by following the commands

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
sta pre_sta.conf
```
![sta run](images/pre_sta_run.png)
![Presynth](images/presynth%20op.png)

## Run OpenLane by minimizing FANOUT

To reduce the fanout of each cell, we adopt another `SYNTH_STRATEGY` by following the commands

```bash 
prep -design picorv32a -tag 29-10_09-04 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
set ::env(SYNTH_SIZING) 1
# Setting new value for Fanout
set ::env(SYNTH_MAX_FANOUT) 4
echo $::env(SYNTH_DRIVING_CELL)
run_synthesis
```
![Fanout](images/setting%20max%20fan%20out.png)
![Fanout success](images/synth%20confirm%20with%20max%20fan%20out.png)

## Run OpenSTA again 
Run OpenSTA again on the new fanout design 
```bash
cd Desktop/work/tools/openlane_working_dir/openlane
sta pre_sta.conf
```
![sta run on fanout](images/running%20res_synth.png)
![fanout sta](images/fanout%20sta.png)

### Timing ECO to Minimize Slack
- OR gate of drive strenght 2 is driving 4 cells 
![delay1](images/delay1.png)

To analyze the net and modify the cells follow these commands
```bash 
report_net -connections _11675_
help replace_cell
replace_cell _14514_ sky130_fd_sc_hd__or3_4
report_checks -fields {net cap slew input_pins} -digits 4
```
![Changing pin](images/changing%20pin1.png)

Change in slack observed is 
![Change in delay](images/slack%20dec%201.png)

Similarly, perform the same for other cells with high fanouts and delays
![delay2](images/delay2.png)
```bash 
report_net -connections _11672_
help replace_cell
replace_cell _14510_ sky130_fd_sc_hd__or3_4
report_checks -fields {net cap slew input_pins} -digits 4
```
![change in pin](images/changin%20pin%202.png)

Change in slack observed is
![change in delay](images/slack%20dec%202.png)

![delay3](images/delay3.png)
```bash 
report_net -connections _11643_
help replace_cell
replace_cell _14481_ sky130_fd_sc_hd__or3_4
report_checks -fields {net cap slew input_pins} -digits 4
```
![change in pin](images/slack%20dec3.png)

Change in slack observed is
![change in delay](images/changing%20pin%203.png)

![delay2](images/delay5.png)
```bash 
report_net -connections _11668_
help replace_cell
replace_cell _14506_ sky130_fd_sc_hd__or3_4
report_checks -fields {net cap slew input_pins} -digits 4
```
![change in pin](images/changing%20pin5.png)

Change in slack observed is
![change in delay](images/slackdec5.png)

## Finding how much slack has been reduced
Inorder to figure out how much slack have we reduced from the initial slack value by modifying cells, run this command 
```bash
report_checks -from _29043_ -to _30440_ -through _14506_
```

![Through command](images/through%20command.png)
![Through command Output](images/final%20value%20of%20slack.png)

Clearly it can be observed that the initial value of slack was -23.89 and it has been reduced to -22.27. This shows a significant improvment in slack by 1.62ns. 

# Replacing the Old Netlist

After running the ECO timing checks, we replace the old netlist containing the higher slack value, by creating a new netlist with the lower slack value by following these commands in the OpenSTA environment. 

```bash
help write_verilog
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/picorv32a.synthesis.v
exit
```

//## Verifying if netlist has been updated
In the below image it can be seen that, the cell 
![Confirm overwriting](images/confirming%20overwriting.png)

