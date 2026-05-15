# Automotive Safe-IC Practice 07: Safety Mechanism Map — From Failure Mode to SM / EP Mapping

Author: Darren H. Chen  
Direction: Automotive chip functional safety analysis and fault-injection platform engineering  
Demo: D07_safety_mechanism_map_failure_mode_to_sm_ep  
Tags: ISO 26262, Safe-IC, FMEDA, safety mechanism, diagnostic coverage, endpoint, failure mode, fault injection, evidence database

---

## 1. The Point Where Architecture Meets Evidence

In the previous steps, we built the foundation of a functional safety analysis flow.

D01 defined the analysis input package. D02 established the Base FIT Rate. D03 made the FIT standard explicit. D04 extracted the structural building blocks of the design: endpoint, startpoint, logic cone, and DCE-style artifacts. D05 organized these work products into a common evidence center. D06 used the evidence center to perform what-if safety exploration and estimate the diagnostic effect of candidate safety mechanisms.

D07 is the moment when a safety mechanism stops being just an engineering idea and becomes a traceable mapping.

A safety mechanism is not useful merely because it exists in the design. It becomes useful for functional safety analysis only when we can answer four questions:

```text
Which failure mode does it address?
Which design structure does it protect?
Which endpoint or instance is credited?
Which alarm, checker, or diagnostic path proves observability?
```

This article focuses on the map that connects these answers.

The map is not simply a convenience file. It is the bridge between FMEDA intent, RTL structure, diagnostic coverage estimation, fault-list generation, and later fault campaign execution.

---

## 2. D07 in the Engineering Flow

D07 is named:

```text
Safety Mechanism Map: failure mode to SM / EP mapping
```

Its role is to turn the result of D06 safety exploration into a reviewable safety mechanism mapping package.

At the end of D06, we may have several what-if scenarios:

```text
endpoint parity
protocol-visible parity
high-risk cone duplication
triplication for high ASIL targets
mixed balanced protection
```

But a what-if scenario is still a proposal. It says that a protection strategy may improve diagnostic coverage. D07 turns this proposal into an explicit mapping:

```text
failure_mode -> sub_part -> endpoint/instance -> safety_mechanism -> alarm/diagnostic path
```

This mapping is then consumed by later steps:

```text
D08 fault list generation
D11 fault campaign setup
D14 final metric update
D15 FMEDA data model
D16 top-down FMEDA flow
D17 diagnostic coverage closure
```

The key idea is that a safety mechanism map is not only a tool input. It is also a safety argument artifact.

---

## 3. What a Failure Mode Means in This Context

A failure mode describes how a hardware function can fail from the viewpoint of the safety goal or the FMEDA.

Examples:

```text
counter value stuck
state machine enters illegal state
configuration write is corrupted
memory data bit is flipped
bus response is inconsistent with request
alarm output is not asserted when required
control path takes an unintended transition
```

A failure mode is not the same as a fault.

A fault is a physical or logical defect model:

```text
stuck-at-0
stuck-at-1
transient bit flip
delay fault
memory cell corruption
```

A failure mode is the externally meaningful way the function fails:

```text
wrong data delivered
control decision corrupted
protection not triggered
unsafe command accepted
```

The safety mechanism map exists between these two levels.

It connects a functional failure concern to the structural locations where faults can occur and to the diagnostic mechanism expected to detect or control them.

---

## 4. What a Safety Mechanism Means

A safety mechanism, abbreviated as SM, is a technical measure used to detect, control, correct, or tolerate a fault.

Common examples include:

```text
parity
ECC
CRC
lockstep
duplication
triplication
watchdog
timeout monitor
range checker
protocol checker
transition checker
alarm aggregation
built-in self-test
software-implemented check
```

For D07, we focus on safety mechanisms from a mapping perspective.

A safety mechanism has at least five properties:

```text
mechanism_id
mechanism_type
coverage_scope
alarm_or_observe_path
credited_failure_mode
```

For example:

```text
mechanism_id       = SM_COUNTER_PARITY
mechanism_type     = endpoint parity
coverage_scope     = counter_state endpoints
alarm_or_observe   = counter_parity_error
failure_mode       = counter state corruption
```

