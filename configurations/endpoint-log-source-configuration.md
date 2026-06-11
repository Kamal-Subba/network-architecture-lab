# Endpoint Log Source Configuration

## Purpose

This document provides a brief overview of how Linux endpoints in the lab are configured to forward selected log sources to Wazuh during initial onboarding.
The objective is to collect useful operational and security-relevant telemetry while reducing unnecessary log volume. Log collection is limited to sources that support authentication visibility, system and service review, and package management activity.

---

## Endpoint Log Configuration

Each enrolled Linux endpoint is configured to forward a focused set of local log sources to Wazuh. The selected sources currently include:

* Authentication and privilege-related activity
* General system and service activity
* APT package management history, including installation and removal events

This is done by backing up the existing `ossec.conf` file and modifying the original configuration to include the required `localfile` entries.

```xml
<!-- Custom endpoint log collection -->

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/apt/history.log</location>
</localfile>
```

These entries allow Wazuh to monitor selected endpoint logs without broadly collecting unnecessary system noise.

---

## Journald Collection

Journald collection remains enabled at this stage.
I have left it as-is to monitor for now and see if duplicate logs or noise become an issue.
---

## Verification

Log collection is verified at both the source-log level and the Wazuh agent level during onboarding.

### 1. Confirm source logs are being generated locally

```bash
sudo tail -f /var/log/auth.log
```

This confirms that the endpoint is generating authentication-related events locally, including SSH authentication activity and `sudo` usage.

Expected result:
Authentication or privilege-related log entries appear in real time when relevant activity occurs.

### 2. Confirm the Wazuh agent is processing configured log sources

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

This confirms that the Wazuh agent is running and actively processing configured log sources on the endpoint.

Expected result:
Agent log activity appears in real time, indicating that configured sources are being monitored successfully.

### 3. Confirm events are visible in Wazuh

After local validation, recent events are reviewed in the Wazuh dashboard to confirm that collected logs are being ingested successfully and are available for monitoring and analysis.

```
agent.id:<>
to confirm recent events from the endpoint
```
---

## Collection Approach

The log sources enabled on each endpoint are selected intentionally rather than collected by default in bulk.

The current approach is based on the following principles:

* Prioritize useful security and operational signals
* Reduce unnecessary noise during early onboarding
* Keep endpoint telemetry easy to validate and troubleshoot
* Expand collection only when a project or use case requires additional visibility

This keeps the logging configuration practical for lab operations while still supporting future troubleshooting, monitoring, and security analysis workflows.
