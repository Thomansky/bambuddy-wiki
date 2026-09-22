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

### Only when the bed is cold

Bed leveling and the vision encoder calibration are best run on a cold, thermally settled machine, and a run that falls due right after a print would otherwise start on a warm bed. The row **Only when the bed is below … °C** under the options adds that as a start condition: tick it (30 °C to begin with; anything above 0 up to 120 °C, one decimal at most) and a queued run also waits until the printer reports a bed temperature below the value. It is the last thing checked, after the printer is idle and the plate has been released, and it is re-checked on every scheduler pass, so the run starts on the first pass after the bed has cooled. Unlike the options, the condition is read from the card at that moment, not copied onto the run: raising the value releases a run that is already waiting. Bambuddy reads the same bed temperature the [Bed Cooled notification](notifications.md#print-events) uses; until the printer has reported one, the run waits rather than guesses.

### Trigger

The **Trigger** dropdown decides when a run is queued:

| Trigger | Queues a run when |
|---------|-------------------|
| **Manual** | Only when you click **Run now**. This is the default. |
| **When due** | The item falls due &mdash; after its interval in print hours or calendar days, using the same calculation as the [due status](#due-status). One run per due period: if that run fails or is cancelled, the item stays due but is not retried by itself. Click **Run now**, or **Reset** once you have calibrated by hand. |
| **On a schedule** | The chosen time arrives on one of the chosen weekdays. **Weekdays** are chips (Saturday by default) and **Earliest time** is a time picker (06:00 by default). While nothing is queued the card shows **Next run: …**. A slot that arrives while the previous run is still waiting adds nothing: one run per item at a time. The checkbox **Don't start jobs that would run into the scheduled time** under the row, on by default, keeps the print queue clear of the slot &mdash; see [Order and the print queue](#order-and-the-print-queue). |

The two automatic options say in brackets what actually gates the start &mdash; *(only once the plate has been released)* while **Require plate-clear confirmation** is on, *(once the printer is idle)* otherwise &mdash; because a queued run, whatever queued it, starts only on a printer that passes [that check](#while-a-run-waits). **Run now** works in every mode. The automatic triggers only fire for an item that is enabled, on a printer that is active. A schedule needs at least one weekday and a time, and both automatic triggers need at least one calibration option ticked &mdash; Bambuddy refuses to save the trigger otherwise.

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
| **Waiting: printer busy** | The printer is printing, paused or preparing, or the print queue has claimed it: a job it just dispatched, an upload still in flight, or its post-dispatch hold |
| **Waiting: after Printer Calibration** | Another run is ahead of this one on the same printer &mdash; pending or already running &mdash; and goes first; see [Order and the print queue](#order-and-the-print-queue) |
| **Waiting: plate not released yet** | **Require plate-clear confirmation** is on and the printer is waiting for you to release the plate |
| **Waiting: AMS drying in progress** | A drying session, manual or scheduled, is holding the printer |
| **Waiting: bed still warm (34 °C)** | [Only when the bed is cold](#only-when-the-bed-is-cold) is ticked and the bed, at the temperature shown, is not yet below the value |
| **Waiting: bed temperature unknown** | The condition is ticked and Bambuddy has no bed temperature for the printer yet &mdash; it has not received a status report since Bambuddy started (the last reported value is kept across a short reconnect) |

A run waits as long as it takes and never interrupts anything: a print that is already running finishes first, and a print the queue is in the middle of dispatching &mdash; uploading, or inside its post-dispatch hold &mdash; wins over the calibration. Beyond that the print queue keeps out of the run's way: a printer with a run pending or running takes no new job from the queue until the run has closed, and a scheduled run keeps the queue clear ahead of its slot &mdash; both described under [Order and the print queue](#order-and-the-print-queue). There is no limit on how long a run can wait.

!!! warning "Keep plate-clear confirmation on if parts can be left on the plate"
    With **Require plate-clear confirmation** enabled, the run waits until you press **Clear Plate & Start Next** (or acknowledge over the API), so a bed leveling run never starts with a finished print still on the plate &mdash; see [Clear Plate Confirmation](print-queue.md#clear-plate-confirmation). With it disabled, the run starts as soon as the printer reports idle, finished or failed, whatever is still on the plate. Leveling into a part is a crash, so disable the gate only where the plate is cleared automatically.

While the calibration runs the card reads **Calibration running since …**, and the page refreshes every few seconds for as long as any run is pending or running.

### Order and the print queue

Schedule both [Printer Calibration](#printer-calibration) and [Vision Encoder Calibration](#vision-encoder-calibration) for Sunday noon on every printer and three things have to hold for the weekend to go as planned. They do, since 1.2.6:

**One run at a time, in a fixed order.** Runs on one printer go out one after the other: **Printer Calibration** first, then **Vision Encoder Calibration**, and among runs of the same kind the one whose scheduled time came first. Only the run at the head of that line is looked at on a scheduler pass; every other run on the printer reads **Waiting: after Printer Calibration** (or after whichever item is ahead) until its turn comes, also while the run ahead is still on the printer. The order is deliberate: the levelling routine heats the bed, so with [Only when the bed is cold](#only-when-the-bed-is-cold) ticked on the vision encoder item, that run waits for the bed to cool on its own once the levelling is done.

**The print queue yields to a pending run.** A printer with a run pending or running takes no job from the print queue until the run has closed &mdash; the queue used to grab the idle printer in exactly the gap between the two runs. The held queue row says why: **Maintenance run pending: Vision Encoder Calibration (bed still warm, 45 °C)**, the state in brackets being what the maintenance card shows for that run. A run that is waiting for the printer to come online reserves it too (the queue could not dispatch there anyway), and so does one waiting for the bed to cool &mdash; that is the point, otherwise the bed never cools. If that is not what you want, cancel the run; nothing in the queue is failed or reordered, and the job starts on the next pass after the run has closed. A job for "Any H2S" names the printers it was kept off, the way a busy printer is named: **Maintenance run pending: Printer Calibration (queued) &mdash; H2S-01, H2S-02**, one sentence however many printers are under the same run.

**The queue keeps clear of a scheduled run.** A run scheduled for Sunday noon is only a lower bound if a job started on Saturday afternoon runs straight through it. With **Don't start jobs that would run into the scheduled time** ticked on a scheduled item (the default), the queue starts a job on that printer only when it is expected to be done **15 minutes** before the slot, going by the job's own time estimate from its file. A job whose duration Bambuddy does not know is held from **two hours** before the slot. A held row reads **Scheduled maintenance at Sunday 12:00 — this job would run into it (estimated 3h 50m)** (or *duration unknown*); a shorter job behind it that does fit goes out, so the printer stays productive until the slot. Once the run has been queued the item's next occurrence moves a week out and everything flows again. A job for "Any H2S" is placed on a printer without a slot ahead, and waits with the same reason when there is none.

!!! info "Explicit starts are not held"
    Both holds apply to the queue's *automatic* dispatch only. The :material-play: on a staged queue item starts it regardless of a pending run or a slot ahead &mdash; the run then waits for that print, as it would for any print on the printer &mdash; and anything started on the printer itself is outside Bambuddy's queue to begin with. Queueing a job from the print dialog is *not* such a start, **ASAP** included: ASAP puts the job at the top of the queue and the queue dispatches it from there like any other, holds and all. That is the point of the holds &mdash; a job queued on Saturday evening is dispatched by the scheduler, possibly hours later, and it is exactly that dispatch that must not run into Sunday's slot. The :material-play: is offered on items queued with **Manual start**; an ordinary queued job waits out the run or the slot and goes out by itself afterwards.

### When a run finishes

The printer's own completion event closes the run:

- **Completed** &mdash; a history entry with the note **Automatic calibration** is written and the item's interval is reset, exactly as pressing **Reset** does. The same `bambuddy/maintenance/reset` [MQTT event](mqtt.md#maintenance-events) is published. The card reads **Last run completed …** until the next run.
- **Failed** &mdash; the card reads **Last run failed …** with the printer's error code. The item is not reset and nothing is retried automatically.
- **Cancelled** &mdash; see below. The item is not reset.

The outcome can also reach your notification providers through the **Maintenance Run Finished** event, which is off by default &mdash; see [Notifications](#notifications) below. Every run that was queued reports exactly once, whichever way it ended &mdash; a cancel from the card included, so a shared channel sees it too.

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

Like every calendar-day item it counts as **due** until it has been performed once, so right after the update every H2 printer shows it in red. Its trigger starts on **Manual**, so nothing runs on its own: press **Run now** when the printer is free, or **Reset** if you calibrated from the touchscreen recently. Switching the trigger to **When due** queues a run straight away.

The card is the same as the [Printer Calibration](#printer-calibration) card without the option row: the vision encoder routine takes no options, so there is nothing to tick. Everything else &mdash; the [cold-bed condition](#only-when-the-bed-is-cold), the **Trigger** dropdown with its three modes, the weekday chips and time, **Run now**, **Cancel run**, the waiting reasons, the status line and the history entry with the note **Automatic calibration** &mdash; works exactly as described above, and the two items on one printer never run at the same time: Printer Calibration goes first and the vision encoder run waits behind it with **Waiting: after Printer Calibration** &mdash; see [Order and the print queue](#order-and-the-print-queue).

!!! tip "If you already had a reminder for it"
    Many H2 owners set up a custom "vision encoder" reminder for this before 1.2.6. The new default item replaces it: it is on the same 7-day cadence and it does the calibration rather than only reminding you. Once you are happy with the new card, delete the custom type on the **Settings** tab (or unassign it from the printer) so the two do not nag in parallel.

Under the hood the run starts the printer's own `calibrate_motion_precision` routine; the printer reports it like any other of its jobs, so it is [not treated as a print](#the-printers-own-calibration-is-no-longer-a-print) either &mdash; no archive, no filament deduction, no plate-clear prompt, no cover lookups. Cancelling from the touchscreen is recognised the same way as for Printer Calibration. Should the item ever end up on a printer without a vision encoder (an assignment made by hand, say), the run is refused with **Vision encoder calibration needs an H2-series printer** instead of sending the printer a file it does not have. The routine lives in a model-specific folder on the printer; Bambuddy learns that folder from the printer's own jobs and falls back to a per-model guess for a printer that has not reported one yet. Should the printer refuse the file, the run fails at once with the printer's answer in the status line rather than waiting for a completion that never comes.

---

## :material-bell-ring: Notifications

Get notified when maintenance is due:

1. Go to **Settings** > **Notifications**
2. Enable **Maintenance Due** event
3. Configure your notification provider

### Maintenance Run Finished

Since 1.2.6 a second event, **Maintenance Run Finished**, reports how a [calibration run](#printer-calibration) Bambuddy queued ended: completed, failed (with the error in the message, whether the printer's code or a run Bambuddy lost track of) or cancelled, from the card or at the printer's touchscreen. It is off on every provider, including the ones that existed before the event, so nothing new arrives after the update until you switch it on under **Printer Status** in the provider's event settings. The message template is **Maintenance Run Finished** on the **Templates** tab; its variables are listed with the [other events](notifications.md#variables).

### Muting one item

Every card has a bell next to the item's name. Click it to mute the item: a muted item (slashed bell) is left out of the **Maintenance Due** reminder and sends no **Maintenance Run Finished** message, while its due status, the badge counts, its interval and its automatic trigger carry on unchanged. Click the bell again to unmute. The bell needs the same `maintenance:update` permission as the item's other controls.

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
