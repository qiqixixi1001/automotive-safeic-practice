# Automotive Safe-IC Practice 09: Simulation Safety Context — VCD, Good Machine, FTTI, and Observe Points

Author: Darren H. Chen  
Direction: Automotive chip functional safety analysis and fault-injection verification  
Demo: D09_simulation_safety_context_vcd_good_machine_ftti_observe_point  
Tags: Functional Safety, ISO 26262, Safe-IC, Fault Injection, VCD, Good Machine, FTTI, Observe Point, Fault Campaign, Safety Verification

---

## 1. From Fault List to Runtime Evidence

The previous step, D08, converted structural safety analysis results and safety mechanism mapping decisions into a campaign-oriented fault list. That list answers an important question:

```text
Which faults should be injected?
```

However, a fault list alone does not define a fault campaign.

A fault becomes meaningful only when it is injected into a known execution context. The same stuck-at fault can be irrelevant in one simulation window, dangerous in another, detected in a third, and unresolved in a fourth. The difference is not the fault object itself. The difference is the runtime situation around that fault.

D09 is about that runtime situation.

This article focuses on four concepts:

```text
VCD
Good Machine
FTTI
Observe Point
```

Together, they form the simulation safety context. They connect the static fault list from D08 to the fault campaign setup in D11 and the fault injection execution in D12.

A simple mental model is:

```text
D08: fault population
D09: golden simulation context
D10: alarm and observation boundary
D11: fault campaign input package
D12: fault injection execution
D13: outcome classification
```

D09 does not prove final diagnostic coverage by itself. It defines the runtime reference that later fault injection will use to judge whether a fault is detected, safe, unsafe, or unresolved.

---

## 2. The Role of D09 in the 20-Step Flow

In the core flow, D09 is:

```text
Simulation Safety Context: VCD, Good Machine, FTTI, observe point
```

It follows D08 and precedes D10 / D11.

The flow dependency is deliberate:

```text
D07 Safety Mechanism Map
  -> D08 Fault List Generation
    -> D09 Simulation Safety Context
      -> D10 Alarm List and Observe Point Boundary
        -> D11 Fault Campaign Setup
```

D08 says which faults should be considered. D09 says what normal execution looks like. D10 says how the campaign will observe detection or unsafe behavior. D11 combines these into one executable input package.

Without D09, a fault campaign becomes a collection of injections without a reliable reference. The campaign may still run, but the classification results will be weak because there is no clearly defined golden context, no timing window, and no stable mapping between testbench hierarchy and design hierarchy.

---

## 3. Safety Verification Needs a Golden Runtime Context

Traditional functional verification often answers:

```text
Did the design pass the test?
```

Functional safety verification asks a different question:

```text
If a hardware fault occurs during this test, does the system detect, tolerate, mask, or expose it?
```

To answer that, the verification environment needs a reference run.

The reference run is the no-fault execution of the same design under the same stimulus. This is often called the golden run or good-machine simulation.

A later fault campaign compares each faulty run against this reference context:

```text
good machine state
faulty machine state
alarm behavior
observe point behavior
simulation time
clock cycle
FTTI window
```

If the faulty machine diverges but an alarm fires in time, the fault may be classified as detected. If the faulty machine diverges and no alarm fires, it may be unsafe. If the fault does not affect the relevant machine state, it may be safe. If the tool cannot decide because of missing data, black-box propagation, unknown values, or insufficient stimulus, the result may become unresolved.

D09 builds the foundation for that comparison.

---

## 4. What Is VCD?

VCD stands for **Value Change Dump**.

It is a waveform file format that records signal value changes over simulation time. A VCD file does not normally store every signal at every time point. Instead, it records changes:

```text
time 0: reset = 0, count = 0
time 10: clk toggles
time 20: reset = 1
time 30: enable = 1
time 40: count changes
```

This event-based representation is compact enough for many simulation flows and simple enough to be produced by many simulators.

In a safety verification context, VCD is not just a waveform for human debugging. It becomes a machine-readable record of the golden execution. A fault campaign engine can use it to understand:

```text
which signals toggled
when the design was out of reset
which endpoints had known values
where the DUT is located inside the testbench
what normal state evolution looked like
which signals are available for comparison or forcing
```

That is why D09 treats VCD as a safety input artifact rather than a casual debug file.

---

## 5. VCD as a Data-Exchange Protocol

