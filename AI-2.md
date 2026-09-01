Priority 1 — Detailed Tasks: Complete Readiness UI Integration

This is the highest-value next milestone because the Readiness Engine already exists conceptually and serves as the final provisioning gate. The goal is to make the UI consume readiness results rather than calculate readiness itself.1.md

Task Group 1 — Create Readiness Page
Objective

Add a dedicated page or tab to display readiness information.

Navigation

Add:

Single Switch

Multi Switch

Inventory Manager

Readiness

SKU Manager

Settings

Page Responsibilities

Display:

Readiness Status

Validation Results

Block Reasons

Provisioning Eligibility


Must not display:

Raw commands

Event names

SQL

CLI output


Success:

Readiness page visible

Task Group 2 — Integrate ReadinessPanel
Objective

Embed the Readiness widget into the UI.

Create Widget Instance
self.readiness_panel = ReadinessPanel(
    event_bus=self.bus,
    engineer_mode=False,
)

Add To Layout
layout.addWidget(
    self.readiness_panel
)

Verify

Initial state:

Waiting For Safety Checks

Start Provisioning Disabled


Success:

Panel renders correctly

Task Group 3 — Wire Event Subscription
Objective

Allow readiness updates automatically.

Event
readiness.evaluated

Flow
ReadinessService
      ↓

readiness.evaluated
      ↓

ReadinessPanel
      ↓

UI Update

Verify

Publishing:

bus.publish(
    READINESS_EVALUATED,
    result=result,
)


updates:

Headline

Status

Button State


Success:

No manual refresh required

Task Group 4 — Technician-Friendly Messages
Objective

Hide internal implementation details.

Internal
SKU-001

UI
The detected hardware does not match
this rack configuration.

Verify Mapping

Codes:

DISC-001

INV-001

SKU-001

LLDP-001

PROF-001


All must resolve to user-friendly text.

Success:

No codes shown in Technician Mode

Task Group 5 — Readiness Status Display
Objective

Render individual safety checks.

Example
Inventory Information Verified

Switch Identified

Hardware Matches Rack Configuration

Required Network Connections Detected

Configuration Available

Statuses
Verified

Needs Review

Blocked


Not:

PASS

WARNING

FAIL


for technicians.

Success:

Friendly status language everywhere

Task Group 6 — Provisioning Gate
Objective

Make readiness the sole authority.

Implementation
start_button.setEnabled(
    readiness_result.ready
)

READY
Button Enabled

BLOCKED
Button Disabled


UI must never perform:

if inventory_ok:


or:

if sku_ok:


checks itself.

Success:

All provisioning access goes through ReadinessService

Task Group 7 — Engineer Mode Support
Objective

Expose diagnostics without changing workflow.

Add
Rule Name

Rule Status

Error Code

Technical Details

Example
sku_model

FAIL

SKU-001

Expected SN4700
Found SN2700

Technician Mode

Hide:

Codes

Technical messages


Success:

Technician and Engineer views render correctly

Task Group 8 — Create Block Reason Panel
Objective

Help technicians understand failures.

Example
Provisioning Blocked

The detected hardware does not match
this rack configuration.

Engineer Mode
Provisioning Blocked

Code:
SKU-001

Expected:
SN4700

Detected:
SN2700


Success:

Every blocker explained clearly

Task Group 9 — Add UI Tests
Objective

Validate behavior automatically.

Test Cases
Initial State
Waiting For Safety Checks

Button Disabled

READY
Button Enabled

Ready To Provision

BLOCKED
Button Disabled

Friendly Message Visible

Technician Mode
No Codes

No Technical Details

Engineer Mode
Codes Visible

Technical Details Visible


Success:

Readiness UI reliably testable

Task Group 10 — Main Workflow Integration
Objective

Connect readiness into existing workflow.

Workflow
Load Inventory
      ↓

Discover Device
      ↓

Verify Inventory
      ↓

Evaluate Readiness
      ↓

Update UI

Future

Later phases simply add new readiness inputs:

Prompt Detection
      ↓

Login Validation
      ↓

Topology Correlation
      ↓

Dry Run Validation


without changing the UI architecture.1.md

Definition of Done

Priority 1 is complete when:

✓ Readiness page exists

✓ ReadinessPanel integrated

✓ readiness.evaluated updates UI

✓ Friendly messages displayed

✓ Technician Mode complete

✓ Engineer Mode complete

✓ Start button gated by readiness

✓ UI tests implemented

✓ No raw commands exposed


At that point, the next development priority becomes:

Priority 2

Prompt Detection


which will feed additional state into the existing Readiness Engine rather than requiring a new workflow.1.md

Next Development Priorities (Alpha v3)

Based on the current design baseline, test coverage, Readiness Engine, Engineer Mode design, and AI import package, the next priorities should be:

Priority 1 — Finish Readiness UI Integration

Goal: Connect the existing Readiness Engine to the real application UI.1.md

Deliverables
Readiness page

ReadinessPanel integration

Friendly readiness messages

Provisioning button gating

Block reason display

Engineer Mode details

Success Criteria
READY
    ↓
Start Provisioning Enabled

BLOCKED
    ↓
Start Provisioning Disabled

Priority 2 — Engineer Mode

Goal: Provide diagnostics without exposing technical complexity to technicians.1.md

Deliverables
Engineer Mode setting

Readiness Details page

Event Monitor page

Revision History page

Inventory match details

LLDP details

Technician View
Inventory Verified

Ready To Provision

Engineer View
Inventory PASS

SKU FAIL

SKU-001

Expected SN4700
Found SN2700

Priority 3 — Prompt Detection

Goal: Automatically recognize switch state. This is the next major feature after Readiness UI completion.1.md

New Module
app/discovery/prompt_detector.py

Detect
login:

Password:

admin@sonic:~$

root@sonic:~$

ONIE

GRUB

New Event
prompt.detected

Priority 4 — Login Workflow

Goal: Remove manual switch login.1.md

Deliverables
Login state machine

Authentication workflow

Prompt handling

login.completed event


State flow:

Disconnected
↓
Connected
↓
Login Prompt
↓
Authenticated

Priority 5 — LLDP Topology Correlation

Goal: Validate cabling, not just neighbor presence.1.md

Example

Current:

Neighbor Exists


Future:

Correct Neighbor

Correct Port

Correct Role

Event
topology.correlated

Priority 6 — Multi-Switch Coordinator

Goal: Coordinate MX, NS1, and NS2 together.1.md

New Component
RackCoordinator

Responsibilities
Launch switch workers

Aggregate readiness

Determine rack readiness


Rule:

MX READY
+
NS1 READY
+
NS2 READY
=
RACK READY

Priority 7 — Dry Run Provisioning

Goal: Generate configuration previews without applying them.1.md

Output
Hostname

Management IP

Profile

Validation Results

Configuration Summary

Event
dry_run.completed

Recommended Immediate Sprint

If choosing the next work items right now:

1. Finish Readiness UI

2. Finish Engineer Mode

3. Add pytest-qt UI tests

4. Implement Prompt Detection

5. Begin Login Workflow


That sequence gives you the most value while staying aligned with the project's primary objective: safely validating racks and preventing incorrect provisioning through inventory, discovery, topology validation, and a centralized Readiness Engine.1.md

Priority 2 — Engineer Mode (Detailed Task Breakdown)

Goal: Add advanced diagnostics and troubleshooting visibility for engineers while keeping the technician workflow simple and safe. Engineer Mode should expose why a decision was made, not change the decision itself. The Readiness Engine remains the single source of truth.1.md

Task Group 1 — Engineer Mode Setting
Objective

Create a persistent application setting that controls advanced visibility.

Settings
[general]
engineer_mode = false

UI
☐ Engineer Mode


When enabled:

☑ Engineer Mode

Event
settings.changed

Success Criteria
Setting persists across restarts

