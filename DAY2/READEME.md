In the document you will get to see the floorplan of the picorv32a design using OpenLane flow and generate necessary outputs. We will calculate the area in microns from the floorplan def and visualize the floorplan using the opensource tools `magic`. The placement process for the picorv32a design is done and necessary outputs are generated. We will visualize the congestion aware placement of standard cells using magic again.

# Floorplan Commands

Inorder to carry out the floorplan, we need to continue from the synthesis step in OpenLane flow

```bash
cd openlane
docker
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
```
