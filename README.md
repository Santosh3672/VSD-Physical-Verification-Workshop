![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Workshop-Flyer.jpeg)
# VSD Physical verification using Sky 130 workshop Aug 11-16

## Contents
### Day 1:
#### Introduction
#### Skywater pdk layers
#### Skywater devices
### Day 2
#### Introduction
#### GDS format
#### Extraction
#### DRC (Design rule check)
#### LVS
### Day 3
#### Silicon Manufacturing Process
#### Design rules
#### Frontend rule
#### Wells and taps
#### Deep nwell
#### Device Rules
##### Resistors
##### Capacitor types
##### Miscellaneous DRC rule
#### Density rule
#### Recommended rules
#### ERC (electrical rule check)
#### DRC Labs
### Day 4
#### RTL to GDSII flow
### Day 5
#### Introduction:
#### Netgen
##### Netgen core matching algorithm
##### Netgen prematch analysis
#### Labs

## Day 1:
### Introduction
Skywater 130 : Skywater is name of foundry and it is based on 130nm technology.

PDK (Process design kit) : Bundles of files and documentation needed by chip designer to work with process.

Open\_pdk: Open source EDA tools and libraries. Some of the open source tools in open\_pdk are:

1. Magic: Layout tool used for DRC, extraction, etc
2. Klayout: Used for DRC.
3. Openlane: STA and synthesis tool
4. Xschem: Schematic editor tool
5. Netgen: LVS tool
6. Ngspice: Analog simulation tool

Open\_pdk also adds foundry and 3rd party libraries.

### Skywater pdk layers

There are two layers:

1. Frontend layer: Contains transitror, diffusion and local interface(LI).
2. Backend layer: Contains metal layer.

MiM cap layer: MiM (Metal insulator Metal) is insertion of metal layer between two metal layers with isulator in between them acting as capacitor.

Redistribution metal layer: A copper layer place above the highest metal layer that is used for solder lamp. Not a part of design hence a separate gds is required for this.

### Skywater devices

Contains basic devices like:

i. NPN or PNP transitor
ii. MOSfet (p and n channel)
iii. Resistor
  a. Pollysilicon
  b. Diffusion
  c. Pwell
iv. Reference layout
v. Hidden mask layer

**Skywater libraries**

Library cells are of 3 types:

i. Digital std cell:

Contains different types of cells like low power, low leakage, high speed, etc.

Naming of library: sky130\_vendor\_library-type[-name]

Cellname : libraryname\_\_gatename

ii. I/O cells
iii. Primitive devices:

Contains elements with very specific layout normally provided by foundry. Eg, RF resistor (carefully prepared to avoid noise and signal attenuation), ESD protection devices, parallel plate capacitance.

iv. 3rd party libraries: Like SRAM and NVRAM

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D1_analog_sim_console.png)

Above is the console snippet for analog simulation of an inverter. Extract netlist was used for simulation.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D1_analog_sim_plot.png)

Result of above simulation to verify that the inverter runs as expected in presence of parasitic capacitances as well.

## Day 2

### Introduction

Physical Verification has two parts

DRC (Design rule check): To check if the layout satisfies foundry rules for masks.

LVS (layout vs schematic): To verify that the layout matches a simulation netlist by electrical connectivity.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_1%20PD%20flow%20with%20DRC%20%26%20LVS.png)

Physical design flow in with DRC and LVS

### GDS format

GDS (graphic design system) is a stream format which stores the information about layout. It is stored in ASCII format.

It is stored in format of layer:purpose as integers which are specific to process. Due to this it is hard to understand without a layout tool

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_2%20Data%20in%20GDS%20file.png)

Data in GDS file

Other data like wires and power supply can be found in LEF and DEF files.

Gds read \&lt;file\_location\&gt; is the command to read it in magic

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_3%20and%20gate%20gds%20in%20magic.png)

Two input and gate gds file read in magic

Since it has no port related information we can import lef file after gds file is read it will not overwrite the cells but annotate extra metadata.

Similarly, port order can be annotated from spice file. (though these are not required for a layout)

When we read only lef file without gds file, only place and route details like wire and pins will be seen without transistor present.

### Extraction

