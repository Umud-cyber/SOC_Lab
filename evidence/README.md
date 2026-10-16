# Evidence register

[Back to project](../README.md)

The public set contains **32 original, unedited screenshots** captured on 13–16 October 2026. Descriptive filenames make them easier to navigate. The manifest preserves each original filename and SHA-256 checksum. E28 is intentionally absent: the administrator credential-maintenance screenshot contains password-hash material and is excluded from public version control.

These images show different stages of the build. An early DHCP address, wildcard port mapping or pending agent must not be mistaken for the later state. Screenshot filenames are capture labels, not authoritative event timestamps.

| ID | Evidence | What it establishes |
|---|---|---|
| E01 | [vmware-host-only-dhcp](01-network/vmware-host-only-dhcp.png) | Initial VMware network and DHCP address before the static SIEM address was applied. |
| E02 | [siem-initial-interfaces](01-network/siem-initial-interfaces.png) | Initial siem-host interfaces and Netplan: host-only address 192.168.38.130; NAT 192.168.121.151. |
| E03 | [indexer-kernel-prerequisite](02-platform/indexer-kernel-prerequisite.png) | vm.max_map_count set to 262144; Git 2.43.0 available. An APT source filename warning is also visible. |
| E04 | [dashboard-connectivity-restored](02-platform/dashboard-connectivity-restored.png) | Three Wazuh containers running; host-only 192.168.38.10; Docker bridge 172.18.0.1/16; Dashboard returns HTTP 302. |
| E05 | [ubuntu-agent-active](03-agents/ubuntu-agent-active.png) | Agent 002, ubuntu-web, 192.168.38.20, Ubuntu 24.04.5 LTS, Wazuh 4.14.8: Active. |
| E06 | [windows-enrollment-port](01-network/windows-enrollment-port.png) | Windows route table during setup includes NAT; TCP 1515 succeeds using host-only source 192.168.38.30. |
| E07 | [windows-agent-service](03-agents/windows-agent-service.png) | Windows Wazuh agent installation command and running WazuhSvc service. |
| E08 | [windows-registration-pending](03-agents/windows-registration-pending.png) | Intermediate registration: WIN11 agent 003 is Pending; this is not the final agent identity. |
| E09 | [two-agents-active](03-agents/two-agents-active.png) | Final identities: ubuntu-web 002 and win-client 004, both Active on Wazuh 4.14.8. |
| E10 | [sysmon-local-events](03-agents/sysmon-local-events.png) | Local Sysmon output includes process creation and a PowerShell-to-whoami parent/child relationship. Local and UTC timestamps differ; this is not clock-sync proof. |
| E11 | [windows-events-ingested](04-detections/windows-events-ingested.png) | Discover returns Windows events for agent 004, demonstrating centralized ingestion. |
| E12 | [windows-net-user](04-detections/windows-net-user.png) | Expanded Windows process event: net.exe user; image, command line and integrity level visible. |
| E13 | [windows-administrators-enumeration](04-detections/windows-administrators-enumeration.png) | Expanded process event for net.exe localgroup administrators. |
| E14 | [linux-failed-su](04-detections/linux-failed-su.png) | Failed su event from journald, decoded as su; rule level 5 and failed UID-change description. |
| E15 | [nginx-missing-page](04-detections/nginx-missing-page.png) | Nginx GET /soc-test-missing-page produces HTTP 404 and a level-5 web alert. |
| E16 | [linux-sudo-authentication-failure](04-detections/linux-sudo-authentication-failure.png) | sudo PAM authentication failure from journald; source user umud; rule level 5. |
| E17 | [ports-before-hardening](02-platform/ports-before-hardening.png) | Earlier container state: published ports bound to 0.0.0.0 and IPv6 wildcard; baseline resource sample. |
| E18 | [ports-after-hardening](02-platform/ports-after-hardening.png) | Compose validation and recreation; Dashboard/Manager bound to 192.168.38.10, Indexer to 127.0.0.1. |
| E19 | [agents-after-port-hardening](03-agents/agents-after-port-hardening.png) | Both agents Active after the port-binding change; Windows 11 Pro is visible. |
| E20 | [nginx-nat-off-test](05-recovery/nginx-nat-off-test.png) | GET /soc-test-without-nat reaches Discover. Test context records NAT-off operation; the image alone does not prove route isolation. |
| E21 | [windows-nat-off-test](05-recovery/windows-nat-off-test.png) | Side-by-side net user execution and matching Wazuh process event during the NAT-off test. |
| E22 | [ubuntu-services-after-reboot](05-recovery/ubuntu-services-after-reboot.png) | wazuh-agent and nginx active; /soc-test-after-reboot returns HTTP 404. Login banner contains older cached network information. |
| E23 | [nginx-alert-after-endpoint-reboot](05-recovery/nginx-alert-after-endpoint-reboot.png) | Post-endpoint-reboot HTTP 404 alert; rule 31101, level 5. |
| E24 | [windows-services-after-reboot](05-recovery/windows-services-after-reboot.png) | WazuhSvc and Sysmon64 running; net user executes successfully during the Windows recovery check. |
| E25 | [windows-alert-after-reboot](05-recovery/windows-alert-after-reboot.png) | Matching net.exe user event in Discover after the Windows recovery check. |
| E26 | [siem-containers-after-reboot](05-recovery/siem-containers-after-reboot.png) | Docker active and all three containers Up following SIEM reboot; restricted bindings persist. |
| E27 | [nginx-alert-after-siem-reboot](05-recovery/nginx-alert-after-siem-reboot.png) | Fresh /soc-test-siem-reboot alert after SIEM restart; rule 31101, level 5. |
| E29 | [docker-versions-and-service](02-platform/docker-versions-and-service.png) | Docker 29.9.0, Compose v5.6.0; Docker active and enabled. |
| E30 | [final-container-port-bindings](02-platform/final-container-port-bindings.png) | Final three Wazuh 4.14.8 containers Up; lab-bound Manager/Dashboard and loopback Indexer. |
| E31 | [resources-and-restart-policy](02-platform/resources-and-restart-policy.png) | Resource sample: Dashboard 190.4 MiB, Indexer 1.434 GiB, Manager 332.2 MiB; restart policy always for all three. |
| E32 | [siem-vm-settings](00-virtual-machines/siem-vm-settings.png) | VM settings associated with siem-host: 16 GB RAM, 4 processors, 250 GB disk; NAT and host-only adapters. |
| E33 | [ubuntu-vm-settings](00-virtual-machines/ubuntu-vm-settings.png) | VM settings associated with ubuntu-web: 2 GB RAM, 2 processors, 40 GB disk; NAT adapter is still listed alongside host-only. |

## Viewing the record

Open an image at full resolution to read its event fields. The [manifest](manifest.csv) allows the public images to be checked against the source captures. File hashes establish byte identity, not independent verification of the depicted lab state.

## Evidence boundaries

- The supplied screenshots verify specific observations, not a continuously monitored environment.
- NAT-off test labels and successful alert delivery do not prove permanent endpoint isolation. A final route table and adapter-state capture would be needed.
- Clock synchronization and long-term retention are not established by these captures.
- Screenshots of benign tests do not establish a real compromise or comprehensive detection coverage.
