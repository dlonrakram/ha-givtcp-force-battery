# GivTCP Force Charge / Discharge Buttons

Home Assistant scripts + dashboard buttons for manually triggering an immediate
Force Charge, Force Discharge (export to grid), or Cancel on a GivEnergy
inverter via [GivTCP](https://github.com/britkat1980/giv_tcp) — fully local,
no GivEnergy cloud account needed. This replaces the "Charge Now" / "Discharge
Now" quick-menu button that GivEnergy removed from recent versions of their
mobile app.

## What this does

- **1 Hour Charge** — forces the battery to charge at max power for 60 minutes.
- **1 Hour Discharge** — forces the battery to discharge (export to grid) at
  max power for 60 minutes.
- **Cancel** — cancels whichever of the above is currently running.

Under the hood these call GivTCP's `select.select_option` entities, which GivTCP
auto-creates via MQTT discovery:

- `select.givtcp_ce1234g001_force_charge`
- `select.givtcp_ce1234g001_force_export`

Both accept the options `Normal`, `Running`, `Cancel`, or a duration in minutes
(`2`, `5`, `10`, `15`, `30`, `45`, `60`, `75`, `90`, ... `180`).

## Prerequisites

- GivTCP running and connected to your inverter, with MQTT discovery enabled
  and Home Assistant's MQTT integration picking up the entities above.
- **Confirm your own entity IDs first** — go to
  **Settings → Devices & Services → Entities**, search "force", and check the
  serial number matches. `CE1234G001` below is this inverter's serial; if you
  ever swap the inverter or dongle, update it in
  `packages/givtcp_force_battery.yaml`.

## Install / restore steps

1. Enable HA packages if not already, in `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

2. Copy `packages/givtcp_force_battery.yaml` into your HA config's
   `packages/` folder.

3. **Developer Tools → YAML → Reload Scripts, Groups and Automations**
   (or just restart Home Assistant).

4. Open the dashboard you want the buttons on → **Edit Dashboard → Add Card →
   Manual** → paste the contents of `dashboards/givtcp_force_battery_card.yaml`.

   Note: `theme: Frosted Glass Dark Lite` is whatever HA theme was active when
   this was built — drop that line (or swap it for your own theme name) if you
   don't have it installed.

5. Test each button, then confirm state via the entities'
   `Force_Charge` / `Force_Export` state topics, or just watch the inverter's
   live power reading change.

## Adjusting duration or power

- Duration: change the `option:` value in
  `packages/givtcp_force_battery.yaml` (must be one of the allowed option
  strings above — plain numbers are minutes).
- Power limit: GivTCP also exposes `number.givtcp_<serial>_battery_charge_rate`
  and `number.givtcp_<serial>_battery_discharge_rate` (in Watts, 0 up to your
  inverter's max). These are sticky settings, not per-action — set one before
  running Force Charge/Discharge to cap that action's power, and remember to
  reset it back to max afterwards (or extend the Cancel script to do so) if
  you don't want it affecting scheduled charge slots too.

## Source references

- GivTCP REST implementation: `GivTCP/REST.py` in
  https://github.com/britkat1980/giv_tcp — routes `/forceCharge`,
  `/forceExport`, `/setDischargeRate`, `/setChargeRate`.
- Entity naming: `GivTCP/entity_lut.py` in the same repo (`Battery_Discharge_Rate`,
  `Battery_Charge_Rate`, etc).
