# Automotive Safe-IC Practice 08: Fault List Generation — From Safety Analysis to Fault Campaign Input
Author: Darren H. Chen  
Direction: Automotive chip functional safety analysis and fault injection practice  
Demo: D08_fault_list_generation_from_safety_analysis_to_fault_campaign_input  
Tags: ISO 26262, Functional Safety, Fault List, Fault Injection, Diagnostic Coverage, Safety Mechanism, FMEDA, VCD, Fault Campaign, Evidence Traceability

---

## 1. The Place Where Safety Analysis Becomes Executable Verification

A safety analysis flow can tell us which structures contribute to FIT, which endpoints are safety-relevant, and which safety mechanisms are expected to improve diagnostic coverage. But fault injection needs something more concrete: it needs a list of specific fault targets that can be injected, simulated, classified, and traced back to the safety argument.

That is the purpose of D08.

D08 is not another FIT calculation step. It is the point where the analysis-side view of the design is transformed into a campaign-side input population:

```text
analysis evidence
  -> structural safety model
  -> safety mechanism map
  -> prioritized fault population
  -> fault campaign input package
```

A fault list is the bridge between “this endpoint matters” and “inject a fault here under this safety context and observe the outcome.”

In a mature automotive safety flow, fault list generation should not be treated as a file-format conversion. It is an engineering decision layer. It decides what fault space is represented, which nodes are safety-critical, how permanent and transient faults are encoded, how sampling is controlled, how fault targets are connected back to failure modes, and how the next stages can reproduce the campaign.

---

## 2. D08 in the 20-Demo Flow

D08 is the eighth demo in this Safe-IC functional safety practice sequence. Its specific theme is:

```text
Fault List Generation: from safety analysis to fault campaign input
```

The upstream stages already established the required context:

```text
D01: analysis input package
D02: base FIT rate and FIT contribution
D03: FIT standard selection and mission-profile variants
D04: endpoint, startpoint, cone, and DCE structural model
D05: common evidence database/session organization
D06: what-if safety mechanism exploration and DC estimation
D07: failure mode to safety mechanism / endpoint mapping
```

D08 consumes these artifacts and prepares the fault population that will later be used by:

```text
D09: simulation safety context
D10: alarm list and observe point boundary
D11: fault campaign setup package
D12: fault injection execution
D13: fault outcome classification
D14: result write-back and final metrics
```

This placement is important. A fault list generated too early is often structurally correct but safety-blind. A fault list generated too late is difficult to trace back to FIT contribution, structural endpoints, and FMEDA assumptions. D08 sits in the middle.

---

## 3. What a Fault List Actually Represents

A fault list is a machine-readable representation of a fault population.

At minimum, it describes:

```text
where a fault is injected
what fault model is used
when the fault is injected
how long the fault lasts
how the fault relates to safety mechanisms and failure modes
```

For a permanent stuck-at fault, the list may describe:

```text
target node: u_ctrl.state[2]
fault type: stuck-at-0
injection time: 100 ns
duration: permanent
```

For a transient fault, the list may describe:

```text
target node: u_pipe.valid_q
fault type: bit flip or transient value override
injection time: 250 ns
duration: 2 cycles or a time window
```

In practice, fault lists can exist in multiple levels of maturity:

```text
analysis-native fault list
reviewable fault candidate table
campaign-ready fault list
sampled fault subset
collapsed primary-fault list
fault-list manifest
fault-to-FMEDA trace table
```

D08 should produce more than one file, because different consumers need different views.

---

## 4. Fault List Is Not Diagnostic Coverage

A common misunderstanding is to treat a generated fault list as evidence of safety coverage.

It is not.

A fault list is only the test population. It does not prove that a safety mechanism detects anything. Diagnostic coverage becomes credible only after the fault campaign classifies each fault outcome or after an accepted analytical model supports the credit.

The distinction is:

```text
Fault list:
  what will be tested

Fault campaign:
  what happened when faults were injected

Fault classification:
  detected / safe / unsafe / unresolved

Final metric calculation:
  what coverage and residual risk can be credited
```

D08 therefore should not claim final DC. It should provide a clean, traceable, reviewable fault population.

---

## 5. Inputs Consumed by D08

