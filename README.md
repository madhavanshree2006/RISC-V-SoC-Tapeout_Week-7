<h1 align="center">🟦 RISC-V SoC Tapeout Program — Week 7️⃣</h1>
<p align="center"><img src="./ASSETS/0.png" width="500" alt="OpenROAD banner"/></p>

---

<div align="center">

# 🚀 Week 7 — End-to-End RTL-to-GDS Flow with OpenROAD

🌟 This is **Week 7** of the **VSD RISC-V SoC Tapeout Program** —

I moved from **manual stage-wise physical design** to **full automation** using **OpenROAD-Flow-Scripts (ORFS)**.

I successfully set up the **ORFS environment**, integrated the **VSDBabySoC design**,

and executed **complete synthesis → floorplan → placement → CTS → routing → GDSII generation** flow.

</div>

---

OpenROAD-Flow-Scripts automates:

➡️ Logic synthesis (Yosys)

➡️ Floorplanning (TritonTools)

➡️ Placement & Global/Detailed Placement

➡️ Clock-Tree Synthesis (CTS)

➡️ Routing & DRC/LVS checks

➡️ Final GDSII generation

---

## 🎯 Objectives — Week 7

- Install and configure **OpenROAD-Flow-Scripts** on the local system.
- Integrate **VSDBabySoC RTL + analog macros** into the flow.
- Write a complete **`config.mk`** configuration for ORFS.
- Execute the **full RTL-to-GDS physical design flow**.
- Verify outputs with **GUI screenshots, logs, and reports**.

---
## Contents — Week 7