In this series, the word protocol often means a data contract between flow stages.

VCD is one such contract.

It connects functional simulation with fault injection:

```text
functional simulation
  -> VCD
    -> simulation safety context
      -> fault injection engine
```

The contract includes more than the `.vcd` file itself. It also includes:

```text
DUT hierarchy path
top module name
clock definition
reset behavior
timescale
simulation start time
simulation end time
signal naming conventions
which signals are dumped
which signals are required
which signals are optional
```

If any part of this contract is unstable, later fault injection becomes fragile.

For example, if a testbench dumps:

```text
tb.u_dut.count[7:0]
```

but the fault campaign configuration expects:

```text
tb_top.dut_i.count[7:0]
```

then the waveform exists, but the context is not usable.

D09 therefore treats VCD preparation as a controlled interface, not merely a simulator option.

---

## 6. Good Machine Simulation

Good Machine Simulation is the fault-free reference execution.

It answers:

```text
What should the design do when no fault is injected?
```

The good machine context is used later as a comparison baseline. It defines the expected state trajectory of endpoints, alarms, observe points, and key internal nodes.

A good machine context should be reproducible:

```text
same RTL or netlist boundary
same testbench
same reset sequence
same stimulus
same compile options
same clock definition
same dumped signals
same simulation time window
same DUT hierarchy
```

If two good-machine runs produce different VCD content for the same intended test, the fault campaign evidence becomes questionable. The issue may be caused by random stimulus without a recorded seed, uninitialized memory, race conditions, X-propagation, or simulator configuration differences.

D09 should therefore record a simulation manifest:

```text
test name
simulator wrapper
random seed
DUT path
VCD path
start time
end time
clock period
reset release time
dumped signal groups
expected output artifacts
```

This is not bureaucracy. It is what makes the safety evidence reviewable.

---

## 7. Good Machine Is Not the Same as Passing Functional Simulation

A functional test can pass while still being a poor good-machine source.

For example, a test might check only one final output:

```text
PASS if final count == 8'h12
```

That is enough for a simple functional regression, but it may be insufficient for fault injection because the fault engine may need many intermediate signals:

```text
state registers
endpoint values
alarm candidates
observe points
valid / ready handshakes
reset status
clock activity
protocol control signals
```

A VCD that dumps only the final output may not give enough information to classify a fault. The good-machine context must support fault outcome classification, not just functional pass/fail.

D09 therefore asks:

```text
Does the VCD contain the signals needed for safety verification?
```

not merely:

```text
Did the functional test pass?
```

---

## 8. The Relationship Between VCD and Fault Outcome

Fault outcome classification depends on comparing faulty behavior against the golden context.

A simplified classification model is:

```text
no meaningful divergence
  -> safe

divergence reaches alarm within FTTI
  -> detected

divergence affects safety-relevant observe point and no alarm fires
  -> unsafe

divergence cannot be fully classified due to missing data, black boxes, X values, or insufficient stimulus
  -> unresolved
```

The VCD is used as the reference for this comparison.

This means a poor VCD can increase unresolved results even if the design has real safety mechanisms. Missing signals, missing reset context, short simulation windows, and inconsistent hierarchy paths can all cause a campaign to produce low-quality evidence.

The problem is not always the safety mechanism. Sometimes it is the simulation context.

D09 exists to reduce that risk before D12 and D13.

---

## 9. FTTI: Fault Tolerant Time Interval

FTTI means **Fault Tolerant Time Interval**.

It is the time interval within which a fault must be detected, controlled, or otherwise prevented from violating a safety goal.

In a fault campaign, FTTI becomes an observation window.

A conceptual timeline:

```text
t0: fault injected
t1: fault propagates
t2: alarm should fire or system should enter safe behavior
t3: FTTI expires
```

If the system detects the fault before `t3`, the fault may receive diagnostic credit. If detection occurs after the FTTI window, the response may be too late for the safety goal.

In digital simulation, FTTI may be represented in:

```text
clock cycles
simulation time units
testbench phases
protocol transactions
```

For example:

```text
FTTI = 64 clock cycles
```

or:

```text
FTTI = 500 ns
```

The exact interpretation must be explicit. D09 should record both the engineering meaning and the file-level representation.

---

## 10. FTTI Is a Safety Requirement, Not a Simulator Timeout

It is easy to confuse FTTI with a simulation runtime limit.

They are different.

