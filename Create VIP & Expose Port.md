Step 1 — Create VIP (Virtual IP)
FortiGate GUI: Policy & Objects → Virtual IPs → Create New → Virtual IP
FieldValueNameVIP-SQL-1433Interfacewan1External IP Address0.0.0.0 (means "any IP on WAN")Map to IPv4 Address192.168.1.x ← your SQL Server's LAN IPEnable Port Forwarding✅ Tick thisProtocolTCPExternal Service Port1433Map to IPv4 Port1433
Click OK
CLI:
bashconfig firewall vip
    edit "VIP-SQL-1433"
        set interface "wan1"
        set extip 0.0.0.0
        set extport 1433
        set mappedip 192.168.1.x
        set mappedport 1433
        set portforward enable
        set protocol tcp
    next
end

Step 2 — Create Firewall Policy
FortiGate GUI: Policy & Objects → Firewall Policy → Create New
FieldValueNameAllow-SQL-1433Incoming Interfacewan1Outgoing Interfacelan (or your internal interface)Sourceall ⚠️ (restrict to trusted IPs — see warning below)DestinationVIP-SQL-1433ServiceMS-SQL or custom port 1433ActionAcceptNAT✅ EnableEnable this policy✅ On
Click OK
CLI:
bashconfig firewall policy
    edit 0
        set name "Allow-SQL-1433"
        set srcintf "wan1"
        set dstintf "lan"
        set srcaddr "all"
        set dstaddr "VIP-SQL-1433"
        set service "MS-SQL"
        set action accept
        set nat enable
        set status enable
    next
end

Step 3 — Verify
From an external network (mobile data), test port 1433 is reachable using an online port checker like portchecker.co:

Host: 211.24.25.245
Port: 1433

Should show Open.
