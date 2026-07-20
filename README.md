# OPNsense-scripts

There are three scripts (so far) in here, which I've created to get some additional functionality out of OPNsense where I wasn't happy with what was available or could be configured.

## 50-wan-watchdog
/usr/local/etc/rc.syshook.d/monitor/50-wan-watchdog

React to OPNsense gateway-monitor events for one or more configured gateways, forcing OPNsense to reset dhclient on the interface and attempt to pull a new IP.

Behavior:
  - When a configured gateway is currently in the exact state "down", run `configctl interface reconfigure <interface>` once per minute until that gateway is no longer exactly "down".

## 60-wan-pushover
/usr/local/etc/rc.syshook.d/monitor/60-wan-pushover

Send a Pushover notification when a gateway passed by the OPNsense monitor syshook changes state according to `pluginctl -r return_gateways_status`

Behavior:
  - Processes only gateway names passed in argument 1 by the monitor syshook.
  - Reads current status from pluginctl JSON.
  - Sends an alert only when the status changes.
  - Suppresses duplicate alerts.
  - Uses the gateway name in the alert.
  - Includes old status and new status in the alert.
  - Handles any non-empty status string, including:
      none, down, force_down, unknown, etc.

## 20-wan-pushover-bootcheck
/usr/local/etc/rc.syshook.d/start/20-wan-pushover-bootcheck

After OPNsense network startup, wait a short stabilization period and then inspect all gateways returned by: `pluginctl -r return_gateways_status`. If a gateway is still not up after boot, send one alert and seed the same state cache used by 60-wan-pushover.

Behavior
  - Wait BOOT_DELAY_SECONDS after startup.
  - Query all gateways from pluginctl JSON.
  - For each gateway:
      * none       -> healthy, no alert
      * anything else -> send alert
  - Always update the shared cache in /var/run/opnsense-wan-pushover/
  - Do not send duplicate alerts if the shared cache already contains the same
    status, which can happen if the monitor script has already run.


*Note: AI (Perplexity) was used to assist with coding and troubleshooting.*
