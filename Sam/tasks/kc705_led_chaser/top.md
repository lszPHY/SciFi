---
Rank: 2
Skills: local_env
BashTime: -1
GPU: no
Slurm: off
CommonStorage: ro
PrescanModel: claude-opus-4-7
Timeout: 6000
ReviewModel: gpt-5.5
TaskGroup: kc705_verilog
---

# KC705 LED Chaser Bitstream

## Context

Create a complete Vivado batch-flow project that builds a bitstream for the
Xilinx KC705 board. The design should turn the KC705 user LEDs on and off in a
visible sequence, one active LED moving across the 8 user LEDs.

This task is for **bitstream generation only**. Do not program the physical
board unless explicitly requested in a separate task.

Use the installed Vivado toolchain:

```bash
/hdd/Xilinx/Vivado/2024.1/settings64.sh
```

Use this Vivado license server for all Vivado/xsim/synthesis commands:

```bash
export XILINXD_LICENSE_FILE=2222@phys-hep.physics.lsa.umich.edu
```

Because SciFi runs inside an Apptainer container with `--cleanenv`, do not
assume license variables from the host shell are inherited. Every script or
bash command that runs Vivado, `xvlog`, `xelab`, or `xsim` must set
`XILINXD_LICENSE_FILE` first.

Use the KC705 FPGA part:

```text
xc7k325tffg900-2
```



Use a non-project Vivado Tcl batch flow. Do not create a GUI project. Do not
search the whole Vivado install tree. Do not install tools. Do not use internet
unless needed only to verify KC705 public documentation.

The KC705 user LEDs are active-high. Use these constraints:

```xdc
# KC705 user LEDs, active-high
# Source: KC705 User Guide UG810, GPIO Connections to FPGA U1.
set_property PACKAGE_PIN AB8  [get_ports {led[0]}]
set_property IOSTANDARD LVCMOS15 [get_ports {led[0]}]

set_property PACKAGE_PIN AA8  [get_ports {led[1]}]
set_property IOSTANDARD LVCMOS15 [get_ports {led[1]}]

set_property PACKAGE_PIN AC9  [get_ports {led[2]}]
set_property IOSTANDARD LVCMOS15 [get_ports {led[2]}]

set_property PACKAGE_PIN AB9  [get_ports {led[3]}]
set_property IOSTANDARD LVCMOS15 [get_ports {led[3]}]

set_property PACKAGE_PIN AE26 [get_ports {led[4]}]
set_property IOSTANDARD LVCMOS25 [get_ports {led[4]}]

set_property PACKAGE_PIN G19  [get_ports {led[5]}]
set_property IOSTANDARD LVCMOS25 [get_ports {led[5]}]

set_property PACKAGE_PIN E18  [get_ports {led[6]}]
set_property IOSTANDARD LVCMOS25 [get_ports {led[6]}]

set_property PACKAGE_PIN F16  [get_ports {led[7]}]
set_property IOSTANDARD LVCMOS25 [get_ports {led[7]}]
```

Use the KC705 differential SYSCLK input:

```xdc
# KC705 SYSCLK, 200 MHz differential clock
# Source: KC705 User Guide UG810, KC705 Board XDC Listing.
set_property PACKAGE_PIN AD12 [get_ports sysclk_p]
set_property IOSTANDARD LVDS [get_ports sysclk_p]
set_property PACKAGE_PIN AD11 [get_ports sysclk_n]
set_property IOSTANDARD LVDS [get_ports sysclk_n]
create_clock -name sysclk -period 5.000 [get_ports sysclk_p]
```

The RTL top module must be named:

```verilog
led_chaser
```

The top-level ports must be:

```verilog
input  wire       sysclk_p,
input  wire       sysclk_n,
output reg  [7:0] led
```

Inside RTL, instantiate `IBUFDS` to convert `sysclk_p/sysclk_n` to a
single-ended internal clock. Use a clock divider so the LED sequence is visible
on hardware. A reasonable default is to advance the LED every about 0.25 second
using the 200 MHz clock. That means approximately 50,000,000 clock cycles per
step.

For simulation, make the divider configurable with a Verilog parameter so the
testbench does not need to simulate millions of cycles. Example:

```verilog
parameter integer STEP_COUNT = 50_000_000
```

The testbench can instantiate:

```verilog
led_chaser #(.STEP_COUNT(4)) dut (...)
```

Use only simple synthesizable Verilog/SystemVerilog compatible with Vivado
2024.1.

## Todo

1. Verify Vivado is available:

   ```bash
   export XILINXD_LICENSE_FILE=2222@phys-hep.physics.lsa.umich.edu
   source /hdd/Xilinx/Vivado/2024.1/settings64.sh
   vivado -version
   xvlog -version
   xelab -version
   xsim -version
   env | grep -E 'XILINXD_LICENSE_FILE|LM_LICENSE_FILE' || true
   ```

   Write the output to `tool_versions.log`. If Vivado is missing or cannot run,
   write `VIVADO_MISSING` to `error.txt` and stop.

