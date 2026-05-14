# Automotive Safe-IC Practice 03: FIT Standards — IEC 62380 vs SN 29500

Author: Darren H. Chen  
Direction: Automotive chip functional safety analysis and fault injection  
Demo: D03_fit_standards_real_engine_from_d01_d02_quiet  
Tags: ISO 26262, FIT, IEC 62380, SN 29500, Mission Profile, FMEDA, Diagnostic Coverage, Safe-IC, Functional Safety

---

## 1. Why the FIT Standard Must Become an Explicit Engineering Control

In a functional safety flow, a FIT number is not just a number.

It is the result of a model, a design boundary, a set of environmental assumptions, a technology setup, and a reporting policy. If any of these inputs is implicit, the result becomes difficult to reproduce and even harder to defend in review.

This article focuses on the third step of the Safe-IC practice series:

```text
D03 — FIT Standards: IEC 62380 vs SN 29500
```

D03 is not an isolated demo. It is a controlled continuation of the previous two steps:

```text
D01: Analysis Input Package
D02: Base FIT Rate
D03: FIT Standards — IEC 62380 vs SN 29500
D04: Structural Building Blocks
```

D01 establishes the design boundary and the safety analysis input package. D02 calculates the base FIT rate and produces the initial contribution evidence. D03 asks a more subtle question:

```text
If the design boundary and base analysis context are fixed,
how does the selected FIT standard affect the interpretation of the result?
```

This is an important question because FIT is used downstream by FMEDA, diagnostic coverage planning, safety mechanism selection, and final residual-risk assessment. A flow that silently relies on a default FIT standard may still produce a report, but it does not produce strong engineering evidence.

The main principle of D03 is therefore:

> The FIT standard is part of the run identity. It must be explicit, reviewable, and traceable.

---

## 2. A Clarification: FIT Standard Is Not the Same as a Demo Variant

Before discussing the demo architecture, we need to clarify an important naming issue.

In this flow, there are only two FIT standard identifiers used by the analysis engine:

```text
iec_62380
sn_29500
```

These are standards or standard-model selectors.

However, a demo may run multiple scenarios such as:

```text
iec62380_passenger_65c
iec62380_passenger_85c
iec62380_motorcontrol_65c
sn29500_reference_65c
```

These are not FIT standards.

They are **variant IDs**.

A variant ID is an engineering label used to identify one specific combination of:

```text
FIT standard
mission profile
temperature assumption
manufacturing-year assumption
technology setup
reference data source
run identity
output directory
database session
```

A better way to express the relationship is:

```text
fit_standard = iec_62380 | sn_29500

variant_id = fit_standard + mission_profile + parameter_set
```

For example:

```text
variant_id:          iec62380_motorcontrol_85c
fit_standard:        iec_62380
mission_profile:     motor-control-like operating profile
parameter_set:       elevated thermal condition
design_boundary:     inherited from D01
base_fit_context:    inherited from D02
```

This distinction matters because a future reviewer should not confuse an experiment label with a safety standard.

---

## 3. What D01 and D02 Contribute to D03

D03 should not create a new design setup from scratch.

A real safety flow is not a set of unrelated scripts. It is a chain of controlled evidence. The D03 demo therefore uses the previous steps as upstream inputs.

### 3.1 D01 Contribution: Stable Analysis Input Package

D01 provides the structural input package:

```text
RTL design files
filelist
top module
clock definition
FIT setup baseline
analysis initialization file
run identity
input inventory
expected output model
common database settings
```

The key value of D01 is not that it contains RTL. The key value is that it fixes the boundary of analysis.

If two FIT-standard experiments use different RTL, different clocks, different file ordering, or different top modules, then the comparison is not clean. The standard would not be the only variable.

D03 therefore treats D01 as the source of truth for the design boundary.

### 3.2 D02 Contribution: Base FIT Context and Evidence

D02 provides the first base FIT evidence:

```text
BFR summary
FIT contribution report
base FIT evidence index
quality gate result
handoff file from D02 to D03
```

D02 answers:

```text
Before we credit any safety mechanism, where is the random hardware failure exposure?
```