A robust D08 implementation reads from the preceding stages rather than inventing a new design boundary.

Typical D08 inputs include:

```text
D01 design filelist and clock definition
D02 base FIT and FIT contribution summary
D03 selected FIT standard and run identity
D04 endpoint inventory, startpoint inventory, cone map, DCE catalog
D05 common database/session manifest
D06 safety exploration scenario and candidate SM assignments
D07 failure-mode-to-endpoint map and EP-to-SM map
```

This means D08 should be able to answer:

```text
Which design version is this fault list based on?
Which FIT standard and mission profile are associated with it?
Which endpoints and cones are included?
Which failure modes justify each fault target?
Which safety mechanism is expected to detect or control the fault?
Which later campaign will consume the list?
```

Without these answers, a fault list may run, but it is weak as safety evidence.

---

## 6. Outputs Produced by D08

D08 should produce outputs for both humans and tools.

A practical D08 output set can include:

```text
outputs/fault_scope_inventory.csv
outputs/fault_target_candidates.csv
outputs/permanent_fault_candidates.csv
outputs/transient_fault_candidates.csv
outputs/campaign_fault_list_permanent.list
outputs/campaign_fault_list_transient.list
outputs/campaign_fault_list_all.list
outputs/fault_priority_matrix.csv
outputs/fault_sampling_plan.csv
outputs/fault_list_manifest.csv
outputs/fault_to_failure_mode_trace.csv
outputs/fault_to_sm_trace.csv
outputs/d08_handoff_to_d09.csv
outputs/d08_handoff_to_d10.csv
outputs/d08_handoff_to_d11.csv
outputs/d08_quality_gate.csv
outputs/evidence_index.csv
outputs/demo_summary.md
```

The file names are less important than the separation of concerns:

```text
candidate generation
fault encoding
sampling
traceability
campaign handoff
quality gate
evidence index
```

---

## 7. The Two Worlds: Analysis Fault Space and Campaign Fault Space

Safety analysis and fault campaign execution do not always use the same representation.

Analysis-side fault space is usually tied to:

```text
endpoints
startpoints
logic cones
FIT contribution
diagnostic coverage elements
safety mechanism assumptions
```

Campaign-side fault space is tied to:

```text
simulation hierarchy
fault injection object
fault type
injection time
duration
safety context
alarm and observe boundary
```

D08 maps between them.

```text
Endpoint / cone evidence
  -> concrete injectability target
    -> campaign fault row
      -> traceable fault ID
```

This mapping is not trivial. A single endpoint may imply multiple internal fault sites. A single net may have multiple ports. A permanent stuck-at model may be enough for one analysis goal, while a transient model may be needed for architectural vulnerability or software-visible effects.

---

## 8. Permanent Faults

A permanent fault is a fault that remains active after injection.

The most common permanent digital models are:

```text
SA0: stuck-at-0
SA1: stuck-at-1
```

A stuck-at fault models a node being forced to a constant logic value. This is useful for random hardware faults that behave like persistent defects or stuck conditions.

A campaign-ready row may look like:

```text
# fault_target                         fault_type   injection_time   duration
u_top.u_ctrl.state_q[2]                SA0          100              -1
u_top.u_ctrl.state_q[2]                SA1          100              -1
```

A duration of `-1` is commonly used in campaign-oriented list formats to indicate a permanent fault. The exact convention should be documented in the D08 manifest.

Permanent fault lists are especially important for:

```text
register state corruption
control path stuck values
protocol signal stuck values
alarm path stuck values
configuration bit faults
```

---

## 9. Transient Faults

A transient fault is active only during a limited time window.

Transient faults model effects such as:

```text
single-event upset
temporary value corruption
soft error in a state element
short-lived logic disturbance
timing-window-dependent failure
```

A campaign-ready row may look like:

```text
# fault_target                         fault_type   injection_time   duration
u_top.u_pipe.valid_q                   1:0          250              2
u_top.u_pipe.data_q[7]                 SA1          310              1
```

For transient campaigns, the injection time and duration are not secondary details. They define the workload phase under which the fault is tested.

That is why D08 does not fully close transient campaign setup by itself. It prepares the target population and hands it to D09, where VCD, good-machine behavior, time windows, and observe context are formalized.

