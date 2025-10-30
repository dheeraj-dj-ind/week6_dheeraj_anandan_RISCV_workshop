In this document we will go through SPICE simulation of a inverter cell whose SPICE deck is obtained from the layout of an inverter on the opensource tool `magic`. We open a custom inverter design in magic and extract the post-layout SPICE deck for SPICE simulation. The document also goes through simple DRC violations that are fixed by updating the `sky130A.tech` file. 

# Custom Inverter Layout
Inorder to open the custom inverter layout on magic, we must clone the github repo given below

```bash 
cd Desktop/work/tools/openlane_working_dir/openlane
git clone https://github.com/nickson-jose/vsdstdcelldesign
cd vsdstdcelldesign
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .
ls -ltr
```

![VSDLIB](images/vsdlib.png)

Run the command below to open the inverter layout in magic 

```bash
magic -T sky130A.tech sky130_inv.mag &
```

![Inverter](images/inverter.png)

## Identification of NMOS and PMOS

To indentify NMOS and PMOS in the layout, simply hover over the white box shown in the image and press 's'. On the `tkcon 2.3 Main` file you can see it's description

![NMOS](images/nmos.png)
![PMOS](images/pmos.png)

## Indentifying Correct Connections
In order to check the proper connectivity of the drain terminals of the transistor with the output port, the source terminal of PMOS with VDD and source terminal of NMOS with GND, simply hover over the terminals and press 's' two times and you will get the desired output as shown 

- Output Port Connectivity
![output](images/output.png)

- PMOS Source with VDD
![PMOS SRC](images/pmos%20src.png)

- NMOS Source with GND
![NMOS SRC](images/nmos%20src.png)

## Deleting Parts to View DRC
In magic, with a predefined layout, we can delete parts to check the DRC violations as shown below:

![DRC Violation](images/drc%20vio.png)

## SPICE Extraction from Layout

We can extract the SPICE deck/netlist from the layout by following these commands. Make sure to type them in the `tkon 2.3 Main` window.

```bash
pwd
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```

- SPICE Extraction in tkon
![SPICE Extraction](images/ext%20and%20spice%20con.png)

The .spice file obtained is stored in the directory as shown below

![Spice directory](images/spice%20directory.png)

The obtained netlist is shown below

![Netlist](images/netlist.png)

## SPICE Simulation
Inorder to run the SPICE simulation, we update the obtained inverter netlist, and add:
- Update the scale option according to the size of a `root box` which is `0.01u`
- Include library files `pshort.lib` and `nshort.lib`
- Add `DC supply` voltage of 3.3V and `Pulsating supply` of 3.3V
- Update the model names of trasistors to `pshort_model.0` and `nshort_model.0`
- Capacitance - `C6` to smoothen out sharp edges. The update netlist is shown below:
- Add command to perform `transient analysis`.
![Root Box](images/box%20cell%20unit.png)
![Updated netlist](images/updated%20spice.png)

After saving the file, we go back to the terminal and perform SPICE simulation in `ngspice`. Followi these commands:

```bash
ngspice sky130A_inv.spice # enters into ngspice environment
plot y vs time a # Performs transient analysis
```
![ngspice](images/ngspice.png)

Upon running the command, we obtain the following result:
![transient analysis](images/tran%20ana.png)

## Characterization of Inverter
From the transient analysis output, we can characterize the cell by finding it's:
- Rise Transistion Time
- Fall Transistion Time
- Rise Delay 
- Fall Delay 

Let's look at each of them one by one.

### Rise Transition Time
It is defined as the difference between output rise time at 80% voltage and 20% voltage. 
- 80% Vin = 2.64V   Time at 2.64V = 2.24669 ns
- 20% Vin = 0.66V   Time at 0.66V = 2.18234 ns

Therefore, Rise Transistion Time = 2.24669 - 2.18234 = 0.28456 ns

- Rise Transition at 20%
![Rise 20%](images/rise%2020%25.png)

- Rise Transition at 80%
![Rise 80%](images/rise%2080%25.png)