D03 then changes the FIT standard and selected reliability assumptions while keeping the upstream design context traceable.

### 3.3 Why This Dependency Matters

Without this dependency, D03 would become a toy comparison:

```text
run standard A on one input
run standard B on another input
compare numbers
```

That would be weak.

A reviewable D03 flow should be closer to:

```text
same D01 design boundary
same D02 base evidence chain
controlled FIT-standard selection
controlled mission-profile variants
separate output evidence per variant
common comparison table
handoff to structural safety modeling
```

This makes D03 a methodology demo instead of a loose experiment.

---

## 4. FIT, BFR, DC, and Residual FIT

D03 is about FIT standards, but the standard only makes sense when placed inside the broader safety-analysis language.

### 4.1 FIT

FIT means **Failure In Time**.

A common interpretation is:

```text
1 FIT = 1 failure per 10^9 hours
```

In chip-level functional safety work, FIT is used to quantify random hardware fault exposure. It is not a pass/fail flag. It is a rate that must be interpreted against safety goals, ASIL budgets, failure modes, and diagnostic coverage.

### 4.2 Base FIT Rate

Base FIT Rate, or BFR, is the failure-rate baseline before safety mechanisms are credited.

Conceptually:

```text
BFR = random hardware failure exposure before diagnostic credit
```

D02 establishes this baseline. D03 then examines how the selected reliability model affects the interpretation of that baseline.

### 4.3 Diagnostic Coverage

Diagnostic Coverage, or DC, describes how much of the relevant fault population is detected or controlled by safety mechanisms.

A simplified view is:

```text
DC = covered relevant faults / total relevant faults
```

A FIT-weighted view is:

```text
FIT-weighted DC = covered FIT / total relevant FIT
```

In real flows, DC is not just a percentage. It depends on:

```text
fault model
fault population
failure mode
safety mechanism
observe point
alarm definition
fault classification policy
FTTI window
simulation stimulus
unresolved-fault handling
```

### 4.4 Residual FIT

Residual FIT is the risk left after diagnostic credit.

A simple teaching formula is:

```text
Residual FIT = Base FIT × (1 - DC)
```

In an FMEDA, the actual calculation may be split by:

```text
part
sub-part
failure mode
safety mechanism
single-point fault
latent fault
safe fault
detected fault
residual fault
ASIL target
```

The FIT standard affects the input side of this chain. If the base FIT changes, the residual FIT and prioritization may change downstream.

---

## 5. IEC 62380: Mission Profile as a First-Class Input

IEC 62380 is commonly used for reliability prediction of electronic components and equipment. In a chip-level analysis flow, it encourages the engineer to think in terms of physical operating conditions rather than treating a component as a context-free object.

A simplified conceptual decomposition is:

```text
Total FIT ≈ die contribution + package contribution + EOS contribution
```

The exact implementation is tool-specific and data-specific, but the engineering idea is clear: failure-rate prediction depends on how the component is used.

Important concepts include:

```text
mission profile
junction temperature
ambient temperature
operating time ratio
non-operating time ratio
thermal cycling
package stress
technology-dependent base rate
electrical overstress contribution
manufacturing-year related factors
```

### 5.1 Mission Profile

A mission profile describes how the part is expected to live during operation.

It may answer questions such as:

```text
How long is the device powered on?
How long is it dormant?
What temperatures does it see?
How often does it cycle between thermal states?
Is it in a passenger-compartment-like environment?
Is it in a motor-control-like environment?
Is the package exposed to stronger thermal stress?
```

The same RTL design can produce different predicted reliability results if the environmental assumptions change.

This is why D03 does not simply toggle a standard and stop. It also makes the mission-profile assumption visible as a variant parameter.

### 5.2 Why Temperature Matters

Temperature affects semiconductor reliability because many physical failure mechanisms are temperature dependent.

For a safety analysis engineer, the practical lesson is:

```text
Do not compare FIT values without checking the mission profile and thermal assumptions.
```

A lower FIT number under a mild mission profile does not automatically mean the design is safer than another run under a harsher mission profile. The context is part of the evidence.

