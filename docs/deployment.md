# Deployment record

[Back to project](../README.md)

This document describes the build established by the supplied evidence. Commands are operational guidance for this lab; they do not imply that every installation step was captured. The running deployment's private configuration was not supplied as source files.

## 1. Establish the network

The initial SIEM interface record has DHCP enabled for both interfaces and host-only address `192.168.38.130`. The recovered state later shows `ens34` at `192.168.38.10/24`, while `ens33` uses NAT address `192.168.121.151/24`.

Check the actual interfaces before editing networking:

```bash
hostname
ip -br addr
ip route
```

The agent target is the host-only address `.10`. The Windows enrollment test confirms TCP 1515 is reachable from `.30`. Enrollment reachability alone does not prove continuous event transport; the Active agent and indexed event checks complete that validation.

## 2. Prepare the platform

The captured Indexer prerequisite is `vm.max_map_count = 262144`. E03 shows it being loaded from `/etc/sysctl.d/99-wazuh.conf`. To recreate that specific setting on a lab host:

```bash
printf '%s\n' 'vm.max_map_count=262144' | sudo tee /etc/sysctl.d/99-wazuh.conf
sudo sysctl -p /etc/sysctl.d/99-wazuh.conf
sysctl vm.max_map_count
```

Docker and Compose are verified later as `29.9.0` and `v5.6.0`. The Wazuh images in this record are pinned to `4.14.8`. For a new deployment, consult the version-appropriate upstream installation instructions and preserve the chosen version; do not silently substitute today's upstream main branch for this recorded build.

The lab deployment directory is `~/wazuh-docker/single-node`. Its certificates, credentials and persistent data are maintained separately from this documentation repository.

## 3. Validate the stack

```bash
cd ~/wazuh-docker/single-node
sudo docker compose config --quiet
sudo docker compose up -d
sudo docker compose ps -a
```

The three expected containers are:

- `single-node-wazuh.manager-1`
- `single-node-wazuh.indexer-1`
- `single-node-wazuh.dashboard-1`

Names can differ if the Compose project name changes. The service names are preferable for Compose commands because they do not include the generated project prefix.

## Restricting published ports

[E17](../evidence/02-platform/ports-before-hardening.png) shows the initial wildcard mappings. The later configuration binds external lab services to `192.168.38.10` and the Indexer host API to `127.0.0.1`.

The [port example](../configs/port-bindings.example.yml) records these values. It is a partial reference, not a complete Compose file. Replace the corresponding `ports` lists in the private deployment. Do not add it as an unchecked Compose override: merged port lists can leave earlier wildcard mappings in place.

After editing:

```bash
cd ~/wazuh-docker/single-node
sudo docker compose config --quiet
sudo docker compose up -d
sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

`docker compose restart` alone does not apply edited Compose settings. Review the actual bindings after `up -d`; a syntactically valid file can still expose the wrong interface.

The post-change check confirms both agents Active in [E19](../evidence/03-agents/agents-after-port-hardening.png). Fresh-event tests then check that the collection path still works.

## 4. Connect the Linux endpoint

`ubuntu-web` runs Ubuntu 24.04.5 LTS, the Wazuh agent and Nginx 1.24.0. The agent's final identity is `002`, with host-only IP `.20`.

The public evidence identifies two working sources: journald authentication events and `/var/log/nginx/access.log`. The [Nginx source fragment](../configs/nginx-localfile.example.xml) illustrates the latter. The full journald configuration was not supplied and is not reconstructed as an exact export.

```bash
systemctl is-active wazuh-agent nginx
sudo tail -n 10 /var/log/nginx/access.log
curl -i http://127.0.0.1/soc-test-missing-page
```

The expected response is 404. Find the matching URL in Discover to validate the complete path.

## 5. Connect the Windows endpoint

The agent installation capture names the Manager `.10` and the agent `win-client`. The initial pending `WIN11` registration is superseded by Active agent `004` in the later screenshots. The exact steps that replaced the earlier record are not visible, so this document does not present an unverified deletion or re-enrollment command.

Sysmon local events must be checked separately from the Wazuh agent service. The [Sysmon source fragment](../configs/sysmon-localfile.example.xml) illustrates event-channel collection, not the Sysmon event-filter policy itself.

```powershell
Get-Service WazuhSvc, Sysmon64
net user
net localgroup administrators
```

Verify `net.exe` events in Discover. Record the image and command line together; a process name alone does not identify what the process did.

## Acceptance checkpoints

| Check | Evidence |
|---|---|
| Host-only SIEM address and Docker bridge available | E04 |
| Both correct agent identities Active | E09, E19 |
| Windows process events centrally searchable | E12, E13 |
| Linux authentication events searchable | E14, E16 |
| Nginx access-log request searchable | E15 |
| Restricted published bindings | E18, E30 |
| Fresh events after recovery checks | E23, E25, E27 |

The [evidence register](../evidence/README.md) maps each ID to its full-resolution image.

References: [Wazuh Docker deployment](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html), [Compose restart](https://docs.docker.com/reference/cli/docker/compose/restart/).
