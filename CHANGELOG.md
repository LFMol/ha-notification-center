# Changelog

## 1.0.0 — 2026-10-28

First public release.

- **One inbox** for all alerts, built on a Local To-do list.
- **Five priorities**: P1 🔴 Critical … P5 🟣 Info, with matching push urgency (P1 high priority / time-sensitive, P5 quiet). Default priority is 3.
- **`person_notify`** decides who gets a push, e.g. `"1,3,5"`, `all` or `none`.
- **Up to five people**. Each slot is a Home Assistant user plus a phone, chosen from dropdowns on the People tab. Picking a user fills in their phone automatically.
- **Acknowledge / Done** buttons on every push. The inbox shows who handled an item and when. Done completes it and clears every phone.
- **Dedup and cooldown**: an open item with the same `dedup_key` is updated instead of duplicated, and repeats within the cooldown are ignored.
- **`script.nc_set_status`** for NFC tags, voice or other automations.
- **Dashboard** with three tabs:
  - Inbox: status header and a "Send notification" form with named recipient switches.
  - People: a user and phone per slot, with a test button.
  - Settings: switches, dropdowns, a setup check and a usage example.
- **Two dashboard variants:** `notification_center_core.yaml` (built-in cards only, no HACS) and `notification_center.yaml` (foldable row, needs fold-entity-row).
- **Easy installation:** one YAML edit, and the dashboard is pasted via the Raw configuration editor.
- Requires **Home Assistant 2025.9** or newer. Validated on 2025.9.4 and 2026.10.4.