### Fall Transision Time
It is defined as the difference between output rise time at 80% voltage and 20% voltage. 
- 80% Vin = 2.64V   Time at 2.64V = 4.05318 ns
- 20% Vin = 0.66V   Time at 0.66V = 4.09554 ns

Therefore, Fall Transistion Time = 4.09554 - 4.05318 = 0.04236 ns

- Fall Transition at 80%
![Fall 80%](images/fall%2080%25.png)

- Fall Transistion at 20%
![Fall 20%](images/fall%2020%25.png)

### Rise Propogation Delay 
It is defined as the the difference in timeing values at 50% voltage between output and input during the positive edge. At 50% Vin which is equal to 1.65V 
- Input Signal Time at 1.65V = 2.14989 ns
- Outpu Signal Time at 1.65V = 2.21121 ns

Therefore, the Rise Propogation Delay = 2.21121 - 2.14989 = 0.06132 ns

- Delay at 50% Vin
![Rise Prop](images/rise%20prop%20delay.png)

### Fall Propogation Delay
It is defined as the the difference in timeing values at 50% voltage between input and output during the negative edge. At 50% Vin which is equal to 1.65V 
- Input Signal Time at 1.65V = 4.0779 ns
- Outpu Signal Time at 1.65V = 4.04986 ns

Therefore, the Rise Propogation Delay = 4.0779 - 4.04986 = 0.02804 ns

The values obtained for the calculations are shown below
![Calc Values](images/timing%20values.png)

# Magic Examples
This section of tasks describes how to install the examples of layouts to check for DRC violations along with changed made to correct the DRC violations. 

To get a good understanding of the Magic tool follow this link
[link](http://opencircuitdesign.com/magic/index.html)


Command to install the examples is as follows:

```bash
cd
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz
tar xfz drc_tests.tgz
cd drc_tests
ls -al
gvim .magicrc
magic -d XR &
```

- Installing the tar-ball package
![Installing the package](images/install%20examples.png)

- Magicrc file 
![Magic rc file](images/magicrc.png)

- List of examples downloaded
![Examples](images/examples.png)

- Command to open Magic tool
![Command](images/opening%20magic.png)

## Incorrectly Implemented poly.9

- Poly Rules 
![Poly Rules](images/poly%20rules.png)

For this we load the `poly.mag` file as shown below
![Loding poly.mag](images/poly_mag%20load.png)
![Polymag file](images/poly_mag.png)

The DRC violation for the zoomed in images is shown below
![Violation](images/poly_9_vio.png)

### Lines Added in sky130A.tech file
Inorder for these viloations to be fixed we add/update a few lines inside the sky130A.tech file.
- Line updation 1
![Line update 1](images/added%20command1.png)

- Line Updation 2
![Line Update 2](images/added%20command%202.png)

Upon saving the `sky130A.tech` file, we can simply use the command given below so that it can be used in the existing magic session in the `tkon` window. 

```bash
tech load sky130A.tech
drc check
drc why
```

### Additional Assignment
![Additional Assignment fail](images/extended%20assignment.png)
![Additional Assignment pass](images/extended%20poly9%20success.png)

# Incorrectly Implemented nwell.4 rules
- nwell rules
![nwell](images/nwell%20rules.png)

No DRC violation even though no tap present 
![drc vio1](images/nwell%20assignment%202.png)

We update the `sky130A.tech` file to fix the DRC violation

- Updating sky130A.tech file
![Update sky130A.tech file](images/updating%20skytech%20for%20assignment%202.png)
![Update sky130A.tech file](images/updating%20skytech%20for%20assignment%202%202.png)
![Update sky130A.tech file](images/updating%20skytech%20for%20assigment%202%203.png)

To include updated .tech file use the commads given below in the tkon window

```bash
tech load sky130A.tech
drc check
drc why
```
- Output of DRC why
![Nwell DRC](images/assignment%202%20fail.png)

- Placing TAP and checking DRC
![Nwell tap DRC](images/assignment%202%20pass.png)
