# **1. Capture overview and infected host** 

|**Screenshot**|**What it shows**|**Report section**|
|---|---|---|
|Protocol Hierarchy|Traffic mix: HTTP is 80.8% of bytes; 639 'Data' items = 637 decoy POSTs<br>+trow.exe+t64.bin.|Type of Infection and<br>Scope|
|IPv4 Conversations|381 IPv4 conversations, all involving 192.168.1.96, most starting around<br>13:44.|Infected System|
|IPv4 Endpoints|382 IPv4 endpoints (backs up the conversations view).|Infected System|
|Ethernet Endpoints|Only two devices: the Dell PC (00:15:c5:de:c7:3b) and the Cisco router,<br>so this is a single-host capture.|Infected System|
|Internal IP Addresses in<br>Alerts|Across the 673 Snort and 842 Suricata alerts, the only internal host is<br>192.168.1.96 (apart from 3 DNS alerts involving the router, 192.168.1.1).|Infected System|
|Computer Name from DHCP|Frame 213: hostname FlashGordon-PC, IP 192.168.1.96, MAC address<br>and vendor class'MSFT 5.0'(Windows).|Infected System|
|NetBIOS Name Queries|NetBIOS lookups for unrelated domains from 13:44:03, a fallback after<br>DNS failures; no name registrations.|Type of Infection and<br>Scope|



# **2. How the infection happened** 

|**Screenshot**|**What it shows**|**Report section**|
|---|---|---|
|gerv.gun Download Request|Frame 6, 13:38:32: GET /gerv.gun from matied[.]com with the<br>'WinHttp.WinHttpRequest.5'User-Agent, i.e. a script, not a browser.|How It Was Infected|
|gerv.gun Download Stream|200 OK, 241,664 bytes; the body starts 'MZ ... This program cannot be<br>run in DOS mode', so it is an executable disguised as .gun.|How It Was Infected|
|Suricata First Alerts|Suricata's first three alerts are all on the gerv.gun download, including<br>'WinHttpRequest Downloading EXE' (Suricata times run about 10 h 36<br>min after packet times).|How It Was Infected|
|Google Connection Check|Frame 227: TLS Client Hello to www.google.com one second before the<br>first centler check-in (probably a connectivity check).|How It Was Infected|
|Centler Check-in Request|Frame 298, 13:43:50: POST /auth/ajax/847598782/?min=data to<br>centler[.]at; IE11 User-Agent with 'Windows NT 6.1; WOW64' (Windows 7,<br>64-bit).|How It Was Infected|
|Centler Encrypted Reply 1|Frame 302: 302 redirect to /auth/index.php with a 32-byte encrypted body;<br>the redirect does not point to the downloads.|Technical Reasoning|
|Centler Encrypted Reply 2|Frame 306: 302 reply with a 144-byte encrypted body, 0.46 s before the<br>next download.|How It Was Infected|
|trow.exe Download Request|13:43:52: GET /oud/trow.exe from lounge-haarstudio[.]nl with a fake IE8<br>User-Agent.|How It Was Infected|
|wp.exe Download Request|13:43:54: GET /wp.exe from vantagepointtechnologies[.]com with a fake<br>IE8 User-Agent.|How It Was Infected|
|Files Downloaded Over HTTP|Wireshark Export Objects list: trow.exe, wp.exe, the centler POSTs and<br>hundreds of decoy'/'objects.|How It Was Infected|
|Non-Decoy Web Traffic|With the decoys filtered out, only 8 web items remain: gerv.gun, two<br>Google TLS sessions, two centler POSTs, trow.exe, wp.exe and t64.bin.|How It Was Infected|



# **3. Malware identification** 

