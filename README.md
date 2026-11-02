# Battery node with an industrial-style BMS (EEE40016 Group 3)

Battery, BMS and energy-management node written by **Gia Thanh Binh Duong** for the
EEE40016 Design of Smart Power Grids group project (Swinburne University of Technology, 2026).
The project co-simulates a street-level LV feeder in which a digital twin sends dynamic
operating envelopes to EV charge points over degraded telemetry. This repository contains
**only the battery node**. The digital twin, the DOE controllers, the communication layer
and the EV charge-point controller were written by other team members and are not included.

## Files

| File | What it is |
| --- | --- |
| `gia_battery.py` | **The file the project runs.** The adapter that puts the BMS behind the team's battery interface (`response` / `commit` / `status`), with the 50 kVA four-quadrant inverter's AS/NZS 4777.2:2020 Australia A volt-var and volt-watt curves. Pack: 96S156P A123 cells, about 100 kWh. |
| `bms_industrial.py` | The BMS core it uses: state machine (UNINIT → INIT → IDLE → STANDBY → PRECHARGE → NORMAL, ERROR), MOL/RSL/MSL limits with debounced diagnostics, coulomb-counting SoC with OCV rest recalibration, horizon-based state of function (power limits), derating, and HOLD/FALLBACK when commands expire. Also contains the pack plant model (ESC cell model, contactors, fuse). Structure inspired by foxBMS 2; independent Python implementation. Its built-in defaults (96S40P) are only for its stand-alone demo. |
| `a123_params.json` | A123 ANR26650M1A enhanced self-correcting (ESC) cell parameters (OCV, R0, R1, RC, hysteresis vs. temperature). |
| `export_a123_params.py` | How `a123_params.json` was produced from the esctoolbox-python pickle. |
| `test_bms_paths.py` | Scenario tests A–M (taper at high SoC, faults and acknowledge, precharge retries, SoC recalibration, command expiry, ...). |
| `bms_node_ui.py` | Live view of the battery node over MQTT (`--demo` runs without a broker). |

## Run

```bash
python test_bms_paths.py          # tests A-E, G-M pass; F documents a known limitation
python bms_industrial.py          # stand-alone BMS demo
python bms_node_ui.py --demo      # live view with a scripted evening (needs tkinter)
```

Use from Python, as the twin does:

```python
from gia_battery import GiaBattery
b = GiaBattery(soc0=0.6)
p_kw, q_kvar = b.response(v_local=245.0, setpoint_kw=0.0)  # load convention: P > 0 charges, Q < 0 supplies vars
b.commit(p_kw, minutes=5)
print(b.status())   # soc_true, soc_reported (BMS estimate), bms_state, t_max_c, ...
```

`python gia_battery.py <broker> [node] [folder_with_messages.py]` runs it as a stand-alone
MQTT node; that needs the project's shared `messages.py`, which is not in this repository.

## Sources

- A123 Systems, ANR26650M1A datasheet (MD100001-02, 2009): cell voltage, current and temperature figures.
- G. L. Plett, *Battery Management Systems*, Vols. I–II (Artech House, 2015) and esctoolbox-python: ESC cell model, SoC and state-of-power methods.
- foxBMS 2 (Fraunhofer IISB): architecture of the state machine, diagnostics and MOL/RSL/MSL limits.
- AS/NZS 4777.2:2020 (Australia A settings, as published by Jemena): volt-var and volt-watt curves.
- CSIP-AUS: command expiry and loss-of-communication behaviour.
