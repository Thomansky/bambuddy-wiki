---
title: Maintenance Tracker
description: Schedule and track printer maintenance tasks
---

# Maintenance Tracker

Schedule and track maintenance tasks to keep your printers running reliably.

![Maintenance](../assets/maintenance-1.png){ .screenshot }

!!! tip "Looking for the per-printer on/off switch?"
    The **Maintenance Tracker** on this page is for **interval-based scheduled tasks** (clean nozzle every 100 hours, check belts every 200 hours, etc.). If you want to take a single printer **out of service right now** — stop MQTT, drop it from the queue, hide it from notifications — that's **[Maintenance Mode](printer-control.md#maintenance-mode)** on the Printer Control page. The two pair well: when you start a real task from the tracker, you can flip the same printer to Maintenance Mode while the cover is open.

---

## :material-tools: Overview

The maintenance tracker helps you:

- **Schedule** recurring maintenance tasks
- **Track** when maintenance was last performed
- **Get notified** when maintenance is due
- **Log** maintenance history
- **Run** the printer's calibrations for you, by hand, when due, or on a weekday schedule — see [Printer Calibration](#printer-calibration) and [Vision Encoder Calibration](#vision-encoder-calibration)

---

## :material-format-list-checks: Maintenance Types

### Default Types

Bambuddy includes common maintenance tasks:

