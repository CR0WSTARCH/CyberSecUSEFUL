
This is probably going to be the most useful thing I can give you.
Cybersec is a very information dense field and trying to learn everything at once is impossible.

But here are the tools and little titbits of information that I rely on and I think being familiar with them is a good idea.


## Tools


### Quick FYI
A lot of pen test distros come with these tools already on there but in case you don't have them,
the default way of downloading is 

```bash

Package Manager (Debian/Ubuntu/Kali)

bash sudo apt update && sudo apt install <tool_name>

or


pipx install <tool_name>

or

git clone [https://github.com/user/repo-name.git](https://github.com/user/repo-name.git)
cd repo-name
pip install -r requirements.txt  # If it's a Python tool

```

## Nmap

The industry standard for network discovery. Nmap uses raw IP packets to determine what hosts are available on the network, what services (application name and version) those hosts are offering, what operating systems they are running, and what type of packet filters/firewalls are in use.

| **Flag**     | **Mnemonic**            | **Description**                                                                                                                                  |
| ------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `-sC`        | **S**cripts **C**ommon  | Runs default nmap scripts (safe & useful for basic recon).                                                                                       |
| `-sV`        | **S**ervice **V**ersion | Attempts to determine the version of the service running on the port.                                                                            |
| `-p-`        | **P**orts (All)         | Scans all 65,535 ports (default only scans top 1000).                                                                                            |
| `-O`         | **O**S Detect           | Attempts to guess the Operating System based on TCP/IP fingerprinting.                                                                           |
| `-oA <name>` | **O**utput **A**ll      | Saves output in three formats: Normal, XML, and Grepable. **ALWAYS DO THIS.**                                                                    |
| `-T4`        | **T**iming **4**        | Speed template (0-5). T4 is aggressive/fast but usually reliable.                                                                                |
| `-v`         | **V**erbose             | Shows open ports as they are found (don't wait for the scan to finish).                                                                          |
| -sS          | Stealth Scan            | "half-open" technique that identifies open ports without completing the three-way handshake, reducing detection by firewalls and logging systems |

### Common Commands

**The "I need results now" Scan (Initial Enumeration):** Scans top 1000 ports, detects versions, runs default scripts.

Bash

```
sudo nmap -sC -sV -oA initial_scan <target_ip>
```

**The "Full Noise" Scan (Deep Enumeration):** Scans ALL ports. (Don't do this)

Bash

```
sudo nmap -p- -sC -sV -T4 -v -oA full_scan <target_ip>
```

To understand NMAP deeper: learn routing thro proxies and firewall evasion.





##  Ffuf (Fuzz Faster U Fool)
(i didnt know this was an acronym tills i wrote this)


A fast web fuzzer written in Go. It allows you to discover hidden directories, files, and subdomains by throwing thousands of requests at a server per second.

Web servers rarely link to everything public. `admin`, `backup`, and `config` folders are often hidden but accessible if you guess the name. (check robots txt for easy loot)

You use this by supplying a link and some wordlist, then declare where in the link the wordlist should be added.


### 🚩 Critical Flags

| Flag  | Description                                                                                     |
| :---- | :---------------------------------------------------------------------------------------------- |
| `-u`  | **U**RL. Use `FUZZ` as the placeholder for where the wordlist goes.                             |
| `-w`  | **W**ordlist path.                                                                              |
| `-mc` | **M**atch **C**ode. Default is 200,204,301,302,307,401,403.                                     |
| `-fs` | **F**ilter **S**ize. Hides responses of a specific file size (useful to hide custom 404 pages). |
| `-H`  | **H**eader. Useful for VHOST fuzzing or passing Cookies.                                        |
| `-t`  | **T**hreads. Default is 40. Bump to 100-200 for speed (if the server can handle it).            |
| -ac   | automatic filter (quite usefull)                                                                |
| -e    | append extension to end                                                                         |

###  Common Commands

**Directory Discovery (The Standard):**

```bash

ffuf -u [http://target.com](http://target.com) -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.target.com" -fs <size_of_default_response>


ffuf -u [http://target.com/indexFUZZ](http://target.com/indexFUZZ) -w /usr/share/wordlists/seclists/Discovery/Web-Content/web-extensions.txt


```


##  Hydra



A parallelized login cracker which supports numerous protocols to attack. It is fast, flexible, and the default "go-to" tool for brute-forcing in most CTFs and exams.



When you find a login page (SSH, FTP, Web Login) and you suspect weak passwords, Hydra automates the guessing process using wordlists.

### 🚩 Flags

| **Flag** | **Description**                                                                              |
| -------- | -------------------------------------------------------------------------------------------- |
| `-l`     | **L**ogin (Single user). Used when you know the username (e.g., `admin`).                    |
| `-L`     | **L**ist (User list). Used when you are guessing usernames.                                  |
| `-p`     | **P**assword (Single). Used if you want to test one password against many users.             |
| `-P`     | **P**assword List. Path to your wordlist (e.g., `/usr/share/wordlists/rockyou.txt`).         |
| `-s`     | **S**pecific port. Essential if SSH is running on port 2222 instead of 22.                   |
| `-t`     | **T**asks/Threads. Default is 16. Lower this (to 4) if the service crashes or locks you out. |
| `-V`     | **V**erbose. Shows username/password combinations as they are tried (good for debugging).    |

###  Common Commands

**SSH Brute Force:**

Bash

```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<target_ip>
```

**FTP Brute Force:**

Bash

```
hydra -l user -P /usr/share/wordlists/rockyou.txt ftp://<target_ip>
```

**Web Form Login (The Tricky One):**

_Requires identifying the parameter names in Burp Suite first._

Bash

```
hydra -l admin -P rockyou.txt <target_ip> http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Login Failed"
```

---

##  Medusa

**Bio:**

A speedy, parallel, and modular login brute-forcer. Similar to Hydra but aimed at being more stable and thread-safe.

**Why we use it:**

Some services (especially older SSH or FTP servers) crash when Hydra hits them too hard. Medusa is often more stable and handles "thread safety" better, preventing the target service from dying during the scan.

### 🚩 Critical Flags

|**Flag**|**Description**|
|---|---|
|`-h`|**H**ost (Target IP).|
|`-u`|**U**sername (Single).|
|`-U`|**U**ser List (Path to file).|
|`-p`|**P**assword (Single).|
|`-P`|**P**assword List (Path to file).|
|`-M`|**M**odule. The protocol to attack (e.g., `ssh`, `ftp`, `http`).|
|`-f`|**F**irst. Stop scanning after the first valid password is found (saves time).|

###  Common Commands

**SSH Brute Force:**

Bash

```
medusa -h <target_ip> -u root -P /usr/share/wordlists/rockyou.txt -M ssh -f
```

**Check Valid Credentials Across a Network:**

_If you found a password and want to see if it works on other machines._

Bash

```
medusa -H hosts_list.txt -u admin -p "Password123!" -M smbnt
```

---

## Difference between hydra and medusa

Tldr just use hydra

|**Feature**|**Hydra**|**Medusa**|
|---|---|---|
|**Syntax Style**|Protocol-focused (`ssh://target`)|Host-focused (`-h target -M ssh`)|
|**Speed**|Generally faster, but can be unstable.|Slower, but more stable/reliable.|
|**Web Forms**|**Superior.** Handles complex HTTP POST forms better.|Weak support for complex web forms.|
|**Stability**|Known to crash sensitive services if threads are too high.|Better thread management; safer for fragile servers.|
|**Verdict**|**Use Hydra first.** If the service crashes or kicks you out, switch to **Medusa**.||