---

## 10. Net Faults and Port Faults

Fault list generation must distinguish net faults from port faults.

A net fault injects on a net or node. Conceptually, it drives the entire net to a value. This can be useful for certain structural analyses, but it may over-constrain the simulation because all connected loads see the forced value.

A port fault injects at a specific port boundary. It is more localized. In many fault simulation flows, port fault injection is preferred because it avoids forcing an entire net and better isolates the effect at a particular driver/load boundary.

The engineering distinction is:

```text
Net fault:
  target is a net
  effect can cover the whole connected node

Port fault:
  target is a port
  effect is localized to an instance/interface boundary
```

For D08, this means the target table should explicitly track:

```text
fault_target_kind = net | port | endpoint | instance_port | register_bit
```

Do not hide this distinction. It affects campaign runtime, fault equivalence, and interpretation.

---

## 11. Fault Modes and Fault Types

A fault list row should not be just a string. It should carry a normalized fault model.

Typical fault types include:

```text
SA0
SA1
SA0/SA1 pair
transient 0-to-1
transient 1-to-0
bit-flip
transition fault
time-delay fault
path-delay fault
user-defined fault model
```

D08 should at least support the core permanent and transient digital models:

```text
permanent stuck-at faults
transient fault windows
```

Other models can be staged for later enhancement. The important part is to make the model explicit and traceable:

```text
fault_model_id
fault_type
fault_value
injection_time_policy
duration_policy
campaign_support_status
```

---

## 12. Safety-Critical Node Selection

A full design may contain too many potential fault targets.

D08 should not blindly generate everything unless the purpose is exhaustive analysis on a very small design. For realistic SoCs, the initial fault population should be guided by safety relevance.

Possible selection signals include:

```text
endpoint criticality
FIT contribution
DCE association
failure mode severity
protocol visibility
safety mechanism coverage intent
D06 risk score
D07 failure-mode-to-endpoint mapping
FMEDA part/sub-part relevance
```

A simple scoring formula can be:

```text
fault_priority =
    normalized_fit_contribution
  * failure_mode_severity_weight
  * endpoint_observability_weight
  * uncoveredness_weight
  * protocol_visibility_weight
```

This is not a replacement for formal safety metrics. It is a prioritization method for constructing a useful campaign population.

---

## 13. Endpoint-Driven Fault Scope

An endpoint-driven fault scope starts from the endpoint inventory:

```text
endpoint_id
endpoint_signal
endpoint_class
owning_instance
bit_index
related_failure_mode
related_safety_mechanism
```

Then it expands each endpoint into candidate fault targets.

For example:

```text
endpoint: u_ctrl.state_q[2]
failure mode: state corruption
safety mechanism: endpoint parity
fault candidates:
  u_ctrl.state_q[2] SA0 permanent
  u_ctrl.state_q[2] SA1 permanent
  u_ctrl.state_q[2] transient bit flip
```

Endpoint-driven scoping works well when:

```text
the endpoint is directly safety-visible
a safety mechanism is attached to the endpoint
the fault campaign intends to validate detection of endpoint corruption
```

---

## 14. Cone-Driven Fault Scope

A cone-driven fault scope starts from the logic cone between startpoints and endpoints.

This is useful when a fault inside combinational logic can corrupt an endpoint even if the endpoint storage itself is protected.

A cone record may include:

```text
cone_id
endpoint_id
startpoint_id
intermediate_nodes
logic_depth
estimated_fit_weight
safety_mechanism
```

A cone-driven fault list may target:

```text
selected internal cone nets
instance input/output ports
primary startpoints
endpoint fan-in boundary
```

Cone-driven scoping is especially useful for mechanisms such as:

```text
cone duplication
cone startpoint duplication
triplication
protocol-level protection
```

A practical campaign may not inject every internal cone net. It may choose a representative, weighted, collapsed, or sampled subset.

---

## 15. Failure-Mode-Driven Fault Scope

D07 links failure modes to endpoints and safety mechanisms. D08 should preserve that trace.

For every generated fault, D08 should be able to answer:

```text
Which failure mode does this fault exercise?
Which endpoint or cone does it affect?
Which safety mechanism is expected to detect or control it?
Which FMEDA row will later receive the result?
```