Task Group 2 — Feature Flag Service
Objective

Create a centralized feature-access mechanism.

New Component
FeatureFlags


Example:

feature_flags.engineer_mode

Purpose

Prevent pages from reading settings directly.

Success Criteria
Feature visibility controlled from one location

Task Group 3 — Dynamic Navigation
Objective

Show engineer-only pages when Engineer Mode is enabled.

Technician Navigation
Single Switch

Multi Switch

Inventory Manager

Inventory Mapping

SKU Manager

Readiness

Settings

Engineer Navigation

Add:

Readiness Details

Event Monitor

Revision History


Future:

Topology View

Success Criteria
Engineer pages hidden unless enabled

Task Group 4 — Readiness Details Page
Objective

Show rule-level readiness information.

Technician View
Inventory Verified

Hardware Verified

Ready To Provision

Engineer View
Inventory Rule   PASS

Discovery Rule   PASS

SKU Rule         FAIL

Code: SKU-001

Expected SN4700
Found SN2700

Success Criteria
All readiness rules visible

Task Group 5 — Event Monitor
Objective

Expose EventBus traffic for troubleshooting.

New Page
Event Monitor

Data

Subscribe to:

*

Example Output
08:14:22

inventory.saved

rack_serial=RACK001

-------------------

08:14:24

inventory.verified

state=VERIFIED

role=NS1

Success Criteria
Engineers can trace workflow execution

Task Group 6 — Inventory Match Diagnostics
Objective

Show how identity verification was determined.

Example
Inventory Verification

State:
VERIFIED

Role:
NS1

Method:
Serial + MAC

Conflict Example
State:
CONFLICT

Serial → NS1

MAC → NS2

Success Criteria
Role matching decisions explainable

Task Group 7 — LLDP Details Panel
Objective

Expose parsed LLDP information.

Example
Local:
Ethernet96

Neighbor:
NS2

Remote Port:
Ethernet104

Management IP:
192.168.0.4


Future:

Topology Correlation Results

Success Criteria
Parsed LLDP visible without raw command output

Task Group 8 — Revision History Page
Objective

Display SKU revision history.

Example
SKU      Version     Type

119684   2.0         Major

119684   1.1         Minor

119684   1.0         Initial

Additional Data
Date

Changes

Archive Location

Success Criteria
Engineers can audit configuration evolution

Task Group 9 — Enhanced Block Reason Display
Objective

Expose internal blocking details.

Technician
Provisioning Blocked

The detected hardware does not match
this rack configuration.

Engineer
Provisioning Blocked

Code:
SKU-001

Expected:
SN4700

Detected:
SN2700

Success Criteria
All blocking decisions traceable

Task Group 10 — Logging View
Objective

Provide a user-facing diagnostic log.

Categories
Discovery

Inventory

Readiness

Provisioning

Example
08:32:11

Discovery completed

Model:
SN4700

Serial:
NS1001

Success Criteria
Troubleshooting possible without source code access

Task Group 11 — Engineer Mode Tests
Objective

Verify visibility and behavior.

Test Cases
Feature Flag
Engineer Mode OFF

Pages Hidden

Navigation
Engineer Mode ON

Pages Visible

Readiness Details
Rule Results Visible

Event Monitor
Events Captured

Technician Isolation
Codes Hidden

Diagnostics Hidden

Success Criteria
No engineer-only information leaks into Technician Mode

Definition of Done

Priority 2 is complete when:

✓ Engineer Mode setting exists

✓ Feature flag implemented

✓ Readiness Details page implemented

✓ Event Monitor implemented

✓ Inventory diagnostics visible

✓ LLDP diagnostics visible

✓ Revision History visible

✓ Enhanced blocker details visible

✓ Engineer tests passing

✓ Technician workflow unchanged

What Comes Next

Once Engineer Mode is complete, the next highest-priority feature is:

Priority 3

Prompt Detection


This adds switch-state awareness (login:, Password:, shell prompt, ONIE, GRUB) and provides a new input to the Readiness Engine without changing the existing workflow.1.md

