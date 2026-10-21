# Windows Battery Level by WMI - Zabbix Template

A lightweight Zabbix monitoring template designed to track the remaining battery capacity of Windows devices (laptops, mobile workstations, or battery-backed endpoints) using Windows Management Instrumentation (WMI).

Compatible with **Zabbix 7.4+**.

---

## Features

- **Native WMI Data Collection**: Retrieves real-time battery charge percentage via `Win32_Battery` without third-party utilities.
- **Two-Tier Alerting**: Includes distinct Warning (Average) and Critical (High) triggers.
- **Flapping Prevention (Hysteresis)**: Separate problem and recovery expressions ensure alerts do not rapidly cycle during alternating load or charging states.
- **Trigger Dependency**: The low battery alert is automatically suppressed when the critically low trigger fires to reduce alert noise.
- **Preconfigured Visualizations**: Includes an SVG graph widget dashboard and standard graph for visual charge tracking.

---

## Requirements

- **Zabbix Server / Proxy**: Version 7.4.0 or higher.
- **Zabbix Agent / Agent 2**: Installed and running on the target Windows host.
- **WMI Permissions**: The Zabbix agent service account must possess permissions to query the `root\cimv2` namespace.
- **Hardware Support**: Target endpoint must possess a battery subsystem reporting under the `Win32_Battery` WMI class (standard on laptops and mobile devices).

---

## Installation

1. Download [`template_Battery Level by WMI.yaml`](./template_Battery%20Level%20by%20WMI.yaml) from this repository.
2. In the Zabbix Web UI, navigate to **Data collection** > **Templates**.
3. Click **Import** in the upper-right corner.
4. Select the `.yaml` file and ensure **Update existing** and **Create new** options are checked.
5. Click **Import**.
6. Link the template `Windows Battery Level by WMI` to your target Windows hosts or host groups.

---

## Macros

The template uses the following user macros to control alerting thresholds. You can customize these at the template or host level:

| Macro | Default | Description |
| :--- | :--- | :--- |
| `{$BATTERY.WARN.MIN}` | `20` | Minimum battery percentage before triggering a Warning alert. |
| `{$BATTERY.CRIT.MIN}` | `10` | Minimum battery percentage before triggering a Critical alert. |
| `{$BATTERY.HYST.OFFSET}` | `5` | Offset percentage added to thresholds before an alert recovers (prevents flapping). |

---

## Items

| Name | Key | Type | Interval | Units |
| :--- | :--- | :--- | :--- | :--- |
| Windows: Battery level | `wmi.get[root\cimv2,"Select EstimatedChargeRemaining from Win32_Battery"]` | Zabbix agent | Default (1m) | `%` |

---

## Triggers

| Severity | Name | Problem Expression | Recovery Expression |
| :--- | :--- | :--- | :--- |
| **High** | Windows: Battery level is critically low | `last(...) < {$BATTERY.CRIT.MIN}` | `last(...) >= ({$BATTERY.CRIT.MIN} + {$BATTERY.HYST.OFFSET})` |
| **Average** | Windows: Battery level is low | `last(...) < {$BATTERY.WARN.MIN}` | `last(...) >= ({$BATTERY.WARN.MIN} + {$BATTERY.HYST.OFFSET})` |

> **Note on Dependencies**: `Windows: Battery level is low` depends on `Windows: Battery level is critically low`. When battery level drops into the critical range, the warning alert is suppressed to keep notifications clean.

---

## Dashboards & Graphs

- **Dashboard**: `Battery Level by WMI` (Features an SVG Graph widget with legend statistics).
- **Graph**: `Battery Level by WMI` (Gradient line rendering showing charge levels).

---

## Author

- **Developer:** Warawuth Phralabraksa
- **GitHub:** [@shamankingx](https://github.com/shamankingx)
- **Email:** shamankingx@hotmail.com

---

## License

This project is open-source and available under the [MIT License](LICENSE).
