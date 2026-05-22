
## 1. Global Brute-Force Protection (Admin Lockout)

FortiOS hides default values from standard show commands. The lockout threshold of 3 attempts is active by default but can be verified alongside custom durations via the CLI.

- **Policy:** Limit failed administrative logins to 3 attempts, with a 5-minute (300 seconds) cool-down penalty box.
    
- **Configuration Interface:** CLI Only.
    

Plaintext

```
config system global
    set admin-lockout-threshold 3
    set admin-lockout-duration 300
end
```


### Verification & Management Commands

- **View Hidden Settings:** get system global | grep lockout or show full-system global | grep lockout
    
- **Manual Unlock (by Username):** execute vpn login-id close
    
- **Manual Unlock (by Quarantined IP):** diagnose user quarantine delete src <banned_ip>

## 2. Perimeter Management Plane Hardening

Because the upstream ISP router forwards all traffic to the FortiGate via a **DMZ Passthrough / Double NAT** mapping, the FortiGate WAN interface must be hardened against automated public probing.

Plaintext

```
config system interface
    edit "wan1"
        set allowaccess ping
    next
end
```

_(Removes HTTPS, SSH, and HTTP management protocols from the public-facing port, leaving only ICMP ping active)._

### Administrative Safety Countermeasures

1. **Change Default Ports:** Shift HTTPS management away from TCP 443 to a non-standard alternative (e.g., 8443) via **System > Settings**.
    
2. **Enforce Trusted Hosts:** Explicitly restrict administrative account access to internal subnet structures to ensure external actors cannot reach the login portal.
    

Plaintext

```
config system administrator
    edit "admin"
        set trusthost1 192.168.1.0 255.255.255.0
    next
end
```