|**Screenshot**|**What it shows**|**Report section**|
|---|---|---|
|VirusTotal gerv.exe Result|55/70 engines; family labels Shiotob / URLZone (Bebloh), a banking trojan<br>and downloader.|Type of Infection and<br>Scope|
|VirusTotal gerv.exe Details|Compiled 26 Jun 2017 19:03 UTC; first submitted to VirusTotal 26 Jun<br>2017 22:38 UTC, the night before the capture.|Type of Infection and<br>Scope|
|VirusTotal trow.exe Result|66/71 engines; family labels point to the Cutwail/Pushdo spam bot.|Type of Infection and<br>Scope|
|VirusTotal trow.exe Details|First submitted to VirusTotal on 27 Jun 2017 at 18:34 UTC, after this<br>infection.|Type of Infection and<br>Scope|
|VirusTotal wp.exe Result|60/71 engines; family labels point to Ursnif (Gozi-ISFB), a banking trojan.|Type of Infection and<br>Scope|
|VirusTotal wp.exe Details|First submitted to VirusTotal on 27 Jun 2017 at 05:37 UTC; its 2015<br>compile time is implausible.|Type of Infection and<br>Scope|
|t64.bin Download Request|Frame 866, 13:43:58: GET /img/t64.bin from rts21[.]co[.]jp with no User-<br>Agent, 4.6 s after wp.exe.|Type of Infection and<br>Scope|
|t64.bin Download Stream|200 OK, 3,162,369 bytes, encrypted (no MZ header); Last-Modified 20 Jun<br>2017.|Type of Infection and<br>Scope|
|VirusTotal t64.bin Result|0/62 engines: nothing detects the encrypted module (community score<br>-27), which fits an encrypted Tor module.|Type of Infection and<br>Scope|



# **4. Spam bot behaviour and scope** 

|**Screenshot**|**What it shows**|**Report section**|
|---|---|---|
|Decoy POST Requests|Decoy POSTs to '/' from 13:44:01.381: octet-stream bodies, a fake IE6<br>User-Agent and encoded data.|<br>Type of Infection and<br>Scope|
|Decoy POST Count|Capture File Properties: 637 decoy POSTs in 41.026 s.|Type of Infection and<br>Scope|
|Decoy POST Servers|Conversations for the decoy filter: 294 servers over 596 connections.|Type of Infection and<br>Scope|
|Fake IE6 User Agent Count|http.user_agent contains "MSIE 6.0" = 637: every decoy and nothing else.|Type of Infection and<br>Scope|
|Mail Server Traffic|smtp filter: connections to mail servers worldwide from 13:44:12; Mail.Ru<br>refuses them with 421.|Type of Infection and<br>Scope|
|Mail Servers Contacted|Wireshark smtp conversations: 51 servers / 81 connections (SMTP-<br>recognised traffic only; tcpdump counts 239 attempts, 80 data connections,<br>58 servers).|Type of Infection and<br>Scope|
|Junk Sent to Mail Server|TCP stream: encoded junk sent before the server's 220 greeting; the<br>server replies 421'lost input connection'and no email is sent.|Type of Infection and<br>Scope|
|Suspected Control Server<br>Traffic|TCP stream 408 to 212.83.152[.]59:25: the client speaks first with binary<br>requests and gets about 144 kB of binary data back; not SMTP.|Type of Infection and<br>Scope|
|No Email Sent Check|Filter for MAIL FROM, RCPT TO and message data returns 0: no email<br>was delivered.|Type of Infection and<br>Scope|
|Failed DNS Lookups|11 NXDOMAIN replies, including myip.opendns.com sent straight to<br>OpenDNS (208.67.222.222) to learn the public IP.|Type of Infection and<br>Scope|
|Failed DNS Lookups Count|Capture File Properties: 11 NXDOMAIN replies; the failed names are real<br>but dead sites, not random (DGA) names.|Technical Reasoning|
|Sinkhole Responses|Six packets: 'Server: sinkhole' from 144.217.254[.]90, plus replies from<br>sinkholes 212.61.180[.]100 and 160.16.95[.]189.|Type of Infection and<br>Scope|
|Unusual Ports Check|Filter for TCP SYNs outside 80/25/443 and UDP outside<br>53/137/5355/67/68 returns nothing: no Tor or other control ports in use.|Technical Reasoning|
|Spread to Other Computers<br>Check|37 distinct mail servers behind Snort's 124:1 alerts; the Lab 8 checks for<br>traffic to or from other internal hosts both return 0.|Type of Infection and<br>Scope|