A simulation runtime limit answers:

```text
How long should the simulator run?
```

FTTI answers:

```text
How quickly must the system react to a fault?
```

A campaign may simulate longer than FTTI for debugging or classification reasons, but diagnostic credit depends on whether the response happens inside the fault tolerant interval.

For example:

```text
simulation duration = 10,000 cycles
FTTI = 128 cycles
fault injected at cycle 1,000
alarm fires at cycle 1,300
```

The alarm fired within the full simulation duration, but not within the 128-cycle FTTI. Depending on the policy, that may not be credited as timely detection.

D09 should separate:

```text
simulation duration
fault injection time
post-injection observation window
FTTI
alarm deadline
observe point deadline
```

This distinction becomes important in D13 when detected, safe, unsafe, and unresolved outcomes are classified.

---

## 11. Observe Point

An observe point is a signal or state element used to judge whether a fault has reached a safety-relevant boundary.

It is not necessarily an alarm.

An alarm says:

```text
the safety mechanism detected something
```

An observe point says:

```text
this is where we check whether behavior became relevant to safety
```

Examples:

```text
critical_output_o
safe_state_o
actuator_cmd_o
protocol_response_valid
error_status_o
state_reg
```

Observe points help define what counts as observable divergence. A fault that changes an internal node but never reaches an observe point may be less important than a fault that changes a safety-critical output.

In some campaign configurations, a fault may be considered failed only if it reaches an observe point. In other configurations, endpoint mismatch alone may be enough to classify a divergence. D09 should document the intended observation policy, while D10 will refine alarm and observe point lists.

---

## 12. Alarm vs. Observe Point

Alarm and observe point are related but not interchangeable.

An alarm is usually generated by a safety mechanism:

```text
parity_error
ecc_error
lockstep_mismatch
protocol_check_fail
timeout_alarm
```

An observe point may be a system behavior signal:

```text
brake_cmd
torque_request
valid_response
safe_state
output_data
```

A simplified relationship:

```text
fault occurs
  -> internal deviation
    -> safety mechanism detects deviation
      -> alarm fires

fault occurs
  -> internal deviation
    -> propagates to safety-relevant output
      -> observe point changes
```

For a detected fault, alarm timing matters. For an unsafe fault, observe point impact matters.

D09 introduces the concept of observation boundaries. D10 will turn that into explicit alarm and observe point artifacts.

---

## 13. Simulation Start Time

Fault injection should not usually begin during reset or before the design has entered a meaningful operational state.

D09 should define:

```text
simulation_start_time
reset_release_time
first_valid_injection_time
```

A common mistake is to inject faults too early. If the design is still in reset, many faults may be masked, overwritten, or irrelevant. That can inflate safe-fault counts or create misleading outcomes.

Another mistake is to inject too late. If the simulation window is nearly over, there may not be enough time for fault propagation and alarm response.

A robust D09 package includes a candidate injection window:

```text
reset stable before injection
input stimulus active
clock active
safety-relevant transactions visible
enough remaining time for FTTI
```

This is part of the simulation safety context.

---

## 14. DUT Path and Error Injection Instance

A VCD is usually generated by a testbench, not by the DUT alone.

The hierarchy may look like:

```text
tb_top
  u_env
  u_scoreboard
  u_dut
    u_core
    u_timer
```

The fault engine must know which hierarchy corresponds to the design under analysis. This is often represented as an error injection instance or DUT path.

For example:

```text
vcd_dut_path = tb_top.u_dut
```

If the DUT path is wrong, the fault engine may fail to map VCD signals to design nodes even when the VCD contains the right data.

D09 therefore validates the hierarchy contract:

```text
analysis top name
testbench DUT instance path
VCD signal prefix
filelist design hierarchy
fault list naming style
```

The goal is to avoid late-stage campaign failures caused by path mismatch.

---

## 15. Timescale and Clock Alignment

VCD stores time in simulation units. The campaign engine may interpret time through:

```text
VCD timescale
clock definition
simulation start time
fault injection time
FTTI cycles
```

A mismatch between time units and cycle-based expectations can break the safety context.

For example:

```text
VCD timescale = 1ps
clock period = 10ns
FTTI = 128 cycles
```

The implementation must know whether to convert:

```text
128 cycles -> 1280 ns -> 1,280,000 ps
```

or whether the fault engine expects cycles directly.

D09 should document:

```text
clock name
clock period
VCD timescale
reset edge
injection start
FTTI in cycles
FTTI in time units
```

This makes the context understandable to both tools and reviewers.

---

## 16. Reset and Initialization

A good-machine context must include enough reset and initialization information.

The following questions matter:

```text
When does reset assert?
When does reset deassert?
Are memories initialized?
Are internal registers known?
Are X values expected?
Is the testbench using random initialization?
Is there a stabilization window before fault injection?
```

Uninitialized state can cause spurious mismatches between good-machine and faulty-machine runs. It can also create unresolved faults because the campaign cannot determine whether a divergence is caused by the injected fault or by unstable initial conditions.

D09 should treat initialization as part of safety evidence.

A context manifest may include:

```text
reset_active_level
reset_assert_time
reset_deassert_time
memory_init_files
random_seed
first_safe_injection_cycle
```

---

## 17. X and Z Values

Digital simulation may contain unknown (`X`) or high-impedance (`Z`) values.

In ordinary functional debug, engineers may tolerate some X values if the final test passes. In fault injection, X propagation can make classification ambiguous.

A fault may appear unresolved because:

```text
an X reaches an observe point
a black-box boundary loses visibility
the VCD does not contain required internal signals
the good-machine reference already has unknown values
```

D09 should not try to hide X issues. It should classify them:

```text
expected X during reset
unexpected X after reset
X only on unused debug signals
X on safety-relevant observe points
X on alarm candidates
```

If X appears after reset on a safety-relevant signal, the campaign context may not be ready.

---

## 18. VCD Signal Coverage

Not all VCD files are equally useful.

A minimal VCD may include:

```text
clk
rst_n
input stimulus
final outputs
```

A safety verification VCD often needs more:

```text
endpoint signals
safety mechanism outputs
alarm candidates
protocol handshakes
observe points
internal state
memory interface signals
transaction-valid indicators
safe-state indicators
```

D09 should generate or validate a signal requirement list.

Example:

```csv
signal_group,required,reason
clock,yes,timing alignment
reset,yes,initialization
endpoint,yes,golden endpoint comparison
alarm_candidate,yes,detection evidence
observe_point,yes,unsafe behavior boundary
protocol_valid_ready,yes,transaction context
debug_signal,no,manual debug only
```

The goal is not to dump every signal blindly. Huge VCD files can slow down later steps. The goal is to dump enough relevant signals to classify faults.

---

## 19. VCD Filtering

Large VCD files can become expensive.

D09 should distinguish:

```text
raw simulation waveform
filtered campaign waveform
indexed signal manifest
context quality report
```

A raw waveform may be useful for debugging, but a filtered waveform is often better for fault campaign execution.

Filtering criteria may include:

```text
keep clock and reset
keep DUT boundary inputs and outputs
keep endpoints from D04
keep fault-list nodes from D08 when needed
keep alarm candidates from D07/D10
keep observe points
keep protocol context signals
drop unrelated testbench debug
drop scoreboard internals
drop unused coverage signals
```

VCD filtering should be reproducible. It should not be an ad hoc GUI operation. A filter manifest should explain why each signal group is kept or removed.

---

## 20. VCD Grading

VCD grading asks:

```text
Is this simulation data useful for fault injection?
```

A VCD may be syntactically valid but weak for safety verification.

D09 can grade VCD quality using criteria such as:

```text
clock exists and toggles
reset exists and releases
DUT hierarchy matches expected path
required endpoint signals exist
alarm candidates exist or are scheduled for D10
observe point candidates exist
simulation duration exceeds injection window + FTTI
fault-list nodes are reachable or mappable
post-reset X ratio is acceptable
protocol transactions occur
```

A simple grade model:

```text
PASS: ready for campaign packaging
WARN: usable but needs review
FAIL: cannot support reliable fault classification
```

This grade is an engineering gate before D11.

---

## 21. Protocol-Visible Safety Context

Automotive SoCs often use bus and handshake protocols. Even a small demonstration design can model protocol-visible behavior:

```text
valid / ready
req / ack
addr / data / write
status / error
interrupt / alarm
```

A fault may not immediately change a final output, but it may corrupt protocol behavior:

```text
valid asserted with wrong data
ready stuck low
ack missing
error response not generated
timeout not triggered
interrupt not raised
```

D09 should capture protocol context signals when they are safety-relevant.

