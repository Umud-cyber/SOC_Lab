# Troubleshooting record

[Back to project](../README.md)

## Dashboard timeout despite running containers

**Symptom.** The Dashboard was not reachable even though the three central containers were running.

**Finding from the earlier lab discussion.** The Docker bridge was missing its IPv4 gateway address, `172.18.0.1/16`. The bridge was identified in that session as `br-b51f001aff40`.

**Recorded recovery.** The address was restored on the bridge, followed by connectivity checks. The supplied [E04](../evidence/02-platform/dashboard-connectivity-restored.png) shows the recovered state: the bridge address is present, the three containers are running and an HTTPS request returns `302 Found` with a login redirect. The actual address-add command is not visible in that screenshot.

For a recurrence, inspect the current state first:

```bash
ip -br addr
sudo docker network ls
sudo docker network inspect single-node_default
```

The network name may differ with the Compose project name. Check the actual subnet, gateway and bridge before changing anything. The historical bridge identifier is not a reusable interface name for a new deployment.

A bridge address disappearing after suspend/resume was discussed during the original troubleshooting. The screenshots do not independently prove the ultimate cause. The documented result is restored connectivity, followed by later reboot validation.

## Windows registration stayed Pending

[E07](../evidence/03-agents/windows-agent-service.png) shows a running agent service. [E08](../evidence/03-agents/windows-registration-pending.png) still shows a Pending `WIN11` agent with ID `003`. [E09](../evidence/03-agents/two-agents-active.png) shows the final Active `win-client` record with ID `004`.

The lesson is to check the complete identity and state rather than treating a running service as successful enrollment. The later indexed process events verify useful collection. The screenshots do not show every action between the two registration states; no exact record-removal procedure is claimed.

## Local Sysmon event absent from Discover

The earlier discussion reports that an `ipconfig` event existed locally but was absent from the alert view. The supplied evidence later shows centralized `net.exe` process events.

Investigate in this order:

1. Verify that Sysmon recorded the expected event, including its timestamp and command line.
2. Check that the agent collects the Sysmon Operational channel.
3. Confirm agent connectivity and review the agent log.
4. Check the Manager's rule match and actual alert logging threshold.
5. Check Indexer delivery and the Discover index pattern/time filter.

A local event's existence identifies one working stage; it does not show that every downstream stage succeeded. Do not add a broad custom alert rule solely to make a screenshot appear without first identifying the missing stage.

## Wildcard Docker port mappings

The baseline in E17 publishes Manager, Dashboard and Indexer ports on all host interfaces. E18 and E30 show the later restricted bindings. The change was applied with Compose, followed by both agent-status and fresh-event checks.

Check actual output after modifying configuration. Restarting existing containers without applying the edited Compose definition can leave the original bindings unchanged.

## Misleading status and timestamps

The Ubuntu reboot-check image includes an old login banner with a NAT address and an older date. A login banner can contain cached information; use live `ip` and `date` output for current state.

The Nginx raw log uses UTC while Discover displays a four-hour offset. The early Windows local Sysmon view has a different offset. No screenshot in this set establishes a reliable cross-host clock-synchronization baseline.

## HTTP 404 shown under a “400 error” rule

Rule `31101` has the captured description `Web server 400 error code.` The actual HTTP status is in `data.id` and the raw log line. For these tests it is `404`. Read the fields before interpreting a generic alert title.

## APT source filename warning

E03 includes a warning about a Docker source-list filename extension. Git is present and the requested sysctl value is correct, but that does not prove the warning was resolved. The later Docker version and running stack show a functioning installed runtime. No unseen APT repair is claimed.

## Credential maintenance

The source set includes an administrator credential-maintenance screenshot with an Indexer configuration update and an authenticated HTTP 200 check. It also exposes password-hash material, so the complete capture is withheld from the public repository. This is a recorded maintenance activity, not a reason to publish authentication material as evidence.
