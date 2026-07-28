# Day 0: The Brochure - TryHackMe Hacker Holidays 2026 Write-up

*Difficulty:* Easy | *Category:* OSINT | *Points:* 30 | *Tools Used:* ExifTool, Google Image Search, CyberChef

---

## 📝 Objective

The goal of this room is to analyze a provided brochure image for embedded clues, apply fundamental OSINT techniques to trace those findings, and locate a hidden social media account. The concierge briefing notes that the image carries an "unmistakable AI fingerprint".

---

## 🔍 Image Analysis & Reconnaissance

* **Initial Check:** I started by downloading the task image and running it through `exiftool` to check for any hidden metadata (like GPS coordinates or embedded creator tags).

<p align="center">
  <img src="YOUR_EXIFTOOL_IMAGE_URL" width="650" alt="exiftool thebrochure.png">
</p>

* **Result:** The metadata was clean and did not reveal anything useful.
* **The Pivot:** A hint provided by `@0xMia` suggested that the image looked "suspiciously perfect," heavily implying AI generation and prompting further external investigation. Following the clues pointing toward Instagram, I decided to run the image through Google Image Search to see where it originated.

---

## 🕵️‍♂️ OSINT & Social Media Tracing

* **Locating the Resort:** The Google Image Search successfully linked the AI-generated brochure image to an Instagram account named *thebytelotusresort*.
* **Following the Breadcrumbs:** I navigated to the resort's Instagram page. To find the hidden connection mentioned in the briefing, I checked the accounts they were interacting with. I looked at their "Following" list and noticed they were only following *one* account.
* **Finding the Target:** That single followed account belonged to *veratheconcierge*.

<table align="center">
  <tr>
    <td align="center">
      <img src="YOUR_BYTE_LOTUS_IMAGE_URL" width="380" alt="thebytelotusresort Instagram"><br>
      <sub><b>thebytelotusresort Instagram</b></sub>
    </td>
    <td align="center">
      <img src="YOUR_VERA_IMAGE_URL" width="380" alt="veratheconcierge Instagram"><br>
      <sub><b>veratheconcierge Instagram</b></sub>
    </td>
  </tr>
</table>

---

## 🔓 Decoding & Exploitation

* **Analyzing the Content:** I checked Vera's account and found 3 posts. Instead of normal captions, the posts contained random strings of text.
* **Identifying the Cipher:** The alphanumeric format of the strings and the presence of equal signs (`=`) at the end heavily indicated that the text was encoded using Base64.
* **Decoding:** I copied the encoded strings and pasted them into *CyberChef*. Using the "From Base64" recipe, I decoded the text, which revealed the final flag.

<p align="center">
  <img src="YOUR_CYBERCHEF_IMAGE_URL" width="700" alt="CyberChef Decoding Base64">
</p>

---

## 🚩 Flag

* **Flag:** `THM{REDACTED}`

---

## 💡 Key Takeaways

* When standard metadata analysis (`exiftool`) yields no results, reverse image searching is an excellent fallback technique for OSINT.
* In social media investigations, analyzing an entity's connections (like their "Following" or "Followers" lists) is just as important as analyzing their direct posts.
