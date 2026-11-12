# Lab 19: The Ping Command

## Lab Objective
Learn how to use the ping command and its main switches to test connectivity step by step.

## Lab Purpose
Ping (Packet Internet Groper) tests whether one device can reach another over an IP network. It sends ICMP Echo Request packets and waits for ICMP Echo Replies. It also measures round-trip time and packet loss.

Firewalls and ACLs often block ICMP, so a failed ping doesn't always mean the device is down. Ping is normally used with traceroute and other tools.

## Lab Tool
- Any Windows PC (a physical PC or a virtual machine such as Windows 10 in VirtualBox).
- If you use a VM, set its network adapter to Bridged or NAT so it can reach the internet.
- Make sure your firewall allows ICMP.

## Lab Topology

```text
[Your PC] ----- [Home Router / Default Gateway] ----- [Internet]
127.0.0.1         e.g. 192.168.1.1                 8.8.8.8, google.com
(loopback)
```

## Lab Walkthrough

### Task 1: Open the command prompt

Type `cmd` into the Windows search box and press Enter.

### Task 2: View the ping switches

Type:

```cmd
ping /?
```

Press Enter. Note these switches:

| Switch | Description |
|---|---|
| `-t` | Pings until you stop it |
| `-n count` | Sets the number of packets |
| `-l size` | Sets the payload size in bytes |
| `-f` | Sets the Don't Fragment flag (IPv4 only) |
| `-a` | Resolves the IP address to a hostname |
| `-w timeout` | Sets the reply timeout in milliseconds |
| `-4 / -6` | Forces IPv4 or IPv6 |

### Task 3: Test your own network stack (loopback)

Run:

```cmd
ping 127.0.0.1
```

**Expected:** Four replies, time<1ms, and a TTL of 128 on most Windows PCs.

A reply means TCP/IP is working on your PC. It does not test your network card or cable.

### Task 4: Test your local network (default gateway)

Run:

```cmd
ipconfig
```

Find the Default Gateway address (for example, `192.168.1.1`).

Run:

```cmd
ping <default gateway address>
```

**Expected:** Four replies with very low times (usually under 5 ms on a wired LAN or a few ms on Wi-Fi).

A reply means your PC can reach your router.

### Task 5: Test internet connectivity (public IP)

Run:

```cmd
ping 8.8.8.8
```

(Google's public DNS server). Use the default four packets only.

**Expected:** Four replies with a TTL that is lower than the starting value (common starting values are 64, 128 and 255).

A reply means your connection to the internet works, without involving DNS.

### Task 6: Test DNS (domain name)

Run:

```cmd
ping google.com
```

The first line shows the name resolved to an IP address, for example Pinging google.com [2a00.1450.x.x.x].

If Task 5 worked but this fails with "Ping request could not find host", you have a DNS problem, not a connectivity problem.

Try:

```cmd
ping -4 google.com
ping -a 8.8.8.8
```

The first command forces IPv4, and the second attempts a reverse lookup.

### Task 7: Continuous ping

Run:

```cmd
ping -t <default gateway address>
```

- Press Ctrl+Break to show statistics without stopping.
- Press Ctrl+C to stop and show the final statistics.

Do this only on your own network. Never run continuous pings against someone else's device or a public server, as it can look like an attack.

### Task 8: Change packet count and size, then find the MTU

Run:

```cmd
ping -n 5 -l 1000 8.8.8.8
```

(Five packets with a 1000-byte payload).

Run:

```cmd
ping -n 5 -l 1472 -f 8.8.8.8
```

This should succeed on a standard Ethernet path.

Run:

```cmd
ping -n 5 -l 1473 -f 8.8.8.8
```

This should fail with "Packet needs to be fragmented but DF set."

**Why 1472?**

The standard Ethernet MTU is 1500 bytes. The IP header (20 bytes) and ICMP header (8 bytes) use 28 of those, leaving 1472 bytes of payload.

Your path MTU is the largest successful payload plus 28. If 1472 fails on your connection (for example, with PPPoE links), lower the size until it works.

### Task 9: Recognize common results

| Result | Description |
|---|---|
| Request timed out | No reply arrived. The host may be down, or ICMP may be blocked. |
| Destination host unreachable | A router (or your own PC) reports it has no route to the destination. |
| Ping request could not find host | DNS could not resolve the name. |
| TTL expired in transit | The packet ran out of hops before reaching the destination. |

## Check Questions

1. **What does TTL measure?**

   The number of hops remaining, not time. Each router reduces it by 1, and the packet is discarded at 0.

2. **What does time= show?**

   The round-trip time in milliseconds.

3. **A ping to 8.8.8.8 works but ping google.com fails. What is the likely problem?**

   DNS.

4. **What does 0% packet loss tell you?**

   Every request received a reply.

5. **Why might a ping fail even though the host is up?**

   A firewall or ACL is blocking ICMP.

## Notes

Pinging in this order (loopback, then gateway, then a public IP, then a domain name) is a standard troubleshooting method. Each step rules out one layer, so the first failure shows you where the problem is.

Ping is also best combined with tracert and nslookup. Many sites and ISPs block ICMP, so use reliable targets such as 8.8.8.8 or 1.1.1.1.

## Conclusion

This lab covers the use of the Windows Ping command to test network connectivity, examine packet loss and round-trip time, investigate DNS resolution, and explore packet size and MTU.
