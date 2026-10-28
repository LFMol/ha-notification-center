# 🔔 Notification Center for Home Assistant

[![Validate](https://github.com/LFMol/ha-notification-center/actions/workflows/validate.yml/badge.svg)](https://github.com/LFMol/ha-notification-center/actions/workflows/validate.yml)
[![Release](https://img.shields.io/github/v/release/LFMol/ha-notification-center)](https://github.com/LFMol/ha-notification-center/releases)
![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2025.9%2B-41BDF5?logo=homeassistant&logoColor=white)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A small **inbox for your smart home**. Every automation sends its alerts to one place, a Local To-do list. Each alert gets one of five priorities and a category icon. Push notifications go to exactly the people you name in each call, and they come with **Acknowledge** / **Done** buttons.

```
🔴 P1 🚨 Alarm — Alarm triggered · 👀 Alex
🟠 P2 📬 Postbox — The postbox was opened
🟡 P3 🏠 Sunscreen — Deployed automatically at 14:02
```

<div align="center">
  <table>
    <tr>
      <td align="center">
        <img width="431" height="437" alt="Screenshot of the Notification Center page" src="https://github.com/user-attachments/assets/15d832ff-c4c8-45d8-83ba-8d0b8416c81f" />
      </td>
    </tr>
    <tr>
      <td align="center"><sub><b>Screenshot 1:</b> The Notification Center page</sub></td>
    </tr>
  </table>
</div>

## Features

- **One inbox.** All alerts are kept in a to-do list, so nothing is lost when you swipe away a phone notification.
- **Five priorities**, from 🔴 P1 Critical to 🟣 P5 Info. The inbox is sorted by priority. P1 breaks through as high priority (time-sensitive on iOS), and P5 arrives silently.
- **You choose who gets the push.** Each call lists the person slots, for example `person_notify: "1,3,5"`. Priority doesn't affect who is notified.
- **Up to five people.** On the People tab, each slot is a Home Assistant user plus a phone. Picking a user fills in their phone automatically.
- **Acknowledge / Done on the phone.** Everyone sees who picked up an alert. Done completes the item and clears the notification on every phone.
- **Dedup and cooldown.** If an open item has the same `dedup_key`, it is updated instead of duplicated. A cooldown ignores repeats.
- **Dashboard included,** with built-in cards only (an optional variant uses fold-entity-row). Three tabs:
  - **Inbox:** status header, inbox list, and a "Send notification" form.
  - **People:** user and phone per slot, plus a test button.
  - **Settings:** switches, a setup check, and how to use it.
- **Returns a result.** Each call returns `status`, `item_key`, `summary` and `pushed_to`.

## Requirements

| What | Why |
|---|---|
| Home Assistant **2025.9 or newer** (tested on 2025.9 and 2026.10) | Template switches with a fixed entity id |
| **Local To-do** integration | The inbox |
| **Home Assistant Companion app** on each phone | Push notifications and action buttons |
| *Optional:* [fold-entity-row](https://github.com/thomasloven/lovelace-fold-entity-row) (HACS) | Only for the dashboard variant with a foldable "Send notification" row |

## Installation (about 10 minutes)

**1. Enable packages.** This is the only YAML edit. Add these two lines to `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

> Already have a `homeassistant:` line? Add only the `packages:` line underneath it, indented by two spaces.
> Already have a `packages:` line? Skip this step.

**2. Copy the package.** Download the latest release from the [Releases page](https://github.com/LFMol/ha-notification-center/releases), or the files directly from this repository. Copy [`packages/notification_center.yaml`](packages/notification_center.yaml) to `/config/packages/`. Create the `packages` folder if it doesn't exist.

**3. Create the inbox.** Go to **Settings → Devices & services → Add integration → Local To-do** and name it `Notification center`.

**4. Check and restart.** Go to **Developer tools / Tools → YAML → Check configuration**, then **Restart**.

**5. Add the dashboard.** No YAML editing is needed:
1. Go to **Settings → Dashboards → Add dashboard → New dashboard from scratch**, name it *Notification Center* and open it.
2. Go to **⋮ → Edit dashboard → ⋮ → Raw configuration editor**.
3. Replace everything with the contents of one of these files, then save:
   - [`dashboards/notification_center_core.yaml`](dashboards/notification_center_core.yaml): built-in cards only. **Choose this one if you don't use HACS.**
   - [`dashboards/notification_center.yaml`](dashboards/notification_center.yaml): the same, but "Send notification" is a foldable row (needs fold-entity-row).

**6. Set up people.** On the **People** tab, pick a **User** for each slot. Their phone is filled in automatically. Then press **Send test notification**.

**7. Check the setup.** The **Settings** tab has a **Setup check** that shows whether everything is in place:

| Setup check | |
|:--|:--|
| Package loaded | ✅ yes |
| To-do list `todo.notification_center` | ✅ found |
| Companion-app phones | ✅ 2 found |
| People with a phone | ✅ 2 |

Optionally, set **Open on tap** to *Notification Center inbox* on the same tab.

## Usage

```yaml
action: script.notification_center_push
data:
  priority: 2
  category: Post
  title: Postbox
  message: The postbox was opened.
  person_notify: "1,2"
  dedup_key: postbox
  cooldown_minutes: 30
  task_description: Please empty the postbox.
  task_due_date: "2026-10-28"
```

| Parameter | Explanation |
|:--|:--|
| `priority` | 1 = 🔴 critical, 2 = 🟠 high, 3 = 🟡 normal *(default)*, 4 = 🟢 low, 5 = 🟣 info. Sets the colour and how urgent the push is. |
| `category` | Home 🏠, Post 📬, Parcel 📦, Network 🌐, Security 🚨, Energy ⚡, System 🛠️. Anything else shows 🏷️. |
| `title` | Short title. Use at least a title or a message. |
| `message` | The text of the notification. |
| `person_notify` | Who gets a phone push: slot numbers from the People tab, e.g. `"1,3,5"`, or `all`. Also accepts names or a list. Empty or `none` = inbox only. |
| `dedup_key` | Same key while the item is still open → the item is updated instead of added again. |
| `cooldown_minutes` | Used with a dedup key: repeats within this many minutes are ignored completely. |
| `task_description` | Extra text on the inbox item. |
| `task_due_date` / `task_due_datetime` | `YYYY-MM-DD` or `YYYY-MM-DD HH:MM:SS`. Date and time wins. |

The script returns a result you can use:

```yaml
status: added | updated | cooldown
item_key: postbox
summary: "🟠 P2 📬 Postbox — The postbox was opened."
pushed_to: [mobile_app_pixel_8]
```

More examples are in [`examples/automations.yaml`](examples/automations.yaml).

### Phone buttons

| Button | Inbox | Other phones |
|---|---|---|
| **Acknowledge** | `… · 👀 Name`, plus "Acknowledged by Name · time" | Silently updated to *Acknowledged by Name*. Only **Done** remains. |
| **Done** | Completed, `… · ✅ Name` | Notification cleared |

You can also set the status from an NFC tag or another automation:

```yaml
action: script.nc_set_status
data:
  item_key: postbox
  status: done          # acknowledged | done
  person: 1             # slot number or name, optional
```

## Dashboard

- **Inbox:**
  - A header showing the number of open items, push status and last change, plus **Pause phone pushes**.
  - The inbox list, sorted by priority. Tick an item to complete it.
  - **Send notification:** a fold-out row with priority, category, title, message, dedup key, cooldown (0–60 min) and recipients (named after the People tab).
- **People:** per slot, a **User** and a **Phone** dropdown and a test button. **Refresh phone list** reloads both lists.
- **Settings:**
  - **Pause all phone pushes** and **Hide Acknowledge / Done buttons**.
  - **To-do list** and **Open on tap** dropdowns.
  - A setup check that shows what's missing, and a usage example.

## How it works

- Each inbox item's description ends with `[dedup:<key>] [ts:<YYYYMMDD HHMMSS>] [to:<slots>]`. The script uses this to find, update and rate-limit items, and to update the right phones. Don't remove it.
- 🔴 🟠 🟡 🟢 🟣 are consecutive Unicode characters, so alphabetical order is also priority order.
- Android gets one notification channel per priority (`Notification Center P1 Critical` … `P5 Info`), so each can have its own sound or Do Not Disturb behaviour.

## Entities

| Entity | Purpose |
|---|---|
| `script.notification_center_push` | Main script |
| `script.nc_set_status` | Acknowledge / done (phone buttons, NFC, automations) |
| `script.nc_send_from_dashboard` | "Send notification" on the dashboard |
| `script.nc_refresh_people_lists` | Refills the user, phone, to-do and open-on-tap dropdowns |
| `automation.notification_center_phone_buttons` | Handles the phone buttons |
| `automation.notification_center_refresh_people_dropdowns` | Refreshes the dropdowns at startup and hourly |
| `automation.notification_center_people_dropdown_changed`, `…_settings_dropdown_changed` | Save dropdown choices |
| `input_select.nc_person_{1..5}_select`, `…_notify_select` | User and phone per slot (dashboard) |
| `input_text.nc_person_{1..5}_{name,notify}` | Saved user and phone per slot (used by the script) |
| `switch.nc_recipient_{1..5}` | Recipient switches, named after the people |
| `input_boolean.nc_compose_to_{1..5}` | Stored state of the recipient switches |
| `input_boolean.nc_push_paused`, `input_boolean.nc_buttons_hidden` | Pause pushes / hide buttons (off = normal) |
| `input_select.nc_todo_select`, `input_select.nc_open_url_select` | Settings dropdowns |
| `input_text.nc_todo_entity`, `input_text.nc_open_url` | Saved settings |
| `input_*.nc_compose_*` | "Send notification" form |

## Troubleshooting

- **No push arrives.**
  - Check the **Setup check** on the Settings tab.
  - Check that the call includes `person_notify` and that the slot has a phone.
  - Check that pushes aren't paused.
  - Look at the `pushed_to` value the script returns.
- **No buttons on the push.** Make sure **Hide Acknowledge / Done buttons** is off. On Android, swipe the notification open.
- **Buttons do nothing.** Make sure `automation.notification_center_phone_buttons` is enabled. Buttons only work on items that are still open.
- **Items duplicate.** Pass a `dedup_key`.

## Contributing

Issues and pull requests are welcome. Every change is checked automatically by the **Validate** workflow, which runs Home Assistant's own configuration check on the minimum supported version (2025.9) and the current version.

**Releasing:** bump the number in `VERSION`, add a matching `## x.y.z` section to `CHANGELOG.md` and push to `main`. The **Release** workflow then creates the tag, the GitHub release and the download zip.

## License

MIT, see [LICENSE](LICENSE).