A trace table can look like:

```text
fault_id
fault_target
fault_type
failure_mode_id
endpoint_id
mechanism_id
expected_alarm_group
fmda_part
fmda_sub_part
```

This avoids a common weakness in fault injection flows: thousands of faults are simulated, but the results are difficult to connect back to FMEDA.

---

## 16. From Safety Mechanism Map to Fault Population

D07 produces a safety mechanism map. D08 uses it to decide fault generation scope.

A simplified mapping is:

```text
EP-to-SM row:
  endpoint = u_ctrl.state_q[2]
  safety mechanism = endpoint parity
  alarm = sm_alarm_state

D08 expansion:
  generate SA0 and SA1 permanent faults on u_ctrl.state_q[2]
  generate transient bit-flip candidates if transient campaign is enabled
  associate all rows with sm_alarm_state as the expected alarm group
```

For cone-level mechanisms:

```text
EP-to-SM row:
  endpoint = u_pipe.result_q[7]
  safety mechanism = cone duplication
  alarm = sm_alarm_cone

D08 expansion:
  include selected fan-in cone targets
  include endpoint boundary targets
  associate rows with the same failure mode and mechanism
```

The safety mechanism map therefore acts as a policy input, not merely as annotation.

---

## 17. Campaign-Ready Fault List Format

A campaign-ready fault list often needs more columns than an analysis-generated native list.

A practical normalized format is:

```text
fault_target fault_type injection_time duration
```

Example:

```text
u_top.u_ctrl.state_q[0] SA0 100 -1
u_top.u_ctrl.state_q[0] SA1 100 -1
u_top.u_ctrl.state_q[1] 1:0 250 2
```

The first column identifies the injection target.

The second column identifies the fault type or value.

The third column identifies the injection time.

The fourth column identifies the duration. A negative duration can represent permanent injection, while a positive duration represents a transient window.

D08 can generate both:

```text
analysis-native list:
  target only, or target + fault model

campaign-ready list:
  target + type + injection time + duration
```

This distinction is crucial because analysis outputs may not contain timing information. D09 and D11 will complete or refine time-window settings based on simulation context.

---

## 18. Injection-Time Policy

D08 may not know the final VCD time windows yet, but it should define an injection-time policy.

Typical policies are:

```text
fixed_time
phase_relative_time
random_time_with_seed
per_fault_window
defer_to_D09
defer_to_D11
```

For example:

```text
fault_id,target,type,injection_time_policy,duration_policy
F0001,u_ctrl.state_q[0],SA0,fixed_time_100,permanent
F0002,u_pipe.valid_q,TRANSIENT,defer_to_D09,one_cycle
```

The policy is more important than the placeholder value. It tells downstream stages how to finalize the list.

D08 should not pretend that transient timing is final unless it has access to the simulation safety context.

---

## 19. Permanent and Transient List Separation

Even if a unified list is produced, D08 should preserve separate permanent and transient lists.

Recommended outputs:

```text
campaign_fault_list_permanent.list
campaign_fault_list_transient.list
campaign_fault_list_all.list
```

The separation helps because permanent and transient campaigns usually differ in:

```text
fault duration
fault model
simulation time window
classification interpretation
runtime cost
sampling policy
report grouping
```

It also helps D13 classification later. Permanent and transient faults may map to different diagnostic coverage evidence.

---

## 20. Fault Collapsing

Fault collapsing reduces a fault population by grouping equivalent faults.

For example, in simple logic:

```text
buffer input SA0 may be equivalent to output SA0
inverter input SA0 may be equivalent to output SA1
AND output SA0 may be equivalent to an input SA0
```

The goal is not to hide faults. The goal is to avoid simulating many faults that are equivalent under the selected model.

A collapsed fault record should preserve:

```text
primary_fault_id
equivalent_fault_id
equivalence_rule
collapsing_scope
collapsing_status
```

The campaign may simulate only primary faults, but the evidence trace must still know which equivalent faults are represented.

A safe D08 design should never collapse faults silently.

---

## 21. Primary Faults and Equivalent Faults

After collapsing, the flow distinguishes:

```text
primary fault:
  the representative fault actually selected for campaign

equivalent fault:
  a fault represented by the primary fault under the collapsing rule
```

The campaign-ready list may contain only primary faults, while an audit-friendly equivalence table stores:

```text
primary_fault_id
primary_target
primary_type
equivalent_target
equivalent_type
reason
```

This is important for explaining why some original fault candidates were not directly injected.

---

## 22. Statistical Sampling

For large designs, exhaustive fault injection may be impractical.

D08 can define a statistical sampling plan:

```text
sample size
coverage goal
confidence interval
random seed
sampling domain
sampling exclusions
repeatability constraints
```

A sampling record should include:

```text
sampling_plan_id
input_fault_count
selected_fault_count
coverage_goal
confidence_interval
random_seed
selection_algorithm
```

The seed matters. Without it, a sampled campaign cannot be reproduced.

Sampling is not merely a runtime trick. It is part of the safety argument, because it affects how campaign results are interpreted.

---

## 23. AVF-Oriented Fault Lists

Architectural Vulnerability Factor, or AVF, estimates the portion of transient faults that can affect architectural or program-visible behavior.

An AVF-oriented fault list typically requires:

```text
transient fault targets
random or workload-relative injection time
simulation start time
simulation end time
repeatable random seed
```

D08 can prepare the population and sampling plan, but D09 must provide the simulation safety context, and D11 must package the final campaign.

AVF-oriented lists are useful when:

```text
the target is transient error masking
the workload matters
software-visible behavior matters
architectural state propagation matters
```

D08 should mark AVF lists as a specialized transient campaign input, not as a generic permanent fault list.

---

## 24. Exclusion Rules

Not every structural node should become a campaign fault.

D08 may apply exclusions such as:

```text
test-only logic
debug-only signals
constant-tied nets
clock and reset distribution
unreachable logic
black-box internals
non-safety-relevant diagnostic-only plumbing
unsupported fault injection targets
```

Every exclusion should be logged.

Recommended file:

```text
outputs/fault_exclusion_report.csv
```

Fields:

```text
target
reason
source_rule
review_status
```

An exclusion without reason weakens traceability.

---

## 25. Inclusion Rules

D08 should also record why targets are included.

Inclusion reasons can include:

```text
high FIT contribution
mapped failure mode
D06 selected scenario
D07 EP-to-SM binding
protocol-visible endpoint
FMEDA critical sub-part
manual review override
```

A good fault target table includes:

```text
fault_id
target
include_reason
source_artifact
source_row_id
review_status
```

This makes the list defensible.

---

## 26. Alarm and Observe-Point Awareness

D08 does not finalize alarm list and observe points. That is D10.

However, D08 should preserve expected alarm groups and observe intent.

For example:

```text
fault target: u_ctrl.state_q[2]
expected mechanism: endpoint parity
expected alarm group: sm_alarm_state
observe intent: state corruption should reach alarm or safety-visible endpoint
```

This handoff helps D10 build:

```text
alarm list
observe point specification
alarm-to-fault mapping
fault outcome observation boundary
```

Without D08 preserving alarm intent, D10 would have to infer it again.

---

## 27. Safety Context Awareness

Fault injection is meaningful only under a safety context.

A fault injected during inactive reset may not test the intended safety function. A transient injected when the relevant pipeline is idle may be safe for reasons unrelated to the safety mechanism.

D08 therefore should tag faults with context requirements:

```text
requires_post_reset_context
requires_active_protocol_phase
requires_valid_data_phase
requires_error_response_window
requires_software_observable_state
```

D09 will later translate these requirements into VCD and good-machine context.

D08 should not over-specify timing. It should specify intent.

---

## 28. Filelist and Hierarchical Name Stability

Fault list targets are fragile if hierarchy names are unstable.

A reliable D08 flow should capture:

```text
design top module
filelist hash
RTL snapshot hash
DCE source
selected FIT standard
selected design boundary
naming normalization rule
```

If synthesis or hierarchy changes later, the fault list may no longer be valid.

A D08 quality gate should check:

```text
all target names are non-empty
target names are unique or intentionally duplicated
target paths match the current design boundary
port/net classification is present
bit indices are normalized
```

