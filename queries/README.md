# Discover query guide

[Back to project](../README.md)

Use **DQL mode** in Discover. Select `wazuh-alerts-*` as shown in the captures, or the corresponding `wazuh-alerts-4.x-*` index pattern available in your deployment. Set an absolute time range covering the test before searching historical evidence.

These are reusable searches derived from the captured field names. They are not a saved-object export or a claim that each query was executed against the live lab during repository preparation.

## Start with the endpoint

```text
agent.name:"win-client"
```

```text
agent.name:"ubuntu-web"
```

The final IDs in this build are Windows `004` and Linux `002`. Name-based searches are easier to reuse; IDs can change after re-enrollment.

## Windows process creation

Limit Event ID 1 to the Sysmon channel so that an event ID from a different provider is not mistaken for Sysmon:

```text
agent.name:"win-client" AND data.win.system.channel:"Microsoft-Windows-Sysmon/Operational" AND data.win.system.eventID:"1"
```

Find the executable used in the account-enumeration tests:

```text
agent.name:"win-client" AND data.win.eventdata.originalFileName:"net.exe"
```

Narrow by command-line content:

```text
agent.name:"win-client" AND data.win.eventdata.originalFileName:"net.exe" AND data.win.eventdata.commandLine:*localgroup*
```

For `net user`, start with the executable query and inspect `data.win.eventdata.commandLine`. Do not confuse `net user` with commands that create or modify accounts; the captured command only lists them.

| Field | Why it matters |
|---|---|
| `data.win.eventdata.image` | Executable path |
| `data.win.eventdata.originalFileName` | Original filename metadata; useful search aid, not a trust guarantee |
| `data.win.eventdata.commandLine` | Actual requested operation |
| `data.win.eventdata.parentImage` | Process that launched it |
| `data.win.eventdata.parentCommandLine` | Parent execution context |
| `data.win.eventdata.user` / `parentUser` | User context when present |
| `data.win.eventdata.integrityLevel` | Process integrity level |
| `data.win.eventdata.processGuid` | Correlation identifier beyond a reusable PID |

## Linux authentication failures

Broad authentication-failure search:

```text
agent.name:"ubuntu-web" AND rule.groups:"authentication_failed"
```

Failed sudo/PAM event:

```text
agent.name:"ubuntu-web" AND predecoder.program_name:"sudo" AND rule.level:5
```

Failed user switch:

```text
agent.name:"ubuntu-web" AND predecoder.program_name:"su" AND rule.level:5
```

Read `full_log`, `location`, `decoder.name`, `predecoder.program_name` and user/terminal fields together. A broad PAM description does not identify the authentication service by itself.

## Nginx test requests

All captured web-rule matches:

```text
agent.name:"ubuntu-web" AND rule.id:"31101"
```

Only actual 404 responses:

```text
agent.name:"ubuntu-web" AND rule.id:"31101" AND data.id:"404"
```

Specific test stages:

```text
agent.name:"ubuntu-web" AND data.url:"/soc-test-missing-page"
```

```text
agent.name:"ubuntu-web" AND data.url:"/soc-test-without-nat"
```

```text
agent.name:"ubuntu-web" AND data.url:"/soc-test-after-reboot"
```

```text
agent.name:"ubuntu-web" AND data.url:"/soc-test-siem-reboot"
```

The captured source IP is `127.0.0.1` because curl ran locally. It is not the SIEM address or an external attacker address.

## When a search returns no results

Check the time range, timezone, correct agent identity and index pattern first. Remove the most restrictive condition and confirm that the expected field exists in an expanded document. Then trace source generation, agent collection, Manager rule matching and Indexer delivery. An empty result alone does not identify which stage failed.
