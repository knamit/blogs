---
author: "Namit Khurana"
title: "Bandit Part 4"
date: "2024-01-11"
description: "Contains posts related to my learning Bandit journey. Blog posts will contain the walkthrough of the levels and the commands used to solve the levels."
tags: ["bandit", "overthewire", "linux", "security"]
categories: ["Learning Bandit"]
series: ["Learning Bandit"]
cover:
  image: images/cover4.png
---

# Blog 4: Getting Started with Bandit
## Levels Overview

- **Levels Covered:** 16–20
- **Theme:**  Nmap scanning, file comparison, SSH key authentication, and using setuid binaries.
- **Skills Learned:**
   - Scanning for open ports using Nmap
   - Comparing files using `diff`
   - Using SSH with pseudo-terminal
   - Utilizing setuid binaries for file access
   - Setting up listeners with Netcat

---
## Level-by-Level Walkthrough

### Level 16: Nmap

- **Objective:** Find the password hidden in a file.
- **Challenge:** The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.
- **Commands/Tools Introduced:** `nmap`

#### Solution:

1. **Connecting to the Server:**
   Connect to the Bandit Level 16 server using the password obtained from the previous level.

2. **Scanning for Open Ports:**
   We use `nmap` to scan the ports in the range 31000 to 32000 on localhost.
   ```bash
   nmap -sV -T4 localhost -p 31000-32000
   ```
   Here, `-sV` is used to determine service version, and `-T4` is the timing template for the scan.

3. **Identifying the Correct Port:**
   We find that port 31790 is running an SSL service.

4. **Submitting the Password:**
   We submit the password for Bandit Level 16 to port 31790 on localhost using `openssl`.
   We get a private key that can be used to log in to the next level.

[Level 16 Screenshot](images/Level%2017.txt)
---

### Level 17: Diff

- **Objective:** Find the password hidden in a file.
- **Challenge Description:** The password for the next level can be retrieved by comparing two files in the directory. The password is the only line that is different between the two files.
- **Commands/Tools Introduced:** `diff`

#### Solution:

1. **Connecting to the Server:**
   Connect to the Bandit Level 17 server using the password obtained from the previous level.

2. **Comparing Files:**
   We compare two files in the directory using `diff`.
   ```bash
   diff passwords.old passwords.new
   ```
   This reveals the password for the next level.

[Level 17 Screenshot](images/level%2018.txt)

---

### Level 18: SSH Key Authentication

- **Objective:** Find the password hidden in a file.
- **Challenge Description:** The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.
- **Commands/Tools Introduced:** `ssh`

#### Solution:

1. **Connecting to the Server:**
   Connect to the Bandit Level 18 server using the password obtained from the previous level.

2. **Reading the Password:**
   The issue here is that when we log in, we are immediately logged out due to a modification in the `.bashrc` file. To overcome this, we can specify a pseudo-terminal to run a shell instead of executing the `.bashrc` file.
   ```bash
   ssh bandit18@bandit.labs.overthewire.org -p 2220 -t /bin/sh 
   ```
   This logs us in and we can read the password for the next level.

[Level 18 Screenshot](images/Level%2019.txt)

---

### Level 19: Setuid Binary

- **Objective:** Find the password hidden in a file.
- **Challenge Description:** To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.
- **Commands/Tools Introduced:** `setuid`

#### Solution:

1. **Connecting to the Server:**
   Connect to the Bandit Level 19 server using the password obtained from the previous level.

2. **Using the Setuid Binary:**
   We have a setuid binary in the home directory. Running it without arguments gives us a hint on how to use it.
   ```bash
   bandit19@bandit:~$ ./bandit20-do 
   Run a command as another user.
   ```
   We can use this binary to read the password for the next level.

3. **Reading the Password:**
   We use the setuid binary to read the password for the next level.
   ```bash
   ./bandit20-do cat /etc/bandit_pass/bandit20
   ```
   This reveals the password for Bandit Level 20.

[Level 19 Screenshot](images/Level%2020.txt)

---

### Level 20: Setuid Binary with Netcat

- **Objective:** Find the password hidden in a file.
- **Challenge Description:** There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).
- **Commands/Tools Introduced:** `setuid`, `nc`, `tmux`

#### Solution:

1. **Connecting to the Server:**
   Connect to the Bandit Level 20 server using the password obtained from the previous level.

2. **Using Tmux:**
   Instead of running 2 SSH sessions, we can use `tmux` to split the terminal into two panes.

3. **Setting Up the Listener:**
   In one pane, we set up a listener on a port using `nc`.
   ```bash
   nc -l -p 1234
   ```

4. **Running the Setuid Binary:**
   In the other pane, we run the setuid binary with the port number as an argument.
   ```bash
   ./suconnect 1234
   ```

5. **Sending the Password:**
   We send the password for Bandit Level 20 to the listener using `nc`.
   ```bash
   nc -l -p 1234
   ```
   This reveals the password for Bandit Level 21.

[Level 20 Screenshot](images/Level%2021.txt)

---


## Key Takeaways

- **Nmap (Level 16):**
  - Use `nmap -sV -T4 localhost -p 31000-32000` to scan for open ports and determine which ones run SSL services.
  - Identify the correct port that will provide the next credentials.

- **Diff (Level 17):**
  - Use `diff file1 file2` to compare two files and identify the differences, revealing the password.

- **SSH Key Authentication (Level 18):**
  - When logging in with SSH fails due to `.bashrc` issues, use `ssh -t user@host -p port /bin/sh` to bypass the logout.

- **Setuid Binary (Level 19):**
  - Execute a setuid binary without arguments to learn how to use it.
  - Use the binary to access protected files, such as passwords stored in `/etc/bandit_pass`.

- **Setuid Binary with Netcat (Level 20):**
  - Utilize `tmux` to manage multiple terminal sessions efficiently.
  - Set up a listener with `nc -l -p port` to receive data and connect it with a setuid binary that checks the password.

## Cheatsheet

```bash
# Level 16: Nmap Scan
nmap -sV -T4 localhost -p 31000-32000

# Level 17: Comparing Files
diff passwords.old passwords.new

# Level 18: SSH with Pseudo-Terminal
ssh bandit18@bandit.labs.overthewire.org -p 2220 -t /bin/sh 

# Level 19: Using Setuid Binary
./bandit20-do  # Run without arguments to see usage
./bandit20-do cat /etc/bandit_pass/bandit20  # To read the password

# Level 20: Setuid with Netcat and Tmux
# Start Tmux session and split terminal
nc -l -p 1234  # In one pane, set up a listener
./suconnect 1234  # In the other pane, run the setuid binary
# Send the current password to the listener
```