The same mechanism type can appear in different engineering contexts. Endpoint parity for a counter state register and parity on a protocol payload are not the same safety argument, even if both use parity logic.

---

## 5. Endpoint, Instance, Cone, and Map Granularity

A safety mechanism map can be created at several levels of granularity.

The most common levels are:

```text
endpoint-level mapping
instance-level mapping
cone-level mapping
end-to-end path mapping
```

Endpoint-level mapping is precise. It assigns a mechanism to individual endpoint objects, such as state elements or observable outputs.

Instance-level mapping is coarser. It says that every relevant endpoint under an instance is protected by the same mechanism.

Cone-level mapping focuses on the region of logic between selected startpoints and endpoints.

End-to-end path mapping is used when the protection is not local to one endpoint but spans a generation point and a checking point.

The trade-off is simple:

```text
higher precision -> better reviewability, more mapping work
higher abstraction -> easier authoring, higher risk of over-crediting
```

D07 should not blindly choose the easiest mapping. It should choose the mapping granularity that matches how the safety mechanism actually works.

---

## 6. Why Endpoint Mapping Is Often the First Practical Step

Endpoint mapping is practical because endpoints are the places where incorrect state or incorrect output becomes visible to the next sequential boundary or to the outside world.

For RTL and netlist analysis, endpoints often include:

```text
state element inputs
black-box inputs
top-level outputs
protocol response outputs
alarm outputs
status outputs
```

If a fault propagates to an endpoint, the design may preserve or expose an incorrect value. Therefore, endpoints are natural targets for diagnostic coverage modeling.

When a map says:

```text
endpoint A is covered by safety mechanism B
```

it is effectively making this claim:

```text
fault effects reaching endpoint A can be detected or controlled by mechanism B under the stated assumptions
```

This claim must be supported by design structure, failure-mode intent, and later verification evidence.

---

## 7. Failure Mode to Endpoint Is a Many-to-Many Relationship

A common mistake is to assume that one failure mode maps to one endpoint.

In a real design, the relationship is usually many-to-many.

One failure mode may involve many endpoints:

```text
failure mode: bus response corruption
endpoints: response_valid, response_data, response_error, response_id
```

One endpoint may participate in multiple failure modes:

```text
endpoint: control_state[2]
failure modes:
  illegal control transition
  timeout not issued
  wrong arbitration decision
```

Therefore, the D07 map should not be a flat list without context. It needs columns that preserve intent:

```text
failure_mode_id
function_or_sub_part
endpoint_or_instance
safety_mechanism
alarm_signal
coverage_role
mapping_rationale
review_status
```

The map becomes useful only when the reviewer can understand why a particular endpoint is credited under a particular failure mode.

---

## 8. Failure Mode to SM Is Also Not One-to-One

A second common mistake is to assume that one failure mode is addressed by one safety mechanism.

In practice, a failure mode may require several mechanisms:

```text
failure mode: memory data corruption
mechanisms:
  ECC for data correction
  parity for metadata protection
  scrubber for latent error reduction
  alarm path for uncorrectable error reporting
```

The opposite is also true. One safety mechanism may cover multiple failure modes:

```text
mechanism: protocol timeout checker
failure modes:
  request lost
  response missing
  slave not responding
  handshake stuck
```

D07 should represent this explicitly. The map should not force artificial one-to-one relationships if the design architecture is not one-to-one.

A better approach is to treat the map as a graph:

```text
Failure Mode Node
    -> Safety Mechanism Node
        -> Endpoint / Instance Node
            -> Alarm / Observe Node
```

This graph model will later help D15 FMEDA modeling and D17 diagnostic coverage closure.

---

## 9. The Role of the Alarm Signal

An alarm signal is an observable indication that a safety mechanism has detected a fault or an abnormal condition.

Examples:

```text
parity_error
ecc_uncorrectable
crc_mismatch
lockstep_mismatch
watchdog_timeout
protocol_violation
range_check_error
state_transition_error
```

In early what-if exploration, the alarm may not yet exist in the RTL. In that case, the mapping can use a placeholder such as:

```text
alarm_signal = NULL
```

But this should not be treated as final proof.

A placeholder alarm is acceptable for architectural exploration. It is not sufficient for signoff. Later stages must resolve it into a real alarm, observe point, fault outcome rule, or diagnostic path.

D07 should therefore distinguish two states:

```text
planned diagnostic path
implemented diagnostic path
```

This distinction prevents the map from giving final diagnostic credit to a mechanism that is only proposed.

---

## 10. The Difference Between Detection and Control

Detection means the system notices the fault effect.

Control means the system prevents the fault effect from violating a safety goal or moves the system to a safe state.

These are not identical.

For example:

```text
parity detects corrupted data
ECC may correct corrupted data
lockstep detects divergent execution
TMR can vote and continue operation
watchdog detects missing progress
reset or safe shutdown controls the hazard
```

D07 should capture the role of the mechanism:

```text
detect-only
correct-and-continue
detect-and-reset
detect-and-degrade
detect-and-isolate
fail-operational
fail-safe
```

This matters because diagnostic coverage is not only about whether an alarm toggles. It is about whether the hazardous effect is avoided or controlled within the required time constraints.

---

## 11. Primary and Latent Safety Mechanisms

A primary safety mechanism protects the function.

A latent safety mechanism protects the primary safety mechanism or detects whether the primary mechanism has become unavailable.

Example:

```text
primary SM: data path parity
latent SM: periodic parity-checker self-test
```

Another example:

```text
primary SM: lockstep comparator
latent SM: comparator stuck-at monitor
```

This distinction becomes important for latent fault metric reasoning. A safety architecture that protects a function but leaves the safety mechanism itself unprotected may still have latent risk.

D07 can prepare for this by including fields such as:

```text
sm_role = primary | latent | supporting
protects_sm = <primary_sm_id>
latent_check_interval
```

The mapping package does not need to solve the full LFM closure, but it should not lose the relationship between primary and latent mechanisms.

---

## 12. Local Protection vs End-to-End Protection

A local safety mechanism protects a local endpoint, register, or instance.

Examples:

```text
register parity
local ECC
state encoding check
local range check
```

An end-to-end safety mechanism protects a path between a generation point and a checking point.

Examples:

```text
CRC from packet generation to packet consumption
request/response tag consistency
bus payload parity across an interconnect
transaction ID protection from master to slave
```

End-to-end protection is common in bus fabrics, NoC paths, DMA paths, sensor data paths, and control-message protocols.

D07 should not flatten end-to-end protection into random endpoint assignments without preserving the generation/check relationship.

A useful end-to-end map includes:

```text
generation_point
check_point
protected_payload
protected_control_fields
intermediate_instances
covered_failure_mode
alarm_signal
```

This is especially important for protocol-visible safety mechanisms.

---

## 13. Protocol-Visible Endpoints

A protocol-visible endpoint is a signal or state element whose fault effect is visible at a protocol boundary.

Examples:

```text
valid
ready
request
acknowledge
response
error
address
data
write_enable
byte_enable
transaction_id
interrupt
alarm
```

Protocol-visible endpoints matter because many automotive chips are built from IP blocks connected through standardized or semi-standardized bus protocols.

A fault in a protocol signal can produce failure modes such as:

```text
transaction lost
transaction duplicated
wrong address accepted
write applied to wrong register
error response suppressed
status bit reported incorrectly
interrupt missed
```

For D07, protocol-visible endpoints need a stronger mapping rationale than generic register endpoints. The map should explain how the mechanism observes or controls protocol behavior.

Examples:

```text
valid/ready handshake timeout -> timeout monitor
address/control field corruption -> parity or CRC
response mismatch -> protocol checker
transaction ID mismatch -> ID consistency checker
```

---

## 14. Failure Mode Library vs Design-Specific Failure Mode

A failure mode library provides reusable categories.

Examples:

```text
state corruption
data corruption
control corruption
address corruption
timeout
stale value
illegal transition
missing alarm
spurious alarm
```

A design-specific failure mode binds the category to a concrete function.

Examples:

```text
FM_COUNTER_STATE_CORRUPTION
FM_TIMER_TIMEOUT_MISSED
FM_STATUS_REGISTER_STALE
FM_BUS_WRITE_ADDRESS_CORRUPTION
```

D07 should support both.

The library-level category makes the methodology reusable. The design-specific failure mode makes the map reviewable.

A good map might contain:

```text
failure_mode_category = state corruption
failure_mode_id       = FM_COUNTER_STATE_CORRUPTION
sub_part              = counter_control
endpoint              = u_counter.count[3:0]
safety_mechanism      = SM_COUNTER_PARITY
```

This structure is much more useful than a mechanism list with no failure-mode context.

---

## 15. FMEDA Part and Sub-Part Context

FMEDA models rarely operate directly on raw RTL signals.

They usually organize the design into:

```text
part
sub-part
function
failure mode
safety mechanism
metric contribution
```

D04 and D05 already prepared structural and evidence context. D07 should bind that context to FMEDA modeling.

Example:

```text
part       = control_unit
sub_part   = counter_state_machine
function   = count and alarm generation
failure    = state corruption
endpoint   = u_counter.count_reg[3]
sm          = endpoint parity
alarm       = counter_parity_error
```

This mapping allows D15 to build a meaningful FMEDA data model later.

Without this binding, the FMEDA becomes a spreadsheet disconnected from actual design structure.

---

## 16. What the D07 Demo Should Produce

The D07 demo should not simply generate one text file.

It should produce a mapping package.

A practical output set includes:

```text
outputs/failure_mode_library.csv
outputs/failure_mode_to_subpart_map.csv
outputs/sm_catalog.csv
outputs/ep_to_sm_map.csv
outputs/instance_to_sm_map.csv
outputs/end_to_end_sm_map.csv
outputs/alarm_path_map.csv
outputs/mapping_consistency_checks.csv
outputs/d07_quality_gate.csv
outputs/d07_handoff_to_d08.csv
outputs/d07_handoff_to_d15.csv
outputs/evidence_index.csv
outputs/demo_summary.md
```

The goal is not to claim final safety closure. The goal is to create a traceable map that later steps can consume.

---

## 17. Proposed Map Schema

A robust EP-to-SM map can use the following conceptual schema:

```text
map_id
failure_mode_id
part_id
sub_part_id
mapping_scope
endpoint_or_instance
startpoint_context
cone_id
safety_mechanism_id
safety_mechanism_type
alarm_signal
coverage_kind
coverage_credit_policy
source_scenario
rationale
review_status
```

Where:

```text
mapping_scope = endpoint | instance | cone | end_to_end
coverage_kind = permanent | transient | both
review_status = proposed | reviewed | implemented | verified
```

The map should be machine-readable, but it should also be understandable by a human reviewer.

Do not hide the rationale in a script. Put it in the evidence file.

---

## 18. Example Mapping Record

A simplified mapping record may look like this:

```csv
map_id,failure_mode_id,sub_part_id,mapping_scope,endpoint_or_instance,safety_mechanism_id,alarm_signal,coverage_kind,review_status
MAP_0001,FM_COUNTER_STATE_CORRUPTION,counter_state,endpoint,u_counter.count_reg[0],SM_COUNTER_PARITY,counter_parity_error,both,proposed
MAP_0002,FM_COUNTER_STATE_CORRUPTION,counter_state,endpoint,u_counter.count_reg[1],SM_COUNTER_PARITY,counter_parity_error,both,proposed
MAP_0003,FM_COUNTER_ALARM_MISSED,alarm_logic,cone,CONE_ALARM_PATH,SM_CONE_DUPLICATION,alarm_compare_error,permanent,proposed
```

The first two rows map individual endpoints to a parity mechanism. The third row maps a logic cone to duplication.

The important point is not the exact CSV format. The important point is that the map preserves:

```text
failure intent
structural target
mechanism identity
alarm observability
review state
```

---

## 19. How D07 Uses D06 Safety Exploration

D06 estimated the diagnostic benefit of candidate safety mechanisms.

D07 should consume D06 outputs such as:

```text
candidate endpoint risk score
candidate SM assignment
what-if DC estimate
residual FIT estimate
safety exploration decision matrix
reviewable SM files
```

D07 then transforms these into a more formal mapping package.

The transformation is conceptually:

```text
candidate assignment -> reviewed map candidate
scenario -> mechanism family
risk score -> mapping priority
residual FIT impact -> review order
SM file -> tool-consumable map
```

D07 should not blindly accept every D06 recommendation. It should classify mappings as:

```text
accepted
needs review
rejected
requires design change
requires alarm implementation
requires protocol interpretation
```

This makes the map an engineering artifact, not just an automatic output.

---

## 20. Consistency Rules for the Map

A safety mechanism map should pass basic consistency checks before it is used by D08 or D15.

Examples:

```text
every mapped endpoint exists in the D04 endpoint inventory
every mapped instance exists in the D04 structure catalog
every failure mode is assigned to a part or sub-part
every safety mechanism exists in the SM catalog
every non-NULL alarm exists as an endpoint or observe point candidate
every end-to-end map has both generation and check points
no endpoint is credited twice with incompatible mechanisms
no placeholder alarm is treated as final verification evidence
```

These checks do not prove safety. They prevent obvious traceability mistakes.

A failed consistency check should block downstream automation.

---

## 21. Avoiding Over-Crediting

Over-crediting happens when the map assigns more diagnostic coverage than the design can justify.

Common causes:

```text
mapping an entire instance when only one endpoint is protected
crediting an alarm that is not connected to the safety response
using parity credit for a multi-bit corruption case without justification
assuming a protocol checker covers data payload corruption
assuming duplication covers common-cause faults
crediting a placeholder alarm as if it were implemented
```

D07 should enforce a conservative policy:

```text
If the mechanism is proposed but not implemented, mark it as proposed.
If the alarm is not real, mark it as NULL or planned.
If the scope is uncertain, map at endpoint level rather than instance level.
If the mechanism detects but does not control the hazard, do not describe it as corrective.
```

This is a core safety engineering discipline.

---

## 22. Handling Multiple Mechanisms on the Same Endpoint

Sometimes one endpoint is covered by more than one mechanism.

Example:

```text
state register protected by parity
state transition protected by illegal-transition checker
state machine monitored by watchdog
```

This can be valid, but the map must preserve the difference in coverage roles.

Suggested fields:

```text
mechanism_priority
coverage_dimension
common_cause_assumption
aggregation_policy
```

Example:

```text
parity -> bit-level corruption detection
transition checker -> illegal state transition detection
watchdog -> temporal progress detection
```

These are not identical coverage claims. They should not be merged into a single generic “protected” label.

---

## 23. Safety Mechanism Catalog

D07 should maintain a safety mechanism catalog.

A catalog entry may include:

```text
sm_id
sm_type
mechanism_family
coverage_target
fail_safe_or_fail_operational
requires_alarm
supports_latency_budget
supports_end_to_end_mapping
applicable_failure_modes
review_notes
```

Example:

```text
sm_id                  = SM_ENDPOINT_PARITY
mechanism_family       = parity
coverage_target        = endpoint state/data corruption
fail_safe_or_operational = fail-safe
requires_alarm         = yes
supports_end_to_end    = no
```

The catalog prevents inconsistent naming.

Without a catalog, one engineer may write:

```text
PARITY
```

another may write:

```text
EndpointParity
```

and another may write:

```text
EP_PAR
```

The tool may treat these as different mechanisms. The FMEDA reviewer may not know whether they are intended to be the same.

---

## 24. Alarm Path Map

An alarm path map connects a safety mechanism to a signal or event that can be observed during verification or fault campaign.

A useful alarm path map includes:

```text
alarm_id
alarm_signal
source_sm_id
source_endpoint_or_instance
aggregation_path
visible_at_top
fault_campaign_observable
safe_state_action
latency_requirement
```

This prepares D10 and D11.

D10 will focus on alarm list and observe point boundaries. D11 will package these into fault campaign setup.

If D07 does not preserve alarm information, later fault campaign setup becomes guesswork.

---