| Type | Default Interval | Applies To |
|------|-----------------|------------|
| **Clean Build Plate** | Every 25 hours | All printers |
| **Clean Nozzle/Hotend** | Every 100 hours | All printers |
| **Check Belt Tension** | Every 200 hours | All printers |
| **Check PTFE Tube** | Every 500 hours | All printers |
| **Lubricate Carbon Rods** | Every 50 hours | X1/P1 |
| **Clean Carbon Rods** | Every 100 hours | X1/P1 |
| **Lubricate Steel Rods** | Every 50 hours | P2S |
| **Clean Steel Rods** | Every 100 hours | P2S |
| **Lubricate Linear Rails** | Every 50 hours | A1/H2D |
| **Clean Linear Rails** | Every 100 hours | A1/H2D |
| **[Printer Calibration](#printer-calibration)** | Every 100 hours | All printers — Bambuddy runs it for you |
| **[Vision Encoder Calibration](#vision-encoder-calibration)** | Every 7 days | H2S/H2D/H2D Pro/H2C — Bambuddy runs it for you |

### Hiding Default Types

If a default maintenance type isn't relevant to your setup, you can remove it:

1. Go to **Settings** > **Maintenance**
2. Click the :material-delete: icon next to the default type
3. Confirm deletion

Hidden default types can be restored at any time — click **Restore Default Tasks** to bring them all back.

### Custom Types

Create your own maintenance tasks:

1. Go to **Settings** > **Maintenance**
2. Click **Add Maintenance Type**
3. Enter name and description
4. Set default interval
5. Click **Save**

![Maintenance Settings](../assets/maintenance-2.png){ .screenshot }

---

## :material-calendar-clock: Interval Types

### Print Hours

Schedule based on actual print time:

- Example: Lubricate every 100 hours
- Accumulated from print durations

### Calendar Days

Schedule based on calendar time:

- Example: Check belts every 30 days
- Counts calendar days regardless of usage

---

## :material-plus-circle: Setting Up Maintenance

### Per-Printer Configuration

Each printer has its own maintenance schedule:

1. Go to **Maintenance** page
2. Select your printer
3. Click **Configure**
4. Enable/disable maintenance types
5. Adjust intervals as needed
6. Click **Save**

### Interval Customization

Override default intervals per printer:

| Printer | Clean Build Plate |
|---------|:-----------------:|
| Workshop X1C | Every 25 hours |
| Office P1S | Every 15 hours |
| Garage A1 | Every 50 hours |

---

## :material-check-circle: Logging Maintenance

### Marking Complete

When you perform maintenance:

1. Go to **Maintenance** page
2. Find the due/overdue item
3. Click **Mark Complete**
4. Optionally add notes
5. Counter resets

### Logging Details

Add information to maintenance logs:

| Field | Description |
|-------|-------------|
| **Date** | When performed |
| **Notes** | What you did |
| **Parts** | Any parts replaced |

---

## :material-alert: Due Status

### Status Indicators

| Status | Meaning |
|:------:|---------|
| :material-check-circle:{ style="color: #4caf50" } OK | Not due yet |
| :material-alert:{ style="color: #ff9800" } Due Soon | Approaching due |
| :material-alert-circle:{ style="color: #f44336" } Overdue | Past due |

### Due Soon Threshold

Configure when "Due Soon" triggers:

- Default: 80% of interval
- Example: Clean Build Plate every 25 hours → "Due Soon" at 20 hours

---

## :material-target: Printer Calibration

Since 1.2.6 ([#3127](https://github.com/maziggy/bambuddy/issues/3127)) two maintenance types are more than a reminder. **Printer Calibration** is a task Bambuddy performs itself: it tells the printer to run its own calibration routine &mdash; the bed leveling, vibration compensation and motor noise cancellation you would otherwise start from the touchscreen &mdash; and marks the item performed when the printer reports it finished. The second one, [Vision Encoder Calibration](#vision-encoder-calibration), works the same way on the H2 series and is described below.

**Printer Calibration** is a default type (every 100 print hours, all printers), so it sits on every printer's card like the other defaults and can be hidden and restored the same way. On the **Settings** tab it carries a **Runs a calibration** badge. Only these two types can run anything: custom types are reminders only.

### Calibration options

The card lists the routines a run performs. Tick the ones you want; they are sent to the printer as one command.

| Option | Notes |
|--------|-------|
| **Bed leveling** | On by default |
| **Vibration compensation** | On by default |
| **Motor noise cancellation** | On by default |
| **Nozzle offset** | Dual-nozzle printers only (H2D, H2D Pro, H2C, X2D). The checkbox is not shown on other models |
| **High-temperature bed leveling** | |
| **Micro Lidar** | Lidar calibration of the X1 series |
| **Nozzle clumping detection** | |

Apart from **Nozzle offset**, every option is offered on every model; the printer ignores the ones its hardware does not have. At least one option must be ticked before a run can be queued &mdash; otherwise Bambuddy refuses with **Select at least one calibration option**. The options are copied onto a run when it is queued, so changing them afterwards does not alter a run that is already waiting.

### Trigger

The **Trigger** dropdown decides when a run is queued:

| Trigger | Queues a run when |
|---------|-------------------|
| **Manual** | Only when you click **Run now**. This is the default. |
| **When due** | The item falls due &mdash; after its interval in print hours or calendar days, using the same calculation as the [due status](#due-status). One run per due period: if that run fails or is cancelled, the item stays due but is not retried by itself. Click **Run now**, or **Reset** once you have calibrated by hand. |
| **On a schedule** | The chosen time arrives on one of the chosen weekdays. **Weekdays** are chips (Saturday by default) and **Earliest time** is a time picker (06:00 by default). While nothing is queued the card shows **Next run: …**. A slot that arrives while the previous run is still waiting adds nothing: one run per item at a time. |

**Run now** works in every mode. The automatic triggers only fire for an item that is enabled, on a printer that is active. A schedule needs at least one weekday and a time, and both automatic triggers need at least one calibration option ticked &mdash; Bambuddy refuses to save the trigger otherwise.

!!! info "Timezone"
    The scheduled time is the server's local time, read from the `TZ` environment variable &mdash; the same rule as for [scheduled local backups](backup.md#scheduled-local-backups). Set `TZ` in `docker-compose.yml` (e.g. `TZ=Europe/Berlin`) to match your wall clock; without it, times are UTC.

### Run now

**Run now** queues a run straight away (**Calibration queued**) and leaves the rest to the scheduler; it does not send anything to the printer itself. The button is greyed out while a run is already pending or running for the item, while the item is disabled, and without the `maintenance:update` permission when authentication is enabled.

### While a run waits

A queued run starts on the next scheduler pass on which the printer is free &mdash; connected, idle, and, with **Require plate-clear confirmation** on, with its plate released. Until then the card says why it has not started:

| Card says | Meaning |
|-----------|---------|
| **Queued – starts as soon as the printer is idle** | Just queued; the scheduler has not looked at it yet |
| **Waiting: printer offline** | The printer is not connected, or the command could not be sent |
| **Waiting: printer busy** | The printer is printing, paused or preparing, or the print queue has claimed it: a job it just dispatched, an upload still in flight, or its post-dispatch hold. Another calibration run on the same printer counts too |
| **Waiting: plate not released yet** | **Require plate-clear confirmation** is on and the printer is waiting for you to release the plate |
| **Waiting: AMS drying in progress** | A drying session, manual or scheduled, is holding the printer |

A run waits as long as it takes and never interrupts anything: a print that is already running finishes first, and a print the queue is about to dispatch wins over the calibration. Once the calibration is running, the print queue treats the printer as busy, so prints queued behind it start when it finishes. There is no limit on how long a run can wait.

!!! warning "Keep plate-clear confirmation on if parts can be left on the plate"
    With **Require plate-clear confirmation** enabled, the run waits until you press **Clear Plate & Start Next** (or acknowledge over the API), so a bed leveling run never starts with a finished print still on the plate &mdash; see [Clear Plate Confirmation](print-queue.md#clear-plate-confirmation). With it disabled, the run starts as soon as the printer reports idle, finished or failed, whatever is still on the plate. Leveling into a part is a crash, so disable the gate only where the plate is cleared automatically.

While the calibration runs the card reads **Calibration running since …**, and the page refreshes every few seconds for as long as any run is pending or running.

### When a run finishes

The printer's own completion event closes the run:

- **Completed** &mdash; a history entry with the note **Automatic calibration** is written and the item's interval is reset, exactly as pressing **Reset** does. The same `bambuddy/maintenance/reset` [MQTT event](mqtt.md#maintenance-events) is published. The card reads **Last run completed …** until the next run.
- **Failed** &mdash; the card reads **Last run failed …** with the printer's error code. The item is not reset and nothing is retried automatically.
- **Cancelled** &mdash; see below. The item is not reset.

No notification is sent for a run; check the card, or subscribe to the MQTT event.

A run that is still "running" two hours after it started has lost its completion event &mdash; Bambuddy restarted mid-run with the printer offline since, say. It is closed as failed with **Lost track of the run: no completion was reported**, so the item is never blocked forever.

### Cancelling a run

**Cancel run** on the card withdraws the pending or running run (**Calibration run cancelled**):

- A **pending** run is simply removed; nothing is sent to the printer.
- A **running** run is stopped on the printer &mdash; but only when the printer is, at that moment, actually on the calibration. If the run has outlived the calibration (a completion missed across a restart, or a command the firmware never acted on) and the printer is by now on somebody's print, Bambuddy closes the run and leaves the printer alone.

Cancelling at the printer's touchscreen works too. The printer reports the calibration as failed and sends its cancel code a few seconds later, so Bambuddy holds the verdict for about 15 seconds and then records **Last run cancelled …** rather than a failure. Aborting the calibration back to idle counts as cancelled as well.

### The printer's own calibration is no longer a print

Whatever starts a calibration &mdash; a maintenance run, the touchscreen or Bambu Studio &mdash; Bambuddy now treats it as the printer's own job, not as a print:

- **No archive**, and no print-started or print-completed notification.
- **No filament deduction.** A calibration that was cut short used to book a full spool against every loaded AMS slot &mdash; [#3081](https://github.com/maziggy/bambuddy/issues/3081) saw 1 kg deducted from each spool after a bed leveling run was powered off. A calibration consumes nothing and now books nothing.
- **No plate-clear prompt** afterwards, because it leaves nothing on the plate, and no finish photo.
- **Not stopped by the [printer kill switch](billing.md#printer-kill-switch)**, which would otherwise cancel the run a schedule had just started.

Only a run Bambuddy queued marks the item performed. A calibration you start from the touchscreen is not tracked: press **Reset** on the card afterwards.

!!! note "What a run does not cover"
    - **Pressure advance (flow dynamics) calibration** is not part of a maintenance run. The K-profile line is a different printer routine, and a run never closes on it. See [K-Profiles](k-profiles.md).
    - **The vision encoder calibration** is its own item, described next. A Printer Calibration run only waits for the bed leveling / vibration / motor noise routine the options above belong to, and a Vision Encoder Calibration run only for the vision encoder routine: neither closes on the other's job.

---

## :material-eye-check: Vision Encoder Calibration

The H2 series (H2S, H2D, H2D Pro, H2C) has a vision encoder that Bambu Lab recommends recalibrating regularly &mdash; the **Motion precision** calibration on the printer's **Calibration** screen. Since 1.2.6 it is the second task Bambuddy can perform itself: **Vision Encoder Calibration** is a default type, every **7 days**, that only appears on the H2-series printers. Other models never get the item; an H2D added later gets it automatically, like the other defaults.

The card is the same as the [Printer Calibration](#printer-calibration) card without the option row: the vision encoder routine takes no options, so there is nothing to tick. Everything else &mdash; the **Trigger** dropdown with its three modes, the weekday chips and time, **Run now**, **Cancel run**, the waiting reasons, the status line and the history entry with the note **Automatic calibration** &mdash; works exactly as described above, and the two items on one printer never run at the same time: a run waits with **Waiting: printer busy** while the other calibration is on the printer.

!!! tip "If you already had a reminder for it"
    Many H2 owners set up a custom "vision encoder" reminder for this before 1.2.6. The new default item replaces it: it is on the same 7-day cadence and it does the calibration rather than only reminding you. Once you are happy with the new card, delete the custom type on the **Settings** tab (or unassign it from the printer) so the two do not nag in parallel.

Under the hood the run starts the printer's own `calibrate_motion_precision` routine; the printer reports it like any other of its jobs, so it is [not treated as a print](#the-printers-own-calibration-is-no-longer-a-print) either &mdash; no archive, no filament deduction, no plate-clear prompt, no cover lookups. Cancelling from the touchscreen is recognised the same way as for Printer Calibration. Should the item ever end up on a printer without a vision encoder (an assignment made by hand, say), the run is refused with **Vision encoder calibration needs an H2-series printer** instead of sending the printer a file it does not have.

---

## :material-bell-ring: Notifications

Get notified when maintenance is due:

1. Go to **Settings** > **Notifications**
2. Enable **Maintenance Due** event
3. Configure your notification provider

### Notification Timing

- **Due Soon**: When threshold reached
- **Overdue**: When interval exceeded

[:material-arrow-right: Notification setup](notifications.md)

---

## :material-history: Maintenance History

View past maintenance for each printer:

### History Log

| Date | Type | Notes |
|------|------|-------|
| Dec 14 | Clean Build Plate | IPA wipe |
| Dec 10 | Lubricate Carbon Rods | Rails and screws |
| Dec 1 | Clean Nozzle/Hotend | Replaced with 0.4mm |

### Exporting History

Export maintenance logs for records:

- CSV format
- Date range selection
- Per-printer or all printers

---

## :material-printer: Per-Printer View

### Dashboard

Each printer shows maintenance status:

```
Workshop X1C
─────────────────────────────
✅ Clean Build Plate     12 hours until due
⚠️ Lubricate Carbon Rods  8 hours until due
❌ Check Belt Tension    OVERDUE
✅ Clean Nozzle/Hotend   65 hours until due
```

### Quick Actions

- **View Details**: See full maintenance info
- **Mark Complete**: Log completed maintenance
- **View History**: See past maintenance

---

## :material-lightbulb: Maintenance Tips

!!! tip "Bed Cleaning"
    Clean with IPA between prints for best adhesion. Deep clean with dish soap weekly.

!!! tip "Lubrication"
    Use appropriate grease for linear rails. Don't over-lubricate.

!!! tip "Belt Tension"
    Belts should be firm but not overly tight. Use the "guitar string" test.

!!! tip "Nozzle Inspection"
    Check for wear, especially with abrasive filaments. Replace when worn.

!!! tip "Filters"
    HEPA and carbon filters lose effectiveness over time. Replace as recommended.

!!! tip "Document Everything"
    Add notes when logging maintenance to build a history of what works.

---

## :material-cog: Best Practices

### Regular Schedule

- Check maintenance status weekly
- Don't skip overdue items
- Build maintenance into your routine

### Preventive vs Reactive

- Preventive: Follow intervals
- Reactive: Fix when broken
- **Preventive is better!**

### Track Everything

- Log all maintenance
- Note any issues found
- Track parts replaced