It is a process where parasitic information is &quot;extracted&quot; from the layout (as it contains metal layers and spacing information) and then it is converted to a netlist where these parasites (R, L, C or K) are incorporated in the.

Extraction in magic:

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_4%20Extraction%20flow%20magic.png)

Extraction flow in magic

For R extraction extresist is also required to be run.

extract commands:

extract do local

extract all : creates .ext file

ext2spice lvs : sets sane option

ext2spice cthresh \&lt;value\&gt; : suppres capacitance below specified value

ext2spice : creates .spice file.

### DRC (Design rule check)

Magic has 3 types of DRC setting which can be set in the console:

1. Full : Checks complete design (slowest)
2. Fast : Checks near the region where changes are done (fast)
3. routing : Checks metal (fastest)

For large design full DRC after every small changes will take a too much time, fast can be used.

DRC rules:

1. Edge based rules: checks edges of elements (fast method)

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_5%20edge%20based%20DRC%20rule.png)

1. Boolean geometry rules: performs Boolean operator on the layout elements.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_6%20boolean%20based%20DRC%20rule.png)

XOR operator is used to find difference between two version of same the layout. It is used to check if only the intended portion of design is changed or if any cell is misplaced by mistake.

To do XOR operation in magic:

1. Flatten the current modified layout and save it in a new file (say newfile)
2. Load the layout with which the current layout needs to be compared
3. Run xor command with the newfile as argument
4. To view the result load the newfile

In the lab I intentionally misplaced an and cell in both horizontal and vertical direction. The XOR produces following result.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_7%20XOR%20operation%20in%20magic.png)

### LVS

It compares the spice(netlist) files of layout and simulation to check if the two netlists are equivalent in terms of electrical connectivity.

Layout tools can&#39;t perform LVS and LVS tools don&#39;t know how to handle layout files. So we use spice tools produced from extract for LVS.

For LVS it is better if the two spice files have same hierarchy.

If they have different hierarchy LVS tool will flatten them and compare.

Netgen tool is used to run LVS following command is used

netgen -batch lvs &quot;layout.spice\_loc designsubckt&quot; &quot;reference.spice\_location refsubckt&quot;

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D2_8%20C%20extracted%20LVS%20result.png)

Snippet of LVS run between C extracted layout and library cell spice model

## Day 3

### Silicon Manufacturing Process

It si important to understand manufacturing process as it will help us understand about DRC rules.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_1_planar_process.png)

In planar process the materials are deposited in top of or implanted into a substrate (silicon wafer)

Transistors and resistors, cap and diode are drawn to wafer by combination of light, photo resistive materials and masks.

Masks are high resolution elements, covers some area and exposes some area so that they can be doped, or removed or different task can be done on that and chip can be made.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_2_Opical.png)

On exposure to light photoresist is broken down and removed then doping can be done.

Percentage yield: Percentage of chips manufactured in foundry that works correctly.

Let us discuss about a design rule of spacing: if spacing is more then there is no issue but if it is reduced further then there is a chance of short circuit.

So, the spacing rules are decided where the probability of failure is constant 1ppm if we go below that then space of chip will increase.

If we violate it then foundry will not accept this as their yield will be reduced.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_3_Failure_rate_DRC.png)

### Design rules

Width rule: If wire width too small then chance of open circuit. Mostly wires are routed at minimum width (may have high resistance). nwell width should also be above a limit.

Spacing: If spacing too small then high chances of short circuit. Even if 2 wires run parallel for some minimum distance then they should have more spacing (run length rule) it is not present in skywater130.

Wide-spacing rule: If a wire is wider than a specific amount then other wire of any width should keep a larger distance (as they carry more current).

Other related rules are: Notch rule, Minimum and maximum metal area rule, minimum hole area rules, contact cut (via) rules, etc.

Magic is designer friendly tool it hides some details like contact cuts inside a via to the user as internally it is satisfying design rules off skywater. If magic is showing no DRC violations, then it is fine for skywater.

Local interconnect rule:

Local interconnect is placed in between polysilicon and metal layer and had intermediate resistance 100 times more than metal.

So it is only used for local connection (like within a cell) hence it has rule that aspect ratio should be less than 10 (more like a design guideline to avoid IR drop)

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_4_LI.png)