## 25. Protocol-Aware Mapping Example

Consider a simple request/response interface.

Important protocol-visible endpoints may include:

```text
req_valid
req_ready
req_addr
req_write
req_wdata
rsp_valid
rsp_ready
rsp_error
rsp_rdata
```

Failure modes might include:

```text
FM_REQ_ADDR_CORRUPTION
FM_WRITE_DATA_CORRUPTION
FM_RESPONSE_ERROR_SUPPRESSED
FM_HANDSHAKE_STUCK
```

Possible mechanisms:

```text
address/control parity
write-data parity
response consistency checker
timeout monitor
protocol assertion checker
```

Mapping examples:

```text
FM_REQ_ADDR_CORRUPTION -> req_addr endpoint group -> address parity -> addr_parity_error
FM_WRITE_DATA_CORRUPTION -> req_wdata endpoint group -> data parity -> data_parity_error
FM_RESPONSE_ERROR_SUPPRESSED -> rsp_error endpoint -> response checker -> rsp_checker_error
FM_HANDSHAKE_STUCK -> req_valid/req_ready relation -> timeout monitor -> bus_timeout_alarm
```

Notice that the last example is not a single endpoint property. It is a temporal protocol relation. A good D07 map must allow this.

---

## 26. River-Style or End-to-End Mapping

Some mechanisms protect a data item from one point to another.

For example:

```text
sensor sample generated at input interface
sample transported through internal bus
sample transformed by processing block
sample consumed by control decision logic
```

A local endpoint map may not be enough. The safety mechanism may be a CRC or sequence counter carried across the path.

An end-to-end mapping needs:

```text
source point
sink point
protected object
intermediate path
checker location
alarm path
coverage assumption
```

This is useful for:

```text
NoC packets
DMA transfers
sensor frames
configuration register writes
control messages
safety island communication
```

D07 should allow both local EP maps and end-to-end maps because modern automotive chips contain both local logic and long safety-critical data paths.

---

## 27. Map Versioning and Identity

Safety mechanism maps evolve.

A map may start as an architectural proposal, then become an implemented RTL map, then a fault campaign input, then a final FMEDA evidence source.

D07 should assign stable identities:

```text
map_version
source_scenario
source_evidence_session
input_design_hash
endpoint_inventory_hash
sm_catalog_hash
review_owner
review_date
```

The exact hashing mechanism can be simple at demo level, but the concept is important.

Without identity, two maps with the same filename may refer to different design versions or different endpoint inventories.

That breaks evidence traceability.

---

## 28. Quality Gate for D07

A D07 quality gate should check at least:

```text
failure mode library exists
failure mode to sub-part map exists
SM catalog exists
EP-to-SM map exists
all mapped endpoints are known
all safety mechanism IDs are known
all failure mode IDs are known
all non-NULL alarm signals are known or explicitly marked as planned
handoff files are generated for D08 and D15
no critical consistency error exists
```

Warnings may be allowed for planned alarms or proposed mechanisms, but they must be visible.

A possible gate policy:

```text
FAIL: missing endpoint, unknown SM, unknown failure mode, broken handoff
WARN: planned alarm, coarse instance mapping, incomplete end-to-end path
PASS: consistent, reviewable, downstream-ready
```

The goal is not to make every entry perfect. The goal is to avoid invisible assumptions.

---

## 29. Suggested Demo Directory Structure

The D07 demo can use a structure like this:

```text
D07_safety_mechanism_map_failure_mode_to_sm_ep/
  scripts/
    run_demo.csh
    run_demo.sh
  tools/
    build_safety_mechanism_map.py
  inputs/
    from_D04/
    from_D05/
    from_D06/
    failure_mode_library/
    mapping_policy/
  outputs/
    failure_mode_library.csv
    failure_mode_to_subpart_map.csv
    sm_catalog.csv
    ep_to_sm_map.csv
    instance_to_sm_map.csv
    end_to_end_sm_map.csv
    alarm_path_map.csv
    mapping_consistency_checks.csv
    d07_quality_gate.csv
    d07_handoff_to_d08.csv
    d07_handoff_to_d15.csv
    evidence_index.csv
    demo_summary.md
```