### 5.3 Why IEC 62380 Variants Are Useful

In D03, IEC 62380 variants are not new standards. They are controlled scenario labels.

For example:

```text
iec62380_passenger_65c
iec62380_passenger_85c
iec62380_motorcontrol_65c
iec62380_motorcontrol_85c
```

These names express an experiment design:

```text
standard:      IEC 62380 model
profile:       passenger-like or motor-control-like
temperature:   selected comparison point
```

This makes it easy to compare how the same design responds to different reliability assumptions.

---

## 6. SN 29500: Reference Failure Rate and Operating-Condition Adjustment

SN 29500 is another reliability prediction approach commonly used in automotive electronics contexts. A useful way to understand it is to start from reference failure rates and then adjust them for operating conditions.

Key concepts include:

```text
reference failure rate
component category
technology category
operating temperature
stress condition
environment condition
conversion from reference condition to actual condition
```

The most important methodological difference is that SN 29500 often feels more like:

```text
start with reference data
adjust based on operating assumptions
```

Whereas IEC 62380 often feels more mission-profile and physical-environment driven.

This is a simplification, but it is useful for engineering communication.

### 6.1 Why SN 29500 Should Be Compared Explicitly

If a project uses SN 29500, the standard should not be buried in a hidden setup file. It should appear in:

```text
run identity
variant matrix
analysis configuration
comparison table
evidence index
FMEDA handoff
```

A comparison table that only shows numbers is incomplete.

A useful comparison table should show at least:

```text
variant_id
fit_standard
mission_profile
temperature assumption
manufacturing-year assumption if applicable
native output directory
summary report
DCE-style artifact
database session
diagnostics status
handoff status
```

This is why D03 produces comparison and evidence files rather than just one printed result.

---

## 7. Controlled Comparison: What Must Stay Fixed and What May Change

A standard comparison is meaningful only when the controlled variables are clear.

### 7.1 Fixed Inputs

D03 keeps the following fixed through D01 and D02:

```text
RTL design boundary
top module
filelist source
clock definition
base input package
D02 base evidence chain
common run structure
evidence collection policy
```

These fixed inputs allow the comparison to focus on the selected standard and reliability assumptions.

### 7.2 Variable Inputs

D03 allows these inputs to vary by variant:

```text
fit_standard
mission_profile_type
mission_profile_file
temperature-related parameter
manufacturing-year parameter
standard-specific reference data
output directory
database session name
```

### 7.3 Output Isolation

Each variant writes into its own output location:

```text
outputs/native/<variant_id>/
outputs/db/<variant_id>.fdb::<session_name>
logs/run_<variant_id>.console.log
```

This prevents one run from overwriting another and makes review easier.

The comparison layer should never rely on “whatever report was produced last.” It should know which variant produced which report.

---

## 8. Demo Architecture

The D03 demo is structured as a small orchestration flow around a real safety analysis engine.

It does not replace the engine. It prepares inputs, generates per-variant configurations, invokes the engine through a stable interface, and collects evidence.

### 8.1 External Environment

The demo assumes the environment is prepared outside the demo:

```csh
setenv D01_ROOT /path/to/D01_analysis_input_package
setenv D02_ROOT /path/to/D02_base_fit_rate
setenv SAFEIC_ANALYSIS_ENGINE /path/to/safety-analysis-engine
```

The demo does not set Python internally. The external environment is responsible for providing a suitable `python3`.

This keeps the demo closer to a real EDA environment where project-wide tool versions are managed outside the individual demo directory.

### 8.2 Run Entry

The run entry remains simple:

```csh
csh scripts/run_demo.csh
```

The script performs four conceptual steps:

```text
check upstream roots
prepare D03 variant inputs
run all configured variants
collect comparison and evidence
```

### 8.3 Variant Matrix

The variant matrix is a CSV-like control file.

Conceptually:

```text
variant_id,fit_standard,mission_profile,temperature,mfg_year,description
iec62380_passenger_65c,iec_62380,passenger,65,default,IEC 62380 passenger-like baseline
iec62380_motorcontrol_85c,iec_62380,motor_control,85,default,IEC 62380 motor-control-like elevated condition
sn29500_reference_65c,sn_29500,reference,65,default,SN 29500 reference comparison
```

