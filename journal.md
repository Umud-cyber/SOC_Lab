# Lab journal

[Back to project](README.md)

This journal follows the supplied capture sequence and the earlier lab discussion. Dates refer to capture filenames unless an explicit event timestamp is stated. It records observed work without inventing unrecorded commands or durations.

## 13 October 2026 — Network and platform preparation

The SIEM host was inspected with `hostname`, `ip -br addr`, `ip route` and the Netplan configuration. Both interfaces initially used DHCP: the host-only address was `192.168.38.130`, and the NAT address was `192.168.121.151`. VMware's host-only network settings were recorded.

The Indexer kernel prerequisite was set and checked at `vm.max_map_count = 262144`. Git 2.43.0 was available. An APT filename-extension warning appeared; the evidence does not show its resolution.

Evidence: E01–E03.

## 14 October 2026 — Stack connectivity and endpoint telemetry

The working SIEM state used `192.168.38.10` on the host-only interface. All three Wazuh containers were running. The Docker bridge had `172.18.0.1/16`, and an HTTPS check returned a login redirect after the earlier connectivity troubleshooting.

The Ubuntu agent was Active as `ubuntu-web`, ID `002`, at `.20`. Windows reached the Manager's enrollment port from `.30`. Its local Wazuh service was running, while an intermediate registration still appeared Pending as `WIN11`, ID `003`. The later inventory showed the final `win-client`, ID `004`, Active alongside Ubuntu.

Local Sysmon output was inspected, and Windows events were located in Discover. The `net user` and `net localgroup administrators` process records demonstrated centralized process telemetry. A failed `su` attempt produced a Linux level-5 authentication alert.

Evidence: E04–E14.

## 16 October 2026 — Detection checks, exposure reduction and recovery

An Nginx request to `/soc-test-missing-page` generated a 404 alert. A failed sudo/PAM authentication event was also located at level 5.

The initial Docker port output still exposed services on wildcard interfaces. The Compose configuration was then validated and reapplied with Dashboard and Manager bound to `192.168.38.10`, and the Indexer bound to `127.0.0.1`. Both endpoint agents were Active after the change.

The recorded NAT-off sequence produced a distinctive Nginx test alert and a Windows `net user` process event. Ubuntu and Windows recovery checks then showed running services and fresh indexed events. Following a SIEM reboot, all three containers were running with the restricted bindings, and `/soc-test-siem-reboot` produced a new rule-31101 alert.

Later captures recorded Docker/Compose versions, Docker's active/enabled state, final port bindings, a resource sample and `always` restart policies. The VM settings associated with the SIEM and Ubuntu endpoint were recorded too.

Administrator credential maintenance was also performed. Its source screenshot contains password-hash material and is excluded from the public evidence set.

Evidence: E15–E27 and E29–E33.

## Remaining evidence to collect

| Question | Additional evidence needed |
|---|---|
| Are both endpoints permanently isolated from NAT? | Final adapter connection states and live route tables, plus internal/external connectivity checks |
| Are clocks synchronized across all three systems? | Current Linux and Windows time-source/status output and Dashboard timezone |
| Did historical alerts survive a defined interval and restart? | The same known pre-restart document retrieved afterward |
| Can the lab be restored from backup? | A separate backup/restore test and fresh-event validation |
| What are the Windows VM hardware allocations? | A clearly identified Windows VM settings capture |

These items refine the record; they are not represented as completed tests.
