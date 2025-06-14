# Cisco NetFlow Configuration and Verification Lab

This lab guide provides step-by-step instructions for configuring and verifying NetFlow on a Cisco router using Cisco IOS. The lab assumes a basic understanding of Cisco IOS commands and networking concepts. The goal is to configure NetFlow to collect and export network traffic data to a NetFlow collector for analysis.

## Lab Objectives
- Configure NetFlow on a Cisco router.
- Export NetFlow data to a collector.
- Verify NetFlow configuration and operation.
- Monitor and troubleshoot NetFlow traffic.

## Lab Topology
- **Router**: Cisco Router (e.g., Cisco 2900 series) running Cisco IOS.
- **Interfaces**:
  - GigabitEthernet0/0: Connected to internal network (192.168.1.0/24).
  - GigabitEthernet0/1: Connected to external network.
- **NetFlow Collector**: A server running NetFlow analysis software (e.g., SolarWinds, PRTG) with IP address 192.168.1.100.
- **Management PC**: Connected to the router for configuration and verification.

## Prerequisites
- Cisco router with NetFlow support (IOS version 12.4 or later).
- Access to the router via console or SSH.
- NetFlow collector software installed and running on the server.
- Basic knowledge of Cisco IOS CLI.

## Lab Steps

### Step 1: Access the Router
1. Connect to the router using a terminal emulator (e.g., PuTTY, Tera Term) or console cable.
2. Enter privileged EXEC mode:
   ```
   enable
   ```
3. Enter global configuration mode:
   ```
   configure terminal
   ```

### Step 2: Configure NetFlow on the Interface
1. Enable NetFlow on the interface(s) where traffic will be monitored (e.g., GigabitEthernet0/0).
   ```
   interface GigabitEthernet0/0
   ip flow ingress
   ip flow egress
   exit
   ```
   - `ip flow ingress`: Captures incoming traffic.
   - `ip flow egress`: Captures outgoing traffic.

### Step 3: Configure NetFlow Export
1. Specify the NetFlow version (Version 9 is recommended for modern deployments).
   ```
   ip flow-export version 9
   ```
2. Define the destination IP address and port for the NetFlow collector (e.g., 192.168.1.100, port 2055).
   ```
   ip flow-export destination 192.168.1.100 2055
   ```
3. (Optional) Specify the source interface for NetFlow packets to ensure consistent export.
   ```
   ip flow-export source GigabitEthernet0/0
   ```

### Step 4: Configure NetFlow Cache
1. Set the NetFlow cache timeout values to control how often flows are exported.
   ```
   ip flow-cache timeout active 1
   ip flow-cache timeout inactive 15
   ```
   - `active 1`: Exports active flows every 1 minute.
   - `inactive 15`: Exports inactive flows after 15 seconds of inactivity.

### Step 5: Enable NetFlow Top Talkers (Optional)
To identify high-traffic flows, enable the Top Talkers feature.
```
ip flow-top-talkers
 top 10
 sort-by bytes
```

### Step 6: Save the Configuration
Save the configuration to ensure it persists after a reboot.
```
write memory
```

### Step 7: Verify NetFlow Configuration
1. Verify that NetFlow is enabled on the interface:
   ```
   show ip flow interface
   ```
   - Expected output:
     ```
     GigabitEthernet0/0
       IP flow ingress
       IP flow egress
     ```
2. Check the NetFlow export configuration:
   ```
   show ip flow export
   ```
   - Expected output:
     ```
     Flow export v9 is enabled for main cache
       Export source and destination details:
       Source: GigabitEthernet0/0 (192.168.1.1)
       Destination: 192.168.1.100 (2055)
     ```
3. Display the NetFlow cache to verify captured flows:
   ```
   show ip cache flow
   ```
   - This command shows active flows, including source/destination IPs, ports, and byte counts.
4. Verify Top Talkers (if enabled):
   ```
   show ip flow top-talkers
   ```
   - Displays the top 10 flows sorted by bytes.

### Step 8: Verify NetFlow Data on the Collector
1. Log in to the NetFlow collector software (e.g., SolarWinds, PRTG).
2. Confirm that the collector is receiving NetFlow data from the router.
3. Check for flow statistics, such as top talkers, protocols, or traffic patterns.

### Step 9: Troubleshoot NetFlow Issues
If NetFlow data is not reaching the collector, perform the following checks:
1. Verify connectivity to the collector:
   ```
   ping 192.168.1.100
   ```
2. Check for access control lists (ACLs) or firewalls blocking UDP port 2055.
3. Ensure the correct source interface is used for export:
   ```
   show running-config | include ip flow-export
   ```
4. Verify that traffic is passing through the monitored interface:
   ```
   show interface GigabitEthernet0/0
   ```

## Sample Configuration
Below is the complete NetFlow configuration for reference:
```
interface GigabitEthernet0/0
 ip flow ingress
 ip flow egress
!
ip flow-export version 9
ip flow-export destination 192.168.1.100 2055
ip flow-export source GigabitEthernet0/0
ip flow-cache timeout active 1
ip flow-cache timeout inactive 15
!
ip flow-top-talkers
 top 10
 sort-by bytes
```

## Lab Verification Checklist
- [ ] NetFlow is enabled on the correct interface(s).
- [ ] NetFlow export is configured with the correct destination IP and port.
- [ ] NetFlow cache settings are applied.
- [ ] Top Talkers is configured (if required).
- [ ] Configuration is saved to NVRAM.
- [ ] NetFlow data is visible in the router's cache (`show ip cache flow`).
- [ ] NetFlow data is received by the collector.
- [ ] Troubleshooting steps are followed if issues arise.

## Notes
- Ensure the NetFlow collector is configured to listen on the correct port (e.g., 2055).
- NetFlow can generate significant traffic, so monitor the router's CPU and memory usage with the `show processes cpu` and `show memory statistics` commands.
- For advanced analysis, consider using Flexible NetFlow for customized flow records.

## Additional Resources
- Cisco NetFlow Configuration Guide: [Cisco Official Documentation](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-netflow/tsd-products-support-series-home.html)
- NetFlow Collector Software: SolarWinds, PRTG, or open-source tools like nfdump.

This completes the Cisco NetFlow configuration and verification lab. Save this file for reference and ensure all configurations are tested in a lab environment before applying to production.