# **5. Detection analysis (Snort and Suricata)** 

|**Screenshot**|**What it shows**|**Report section**|
|---|---|---|
|Suricata Alert Summary|The 10 Suricata alert types with counts (823 Pushdo, sinkholes, IE6,<br>Conficker, WinHttpRequest).|Detection Analysis|
|Snort Mail Alert Destinations|Snort 124:1 'command overflow' alerts all go to mail servers on port 25;<br>none for 212.83.152[.]59.|Detection Analysis|
|Snort Default Run|VM default configuration: 72preprocessor alerts and no Pushdo alerts.|Detection Analysis|
|Snort Rules Switched Off|Rules 29891, 15306, 11192, 23832 and 15168 are commented out; 8,461<br>rules loaded.|Detection Analysis|
|Pushdo Rules Search|grep for 'pushdo' in /etc/snort/rules: sid-msg.map entries for older Pushdo<br>rules.|Detection Analysis|
|Pushdo Rule Disabled in VM|Rule 29891 is in the ruleset but commented out (#); it belongs to the max-<br>detect/security policies only.|Detection Analysis|
|Pushdo Rule Full Text|Rule 29891 requires an exact header sequence, including 'Connection:<br>Keep-Alive'.|Detection Analysis|
|Snort Pushdo Rule Switched<br>On|Rule 29891 enabled: 570 alerts = 72 + 498, reproducing the provided<br>Snort alert file.|Detection Analysis|
|Decoy POSTs With Keep-<br>Alive|514 decoy POSTs carry a Connection header.|Detection Analysis|
|Decoy POSTs Without Keep-<br>Alive|123 decoy POSTs have no Connection header, so rule 29891 can never<br>match them.|Detection Analysis|
|Keep-Alive Exact Match<br>Check|http.connection == "Keep-Alive" = 514, confirming the 514/123 split.|Detection Analysis|
|Snort Custom Rules Results|Text of the custom rules (3000001-3000004) and their alert counts:<br>3000001 = 621, 3000002 = 80, 3000004 = 1; 29891 = 498.|Detection Analysis|
|Snort Custom Rules Checks|3000004 fired on gerv.gun; 3000002 caught 212.83.152[.]59 and 58 mail<br>servers; 662 POST packets = 637 requests over 596 connections.|Detection Analysis|
|Snort Totals and Missed<br>Requests|Port 25 totals (239 attempts, 80 data connections, 58 servers) and the 16<br>single-POST connections missed by both Pushdo rules.|Technical Reasoning|
|Snort False Alarm Test|Custom rules run against 13 lab captures: 2,784 alerts from other rules,<br>none from the custom rules.|Detection Analysis|
|Suricata Alert Checks|The Conficker alert matches an LLMNR packet (a false positive);<br>Suricata's 823 Pushdo alerts fall on 584 connections, 210 with two or more<br>(duplicates).|Detection Analysis|



# **6. AI use logs** 

|**Screenshot**|**What it shows**|**Report section**|
|---|---|---|
|Screenshot 2026-11-11 at<br>18.19.17|AI chat log 1: asking Claude to explain the CW1 brief in simple terms.|AI Reflection|
|Screenshot 2026-11-11 at<br>18.19.40|AI chat log 2: asking which module topics to revisit for the coursework.|AI Reflection|
|Screenshot 2026-11-11 at<br>18.19.51|AI chat log 3: asking for a checklist of screenshots for the evidence folder.|AI Reflection|
|Screenshot 2026-11-11 at<br>18.20.01|AI chat log 4: asking for a proposed report structure.|AI Reflection|
|Screenshot 2026-11-11 at<br>18.20.25|AI chat log 5: asking for a grammar check of the final report (tracked<br>changes, wording unchanged).|AI Reflection|
|Screenshot 2026-11-11 at<br>18.32.24|Claude account settings showing the free plan.|AI Reflection|