### Frontend rule

Rules related to process devices like transistor, resistor, etc.

Pdk generates devices which satisfies all these rules.

In magic poly layer and ndiff layers are different when they are put one above another a devife is formed.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_5_Polydiff.png)

Some of the frontend rules are:

Minimum width (both poly and diff)

Poly must extend the diff by a certain minimum length to account for misalignment of diff layer.

Poly and diff of different gates should have min. spacing.

Different version of gates (low vt, high vt, etc) have different symbols and have different rules.

The gates have no drc check as they are imported from libraries, if we create without these libraries then there will be error.

### Wells and taps

Normally a well is already present, we add a diffusion layer and poly above it above which a metal is added as gate, in diff only we have source and drain. Say n well and pdiff so here we add a n type diff called ntap which is electrically connected to nwell and hence it is provided voltage to reverse bias the mos and reduce leakage current.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_6_tapcell.png)

There should be spacing between ntap and pdiff layer. Which is also included in sets of rules.

### Deep nwell

In normal CMOS the nfet is connected to the p-substrate and is affected by noise from all different gates, thought nwell is isolated from it (due to the Vdd and Gnd provided causing reverse-bias).

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_7_CMOS.png)

With a deep nwell as shown below the noise is avoided.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_8_CMOS_nwell.png)

Now a deep nwell is used which is sandwitched by p-substrate and p-well, ofcourse with nwell present. Here both nfet and pfet are free from noise.

Some of its design rules:

- Min width of deep nwell.
- There should be a fence that should overlap nwell with a min width.
- Huge min spacing.

### Device Rules

#### Resistors

p-diff resistor: not used as they have pn junction and leaks current

polyresistors: these are used (XHR extra high resistor 350ohm/sq, UHR ultra 2000 ohm/sq)

#### Capacitor types

1. Varactors: it is like mosfet but with small differences (rules similar to mosfet)
2. MOScap: mosfet with wiring changes (rules similar to mosfet)
3. Vertical parallel plate: also called MoM metal oxide metal DRC rules same as metal layer
4. MiM : metals between metal layers (have rules that are helpful to get desired capacitance)

#### Miscellaneous DRC rule

- Off grid error (cells should be on grid point)
- Angle limitation
- Seal ring rules

Latchup rules: Due to Latchup effect (shorts device and can&#39;t be stopped till power is removed) we add tap cells to avoid it.

There are rules for distance between ndiff and ptap.

Antenna rules: applies to large wires with no connection in between, they can accumulate charge and and can have large voltage developed and breakdown can occur.

To fix it a diode(parasitic) can be placed

Other fix is to change metal layers.

Stress rule: metal delamination, causes metal cracking due to mechanical stress, to avoid it metals are slotted in direction of current.

### Density rule

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_9_uneven_metal.png)

Oxide layer forms on metal layer during fabrication process.

They are polished, we can see bumps there due to uneven metal, the bumps should be very less hence there is a metal density rule to avoid it.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_10_dense_metal.png)

### Recommended rules

These rules help to increase yield and robustness. Eg. using redundant vias.

Test chip and production chip: Test chip can have high yield loss, but production can&#39;t.

Manufacturing rules: violation cause immediate rejection by foundry.

Test chip needs to obey at least the manufacturing rules.

All rules discussed are included in manufacturing rules but latchup rule , antenna and recommended rules can be ignored with some risk.

### ERC (electrical rule check)

These are condition due to which circuit may get damaged due to electrical property.

Eg Electromigration, Overvoltage conditions.

### DRC Labs

Source the script &quot;./run\_magic&quot; to open magic with the current process

Magic shortcuts to zoom:

&#39;z&#39; to zoom in &#39;shift + z&#39; to zoom out &#39;ctr + z&#39; to zoom to area selected &#39;v&#39; to go to normal view mode

To fix drc violations.

Width violation: select the wire then see why it is violating using drc report then get required length then extend it using middle mouse click

For spacing select a module and move it with 2,4,6,8 keys, else we can move them using &quot;move e/w/n/s length&quot; command.

To avoid notch rule select the area and shift it by selecting shift+a and 2468 keys.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_11_spacingLab.png)

Drc fixed snapshot Exercise 1

