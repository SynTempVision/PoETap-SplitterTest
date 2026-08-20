# PoE Tap/Splitter Test

Investigating and fixing camera network instability (`eth0` flapping) linked to the PoE
splitter hardware, and building a repeatable qualification test for any new PoE splitter
model before it goes into production.

**Note on terminology:** "PoE Tap" and "PoE Splitter" are being used interchangeably here -
both refer to the inline device that taps power off the PoE Ethernet cable and splits it
into a separate DC power output for the camera. Worth confirming with whoever specs these
parts if there's a meaningful distinction between the two terms for a given vendor/product
line.

## Goal

1. Identify what's actually causing the network instability.
2. Determine the fix.
3. Build a test process every new PoE splitter model has to pass before it's approved for
   production installation.

## Evidence so far

- **Symptom**: the Pi's kernel log shows the `eth0` link repeatedly going down/up
  (`bcmgenet ... eth0: Link is Down` / `Link is Up`). Wireshark has also been used to
  observe this - see the burn-in test plan below for capturing this properly alongside the
  kernel log going forward.
- **Splitter swap result**: 2 cameras (cam 10 / `sv-159`, cam 24 / `sv-142`) had their PoE
  splitters replaced and have **not** shown `eth0` flapping since:
  - **Before**: Anivision AVPS05, 5V 2.4A
  - **After**: Model HX-PD08SAT/G, 5V 3.5A
  
  This is real, positive evidence for the underpowered-splitter theory - matches the
  existing leading theory from `PoESplitter/PoE_Splitter_CameraFlapping_Investigation.docx`
  (2.4A splitter power/heat).
- **Related prior finding**: the RamOS weekend reboot test (`RamOS/RebootLoopTest/
  WEEKEND_TEST_FINDINGS.md`) found the same kind of simultaneous USB+network instability
  across 4 independent units on the same bench, concluding the most likely cause was
  something shared at the hardware/power level (power rail, PoE source) rather than four
  independent camera failures - consistent with a splitter-power root cause rather than a
  per-camera one.

## Tentative test plan (draft)

1. **Load test** - check whether output voltage drops below 5V under load.
2. **Current draw check** - measure actual draw on the splitter using an NF-488 PoE
   checker.
3. **Burn-in** - run the splitter for 1 hour, monitoring the Pi's kernel log for `eth0`
   flap events during that window.
4. **Wireshark capture during burn-in** - run a packet capture (`tcpdump`/Wireshark) on the
   camera itself for the duration of the burn-in, in parallel with the kernel log
   monitoring. Note: a physical link-down is a PHY-layer event (the interface just goes
   silent) - Wireshark won't show a special "flap" packet, it'll show a gap in captured
   traffic. Still useful: gives an independent, precise timestamp source to cross-check
   against the kernel log's `Link is Down`/`Link is Up` lines, and shows exactly what
   traffic was happening on the wire right before the drop.

### Open question: is a 1-hour burn-in long enough?

The weekend reboot test's real failure data showed USB/network instability taking
anywhere from **20 minutes to 20+ hours** to show up under sustained load. A 1-hour
burn-in could pass a marginal splitter that would still fail in the field. Worth deciding
whether to extend the burn-in window (and by how much) before treating a pass as
production-approved, or accepting the shorter window as a first-pass screen only.

## Next steps

- [ ] Run the Wireshark/kernel-log capture together during the next burn-in and record the
      actual signature/timestamps observed
- [ ] Decide the real burn-in duration
- [ ] Confirm Tap vs Splitter terminology with whoever specs these parts
- [ ] Build the actual test script/process once the above is settled
