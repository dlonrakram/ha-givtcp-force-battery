# GivTCP Force Charge / Discharge Buttons

Home Assistant scripts + dashboard controls for manually triggering an immediate
Force Charge, Force Discharge (export to grid), or Cancel on a GivEnergy
inverter via [GivTCP](https://github.com/britkat1980/giv_tcp) — fully local,
no GivEnergy cloud account needed. This replaces the "Charge Now" / "Discharge
Now" quick-menu button that GivEnergy removed from recent versions of their
mobile app.

## What this does

- **Force Charge** — forces the battery to charge at max power for a duration
  you choose, from 15 minutes up to 4 hours.
- **Force Discharge** — forces the battery to discharge (export to grid) at
  max power for the same selectable duration.
- **Cancel** — cancels whichever of the above is currently running.

The dashboard has a shared **Duration** slider (0.25–4 hours, 15-minute steps)
plus **1 h / 2 h / 3 h / 4 h** preset buttons. The presets only set the slider;
press **Charge** or **Discharge** to start.

Under the hood these call GivTCP's `select` and `number` entities, which GivTCP
auto-creates via MQTT discovery:

| Entity | Purpose |
| --- | --- |
| `select.givtcp_ce1234g001_force_charge` | Force charge using GivTCP's preset durations (2–180 minutes) |
| `select.givtcp_ce1234g001_force_export` | Force discharge/export using the same preset durations |
| `number.givtcp_ce1234g001_force_charge_num` | Force charge for any duration up to 240 minutes |
| `number.givtcp_ce1234g001_force_export_num` | Force discharge/export for any duration up to 240 minutes |

Durations up to 180 minutes use the `select` entities. Durations from
181–240 minutes (3.25–4 hours) use the `number` entities, because GivTCP's
`select` option list stops at 180 minutes.

## Prerequisites

- GivTCP running and connected to your inverter, with MQTT discovery enabled
  and Home Assistant's MQTT integration picking up the entities above.
- A recent GivTCP version that creates the `*_force_charge_num` and
  `*_force_export_num` number entities. If you don't see them, update GivTCP.
- **Set your inverter serial once** — at the top of
  `packages/givtcp_force_battery.yaml`, change the `inverter_serial` YAML anchor
  from the placeholder `CE1234G001` to your own serial. All three scripts use
  that single value, and the entity IDs are lower-cased automatically.
  `CE1234G001` in this repo is a mock, not a real inverter.
- **Confirm your own entity IDs** — go to
  **Settings → Devices & Services → Entities**, search "force", and check the
  serial matches your inverter. If you swap the inverter or dongle, update the
  one `inverter_serial` line.

## Install / restore steps

1. Enable HA packages if not already, in `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copy `packages/givtcp_force_battery.yaml` into your HA config's
   `packages/` folder, then set `inverter_serial` at the top to your own
   inverter serial.

3. **Developer Tools → YAML → Reload Scripts, Groups and Automations**
   (or just restart Home Assistant). This also creates the new
   `input_number.force_duration_hours` helper.

4. Open the dashboard you want the controls on → **Edit Dashboard → Add Card →
   Manual** → paste the contents of
   `dashboards/givtcp_force_battery_card.yaml`.

   Note: `theme: Frosted Glass Dark Lite` is whatever HA theme was active when
   this was built — drop those lines (or swap them for your own theme name) if
   you don't have it installed.

5. Test each control, then confirm state via the entities'
   `Force_Charge` / `Force_Export` state topics, or just watch the inverter's
   live power reading change.

## Using the controls

1. Set the **Duration** slider, or tap a preset (`1 h`, `2 h`, `3 h`, `4 h`).
2. Press **Charge** or **Discharge**.
3. Press **Cancel** to stop early.

GivTCP runs the action for the selected time and then restores the previous
charge schedule, power limits and target SOC automatically. Starting another
force charge while one is already running extends the end time rather than
creating a duplicate job.

You can also call the scripts directly, for example from an automation:

```yaml
action: script.force_charge_battery
data:
  hours: 2.5
```

If `hours` is omitted, the script uses the dashboard slider.

Other options in `packages/givtcp_force_battery.yaml`:

- Change the slider limits/steps under `input_number.force_duration_hours`.
  GivTCP's `number` entities cap this at 240 minutes, so 4 hours is the maximum
  without bypassing them via GivTCP's MQTT control topic.
- Change the preset values in the dashboard card.

## Behaviour notes

- Force charge/discharge runs at max power unless you separately change
  `number.givtcp_<serial>_battery_charge_rate` or
  `number.givtcp_<serial>_battery_discharge_rate`. Those are sticky settings,
  not per-action — remember to reset them if you don't want them affecting
  scheduled slots too.
- Force charge does not stop early at a target SOC; it runs for the selected
  duration (or until you press Cancel).
- GivTCP's `sensor.givtcp_<serial>_charge_time_remaining` and
  `sensor.givtcp_<serial>_discharge_time_remaining` sensors show the remaining
  time. Add them to the card if you want a countdown.

## Source references

- GivTCP REST implementation: `GivTCP/REST.py` in
  https://github.com/britkat1980/giv_tcp — routes `/forceCharge`,
  `/forceExport`, `/setDischargeRate`, `/setChargeRate`.
- Entity naming: `GivTCP/entity_lut.py` in the same repo (`Battery_Discharge_Rate`,
  `Battery_Charge_Rate`, `Force_Charge_Num`, etc).
- Preset duration list: `GivTCP/GivLUT.py` (`delay_times`), capped at 180
  minutes.