Vias

Vias can also be fixed if its size is not matching, we can resize them like notch.

We can see contact cut by using cif see MCON command (MCON is the element it is to be found from details)

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_12_VIALab.png)

Exercise 2 Fixed (Via DRC)

Area and hole

Minimum area error occurs when we move from li to m2 where only a small m1 is used hence less error.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_13_AreaLab.png)

DRC area error fixed while moving from li to m2 through m1

Wells and taps:

The DRC style should be full, else DRC violation won&#39;t be reported.

Nwell require ntap, to add ntap we can add nsubstrateendiff above an nwell. But still error will be there as it needs to be connected to ground which is done during ERC, so foundry checks for contact to this and assumes that a gnd or pwr is connected to it add nsubstrateencontace.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_14_well_tap_lab.png)

DRC wells and tap fixes

Angle error are mostly due to off grid error. We can simply remove the angled poly to avoid them.

Seal ring: (abstract view)

It is placed in boundary of chip to protect it from external things, like sawing and dusting.

Without adding ring cell we can have hundreds of error.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_15_latchup_lab.png)

Latchup rule fixed by adding tap cell

Antenna check

As it is an ERC we need to do extraction before it then antennacheck command is done

It can be due to high aspect ratio of wire so to avoid it we connect it to diode.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_16_antenna_lab.png)

Antenna violation fixed

Desnity rule

For that we can have under density or overdensity, we are using a python scripts to take care of this.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_17_densiy_lab.png)

Density rule fixed

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D3_18_density_rep.png)

Metal1 density increased from 5% to 58.1% after using the python script

## Day 4
### RTL to GDSII flow
Open lane flow:
 It is a flow to convert RTL to GDSII using RTL file and PDK files as input.

Steps in openlane flow are mentioned below:

**Synthesis**

Here the RTL is converted to Gate level representation

Openlane uses two synthesis tool.

Yosys – for RTL synthesis

ABC – for technology mapping

**STA:**

OpenSTA is used it performs STA on resulting netlist for timing report generation. It uses ideal clocks.

**Floorplanning:**

This step includes:

- Defining core area for macro and rows and tracks
- Placement of input and output
- Welltap and decap cell insertion
- Power distribution network generation

**Placement**

In this process following things are performed

- Global placement
- Various optimization on the design
- Detailed placement

**Clock Tree synthesis**

Uses TritonCTS to create clock distribution network and uses real clock tree after this STA is performed again.

**Global routing**

Uses FastRoute tool and performs fill insertion

**Antenna diode insertion**

It is done to avoid antenna DRC violaion.

**Detailed Routing**

Using tritonRoute tool and creates DEF file.

**Parasitic (RC) extraction**

Done on magic tool following which STA is performed again.

**Physical Verification**

**Tools used are:**

Magic – Performs DRCcheck and antenna check

Klayout – Performs DRC check

Netgen – Performs LVS check

CVC – Performs circuit validity check

GDSII is finally streamed out by magic and klayout. Klayout GDSII streamed from def file and used as a backup.

We can run the openlane flow by two ways.

1. Non interactive method: Here a predefined script will run and will perform all the above-mentioned steps where we cant change anything interactively.
2. Interactive method: Here each steps are run one-by-one and we can modify each steps and change parameters based on previous steps results.

## Day 5

### Introduction:

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_1_PV_flow.png)

Physical verification flow

Unlike DRC, LVS wont be run in foundry, hence it can cause severe damage.

Knowing if it is matching or not is easy but if they don&#39;t match then it is difficult to figure out why.

LVS has two parts layout and netlist.

Netlist can be: spice, LEF/DEF, Verilog,etc

An additional feature of netlist is it should be able to simulate.

For LVS we need to compare two netlists. We need to prepare them.

One is netlist from layout, other netlist depends on our flow. Some of them are mentioned below:

- Traditional LVS: with a schematic which was created as a start of project.
- Alternate LVS (for digital cicruit): RTL can&#39;t be used as it is behavioral description. So we use Syn netlist.

For LVS it is better if the two netlists have same hierarchy. But layout can have more hierarchy deep down since they add some extra things. In that case LVS flattens them and compare.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_2_diffpiname.png)

