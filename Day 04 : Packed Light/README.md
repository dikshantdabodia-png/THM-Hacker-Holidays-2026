# Day 4: Packed Light - TryHackMe Hacker Holidays Write-up

<p align="center">
  <img src="https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge" alt="Difficulty Easy">
  <img src="https://img.shields.io/badge/Category-Forensics-blue?style=for-the-badge" alt="Category Forensics">
  <img src="https://img.shields.io/badge/Tools-Wireshark%20%7C%20CyberChef-orange?style=for-the-badge" alt="Tools Wireshark | CyberChef">
</p>

---

## 📝 Objective

The room provides a network packet capture (`.pcap`) file and hints at investigating traffic on port 8080. The objective is to analyze the network traffic, identify anomalous behavior or exfiltrated data, reverse-engineer any custom obfuscation methods found, and recover the hidden flag.

---

## 🔍 Reconnaissance & Traffic Analysis

* **Initial Inspection:** Opened the provided `.pcap` file in Wireshark. Guided by the room hint, I applied the `http` filter to narrow down and isolate HTTP traffic across port 8080.
* **Spotting the Outlier:** Browsing through the packet list, one specific request immediately stood out. While most traffic consisted of standard web requests, a GET request was retrieving a Python script (`updates.py` with `Content-Type: text/x-python`).

<p align="center">
  <img src="images/day4-wireshark-http.png" alt="Wireshark HTTP filtered traffic showing the Python script and subsequent requests" width="800">
</p>

---

## 🚪 Exploitation / Decoding

* **Analyzing the Malware:** Following the HTTP/TCP stream for `updates.py` revealed the script's raw source code. Inspection showed it functioned as a keylogger and exfiltration tool: it encoded captured input using **Base64** and encrypted it via **XOR** using a hardcoded key (`H0t3lSc@ff0ldy...`).
* **Locating the Payload:** Analyzing the script logic revealed that data was exfiltrated chunk-by-chunk inside HTTP `Cookie` headers. Returning to the capture, I located the sequential exfiltrated chunks passed within the `Cookie: hotel_sess_state=` parameter across subsequent requests.

<p align="center">
  <img src="images/day4-wireshark-stream.png" alt="TCP Stream in Wireshark revealing the Python exfiltration script" width="800">
</p>

* **Data Extraction & CyberChef Decoding:** I extracted and combined the sequential encoded values from the `Cookie` headers into CyberChef and configured the decryption pipeline:
  1. `From Base64`
  2. `XOR` (UTF-8)
* **Key Adjustment:** Entering the full key from the script failed to yield legible text. Through testing, I determined the XOR logic relied only on the key's first character. Updating the XOR key to `H` successfully decrypted the payload and revealed the flag.

<p align="center">
  <img src="images/day4-cyberchef-flag.png" alt="CyberChef recipe successfully decoding the flag using Base64 and XOR" width="800">
</p>

---

## 🚩 Flag

* **Flag:** `THM{REDACTED}`

---

## 💡 Key Takeaways

* **Traffic Outliers:** When inspecting network captures (`.pcap`), filtering by protocol and scrutinizing unusual `Content-Type` responses (e.g., executable scripts masquerading inside web traffic) provides a fast path to the initial payload.
* **Covert Exfiltration Channels:** Malware frequently uses legitimate, innocuous HTTP headers like `Cookie` to conceal exfiltrated data, bypass simple perimeter rules, and blend in with benign noise.
* **Flexibility in Reversing:** Script analysis provides a map, but unexpected execution behavior or flawed logic might require manual adjustments—such as truncating an XOR key—to successfully reverse-engineer data.