The exact field names can evolve, but the principle should not:

```text
standard selection and scenario identity must be machine-readable.
```

### 8.4 Per-Variant Generated Files

For each variant, D03 generates:

```text
variant FIT setup
variant analysis initialization file
variant run command
variant native output directory
variant database session identity
variant log file
```

The demo’s Python script is not a FIT calculator. It is an orchestrator.

Its responsibilities are:

```text
snapshot upstream inputs
resolve filelists
expand variant matrix
generate per-variant setup files
generate per-variant run scripts
collect native outputs
parse key metrics conservatively
build comparison tables
classify diagnostics
produce D04 handoff
```

---

## 9. Why the Demo Uses a Quiet Diagnostics Policy

Real EDA and safety-analysis tools often emit warnings. A mature flow does not simply ignore them, but it also should not fail every run because the word “Warning” appears.

D03 uses a diagnostic classification policy.

### 9.1 Three Classes of Messages

A practical policy separates messages into three groups:

```text
fatal errors
actionable errors
known and classified diagnostics
```

Fatal errors stop the run.

Actionable errors require investigation before trusting the result.

Known diagnostics may be documented and suppressed if they are understood, stable, and not relevant to the demo objective.

### 9.2 Why “Error” in a Safety Term Is Not Always a Tool Error

Functional safety language contains words such as:

```text
permanent error
transient error
error propagation
error detection
error cone
```

These are engineering terms. They may appear in reports even when the run is successful.

Therefore, log classification should not be:

```text
grep -i error log
```

That is too naive.

A better diagnostic policy looks for structured severity patterns such as:

```text
fatal severity
tool-reported error severity
non-zero exit code
missing expected reports
missing database session
unresolved configuration failure
```

At the same time, safety-domain terms should be treated as domain vocabulary unless they appear as structured tool errors.

### 9.3 Known Warning Suppression

In some environments, a known duplicate safety-mechanism definition warning may appear when the default diagnostic coverage definition file is read and overlapping definitions are encountered. If the tool still completes the run, writes the reports, creates the database session, and returns a zero exit code, this warning can be treated as a classified diagnostic rather than a demo failure.

The quiet version of D03 demonstrates a conservative approach:

```text
classify the diagnostic
document the policy
suppress expected noise where appropriate
continue to fail on real errors
```

This is not about hiding problems. It is about separating known noise from evidence-breaking issues.

---

## 10. Success Criteria for D03

D03 should not define success as “the log has no warnings.”

That is too fragile.

A better success definition is:

```text
all required upstream D01 artifacts are available
all required upstream D02 artifacts are available
each variant has a generated run configuration
each variant uses one valid FIT standard identifier
each variant creates an isolated native output directory
each variant returns a zero exit code
expected summary or metric artifacts are discoverable
DCE-style artifacts are discoverable when generated
database session identity is recorded
comparison table is generated
diagnostics table is generated
D04 handoff file is generated
```

This definition makes the demo useful in real engineering work.

---

## 11. Expected Managed Outputs

After a successful D03 run, the managed output layer should contain files similar to:

```text
outputs/
├── run_status.csv
├── fit_standard_real_comparison.csv
├── fit_standard_real_comparison.md
├── evidence_index.csv
├── log_diagnostics.csv
├── log_diagnostics.md
├── d03_handoff_to_d04.csv
├── source_provenance.csv
├── demo_summary.md
├── variant_fit_setups/
├── variant_analysis_configs/
├── variant_commands/
├── native/
└── db/
```

The exact native report names may vary by tool version and configuration. The managed layer should therefore index the reports instead of hardcoding a single expected filename.

The important output is not just the number.

The important output is the evidence graph:

```text
D01 input package
  -> D02 base FIT evidence
    -> D03 standard-specific variant evidence
      -> D04 structural safety model handoff
```

---

## 12. Interpreting the Comparison Table

A FIT-standard comparison table should be interpreted carefully.