This matters because D13 classification may need to know whether the system violated a protocol-level safety contract. D10 will formalize alarms and observe points, but D09 must ensure the VCD contains the protocol signals needed for that analysis.

---

## 22. Transaction Windows

A simulation contains phases:

```text
reset phase
initialization phase
idle phase
transaction phase
response phase
shutdown phase
```

Fault injection is usually most meaningful during transaction phases.

D09 can define transaction windows:

```csv
window_id,start_cycle,end_cycle,description
W_RESET,0,20,reset and stabilization
W_INIT,21,50,configuration
W_ACTIVE_0,51,150,first active transaction
W_ACTIVE_1,151,260,second active transaction
W_IDLE,261,320,idle after transaction
```

This helps D11 and D12 choose injection times.

Injecting during idle can be useful for latent fault analysis, but it should be intentional. Injecting during active protocol windows can test detection under realistic operation.

D09 records those choices.

---

## 23. Multi-VCD Context

A single VCD may not cover all safety-relevant behavior.

D09 can support multiple simulation data files:

```text
reset_and_init.vcd
nominal_operation.vcd
error_response.vcd
high_activity.vcd
low_power_transition.vcd
```

A fault campaign may distribute faults across multiple VCDs or use different VCDs for different scenarios.

The context manifest should describe:

```text
which VCD covers which scenario
which test generated it
which seed was used
which injection window applies
which fault subset it supports
which observe point policy applies
```

This prevents the common problem of treating all waveforms as interchangeable.

---

## 24. Good Machine Consistency Checks

Before injecting faults, D09 should run consistency checks on the good-machine context.

Typical checks:

```text
VCD file exists
VCD is non-empty
timescale is present
clock toggles
reset releases
DUT path exists
top-level signals are mapped
required endpoint signals are dumped
required observe point candidates are dumped
simulation duration is long enough
FTTI can fit after chosen injection time
```

A stronger check compares the VCD against the design boundary:

```text
every required endpoint from D04 has a waveform signal
every high-risk fault scope from D08 has a relevant observation path
every proposed alarm from D07 is either present or marked for D10 review
```

D09 therefore connects static evidence to dynamic evidence.

---

## 25. Force Signals and Context Bridging

Sometimes a VCD does not contain every internal signal needed for exact simulation comparison.

Some fault campaign flows allow selected signals to be forced from simulation data instead of being recomputed internally. This can help bridge mismatches between RTL simulation context and fault propagation context, but it must be used carefully.

A force-list policy should answer:

```text
Which signals are forced?
Why are they forced?
Are they endpoints, internal nodes, or protocol context signals?
Do they affect classification?
Is this a temporary workaround or intended methodology?
```

D09 should not silently force signals. It should produce a reviewable force list if such bridging is needed.

A generic force list format may look like:

```text
top.u_block.signal_a
top.u_block.signal_b
```

The exact syntax depends on the local tool adapter, but the evidence principle is tool-independent:

```text
forced context must be explicit and reviewable
```

---

## 26. Context Package Structure

A D09 demo should produce a context package, not just a VCD.

A practical directory structure:

```text
D09_simulation_safety_context/
  inputs/
    from_D08/
    from_D07/
    from_D04/
    simulation/
  configs/
    simulation_context_manifest.csv
    vcd_signal_requirements.csv
    ftti_policy.csv
    observe_point_candidates.csv
  outputs/
    vcd_inventory.csv
    good_machine_context.csv
    hierarchy_mapping.csv
    signal_presence_matrix.csv
    ftti_window_plan.csv
    observe_point_candidate_map.csv
    simulation_context_quality_gate.csv
    d09_handoff_to_d10.csv
    d09_handoff_to_d11.csv
    evidence_index.csv
    demo_summary.md
  scripts/
    run_demo.csh
```

This structure makes D09 useful even before the fault campaign engine is invoked.

The output files become the bridge to D10 and D11.

---

## 27. Generic D09 Command Flow

A public engineering demo should not depend on a hard-coded private tool path. It can use local configuration and generic entry points.

An illustrative command flow:

```csh
setenv D08_ROOT /path/to/D08_fault_list_generation
setenv D07_ROOT /path/to/D07_safety_mechanism_map
setenv D04_ROOT /path/to/D04_structural_building_blocks

csh scripts/run_demo.csh
```

Inside the demo, the workflow can be:

```text
1. snapshot D08 fault-list handoff
2. snapshot D07 map and alarm proposal
3. snapshot D04 endpoint inventory
4. read or generate simulation metadata
5. inspect VCD inventory
6. build good-machine context manifest
7. check hierarchy mapping
8. build FTTI window plan
9. build observe point candidate map
10. generate D10 and D11 handoff files
```

If a local simulator or waveform generator is configured, the demo may optionally run a reference simulation. If not, it can still validate the context files and produce a reviewable package.

---

## 28. Example Simulation Context Manifest

A compact D09 manifest may look like:

```csv
field,value
demo_id,D09_simulation_safety_context
design_top,toy_counter
testbench_top,tb_toy_counter
dut_path,tb_toy_counter.u_dut
vcd_file,inputs/simulation/toy_counter_good.vcd
vcd_timescale,1ns
clock_signal,tb_toy_counter.clk
reset_signal,tb_toy_counter.rst_n
reset_release_time,20ns
first_injection_time,50ns
simulation_end_time,1000ns
ftti_cycles,64
clock_period,10ns
observe_policy,endpoint_and_observe_point
```

This file answers the reviewer's first questions:

```text
What waveform is being used?
Where is the DUT inside the waveform?
When can faults be injected?
What is the FTTI?
Which signals define timing?
```

D09 should make these answers explicit.

---

## 29. Example VCD Signal Requirement Matrix

D09 can generate a signal requirement matrix:

```csv
signal_name,required,present,group,reason
tb_toy_counter.clk,yes,yes,clock,cycle alignment
tb_toy_counter.rst_n,yes,yes,reset,initialization context
tb_toy_counter.u_dut.count[0],yes,yes,endpoint,state comparison
tb_toy_counter.u_dut.alarm,yes,yes,alarm_candidate,detection signal
tb_toy_counter.u_dut.error_flag,yes,no,alarm_candidate,D10 review needed
tb_toy_counter.u_dut.safe_state,yes,no,observe_point,D10 review needed
```

This is more useful than simply saying:

```text
VCD exists
```

A VCD can exist and still miss critical signals.

The signal matrix tells D10 which alarms and observe points are already visible and which require design, testbench, or dump configuration changes.

---

## 30. Example FTTI Window Plan

D09 should also produce an FTTI plan.

Example:

```csv
scenario_id,injection_start_cycle,injection_end_cycle,ftti_cycles,post_fault_observe_cycles,status
SCN_ACTIVE_0,50,120,64,80,PASS
SCN_ACTIVE_1,150,220,64,80,PASS
SCN_IDLE,260,300,64,80,WARN
```

This plan does not inject faults yet. It defines where fault injection will be meaningful.

A quality gate can check:

```text
injection_end + FTTI <= simulation_end
reset_release < injection_start
clock is active during window
required observe points are present
```

This reduces avoidable failures in D11 and D12.

---

## 31. Observe Point Candidate Map

D09 prepares observe point candidates for D10.

Example:

```csv
observe_point_id,signal_name,source,reason,priority
OP_COUNT,tb_toy_counter.u_dut.count,D04 endpoint inventory,state behavior
OP_ALARM,tb_toy_counter.u_dut.alarm,D07 alarm proposal,detection signal
OP_SAFE_STATE,tb_toy_counter.u_dut.safe_state,manual review,safe-state evidence
OP_PROTOCOL_VALID,tb_toy_counter.u_dut.valid,D07 protocol-visible map,handshake evidence
```

The key idea is that D09 does not finalize all observe points. It identifies which signals are available in simulation context and which should be promoted in D10.

D10 will define the actual alarm list and observe point boundary.

---

## 32. D09 Quality Gate

D09 should have a quality gate.

Suggested checks:

```text
D08 handoff exists
D07 handoff exists
D04 endpoint inventory exists
VCD manifest exists
VCD file exists
DUT path is defined
clock signal is defined
reset signal is defined
FTTI is defined
simulation window is long enough
required endpoint signals are mapped
observe point candidates are generated
D10 handoff is generated
D11 handoff is generated
```

A typical status policy:

```text
PASS: ready for D10/D11
WARN: usable but needs review
FAIL: context cannot support fault campaign setup
```

Warnings are acceptable when they are explicit. Hidden context assumptions are not acceptable.

---

## 33. Handoff to D10

D10 focuses on:

```text
Alarm List and Observe Point: fault outcome observation boundary
```

D09 gives D10:

```text
observe point candidates
alarm candidate signal visibility
signal presence matrix
DUT path
VCD hierarchy mapping
FTTI policy
missing signal list
protocol context signals
```

D10 then decides:

```text
which signals become alarms
which signals become observe points
which missing signals require waveform dump changes
which alarm names map to safety mechanisms
which observe policy should be used
```

D09 therefore prepares the evidence, but D10 finalizes the observation boundary.

---

## 34. Handoff to D11

D11 focuses on the complete fault campaign input package.

D09 gives D11:

```text
VCD list
good-machine context manifest
DUT path
clock / reset timing
simulation start time
FTTI
injection window
signal mapping
context quality status
```

D11 combines this with:

```text
D08 fault list
D10 alarm list
D10 observe point list
filelist
clock definition
fault campaign initialization file
database session settings
execution mode
```

D09 is therefore not an isolated waveform step. It is a required contributor to the full campaign package.

---

## 35. Common D09 Failure Modes

D09 helps catch common issues early:

```text
VCD exists but DUT path is wrong
VCD uses different RTL hierarchy than the fault list
clock is not dumped
reset never releases
simulation ends before FTTI can be observed
alarm candidate not dumped
observe point not dumped
endpoint names do not match D04 inventory
fault nodes from D08 are not visible or mappable
VCD contains excessive X values after reset
testbench random seed is not recorded
simulation is too short or too idle
```

Catching these in D09 is much cheaper than discovering them during a large fault campaign.

---

## 36. Methodology: Context First, Campaign Later

A mature safety verification flow does not jump directly from fault list to fault injection.

It first builds context:

```text
fault list
+ good machine
+ VCD
+ FTTI
+ observe point candidates
+ alarm candidates
+ hierarchy mapping
+ quality gate
```

Only after that should the flow build a fault campaign package.

The practical principle is:

> A fault campaign result is only as credible as the simulation context used to classify it.

D09 operationalizes that principle.

---

## 37. What Demo9 Should Demonstrate

The D09 demo should demonstrate the following:

```text
read D08 campaign fault list
read D07 safety mechanism map and alarm proposal
read D04 endpoint inventory
create or validate a good-machine VCD context
build a VCD inventory
build a signal presence matrix
define FTTI policy
define injection windows
generate observe point candidates
detect missing signals
generate D10 handoff
generate D11 handoff
produce evidence index and quality gate
```

It should not rely on real post-campaign logs. It should not classify detected, safe, unsafe, or unresolved faults yet. That belongs to D13.

D09 prepares the ground for those classifications.

---

## 38. Expected D09 Outputs

A useful D09 output set:

```text
outputs/vcd_inventory.csv
outputs/good_machine_context.csv
outputs/hierarchy_mapping.csv
outputs/signal_presence_matrix.csv
outputs/ftti_policy.csv
outputs/ftti_window_plan.csv
outputs/observe_point_candidate_map.csv
outputs/missing_signal_review.csv
outputs/simulation_context_quality_gate.csv
outputs/d09_handoff_to_d10.csv
outputs/d09_handoff_to_d11.csv
outputs/evidence_index.csv
outputs/demo_summary.md
```

Each file has a role:

```text
vcd_inventory.csv
  what simulation data exists

good_machine_context.csv
  how golden context is defined

hierarchy_mapping.csv
  how testbench hierarchy maps to design hierarchy

signal_presence_matrix.csv
  whether required signals are present

ftti_window_plan.csv
  where fault injection can be observed safely

observe_point_candidate_map.csv
  what D10 should review

d09_handoff_to_d10.csv
  alarm / observe input handoff

d09_handoff_to_d11.csv
  campaign setup input handoff
```

This creates a traceable bridge from D08 to D11.

---

## 39. Closing View

D09 is the transition from static safety evidence to dynamic safety evidence.

D08 produces the fault population. D09 defines the runtime reference. D10 defines observation boundaries. D11 packages the campaign. D12 injects faults. D13 classifies outcomes. D14 feeds the results back into final metrics.

The essential idea is simple:

```text
Do not inject a fault before you know what normal behavior looks like.
```

For automotive chip functional safety, that normal behavior must be captured, mapped, timed, and reviewed. VCD, good machine, FTTI, and observe points are the language of that capture.

D09 turns that language into an engineering artifact.
