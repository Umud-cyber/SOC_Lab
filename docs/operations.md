# Operations runbook

[Back to project](../README.md)

## Start

1. Power on `siem-host` and wait for the OS to finish booting.
2. Check Docker, then bring up the existing deployment:

```bash
cd ~/wazuh-docker/single-node
systemctl is-active docker
sudo docker compose up -d
sudo docker compose ps -a
```

3. Check the Dashboard response:

```bash
curl -k -I --max-time 10 https://192.168.38.10/
```

A login redirect shows that the HTTP application is reachable. It does not prove every component is healthy. A browser session and fresh alert check provide the next level of verification. `-k` bypasses certificate verification for the lab's untrusted certificate; do not treat that as validation of server identity.

4. Start the two endpoints. Confirm `ubuntu-web` (`002`) and `win-client` (`004`) are Active.
5. Run a small read-only Windows test or a distinctive Nginx missing-page request and locate the new event.

## Check health by layer

| Question | Check | Interpretation |
|---|---|---|
| Is Docker running? | `systemctl is-active docker` | Host service state only |
| Are containers running? | `sudo docker compose ps -a` | Process state; not proof of successful ingestion |
| Are ports bound correctly? | `sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'` | Compare with final host-only/loopback bindings |
| Is the web UI reachable? | HTTPS request to `.10` | HTTP response or useful connection failure |
| Are agents connected? | Dashboard endpoint inventory | Verify exact identity, not only count |
| Are local sources working? | Sysmon, journald, Nginx log | Verify the event exists before looking downstream |
| Is new data indexed? | Search a unique URL or recent command | End-to-end evidence |

## Inspect logs

Run from the deployment directory:

```bash
sudo docker compose logs --tail=100 wazuh.manager
sudo docker compose logs --tail=100 wazuh.indexer
sudo docker compose logs --tail=100 wazuh.dashboard
sudo docker stats --no-stream
```

On Ubuntu:

```bash
systemctl is-active wazuh-agent nginx
sudo journalctl -u wazuh-agent --no-pager -n 50
sudo tail -n 50 /var/ossec/logs/ossec.log
```

On Windows, use an elevated PowerShell session when event-log permissions require it:

```powershell
Get-Service WazuhSvc, Sysmon64
Test-NetConnection 192.168.38.10 -Port 1514
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 5 | Format-List TimeCreated, Id, Message
```

Review logs before sharing them; deployment logs and command history can contain private configuration or authentication material.

## Stop

Close active work and shut down `win-client` and `ubuntu-web` through their operating systems. On the SIEM:

```bash
cd ~/wazuh-docker/single-node
sudo docker compose stop
sudo poweroff
```

Avoid deleting containers, images or volumes as a routine shutdown procedure. `down -v` removes named volumes associated with the Compose project, which can destroy the state this lab is intended to retain.

## Restart behavior

The observed policy is `always` for all three containers, and Docker is enabled at boot. A manual stop suppresses immediate automatic restart; Docker can start those containers when its daemon restarts. Explicit `compose up -d` is the normal way to resume the stack during a session.

Restart policies do not guarantee that an application is ready to accept requests. The Indexer may need time to initialize before the Dashboard is usable. Check logs and responses instead of repeatedly restarting the entire stack.

## Changes to configuration

Validate syntax, apply the changed configuration, inspect bindings, and repeat a fresh event test. For Docker Compose settings, use `compose up -d`; `compose restart` does not re-read modified service settings. Keep the same deployment directory and project identity so that the existing volume configuration is used.

## Time checks for the next validation session

The supplied images do not establish synchronized clocks. Capture these outputs with a new test if time accuracy is being assessed:

```bash
date -u
timedatectl status
```

```powershell
Get-Date
w32tm /query /status
w32tm /query /source
```

Record the Dashboard timezone too. Distinguish source UTC time, ingestion timestamp and browser display time before comparing delays.

Reference: [Docker restart policies](https://docs.docker.com/engine/containers/start-containers-automatically/).