- [Task 1: Installing & Setting up OpenROAD-Flow-Scripts (ORFS)](#task-1-installing--setting-up-openroad-flow-scripts-orfs)
- [Task 2: Building OpenROAD](#task-2-building-openroad)
- [Task 3: Verifying Installation](#task-3-verifying-installation)
- [Task 4: ORFS Directory Structure Overview](#task-4-orfs-directory-structure-overview)
- [Task 5: Adding VSDBabySoC Design to ORFS](#task-5-adding-vsdbabysoc-design-to-orfs)
- [Task 6: Writing `config.mk` for VSDBabySoC](#task-6-writing-configmk-for-vsdbabysoc)
- [Task 7: Running the Complete RTL → GDSII Flow](#task-7-running-the-complete-rtl--gdsii-flow)
  - [7.1 Synthesis](#71-synthesis)
  - [7.2 Floorplan](#72-floorplan)
  - [7.3 Placement](#73-placement)
  - [7.4 Clock Tree Synthesis (CTS)](#74-clock-tree-synthesis-cts)
  - [7.5 Routing](#75-routing)
- [Task 8: Final Outputs of Week-7](#task-8-final-outputs-of-week-7)
- [Task 9: Key Learnings & Workflow Achievements](#task-9-key-learnings--workflow-achievements)
- [Task 10: Special Thanks & References](#task-10-special-thanks--references)



---

# 🟩 **1. Installing & Setting up ORFS**

### 🔹 Clone ORFS and Install All Dependencies

```bash
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts

```

<p align="center"><img src="./ASSETS/1.png" alt="image 1"/></p>
<p align="center"><img src="./ASSETS/2.png" alt="image 2"/></p>

```bash
sudo ./setup.sh
```

<p align="center"><img src="./ASSETS/3.png" alt="image 3"/></p>
<p align="center"><img src="./ASSETS/4.png" alt="image 4"/></p>
<p align="center"><img src="./ASSETS/5.png" alt="image 5"/></p>

This installs:

- Yosys
- OpenROAD
- KLayout
- Magic
- Required tech files
- Required Python/TCL dependencies

---

# 🟩 **2. Building OpenROAD**

```bash
./build_openroad.sh --local

```

<p align="center"><img src="./ASSETS/6.png" alt="image 6"/></p>
<p align="center"><img src="./ASSETS/7.png" alt="image 7"/></p>
This compiles OpenROAD locally and sets up all required binaries.

---

# 🟩 **3. Verifying Installation**

```bash
source ./env.sh
yosys -help
openroad -help

```

<p align="center"><img src="./ASSETS/8.png" alt="image 8"/></p>
<p align="center"><img src="./ASSETS/9.png" alt="image 9"/></p>

---

```bash
cd flow
make
```

<p align="center"><img src="./ASSETS/10.png" alt="image 10"/></p>
<p align="center"><img src="./ASSETS/11.png" alt="image 11"/></p>
<p align="center"><img src="./ASSETS/12.png" alt="image 12"/></p>
<p align="center"><img src="./ASSETS/13.png" alt="image 13"/></p>

---

```bash

make gui_final

```

On success, the ORFS GUI opens with all tools properly mapped.

<p align="center"><img src="./ASSETS/14.png" alt="image 14"/></p>
<p align="center"><img src="./ASSETS/15.png" alt="image 15"/></p>
---

# 🟦 **4. ORFS Directory Structure Overview**

The ORFS environment consists of:

```
OpenROAD-flow-scripts/
├── docker            → Docker build support
├── docs              → Documentation
├── flow              → Complete RTL2GDS automation
├── tools             → All required PD tools
├── etc               → Installers, dependency scripts
├── setup_env.sh      → Environment setup

```

<p align="center"><img src="./ASSETS/16.png" alt="image 16"/></p>

Inside `flow/`:

```
flow/
├── designs           → All example designs
├── platform          → Contains tech files (lef/lib/gds)
├── scripts           → OR scripts for each stage
├── tutorials         → Training references
├── util              → Helper scripts

```

<p align="center"><img src="./ASSETS/17.png" alt="image 17"/></p>

---

# 🟦 **5. Adding VSDBabySoC to ORFS**

To integrate the design:

### ✔ Create directories

```
flow/designs/sky130hd/vsdbabysoc
flow/designs/src/vsdbabysoc
```

### ✔ Copy files

Into `designs/sky130hd/vsdbabysoc/`:

- gds/ → avsddac.gds, avsdpll.gds
- lef/ → avsddac.lef, avsdpll.lef
- lib/ → avsddac.lib, avsdpll.lib
- include/ → all `.vh` files
- vsdbabysoc_synthesis.sdc
- macro.cfg
- pin_order.cfg

Into **designs/src/vsdbabysoc/**:

- vsdbabysoc.v
- rvmyth.v
- clk_gate.v

<p align="center"><img src="./ASSETS/18.png" alt="image 18"/></p>
<p align="center"><img src="./ASSETS/19.png" alt="image 19"/></p>

---

# 🟦 **6. Writing `config.mk` for ORFS**

A customized config is written to describe:

- Design RTL files
- Tech platform: `sky130hd`
- Macro integration (LEF/GDS/LIB)
- Floorplan dimensions
- Pin ordering
- Macro placement
- CTS settings

Key snippet:

```makefile
export DESIGN_NICKNAME = vsdbabysoc
export DESIGN_NAME = vsdbabysoc
export PLATFORM    = sky130hd

# export VERILOG_FILES_BLACKBOX = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/IPs/*.v
# export VERILOG_FILES = $(sort $(wildcard $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/*.v))
# Explicitly list the Verilog files for synthesis
export VERILOG_FILES = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/vsdbabysoc.v \
                       $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/rvmyth.v \
                       $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/clk_gate.v

export SDC_FILE      = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/vsdbabysoc_synthesis.sdc

export vsdbabysoc_DIR = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)

export VERILOG_INCLUDE_DIRS = $(wildcard $(vsdbabysoc_DIR)/include/)
# export SDC_FILE      = $(wildcard $(vsdbabysoc_DIR)/sdc/*.sdc)
export ADDITIONAL_GDS  = $(wildcard $(vsdbabysoc_DIR)/gds/*.gds.gz)
export ADDITIONAL_LEFS  = $(wildcard $(vsdbabysoc_DIR)/lef/*.lef)
export ADDITIONAL_LIBS = $(wildcard $(vsdbabysoc_DIR)/lib/*.lib)
# export PDN_TCL = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/pdn.tcl

# Clock Configuration (vsdbabysoc specific)
# export CLOCK_PERIOD = 20.0
export CLOCK_PORT = CLK
export CLOCK_NET = $(CLOCK_PORT)

# Floorplanning Configuration (vsdbabysoc specific)
export FP_PIN_ORDER_CFG = $(wildcard $(DESIGN_DIR)/pin_order.cfg)
# export FP_SIZING = absolute

export DIE_AREA   = 0 0 1600 1600
export CORE_AREA  = 20 20 1590 1590

# Placement Configuration (vsdbabysoc specific)
export MACRO_PLACEMENT_CFG = $(wildcard $(DESIGN_DIR)/macro.cfg)
export PLACE_PINS_ARGS = -exclude left:0-600 -exclude left:1000-1600: -exclude right:* -exclude top:* -exclude bottom:*
# export MACRO_PLACEMENT = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/macro_placement.cfg

export TNS_END_PERCENT = 100
export REMOVE_ABC_BUFFERS = 1

# Magic Tool Configuration
export MAGIC_ZEROIZE_ORIGIN = 0
export MAGIC_EXT_USE_GDS = 1

# CTS tuning
export CTS_BUF_DISTANCE = 600
export SKIP_GATE_CLONING = 1

# export CORE_UTILIZATION=0.1  # Reduce this value to allow more whitespace for routing.
```

---

# 🟧 **7. Running the Complete RTL → GDSII Flow**

Before running:

```bash
cd OpenROAD-flow-scripts
source env.sh
cd flow
```

<p align="center"><img src="./ASSETS/20.png" alt="image 20"/></p>

---

## 🟪 **7.1 Synthesis**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth

```

<p align="center"><img src="./ASSETS/21.png" alt="image 21"/></p>
<p align="center"><img src="./ASSETS/22.png" alt="image 22"/></p>

### Outputs:

- Gate-level netlist
- Synthesis log
- Timing estimation
- Cell usage report

### 1_synth.v

<p align="center"><img src="./ASSETS/23.png" alt="image 23"/></p>

### synth_check.txt

<p align="center"><img src="./ASSETS/24.png" alt="image 24"/></p>

### synth_stat.txt

<p align="center"><img src="./ASSETS/25.png" alt="image 25"/></p>

---

## 🟪 **7.2 Floorplan**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan

```

<p align="center"><img src="./ASSETS/27.png" alt="image 27"/></p>
<p align="center"><img src="./ASSETS/28.png" alt="image 28"/></p>


```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
```

<p align="center"><img src="./ASSETS/29.png" alt="image 29"/></p>
<p align="center"><img src="./ASSETS/30.png" alt="image 30"/></p>

### Achieved:

- Die/Core sizing
- IO pin ordering
- Macro placement
- Power grid generation

---

## 🟪 **7.3 Placement**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place

```

<p align="center"><img src="./ASSETS/31.png" alt="image 31"/></p>
<p align="center"><img src="./ASSETS/32.png" alt="image 32"/></p>
```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place
```

<p align="center"><img src="./ASSETS/33.png" alt="image 33"/></p>
<p align="center"><img src="./ASSETS/34.png" alt="image 34"/></p>
Includes:

- Global placement
- Detailed placement

### Congestion analysis (Heatmap)

📌 Routing Congestion

<p align="center"><img src="./ASSETS/35.png" alt="image 35"/></p>

📌 Estimated COngestion(RUDY)

<p align="center"><img src="./ASSETS/36.png" alt="image 36"/></p>

📌 IR Drop

<p align="center"><img src="./ASSETS/37.png" alt="image 37"/></p>

📌 Pin Density

<p align="center"><img src="./ASSETS/38.png" alt="image 38"/></p>

📌 Placement Density

<p align="center"><img src="./ASSETS/39.png" alt="image 39"/></p>

📌 Power  Density 

<p align="center"><img src="./ASSETS/40.png" alt="image 40"/></p>

📌 Inner  look of the cells

<p align="center"><img src="./ASSETS/41.png" alt="image 41"/></p>

---

## 🟪 **7.4 CTS (Clock Tree Synthesis)**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts

```

<p align="center"><img src="./ASSETS/42.png" alt="image 42"/></p>
<p align="center"><img src="./ASSETS/43.png" alt="image 43"/></p>



```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts
```

<p align="center"><img src="./ASSETS/44.png" alt="image 44"/></p>

Deliverables:

- Balanced clock tree
- Buffer insertion
- Skew optimization
- CTS timing report

---

## 🟪 **7.5 Routing**

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route

```

Produces:

- Global route
- Detailed route
- DRC-clean wiring
- Final routed DEF

---

# 🟩 **8. Final Outputs of Week-7**

✔ Automated end-to-end RTL2GDS for VSDBabySoC

✔ All intermediate results saved: `logs/`, `reports/`, `results/`

✔ GUI screenshots for Floorplan, Placement, CTS, Routing

✔ Verified design timing, congestion, CTS quality

---

# 🎉 **Week-7 Summary**

This week, I successfully:

- Installed OpenROAD-Flow-Scripts
- Built and validated ORFS environment
- Integrated custom SoC + analog macros
- Wrote complete design configuration (`config.mk`)
- Executed synthesis → floorplan → placement → CTS → routing
- Verified each stage with reports and GUI visualization

This completes the **full physical design flow** for VSDBabySoC using **open-source ASIC tools**.
---


## 📒 Key Learnings — Week 7

### 🛠️ Tools and Frameworks

✔ **OpenROAD** → Complete physical design automation

✔ **Yosys** → RTL synthesis integration

✔ **OpenSTA** → Static timing analysis

✔ **TritonTools** → Floorplan, placement, CTS, and routing

✔ **Sky130 PDK** → Technology libraries for ASIC macros

✔ **ORFS environment** → Standardized scripts for repeatable SoC flow

---

### 🔹 Workflow Achievements

1. **Environment Setup & Verification**
    - ORFS installation and dependency resolution.
    - Built OpenROAD locally and verified binaries.
    - Confirmed Yosys, OpenROAD, and TritonTools availability.
2. **Design Integration**
    - Added VSDBabySoC RTL (`vsdbabysoc.v`, `rvmyth.v`, `clk_gate.v`)
    - Integrated analog macros (`avsddac`, `avsdpll`) with LEF/GDS/LIB.
    - Configured floorplan (`pin_order.cfg`, `macro.cfg`) and timing (`vsdbabysoc_synthesis.sdc`).
3. **Full Flow Execution**
    - **Synthesis:** Generated gate-level netlist, timing reports, and cell usage statistics.
    - **Floorplan:** Defined die/core dimensions, pin placement, and macro layout.
    - **Placement:** Performed global and detailed placement; analyzed congestion, IR drop, and density.
    - **Clock-Tree Synthesis:** Balanced clock network with buffer insertion and skew optimization.
    - **Routing:** Global and detailed routing; ensured DRC-clean nets.
    - **Final GDSII:** Complete physical layout ready for tapeout.
  
  ---

> 💡 “Week 7 was the culmination of our physical design journey — automating the full SoC flow and generating a tapeout-ready layout for VSDBabySoC using open-source ASIC tools.” 🚀

----


## 🙏 Special Thanks 👏  
I sincerely thank all the organizations and their key members for making this program possible 💡:  

- 🧑‍🏫 **VLSI System Design (VSD)** – [Kunal Ghosh](https://www.linkedin.com/in/kunal-ghosh-vlsisystemdesign-com-28084836/) for mentorship and vision.  
- 🤝 **Efabless** – [Michael Wishart](https://www.linkedin.com/in/mike-wishart-81480612/) & [Mohamed Kassem](https://www.linkedin.com/in/mkkassem/) for enabling collaborative open-source chip design.  
- 🏭 **[Semiconductor Laboratory (SCL)](https://www.scl.gov.in/)** – for PDK & foundry support.  
- 🎓 **[IIT Gandhinagar (IITGN)](https://www.linkedin.com/school/indian-institute-of-technology-gandhinagar-iitgn-/?originalSubdomain=in)** – for on-site training & project facilitation.  
- 🛠️ **Synopsys** – [Sassine Ghazi](https://www.linkedin.com/in/sassine-ghazi/) for providing industry-grade EDA tools under C2S program.  

--- 
👉 Main Repo Link :  
[https://github.com/madhavanshree2006/RISC-V-SoC-Tapeout-Program](https://github.com/madhavanshree2006/RISC-V-SoC-Tapeout-Program)