Suppose the comparison table shows:

```text
variant A: lower permanent FIT
variant B: higher permanent FIT
variant C: similar transient FIT
```

This does not automatically mean one standard is “better.”

It means:

```text
the selected model and assumptions produce different failure-rate estimates
```

The engineering questions are:

```text
Which standard is required by the customer or safety plan?
Which model is consistent with the product safety case?
Which mission profile best matches the target application?
Which input data is more defensible?
Which output will be used by FMEDA?
How will residual FIT and diagnostic coverage be computed later?
```

D03 is therefore not a contest between standards. It is a method for making the standard choice visible and auditable.

---

## 13. How D03 Prepares D04

D04 focuses on structural building blocks:

```text
endpoint
startpoint
DCE-style artifact
EP-to-SM mapping
diagnostic coverage computation
```

D03 prepares D04 by recording which standard-specific reports and DCE-style artifacts were generated for each variant.

The D04 handoff should include:

```text
variant_id
fit_standard
native output directory
summary report candidates
DCE artifact candidates
database session
diagnostic status
source provenance
```

This allows D04 to answer structural questions without re-running D03 blindly.

For example:

```text
Which endpoints appear under the IEC 62380 run?
Which DCE artifact belongs to the SN 29500 run?
Was the same D01 design boundary used?
Which D02 evidence was inherited?
```

This is how a demo series becomes a flow rather than a collection of isolated examples.

---

## 14. Methodology Lessons

D03 provides several general lessons for safety-oriented EDA flows.

### 14.1 Make Defaults Visible

A default FIT standard may be convenient, but it is risky for evidence.

If a result will feed FMEDA, the selected standard should be visible in:

```text
configuration
report name
comparison table
database session
handoff file
article text
review checklist
```

### 14.2 Separate Standard from Scenario

A FIT standard is not a scenario.

A scenario is a parameterized run under a standard.

This distinction avoids confusing names such as:

```text
iec62380_passenger_65c
```

with actual standard identifiers.

### 14.3 Keep the Design Boundary Fixed

If the design boundary changes between standard comparisons, the comparison becomes ambiguous.

D03 therefore inherits D01 and D02 instead of rebuilding the input package.

### 14.4 Treat Logs as Evidence, Not Decoration

Logs should be classified, indexed, and connected to output artifacts.

A successful flow should know:

```text
which command was run
which configuration was used
where the native output is
which diagnostics were seen
which artifacts were generated
which database session was written
```

### 14.5 Do Not Over-Trust a Single FIT Number

A FIT number is meaningful only in context.

The context includes:

```text
standard
mission profile
temperature
technology data
package data
design boundary
fault model
diagnostic policy
report policy
```

D03 exists to make that context explicit.

---

## 15. Recommended Repository Notes

A good GitHub repository for D03 should include a short README section explaining:

```text
This demo compares two FIT standard selectors:
  iec_62380
  sn_29500

Variant IDs are scenario labels, not standard names.

The demo requires D01_ROOT and D02_ROOT.

The demo uses SAFEIC_ANALYSIS_ENGINE as the external analysis executable.

The demo does not publish real proprietary logs.

The managed outputs are safe for methodology review, but native tool outputs may require local licensing review before publication.
```

This keeps the public repository clean while preserving the engineering value of the demo.

---

## 16. Final Takeaway

The main purpose of D03 is not to prove that one FIT standard is universally better than another.

The purpose is to show how to build a controlled, traceable, and reviewable FIT-standard comparison flow.

The core ideas are:

```text
FIT standard is part of run identity.
Only iec_62380 and sn_29500 are standard selectors in this demo.
Variant IDs are parameterized experiment labels.
D03 must inherit D01 and D02 evidence.
Mission profile and thermal assumptions must be explicit.
Warnings must be classified, not blindly ignored or blindly treated as failures.
The output must be useful for D04 structural safety analysis.
```

A safety flow becomes credible when every metric can be traced back to its inputs, assumptions, configuration, native reports, diagnostics, and downstream use.

D03 is the step where the FIT standard stops being a hidden option and becomes an auditable engineering decision.