Name stability is an engineering requirement, not a cosmetic issue.

---

## 29. Common Database Session Strategy

D08 can work with files, database sessions, or both.

A file-based flow is transparent:

```text
CSV tables
fault list files
manifest
handoff files
quality gates
```

A database-based flow is useful for cross-tool continuity:

```text
analysis metrics
fault lists
fault simulation results
safety mechanism maps
alarm / observe settings
FMEDA mapping
```

A practical D08 strategy is:

```text
files for review and GitHub demo evidence
database/session links for tool flow continuity
```

D08 should preserve `.fdb::session` identity when available, but the public article and demo can describe it as a common safety database session rather than exposing tool-specific invocation details.

---

## 30. D08 Execution Architecture

A vendor-neutral D08 implementation can be organized into four layers.

```text
Layer 1: upstream evidence loader
Layer 2: fault scope builder
Layer 3: fault list encoder
Layer 4: evidence and handoff publisher
```

### Layer 1: Upstream Evidence Loader

Reads:

```text
D04 endpoint / cone / DCE evidence
D06 scenario and SM assignment
D07 failure-mode / EP-to-SM map
```

### Layer 2: Fault Scope Builder

Builds:

```text
candidate targets
inclusion/exclusion reason
priority score
failure-mode trace
mechanism trace
```

### Layer 3: Fault List Encoder

Writes:

```text
permanent list
transient list
combined list
sampled list
collapsed list
```

### Layer 4: Evidence Publisher

Writes:

```text
manifest
quality gate
handoff to D09 / D10 / D11
evidence index
summary
```

This structure keeps D08 extensible.

---

## 31. Reviewable Command Model

A D08 demo can expose commands through neutral environment variables and wrapper names.

Example:

```csh
#!/bin/csh -f

set ROOT = `cd "$0:h/.." && pwd`
cd "$ROOT"

if ( ! $?SAFEIC_ANALYSIS_ENGINE ) then
    echo "[WARN] SAFEIC_ANALYSIS_ENGINE is not configured."
    echo "[WARN] Running file-based fault-list preparation only."
else
    echo "[INFO] Analysis engine is configured."
endif

python3 tools/build_d08_fault_list_package.py \
    --d07-root "$D07_ROOT" \
    --d06-root "$D06_ROOT" \
    --d05-root "$D05_ROOT" \
    --d04-root "$D04_ROOT" \
    --output outputs
```

When a real analysis engine is available, a generated review command can use the engine to produce native fault-list artifacts. The public script should keep that engine behind an environment variable and preserve the command as evidence.

The point is not to hide execution. The point is to keep the demo portable while still supporting a real tool environment.

---

## 32. Fault List Manifest

The manifest is the central identity file for D08.

Recommended fields:

```text
demo_id
top_module
design_boundary_id
input_filelist_hash
fit_standard
mission_profile_id
source_d03_session
source_d04_structural_model
source_d06_scenario
source_d07_sm_map
fault_list_type
fault_count
permanent_fault_count
transient_fault_count
sampled_fault_count
collapsed_fault_count
random_seed
generation_timestamp
```

The manifest answers:

```text
What is this fault list?
Where did it come from?
What does it represent?
How can it be reproduced?
```

---

## 33. Quality Gate

A D08 quality gate should catch structural and traceability problems before D09/D10/D11 consume the list.

Suggested checks:

```text
D07 SM map exists
selected D06 scenario exists
endpoint inventory exists
fault target count > 0
permanent fault list generated
transient fault list generated or explicitly disabled
all fault IDs unique
all targets non-empty
all faults trace to endpoint or failure mode
all campaign rows have valid fault type
all campaign rows have injection-time policy
all permanent faults use permanent duration
all transient faults use non-permanent duration policy
sampling seed recorded if sampling enabled
handoff files generated
```

The quality gate should distinguish:

```text
FAIL: unsafe to continue
WARN: review required but downstream can inspect
PASS: ready for handoff
```

---

## 34. Handoff to D09, D10, and D11

D08 does not end the campaign preparation. It produces structured handoff.

### D08 to D09

```text
fault targets requiring simulation context
transient timing policies
workload phase requirements
good-machine context requirements
```

### D08 to D10

