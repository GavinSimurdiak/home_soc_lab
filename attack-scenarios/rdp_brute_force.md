Simulating a Brute-Force RDP Attack:

For my first real attack scenario, I wanted to generate actual malicious activity against my Windows victim VM and see how it shows up in Splunk, rather than just looking at routine background logs. I picked an RDP brute-force attempt, since Remote Desktop is such a common real-world attack target, and Windows already logs every logon attempt — successful or not — to the Security log I had forwarding from earlier.

I enabled Remote Desktop on the Windows 11 VM and grabbed its IP with `ipconfig`. From Kali, my plan was to use FreeRDP's `xfreerdp` command to attempt logins with the wrong password on purpose, generating failed-logon events I could then detect in Splunk.

This turned into a bigger troubleshooting exercise than I expected, and not on the Splunk side — on the RDP connection itself. My first attempts, using the `+auth-only` flag (meant to just test credentials without opening a full desktop session), kept failing before ever reaching Windows at all. Digging into FreeRDP's verbose logs, I found the actual problem: FreeRDP was automatically trying to negotiate Kerberos authentication, and failing immediately because my Kali machine has no Kerberos realm configured — which makes sense, since this lab has no Active Directory domain at all, just a standalone Windows machine with a local account.

I tried forcing the connection down to the original, bare RDP security layer with `/sec:rdp` to sidestep Kerberos entirely, but Windows immediately reset that connection — modern Windows won't accept that old, unencrypted protocol anymore. The fix that actually worked was `/sec:tls`, which uses TLS encryption for the connection without going through the full NLA/CredSSP/Kerberos negotiation chain.

I also ended up dropping `+auth-only` completely and just running full connection attempts instead, since auth-only mode was giving me ambiguous exit codes I couldn't interpret from the logs alone. With a full connection, the result is unambiguous — either a real Windows desktop appears, or I get a clear "The user name or password is incorrect" dialog.

With the connection method finally sorted, I ran the actual test:

```bash
xfreerdp /v:<windows-ip> /u:socuser /p:WrongPassword /cert:ignore /sec:tls
```

run several times with a wrong password, followed by one attempt with the correct password.

## Results in Splunk

Searching:
```
index=main sourcetype="WinEventLog:Security" EventCode=4625
```

showed exactly the failed attempts I generated. Expanding one event confirmed:
- `Logon Type: 10` (RemoteInteractive — confirms these came in over RDP specifically, not console or network logons)
- `Account For Which Logon Failed: socuser`
- `Failure Reason: Unknown user name or bad password` (`Status 0xC000006D`, `Sub Status 0xC000006A`)
- `Source Network Address: 10.0.2.5` — my Kali VM

I ran the attempts in two separate bursts a bit apart in time: one burst of 6 failed logons and one of 10, both landing on `DESKTOP-GTDV9R5`. [Once you check EventCode=4740 around the second burst: note here whether it actually triggered a real account lockout, since 10 happens to match my configured lockout threshold.]

I also confirmed a real successful login by re-running the connection with the correct password, which opened an actual Windows desktop inside the FreeRDP window — proof the whole loop (attack → telemetry → detection) works end to end, not just the failure half.

## What I learned

- `Logon Type` is one of the most useful fields in the Security log for telling different kinds of logons apart — `10` specifically means RDP, `5` means a service logon (I initially grabbed one of these by mistake thinking it was my RDP session, before catching the difference).
- Getting a simulated attack to actually *reach* the target cleanly was harder than detecting it once it did — almost all my troubleshooting here was on the attacker (Kali/FreeRDP) side, not the SIEM side.
- Windows' own account lockout policy is a real, active defense worth being aware of when testing repeatedly against the same account — it's not just a theoretical control.

Next: turn this into an actual Splunk detection that would flag this pattern automatically.
