# #003: WHEA Event 17 corrected PCIe errors

**Status:** TRACKED, no action unless it escalates
**Opened:** 2026-08-26
**Area:** PCIe / chipset

**Symptom:** WHEA-Logger Event 17 (corrected hardware error) in bursts of 3-6 events, from
PCI Express Root Port #5 (`0:1C.4`, `DEV_7A3C`, chipset port).

Downstream candidates on that port:

- Intel I226-V Ethernet (disconnected, rig is on Wi-Fi)
- VIA USB 3.0 eXtensible Host Controller

Corrected, not fatal. Full PCIe topology in [`specs.json`](../Hardware/PC/specs.json) under `pcie`.

**If it escalates:** investigate ASPM on that root port first (disable in BIOS or via the power
plan), then isolate by disabling the I226-V in Device Manager and watching whether the bursts stop.