D07 should not re-run earlier analysis unless explicitly requested. It should consume prior evidence and build the map.

---

## 30. Neutral Execution Model

A public demo should use a neutral execution wrapper.

Example:

```csh
cd D07_safety_mechanism_map_failure_mode_to_sm_ep

setenv D04_ROOT /path/to/D04_structural_building_blocks
setenv D05_ROOT /path/to/D05_common_database_evidence_center
setenv D06_ROOT /path/to/D06_safety_exploration

csh scripts/run_demo.csh
```

If a real analysis engine is needed later, it should be mapped through environment variables:

```csh
setenv SAFEIC_ANALYSIS_ENGINE /path/to/local/analysis_engine
```

The D07 map itself should remain independent of local installation paths.

A map is evidence. It should not depend on where the tool is installed on one machine.

---

## 31. From D07 to D08: Fault List Generation

D08 will convert safety analysis intent into fault campaign input.

D07 prepares D08 by giving it:

```text
which endpoints are safety-relevant
which endpoints are protected
which endpoints are unprotected
which alarms should be observed
which failure modes matter
which mechanisms are planned or credited
```

This affects fault list generation because not every structural fault is equally important for every safety goal.

A fault list without failure-mode context is just a structural list.

A fault list with D07 context becomes a safety validation target.

---

## 32. From D07 to D15: FMEDA Data Model

D15 will organize parts, sub-parts, failure modes, safety mechanisms, DC, and residual FIT.

D07 provides a direct bridge:

```text
failure mode -> sub-part -> safety mechanism -> endpoint evidence
```

This makes FMEDA less manual.

Instead of typing a safety mechanism into a spreadsheet without structural backing, D15 can reference D07 evidence:

```text
this failure mode is covered by this SM
this SM covers these endpoints
these endpoints came from D04
this scenario was selected in D06
this mapping is stored in D05 evidence center
```

That is the difference between a spreadsheet and an auditable evidence model.

---

## 33. Common Mapping Mistakes

### 33.1 Mapping by Name Similarity

Do not assign a mechanism simply because endpoint names look similar.

### 33.2 Treating Proposed Mechanisms as Implemented

A what-if mechanism is not automatically an RTL mechanism.

### 33.3 Ignoring Alarm Observability

A mechanism with no observable diagnostic path may not support fault campaign detection.

### 33.4 Mixing Failure Mode and Fault Model

A stuck-at fault is not the same thing as a failure mode.

### 33.5 Overusing Instance-Level Mapping

Instance mapping is useful, but it can over-credit coverage if not justified.

### 33.6 Forgetting Protocol Semantics

Protocol relations such as request/response, valid/ready, and timeout cannot always be represented as isolated endpoints.

### 33.7 Losing the D06 Scenario Context

The map should preserve which exploration scenario produced the recommendation.

---

## 34. Review Checklist

A reviewer should be able to answer:

```text
Which failure modes are covered?
Which failure modes are still uncovered?
Which endpoints are protected?
Which endpoints remain unprotected?
Which safety mechanisms are proposed?
Which are implemented?
Which alarms are real?
Which alarms are placeholders?
Which mappings are endpoint-level?
Which mappings are instance-level?
Which mappings are end-to-end?
Which entries are ready for D08 fault list generation?
Which entries are ready for D15 FMEDA modeling?
```

If these questions cannot be answered from the D07 output package, the map is not yet useful enough.

---

## 35. Final Thought

D07 is not a glamorous step, but it is one of the most important engineering steps in the entire flow.

A Base FIT number explains exposure. A structural inventory explains where exposure exists. A what-if exploration proposes how to reduce it. But the safety mechanism map is where the safety argument becomes explicit.

It says:

```text
this failure mode is addressed by this mechanism
this mechanism covers this endpoint or path
this diagnostic signal proves observability
this mapping is traceable to earlier evidence
this output can drive fault list generation and FMEDA modeling
```

That is why D07 is more than a file conversion step.

It is the point where safety intent, design structure, diagnostic architecture, and evidence traceability become one coherent engineering model.
