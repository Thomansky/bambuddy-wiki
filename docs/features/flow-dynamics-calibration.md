---
title: Flow Dynamics Calibration
description: Measure a filament's pressure advance on the printer and store it as a K-profile
---

# Flow Dynamics Calibration

Measure the pressure advance (K value) of the filament in one AMS slot, the
same way Bambu Studio does, and store the result as a K-profile on the printer.

[K-Profiles](k-profiles.md) lets you read, edit, select and delete the profiles
a printer holds. This page is about producing a new one from a real
measurement, rather than typing a number in.

---

## :material-speedometer: What it does

On the H2 series a flow-dynamics calibration is **a print**. The printer lays a
single 30 mm line and, while it does, runs the calibration block that lives in
its own start G-code. Afterwards Bambuddy asks the printer what it measured,
shows you the number next to the one currently stored, and writes it only if
you say so.

The whole run takes **about 7 minutes** and uses **about 0.1 g** of filament.

| Stage | What is happening |
|-------|-------------------|
| **Waiting for the printer** | The run is queued behind whatever the printer is doing |
| **Slicing** | The calibration job is sliced against your printer, process and filament presets |
| **Uploading** | The sliced job is transferred to the printer |
| **Printing** | The line is printed and the measurement runs |
| **Reading the result** | Bambuddy asks the printer for the measured value |
| **Waiting for your decision** | The result is shown; nothing has been written |
| **Saving** | The value is written to the printer and read back to confirm |

---

## :material-printer-3d: Which printers

| Model | Supported |
|-------|-----------|
| **H2S** | Yes |
| H2D / H2D Pro | Not yet |
| X1 / P1 / A1 series | Not yet |

The action is shown on every printer, disabled with a reason where it does not
apply yet — so you can tell "not supported" from "something is wrong".

The X1/P1/A1 family calibrates through a completely different mechanism (a
single printer command, no print), which is a separate piece of work. The H2
firmware refuses that command outright, so there is no fallback between the
two: nothing is sent to a printer that cannot answer it.

H2D and H2D Pro wait on one thing only: on a two-nozzle machine the measured
result has to be matched to the right hotend with certainty. Guessing there
would write a perfectly correct K value against the wrong nozzle, and nothing
in the UI would show it.

---

## :material-server-network: It needs the slicer sidecar

This calibration will not start without a reachable
[server-side slicer](slicer-api.md). That is not a convenience — it is the
design.

The measurement itself is not something Bambuddy writes. It is vendor G-code
that ships inside your printer's own machine preset, gated on a flag the print
job carries, and parameterised by the filament preset (its maximum volumetric
speed and its nozzle temperature). Slicing is how the correct block, for your
exact machine, firmware generation and filament, gets into the job.

The alternative would have been a G-code template maintained by hand, per
model, per nozzle, per firmware generation. Its failure mode is silent drift: a
firmware update changes something, the file still uploads, the printer still
starts it, and it does something physical nobody intended. Bambuddy does not
author motion G-code for a real machine.

If the sidecar is unconfigured or unreachable, the modal says so and Start
stays disabled, with a link to the setting.

---

## :material-play: Starting a run

Open the slot's menu on the printer card — the same menu that holds **Configure
slot** — and pick **Flow dynamics calibration…**. It is offered for AMS slots,
for AMS-HT units and for the external spool.

You can also start one from a row in the **K-Profiles** list: the
:material-speedometer: button re-measures that profile's filament, provided
that filament is currently loaded in a slot. It is disabled with "not loaded in
any slot" otherwise.

The modal shows the printer and nozzle, the slot and its filament, **the K
value stored today**, and what the run will cost in time and filament. Before
Start becomes available you need to:

1. **Confirm the build plate type** that is really on the bed. The job is
   sliced for that plate, and the plate decides part of the first-layer
   behaviour.
2. **Check the preset triplet.** Bambuddy fills in the standard printer,
   process and filament presets for your machine and material. Change the
   filament preset for a third-party spool — this is what makes the
   measurement happen at the right temperature.
