# Configuration references

[Back to project](../README.md)

These files are **minimal documentation examples**, not exported copies of the running lab configuration. The private Compose definition, full `ossec.conf` files, Sysmon filter policy, credentials and certificates were not supplied as files.

| File | Purpose | How to use |
|---|---|---|
| [port-bindings.example.yml](port-bindings.example.yml) | Records the final observed service bindings | Replace matching `ports` lists in the existing private Compose file; not a standalone deployment or unchecked override |
| [nginx-localfile.example.xml](nginx-localfile.example.xml) | Illustrates collection of the observed Nginx access log | Add one `localfile` block inside the Linux agent's existing `ossec_config`, if not already configured |
| [sysmon-localfile.example.xml](sysmon-localfile.example.xml) | Illustrates Sysmon Operational channel collection | Add one block inside the Windows agent's existing `ossec_config`, if not already configured |

Before adding a source, check both local and centrally managed agent configuration to avoid duplicate collection. Preserve the rest of the agent configuration. Restart the agent after a valid change and verify a fresh event.

The Windows example configures Wazuh collection; it does not install Sysmon or define which events Sysmon generates. The Linux example covers Nginx only; journald collection is established by the event evidence, but its exact configuration is not included.

The port fragment intentionally contains no credentials, images, volumes or certificate mounts. It cannot create a functioning SIEM on its own. Use the private working deployment and the appropriate upstream instructions for those settings.

Reference: [Wazuh localfile configuration](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html).