To start Priority 1 (Readiness UI Integration), don't begin with the UI page itself.

Start with the event flow that connects the existing Readiness Engine to the UI.

This minimizes risk and gives you a working vertical slice almost immediately.1.md

Step 1 — Verify the Readiness Event Contract

First ensure you have a stable event definition.

Event:

readiness.evaluated


Payload:

{
    "result": ReadinessResult
}


This event becomes the contract between:

ReadinessService

and

ReadinessPanel


Before writing UI code, confirm:

bus.publish(
    READINESS_EVALUATED,
    result=result,
)


already works.

Goal:

Readiness Engine
    ↓
Event Bus


working independently of any widget.

Step 2 — Create the Readiness Page Shell

Create a new page:

Readiness Page


Do not build all features yet.

Just create:

Headline

Status Area

Start Provisioning Button


Example:

Readiness

Waiting For Safety Checks

[ Start Provisioning ]


Button should start disabled.

self.start_button.setEnabled(False)


Goal:

Page exists

Step 3 — Add ReadinessPanel

Embed the panel.

Current layout:

Inventory Manager

Readiness Panel

Provisioning Actions


Pseudo-code:

self.readiness_panel = ReadinessPanel(
    event_bus=bus
)


Then:

layout.addWidget(
    self.readiness_panel
)


Goal:

Widget visible


without any readiness data yet.

Step 4 — Subscribe to Readiness Events

Now connect the panel.

During initialization:

bus.subscribe(
    READINESS_EVALUATED,
    self.on_readiness_evaluated
)


Handler:

def on_readiness_evaluated(
    self,
    event,
):

    result = event.payload[
        "result"
    ]

    self.render_readiness(
        result
    )


Goal:

Page reacts automatically


No refresh button.

No polling.

Step 5 — Render Friendly Status

Do not show:

PASS

FAIL

WARNING


to technicians.

Translate:

PASS


→

Verified

WARNING


→

Needs Review

FAIL


→

Blocked


Example display:

Inventory Information Verified

Switch Identified

Required Network Connections Detected


Goal:

Technician-friendly language

Step 6 — Add Friendly Block Messages

Use the message catalog.

Example:

BLOCK_MESSAGES = {

    "SKU-001":
        "The detected hardware "
        "does not match this "
        "rack configuration.",

    "LLDP-001":
        "Required network "
        "connections were not "
        "detected.",
}


Result:

Provisioning Blocked

The detected hardware does not
match this rack configuration.


Never:

SKU-001


for technicians.

Goal:

Human-readable failures

Step 7 — Gate the Provision Button

This is the most important UI behavior.

Only readiness controls access.

Never:

if inventory_ok:


Never:

if lldp_ok:


Instead:

self.start_button.setEnabled(
    result.ready
)


Result:

READY
    ↓
Button Enabled

BLOCKED
    ↓
Button Disabled


Goal:

Single source of truth

Step 8 — Add Engineer Mode Toggle Support

You do not need a full Event Monitor yet.

Just support:

engineer_mode=True


and expose:

Rule Name

Code

Technical Detail


Example:

sku_model

FAIL

SKU-001

Expected SN4700
Found SN2700


Goal:

One UI
Two views

Step 9 — Write Tests Before Expanding

Implement these immediately:

Initial state

READY state

BLOCKED state

Button enabled

Button disabled

Technician mode

Engineer mode


These were already identified as the minimum UI test set.2.zip

Step 10 — End-of-Sprint Checkpoint

Priority 1 is complete when:

✓ Readiness page exists

✓ ReadinessPanel integrated

✓ readiness.evaluated updates UI

✓ Friendly messages displayed

✓ Technician mode works

✓ Engineer mode works

✓ Provision button gated

✓ UI tests passing


At that point, your UI becomes the consumer of readiness results, and Priority 2 (Engineer Mode expansion) and Priority 3 (Prompt Detection) can plug into the existing architecture without redesigning the workflow.1.md