```text
expected alarm groups
observe intent
fault-to-alarm association hints
faults requiring explicit observe points
```

### D08 to D11

```text
campaign-ready fault lists
fault list manifest
fault population summary
quality gate result
tool-native and reviewable list paths
```

This keeps each later demo focused.

---

## 35. Typical Mistakes in Fault List Generation

Several mistakes appear repeatedly in safety flows.

### Mistake 1: Treating All Nodes as Equally Important

Exhaustive generation may be possible for small demos, but large designs require prioritization and traceability.

### Mistake 2: Losing Failure-Mode Trace

A fault without failure-mode context is difficult to use in FMEDA.

### Mistake 3: Mixing Permanent and Transient Faults Without Tags

Permanent and transient models require different campaign handling.

### Mistake 4: Treating Analysis-Native Lists as Campaign-Ready Lists

Analysis-native lists may not contain injection time or duration.

### Mistake 5: Sampling Without Seed

Sampling without seed is not reproducible.

### Mistake 6: Ignoring Port vs Net Semantics

Port and net faults can behave differently in simulation.

### Mistake 7: Hiding Exclusions

Every excluded target should have a reason.

---

## 36. Example Directory Structure

A clean D08 demo directory can look like:

```text
D08_fault_list_generation_from_safety_analysis_to_fault_campaign_input/
  README.md
  scripts/
    run_demo.csh
    run_demo.sh
  tools/
    build_d08_fault_list_package.py
    normalize_fault_targets.py
    build_fault_manifest.py
  configs/
    fault_generation_policy.csv
    fault_sampling_policy.csv
    fault_exclusion_rules.csv
  inputs/
    from_D07/
    from_D06/
    from_D05/
    from_D04/
  outputs/
    fault_scope_inventory.csv
    fault_target_candidates.csv
    permanent_fault_candidates.csv
    transient_fault_candidates.csv
    campaign_fault_list_permanent.list
    campaign_fault_list_transient.list
    campaign_fault_list_all.list
    fault_priority_matrix.csv
    fault_sampling_plan.csv
    fault_list_manifest.csv
    fault_to_failure_mode_trace.csv
    fault_to_sm_trace.csv
    d08_handoff_to_d09.csv
    d08_handoff_to_d10.csv
    d08_handoff_to_d11.csv
    d08_quality_gate.csv
    evidence_index.csv
    demo_summary.md
  logs/
```

This makes D08 understandable as a standalone engineering artifact while keeping it connected to the full flow.

---

## 37. Methodology Summary

D08 turns safety analysis output into fault campaign input.

Its core method is:

```text
1. Load structural and safety evidence from D04-D07.
2. Build a fault target population from endpoints, cones, failure modes, and SM maps.
3. Separate permanent and transient models.
4. Encode campaign-ready rows.
5. Preserve fault-to-failure-mode and fault-to-SM traceability.
6. Apply exclusion, prioritization, collapsing, or sampling policies.
7. Publish manifest, quality gate, and handoff files.
```

D08 is successful when it can produce a fault population that is:

```text
traceable
reviewable
reproducible
campaign-ready
connected to FMEDA evidence
```

A good fault list is not just something the fault engine can read. It is a safety evidence object that explains why each injected fault exists in the campaign.

---

## 38. Demo Deliverables

The D08 demo should deliver:

```text
[ ] fault scope inventory
[ ] permanent fault candidate table
[ ] transient fault candidate table
[ ] campaign-ready permanent fault list
[ ] campaign-ready transient fault list
[ ] combined fault list
[ ] fault priority matrix
[ ] sampling plan
[ ] fault-to-failure-mode trace table
[ ] fault-to-SM trace table
[ ] fault list manifest
[ ] D09 handoff
[ ] D10 handoff
[ ] D11 handoff
[ ] evidence index
[ ] quality gate
[ ] summary
```

Once these are in place, the flow is ready to move from structural safety preparation into simulation safety context and fault campaign setup.

D09 will define the safety context. D10 will define observation boundaries. D11 will package the campaign. D12 will execute fault injection. D13 will classify outcomes. D14 will feed those outcomes back into final metrics.

D08 is the point where that future campaign first becomes a concrete list of faults.