2. Write `led_chaser.v`.

   Requirements:
   - Top module name: `led_chaser`.
   - Ports exactly: `sysclk_p`, `sysclk_n`, `led[7:0]`.
   - Instantiate `IBUFDS`.
   - Use parameter `STEP_COUNT`.
   - Drive exactly one LED high at a time.
   - Sequence must move through all 8 LEDs and wrap around.
   - Initialize LED state deterministically.
   - No reset port is required.

3. Write `tb_led_chaser.v`.

   Requirements:
   - Generate a differential clock pair.
   - Instantiate `led_chaser #(.STEP_COUNT(4))`.
   - Simulate enough cycles to observe at least two full LED rotations.
   - Check that exactly one LED is high when not in the initial transient.
   - Check that the sequence advances and wraps.
   - Print `PASS` only if all checks pass.
   - Print clear `ERROR:` lines if any check fails.

4. Write `kc705_led_chaser.xdc`.

   Requirements:
   - Include only constraints needed for `sysclk_p`, `sysclk_n`, and `led[7:0]`.
   - Use the exact pin and IOSTANDARD values listed in the Context section.
   - Include `create_clock -period 5.000` for 200 MHz SYSCLK.
   - Do not invent or add unrelated constraints.

5. Write `simulate.tcl`.

   It should run Vivado xsim in batch mode:

   ```tcl
   set license [getenv XILINXD_LICENSE_FILE]
   puts "XILINXD_LICENSE_FILE=$license"
   set_msg_config -severity WARNING -new_severity INFO
   xvlog -sv led_chaser.v tb_led_chaser.v
   xelab tb_led_chaser -s sim_led_chaser
   xsim sim_led_chaser -runall
   ```

6. Run simulation:

   ```bash
   export XILINXD_LICENSE_FILE=2222@phys-hep.physics.lsa.umich.edu
   source /hdd/Xilinx/Vivado/2024.1/settings64.sh
   vivado -mode batch -source simulate.tcl 2>&1 | tee sim.log
   ```

   If simulation fails, fix RTL/testbench and rerun until `sim.log` contains
   `PASS` and no `ERROR:` lines.

7. Write `build.tcl`.

   It must use non-project mode:

   ```tcl
   set license [getenv XILINXD_LICENSE_FILE]
   puts "XILINXD_LICENSE_FILE=$license"
   set_part xc7k325tffg900-2
   read_verilog led_chaser.v
   read_xdc kc705_led_chaser.xdc
   synth_design -top led_chaser -part xc7k325tffg900-2
   opt_design
   place_design
   route_design
   report_timing_summary -file timing_summary.rpt
   report_utilization -file utilization.rpt
   write_checkpoint -force led_chaser_routed.dcp
   write_bitstream -force led_chaser.bit
   ```

8. Run bitstream build:

   ```bash
   export XILINXD_LICENSE_FILE=2222@phys-hep.physics.lsa.umich.edu
   source /hdd/Xilinx/Vivado/2024.1/settings64.sh
   vivado -mode batch -source build.tcl 2>&1 | tee build.log
   ```

9. Inspect the build result.

   Required checks:
   - `led_chaser.bit` exists and is larger than 1 MB.
   - `build.log` contains no `CRITICAL WARNING` that affects pin constraints,
     unconstrained clocks, wrong part, or failed timing.
   - `build.log` contains no `ERROR:`.
   - `timing_summary.rpt` exists.
   - `utilization.rpt` exists.

   If there is a harmless warning, document it in `README_RESULT.md`.
   If there is any critical issue, fix it or write a clear failure reason.

10. Write `README_RESULT.md`.

    It must include:
    - Vivado version used.
    - Vivado license variable used, without hiding the server address.
    - Target part.
    - Explanation of the LED sequence.
    - List of generated files.
    - Simulation result.
    - Bitstream build result.
    - Any warnings and whether they are safe.
    - Explicit note that the board was **not programmed**.

## Expect

- `tool_versions.log` exists and contains Vivado 2024.1 information.
- `led_chaser.v` exists.
- `tb_led_chaser.v` exists.
- `kc705_led_chaser.xdc` exists.
- `simulate.tcl` exists.
- `sim.log` exists and contains `PASS`.
- `sim.log` contains no `ERROR:`.
- `build.tcl` exists.
- `build.log` exists.
- `build.log` contains no Vivado `ERROR:`.
- `timing_summary.rpt` exists.
- `utilization.rpt` exists.
- `led_chaser.bit` exists and is larger than 1 MB.
- `README_RESULT.md` exists and clearly states:
  - target part is `xc7k325tffg900-2`;
  - output bitstream is `led_chaser.bit`;
  - board programming was not attempted.

# Notes For Reviewer

The reviewer should verify that the generated XDC uses the KC705 pins from this
task, especially:

```text
SYSCLK_P: AD12
SYSCLK_N: AD11
LED0: AB8
LED1: AA8
LED2: AC9
LED3: AB9
LED4: AE26
LED5: G19
LED6: E18
LED7: F16
```

Reject the task if the design targets `xc7vx485tffg1157-1` or any non-KC705
part.

Reject the task if the bitstream is missing, if simulation does not print
`PASS`, or if the XDC pin constraints were invented instead of using the
provided KC705 pin list.
