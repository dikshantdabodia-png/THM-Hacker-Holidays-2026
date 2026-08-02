# Day 5: Beach Bar - TryHackMe Hacker Holidays Write-up

**Difficulty:** Easy
**Category:** Boot2Root
**Tools Used:** Browser Developer Tools, Netcat (nc)

## 📝 Objective
The concierge briefing mentions a beach bar jukebox that takes requests and a DJ who never logs out. The hint suggests that the night-shift developer wired the jukebox straight into the backend. The objective is to gain initial access via the web application, escalate privileges, and completely compromise the server (Boot2Root).

## 🔍 Reconnaissance & Enumeration
*   **Source Code Inspection:** I started by navigating to the Beach Bar website in my AttackBox. Inspecting the page source code revealed a hidden developer comment mentioning a demo login: `dj/dj`.
*   **Initial Access (Web):** I used the credentials `dj` for both the username and password, which successfully logged me into the dashboard.

![Beach Bar Dashboard Logged In](Screenshot 2026-08-02 235243.png)

*   **Application Logic:** Exploring the dashboard, I found that only the "Import" and "Export" features were functional. Clicking "Export" downloaded a YAML (`.yml`) file containing the current playlist structure:
    ```yaml
    playlist:
      name: Season Session
      vibe: golden hour
      tracks:
        - artist: Khruangbin
          title: Maria Tambien
    ```

## 🚪 Initial Access / Exploitation (User)
*   **Testing for Insecure Deserialization:** I suspected the application might be vulnerable to YAML deserialization if it blindly parsed the imported files using a Python backend. I modified the `name` field to test basic Python execution:
    ```yaml
    playlist:
      name:  !!python/tuple ["Season", "Session"]
      ...
    ```
    The application accepted this and reflected it successfully. 
*   **Confirming Code Execution:** To confirm blind code execution, I injected a time delay payload:
    ```yaml
    name: !!python/object/apply:time.sleep [5]
    ```
    The application hung for exactly 5 seconds, confirming Remote Code Execution (RCE).
*   **Enumerating the User Flag:** I used the `subprocess.check_output` payload to run `ls -la /home/bartender` and then `cat /home/bartender/user.txt`, revealing the flag in the UI response.

![Payload enumerating the user directory](Screenshot 2026-08-03 000525.png)
![Reading the user.txt flag via the UI](Screenshot 2026-08-03 000637.png)

*   **Getting a Reverse Shell:** While reading files via the UI worked, getting an interactive shell is much better. I researched YAML reverse shell payloads and found the appropriate syntax.

![YAML Reverse Shell Payload Reference](Screenshot 2026-08-03 001211.png)

*   I set up a Netcat listener on my AttackBox (`nc -nvlp 1234`), modified the YAML file with the reverse shell payload containing my AttackBox IP, and uploaded it. The server connected back, granting me a shell as the `bartender` user!

![Uploading the reverse shell payload and catching the connection](Screenshot 2026-08-03 002020.png)

## 👑 Privilege Escalation (Root)
*   **System Enumeration:** Now that I had a shell, I explored the local directory and found a file named `jukeboxd.py`. 
*   **Finding Hardcoded Credentials in Processes:** Reading through the script logic, it referenced a stream password. I checked the running system processes to see how the jukebox daemon was being executed:
    ```bash
    ps aux | grep jukebox
    ```
*   **Extracting the Password:** The process list revealed that the root user was running the script with the password passed directly in the command line argument (`--stream-pass SunsetSpritz2024!`).
*   **Root Shell:** I simply used the `su root` command, pasted the discovered password, and successfully escalated my privileges to `root`. From there, I navigated to `/root` and read the final flag.

![Escalating privileges to root and capturing the final flag](Screenshot 2026-08-03 002723.png)

## 🚩 Flags
*   **User Flag:** `THM{REDACTED}`
*   **Root Flag:** `THM{REDACTED}`

## 💡 Key Takeaways
*   **Insecure Deserialization:** Directly parsing user-supplied YAML files (specifically using unsafe load functions in Python's `yaml` library) allows attackers to instantiate arbitrary Python objects, leading to immediate Remote Code Execution.
*   **Process Enumeration:** Passing sensitive information like passwords directly as command-line arguments is a critical misconfiguration. Anyone with lower-level shell access can read those arguments using `ps aux` and easily escalate privileges.
