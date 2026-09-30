# Attack Scenario 01 — RDP Brute-Force Login Attempt

**Attacker:** Kali Linux VM
**Target:** Windows 11 VM (monitored endpoint)
**Telemetry source:** Windows Security Event Log (Event ID 4625 — failed logon), forwarded to Splunk via the Universal Forwarder
**Goal:** Simulate repeated failed authentication attempts and detect them in Splunk.

## 1. Setup

- [ ] On the Windows 11 VM: enabled Remote Desktop (Settings → System → Remote Desktop) and confirmed the firewall allows RDP (port 3389) from the lab network.
- [ ] Confirmed the Windows VM's IP address on the lab network (`ipconfig`).
- [ ] Confirmed Splunk is receiving Security log events before starting: `index=main sourcetype="WinEventLog:Security"`.

## 2. Attack

From the Kali VM, attempted RDP logins with an intentionally wrong password:

```bash
xfreerdp /v:<windows-vm-ip> /u:<real-username> /p:wrongpassword1
xfreerdp /v:<windows-vm-ip> /u:<real-username> /p:wrongpassword2
# repeat a handful of times
```

*(Stretch goal once the manual version works: automate this with Hydra —*
`hydra -l <username> -P <small-wordlist> rdp://<windows-vm-ip>` *— to generate a larger, more realistic burst of attempts.)*

## 3. Telemetry observed

In Splunk:

```
index=main sourcetype="WinEventLog:Security" EventCode=4625
```

- [ ] Screenshot of the raw events in Splunk (redact anything sensitive) → save to `screenshots/`
- Note here what fields were populated (e.g. `Account_Name`, `Source_Network_Address`/`Workstation_Name`, `Logon_Type`) and whether they needed the Splunk Add-on for Microsoft Windows to parse cleanly.

## 4. Detection

See detection section with rdp_failed_logon_detection for the Splunk search built to flag this pattern.

## 5. Findings / what I learned

- What Logon Type showed up for RDP attempts (should be 10 — RemoteInteractive)?
- How many failed attempts before the account would realistically lock out, if a lockout policy were configured?
- What would distinguish this from a legitimate user mistyping their password a couple of times (i.e. where's a reasonable alerting threshold)?

## 6. Remediation / hardening notes

- e.g. account lockout policy, RDP Network Level Authentication, restricting RDP to a jump host, MFA.
