# PoE Tap/Splitter Test

"PoE Tap" and "PoE Splitter" are being used interchangeably here

## Goal

1. Identify what's actually causing the network instability.
2. Determine the fix.
3. Build a test process every new PoE splitter model has to pass before it's approved for
   production installation.

  
  - **Before**: Anivision AVPS05, 5V 2.4A
  - **After**: Model HX-PD08SAT/G, 5V 3.5A

## Tentative test plan

1. **Load test** - check whether output voltage drops below 5V under load.
2. **Current draw check** - measure actual draw on the splitter using an NF-488 PoE
   checker.
3. **Burn-in** - run the splitter for 1 hour, monitoring the Pi's kernel log for `eth0`
   Link events during that window.
4. **Wireshark capture during burn-in** - run a packet capture (`tcpdump`/Wireshark) on the
   camera itself for the duration of the burn-in, in parallel with the kernel log
   monitoring. Note: a physical link-down is a PHY-layer event (the interface just goes
   silent) - Wireshark won't show a special "flap" packet, it'll show a gap in captured
   traffic. Still useful: gives an independent, precise timestamp source to cross-check
   against the kernel log's `Link is Down`/`Link is Up` lines, and shows exactly what
   traffic was happening on the wire right before the drop.
5. iperf:

## Pass/Fail Criteria 
- Voltage: ouput must stay abouve 5V continuously under a load ( camera or equivalent load on the PoE tester
- Network link event: it is typically to see one in the boot up process but a cluster over time is considered a fail
- 


### Open question: is a 1-hour burn-in long enough?

The weekend reboot test's real failure data showed USB/network instability taking
anywhere from **20 minutes to 20+ hours** to show up under sustained load. A 1-hour
burn-in could pass a marginal splitter that would still fail in the field. Worth deciding
whether to extend the burn-in window (and by how much) before treating a pass as
production-approved, or accepting the shorter window as a first-pass screen only.

## Next steps

- Get feedback from Tim 
- Decide the real burn-in duration
- Build the actual test script/process once the above is settled
