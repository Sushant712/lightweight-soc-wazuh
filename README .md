# Lightweight SOC for a Logistics SME

**Author:** Sushant Ghimire Khatri  
**Unit:** SCI701  
**Project type:** Academic laboratory prototype

## About the project

This project explores affordable security monitoring for a simulated logistics business with about 50 employees. The prototype uses Wazuh and Sysmon to collect security events from one Windows 11 virtual machine and display alerts in a central dashboard.

The Ubuntu virtual machine runs the Wazuh manager, indexer and dashboard. The Windows virtual machine runs the Wazuh agent and Sysmon.

## Submission files

| File | Purpose |
| --- | --- |
| `local_rules.xml` | Exported local Wazuh rules, including the supplied example and project custom rules |
| `soc-dashboard.ndjson` | Exported SOC dashboard and its related saved objects; originally exported as `export.ndjson` |
| `mitre-coverage.json` | Editable MITRE ATT&CK Navigator layer |
| `README.md` | Project overview and setup instructions |

The final report contains the screenshots, testing evidence, policy and supporting discussion. This repository contains project configuration exports, rather than the full Wazuh software or virtual machines.

## Requirements

- An Ubuntu virtual machine with a supported Wazuh installation.
- A Windows 11 virtual machine with the Wazuh agent and Sysmon.
- Network connectivity between the two virtual machines.
- Administrator access to configure the lab.
- A browser to access the Wazuh dashboard.

Use compatible Wazuh component versions. The original Windows agent configuration and Sysmon configuration have not been verified as included in this submission, so these exports alone do not provide a complete copy of the original environment.

## Setup and use

1. Install the Wazuh manager, indexer and dashboard on Ubuntu using the official Wazuh installation guide.
2. Open the Wazuh dashboard and use its agent deployment instructions to install and enrol the Windows agent. Use your lab server address, then confirm the agent shows as Active.
3. Install Sysmon on Windows using a reviewed Sysmon configuration. Configure the Wazuh agent to collect the Windows Security log and the `Microsoft-Windows-Sysmon/Operational` event channel. Enable the Windows audit settings needed for the selected tests. Restart the agent after configuration changes.
4. Back up `/var/ossec/etc/rules/local_rules.xml` on the Wazuh manager. Review and merge the submitted custom rules into that file. Avoid duplicate rule IDs and keep any existing rules that the environment needs.
5. Test the rules with `/var/ossec/bin/wazuh-logtest` and representative event logs. After successful validation, restart the Wazuh manager using `sudo systemctl restart wazuh-manager`.
6. In the dashboard's saved-object management screen, import `soc-dashboard.ndjson`, including its related objects. If the file is still named `export.ndjson`, use that file instead. Open **SME SOC Monitoring Dashboard** and select a time range containing alerts.
7. Open `mitre-coverage.json` in MITRE ATT&CK Navigator to view the project layer.

For normal use, start both virtual machines, confirm the Windows agent is Active, and review alerts in Threat Hunting or the custom dashboard.

## Custom rules

| Rule ID | Configured purpose | Level | ATT&CK mapping |
| --- | --- | --- | --- |
| 100301 | Repeated failed logins for the same target username within 180 seconds | 12 | T1110.001 |
| 100302 | Local account creation outside 8 am–6 pm | 10 | T1136.001 |
| 100303 | Member added to the local Administrators group | 12 | T1098.007 |

Rule 100301 has `frequency="5"`, `timeframe="180"` and `ignore="180"`. Confirm its observed event count when testing with the installed Wazuh version. The after-hours rule uses the Wazuh manager's time; check its timezone. The group-name match in rule 100303 assumes the English name `Administrators`.

Including a custom rule does not prove that it generated a particular alert. Check the actual rule ID in each alert; built-in rules can also detect these activities.

## Testing and coverage

The project testing covered suspicious PowerShell, repeated failed logins, local account creation, after-hours account creation, administrator-group changes and Windows Defender being disabled. One selected Atomic Red Team PowerShell test was also used. See the final report for evidence and results.

Phishing and business email compromise were covered through a tabletop exercise only. Actual email logs were not connected to Wazuh.

In the Navigator layer, green indicates a tested activity detected in the lab. Yellow is labelled Planned and represents the phishing/BEC tabletop scope. Neither colour means complete coverage of every behaviour within an ATT&CK technique.

## Limitations

- Only one Windows endpoint was monitored.
- The prototype was not deployed in a real organisation.
- Automated endpoint isolation, external ticketing and email/Slack notifications were not implemented.
- The full Atomic Red Team test collection was not run.
- Rules and dashboards need review and tuning before use in another environment.

## Official documentation

- [Wazuh installation guide](https://documentation.wazuh.com/current/installation-guide/index.html)
- [Wazuh custom rules](https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html)
- [Wazuh Windows event collection](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html)
- [Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [MITRE ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)

Only use controlled tests in an authorised laboratory. Keep passwords, private keys and agent authentication keys out of the repository.