Cases where LVS fails: it is assumed VPB and VPWR are same but tool don&#39;t understand it.

Simulation netlist vs LVS netlist

Simulation netlist is used to extract parasitic for more accurate delay calculations. But they are added as components (R,C) hence shouldn&#39;t be used for LVS. As it will affect matching process.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_3_layout_resistance.png)

### Netgen

Tcl based interpreter.

End user only need to use lvs command.

Command is netgen -batch lvs &quot;file1 circuit1&quot; &quot;file2 circuit2&quot; setup\_file output\_file

It is advised to keep design and testbench in separate file as we don&#39;t need to pass tb.

Netgen works with time complexity of O(logN) so easily scalable.

#### Netgen core matching algorithm

The algo runs on iteration. On first iteration it creates list of devices and nets. And it combines both netlist and it keeps tag of which net or device belongs to which netlist.

Then it does hashing where it gives identifier number to devices (matching ones have same number).

Then it gives hash numbers to nets connected to same pins (pins also have hash like their device). So each nets are numbered.

Then it again given a new hash number to the devices based on which net its port is connected to.

This is done in iterations till each hash number has exactly two devices (or the number of partitions i.e. hash number is equal to number of devices).

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_4_partition_algo.png)

Partitions modified with iterations

If they don&#39;t match by the last iteration then these errors will be dumped by tool.

#### Netgen prematch analysis

Some elements like wrapper adds another hierarchy to layout. Netgen counts number of elements and handles them.

Since hierarchical LVS flattens the design if they don&#39;t match. Error in low level hierarchy will consume a lot of time. To avoid it we can fix the low level mismatch first or specify tool not to treat specific cell for flattening.

During pin checking the name of pins are irrelevant but number of pins should be equal (special case when extra pin is not connected anywhere then it can be excluded, here the tool can add a new pin whose name starts with proxy (proxyC))

Finally LVS also performs properties of device like W,L etc.

Fingers of transistor: Width may be very large so it is divided into different fingers to reduce width by number of finger times. But they are equivalent and should be understood by tool.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_5_dev_merge.png)

Similar devices that should be understood by LVS tool

Similarly netgen should also merges series/parallel devices like Resistor, capacitor, etc. So that it wont flag similar elements as not matching.

The partition theory explained earlier might fail if there are symmetrical components in the design like 74LS00 nand chip with 4 nand gates each connected to ports.

- Rule of thumb for LVS :
 Analyze device mismatch first after solving them only go for nets mismatch.
- Solve easy to understand error first then others (like LEC)

### Labs

Lab files are cloned from &quot;[https://github.com/rtimothyedwards/vsd\_lvs\_lab.git](https://github.com/rtimothyedwards/vsd_lvs_lab.git)&quot;

If two netlists have undefined cells but are defined in the same way, then netgen treats them as similar.

After running LVS the rep file is dumped (default name comp.log) check it for net and device mismatch. Device partitions is also reported.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_6_Lab1.png)

In netgen when we read new design the tool will have memory of previous design so we either must close and open it again or use **reinitialize** command.

If we declare a design without definition like before in a subcircuit it will say no device found as subcircuit should have definition. But if we provide subckt name it won&#39;t be a mismatch.

If we change pin order or swap pins then netgen won&#39;t have a problem if pins are consistent but for top level module it will be a problem as it can cause severe problems in future.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_7_lab_mismatch.png)

On swapping pin A and B netlists match but netgen shows that pin matching fails.

If we define a subckt but the definition is empty then netgen treats that as black box. It is done to specify pin order correctly.

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_8_cellrename.png)

LVS run with one cell1 renamed to cell4. Here cell4 and cell 1 are not matched but they remain in the random partition of unmatched cells hence shown as above.

To specify to netgen that we can swap pins (like a resistor or Capacitor). We are using permute command and specify cell and pin name (so that you don&#39;t permute other devices like diode). Like below,

![](https://github.com/Santosh3672/VSD-Physical-Verification-Workshop/blob/main/Pics/D5_9_permute_lab.png)

Report of using permute command to swap pins of a resistor.

If in one of netlist the library cells are not included then it will not know about the pin names and cause pin mismatch. So the libraries should be added.