3. **Tick the confirmation** that the build plate is empty and the printed line
   may be removed afterwards. It is never pre-ticked: the toolhead is about to
   move across the bed.

If a run cannot start, Start is disabled **and the reason is shown next to
it** — printer offline, printer printing, slot empty, sidecar unreachable,
model not supported, a run already in progress. There is no silently dead
button.

---

## :material-check-decagram: The result is yours to accept

When the print finishes, Bambuddy reads the measurement and stops.

The card then shows the stored K next to the measured one, the flow
coefficient, the printer's own confidence figure, and whether saving will
**replace** an existing profile or **create** a new one. Two buttons: **Save to
printer** and **Discard**.

Bambuddy never saves the value on its own, for three reasons:

- the write is persistent printer-side state that changes every future print
  with that filament;
- the printer reports a *confidence* field whose meaning is not yet understood
  — every measurement seen so far reports `0`;
- a seven-minute unattended job that silently rewrites a calibration is the
  wrong default for a tool that runs a print farm.

**Discard writes nothing.** A run left unanswered for 24 hours is cancelled,
and cancelling also writes nothing.

After a save, Bambuddy reads the calibration table back and confirms the value
the printer actually stored. If it does not match, the run is marked failed
rather than reported as saved.

!!! note "Creating a profile is the less-tested path"

    Replacing an existing K-profile is the path this feature was built and
    verified against. When no profile exists yet for that filament and nozzle,
    Bambuddy creates one and then binds it to the calibrated slot — the same
    two steps the manual editor uses, but driven automatically. It is logged
    clearly, and worth a glance at the K-Profiles page afterwards to confirm
    the slot picked it up.

---

## :material-shield-check: Safety

The run moves a real printer, so it is gated in both directions:

- It only starts on an **idle, connected** printer that is not claimed by the
  print queue or a drying run.
- While it runs, the printer is **held**: queued prints and scheduled drying
  wait, and their cards say they are waiting for a flow-dynamics calibration.
- The slot must still hold the **same filament** it was created with, and the
  **same nozzle diameter**, both when the run is picked up and again
  immediately before the job is dispatched.
- The sliced file is checked for the calibration step **before it is
  uploaded**. If it is not there the run fails at Slicing and the file is
  thrown away — nothing reaches the printer. Without this check a broken run
  would look exactly like a normal six-minute print that produces no result.
- A filament preset that failed to resolve is fatal here, not a warning: it
  would measure the wrong filament at the wrong temperature and then offer the
  number as if it were true.
- The uploaded job is removed from the printer when the run ends, whatever the
  outcome.

---

## :material-bell: Notifications

Flow-dynamics calibration has its own notification event, on by default. It
fires when a run reaches **Waiting for your decision** and on every final
state. Seven minutes is long enough that you walked away, and the interesting
moment is the one where Bambuddy has stopped to ask a question.

See [Notifications](notifications.md) to route it.

---

## :material-help-circle: Troubleshooting

| What you see | What it means |
|---|---|
| **Not supported on this printer yet** | The model is not on the list above |
| **The slicer sidecar is not reachable** | Configure or start the sidecar; this calibration cannot run without it |
| **The slicer has no bundled profile for this printer and nozzle** | The sidecar's bundled profiles are older than your machine — update its image |
| Failed at **Slicing**, "contains no flow-dynamics calibration step" | The sliced file could not calibrate, so it was discarded. Usually the wrong printer preset, or a sidecar whose profiles predate the firmware |
| Failed with **"the printer returned no calibration result"** | The print finished but the measurement produced nothing. Nothing was written |
| Failed with **"the printer did not store the value"** | The write was sent and acknowledged, but the read-back did not show it |
| Failed with **"the filament in the slot changed"** | The spool was swapped mid-run. The measurement belongs to the old filament, so it is not written |

---

## :material-arrow-right: Related

- [K-Profiles](k-profiles.md) — read, edit and select the profiles this produces
- [Server-Side Slicing](slicer-api.md) — the sidecar this calibration needs
- [Notifications](notifications.md) — routing the run's events
- [AMS & Humidity](ams.md) — the slot menu this is started from
