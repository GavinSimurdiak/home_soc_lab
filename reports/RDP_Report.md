**GOAL**
The goal of this activity was to simulate a real attack against a Windows VM (where someone is actually trying to guess a password over Remote Desktop). This activity proved that Splunk can actually catch it through log detection, as when the attack happens, Windows logs it, Splunk shows it, and then one can build a rule to flag this behavior in the future.

**Walkthrough**
From Kali, a tool named xfreerdp (a command line program that connects to a Windows machine over Remoted Desktop) was used to try logging on to the Windows 11 Enterprise VM. At first, wrong passwords were attempted to generate logs, but eventually, the correct password was used.

**Problems**
1. Windows Trial License 
The Windows 11 Enterprise free trial 90 day license is nearing its end (10 days left), and I tried to fix it through using /rearm, but after I did this, I had to restart, but everytime I restarted the VM crashed and my rearm "tokens" got used. So, I had to roll back to a previous snapshot where I had 85 days left on the trial, but for some reason, towards the end it went back to 10 days so I will have to look further into fixing this issue. 
2. RDP Connection Failures
At first, when trying to do wrong password RDP connections, the connection was dying because the RDP client was using Kerberos (a Windows domain login method) automatically. To try and fix this, I tried using an older, simpler connection with /sec:rdp, but Windows refused this as it was an outdated protocol. So, the fix was telling the client to use TLS encryption (/sec:tls).
3. Account Lockout
During the above process, when I was trying to troubleshoot, I accidentally used over 10 connections with wrong passwords to the client, which triggered an account lockdown where I could not longer sign in. So, I was yet to do my correct password attempt and was locked out. I logged on to the Windows VM and ran net accounts which showed that I only had to wait 10 minutes to retry, and for the future, I only have 10 attempts before lockout is triggered. 

**Results**
This activity ended up with real proof in Splunk of failed login attempts and a successful login, both of which respectively showed up as correctly labeled events. Also, a detection rule I made in Splunk flaged the suspicious bursts of failures without falsely flagging anything else.

**Learning**
This activity taught me that a lot of the real-problem solving isn't involving Splunk or detection, but rather getting the attack itself to work. Getting a test to actually successfully reach its target is extremely difficult, and troubleshooting these smaller problems using AI or documentation is an effective way to get back on track.