This project has been created as part of the 42 curriculum by *varaniba*

# DESCRIPTION

This project consists of setting up a server by following specific rules. A minimum amount of services will be installed, therefore there will be no graphical interface.

## Debian vs Rocky Linux

## AppArmor vs SELinux

## UFW vs firewalld

## VirtualBox vs UTM

# SETUP

## Preparation:
We will need to prepare certain things beforehand:
- **Hypervisor:** [VirtualBox](https://www.virtualbox.org/)
- **Operating system:** [Debian 13 (Trixie)](https://www.debian.org/download)
- **External storage:** Not needed if the VM is stored in the sgoinfre directory.

## 1. Create the virtual machine:
Using VirtualBox, create a new virtual machine and configure the following resources:

| Resource | Configuration | Justification |
|---|---|---|
| Base Memory (RAM) | 2048 MB | Sufficient for Debian without a graphical interface, while limiting host memory usage. |
| Processors (vCPUs) | 1 | Provides enough processing capacity for the OS and additional services without unnecessarily allocating host resources. |
| Virtual Disk | 30 GB | Provides space for the operating system, partitions, logs, and additional services. |

## 2. Operating System installation:
Born2beRoot allows either Debian or Rocky Linux. For this project, Debian 13 (Trixie) was selected because it is suitable for beginners and has extensive documentation.

The installation must follow these requirements:
- **No graphical interface:** Install Debian without a desktop environment, as required by the subject.
- **Hostname:** varaniba42 — student login followed by 42.
- **Username:** varaniba — student login.
- **Disk partitioning:** Use manual partitioning to create the required partitions and mount points. Partition sizes must be chosen to provide sufficient space for the operating system and services while avoiding unnecessary disk allocation.

| Partition | Size | Justification |
|-----------|------|---------------|
| `/boot` | 1 GB | Stores the kernel and boot files, with room for updates. |
| `/` | 8 GB | Stores Debian, system utilities, and installed software. |
| `swap` | 2 GB | Matches the VM's RAM capacity and provides additional virtual memory. |
| `/home` | 2 GB | Sufficient for a small number of users and their configuration files. |
| `/var` | 6 GB | Accommodates MariaDB databases, WordPress files, and other variable data. |
| `/srv` | 2 GB | Reserves space for data used by additional services. |
| `/tmp` | 1 GB | Provides space for temporary files. |
| `/var/log` | 2 GB | Stores system and service logs separately to prevent them from filling `/var`. |
| **Total** | **24 GB** | Leaves approximately 6 GB for storage overhead and future expansion. |

The remaining usable space is left unallocated within the LVM volume group, allowing logical volumes to be extended later if necessary. These sizes are estimates based on the project's requirements.

## 3. Virtual Machine configuration:
Launch the virtual machine and log in with the user created during installation.
### 3.1 Sudo:
**sudo (superuser do)** allows authorized users to execute commands with root privileges without logging in as root.
- **Root:** Has full administrative privileges. Commands can be executed without sudo.
- **Regular user:** Has limited permissions and must use sudo for administrative tasks, provided the user has been granted permission.

**Install sudo:**
Switch to root user `su`, the `-` loads root's login environment:
```bash
su -
```

**Update the package list and install sudo:**
```bash
apt update
apt install sudo
```

**Verify the installation:**
Once the installation finishes, sudo is installed and available immediately. No reboot or logout is required, the installation can be verified:
```bash
sudo -V
```
**Difference between `apt` and `aptitude`:**

Both are package managers used to install, update, and remove software on Debian.

| `apt` | `aptitude` |
|---|---|
| Primarily command-line based | Command-line and interactive text interface |
| Simple package and dependency management | More advanced dependency resolution options |
| Suitable for everyday package management | Useful for managing more complex package conflicts |

For this project, `apt` is sufficient for installing packages.

### 3.2 Users:

Linux uses user accounts to manage access to files, programs, and system resources.

Born2beRoot requires a regular user with the student's login as username, in addition to the root account.

**Create a new user (as root):**
```bash
adduser <username>
```

The required user (`varaniba`) was already created during Debian installation, so it does not need to be created again.

**Verify that the user exists:**

```bash
id <username>
```

### 3.3 Groups:
Linux groups allow multiple users to share permissions without configuring each user individually.

Born2beRoot requires a regular user to belong to two groups:

- **sudo**:  Allows authorized users to execute commands with root privileges using `sudo`. Membership does not make the user root.

- **user42**: A custom group required by the project. It has no special privileges by default and demonstrates Linux group management.

**Create a new group (as root):**

```bash
addgroup <groupname>
```

**Verify that a group exists:**

```bash
getent group <groupname>
```

**Add a user to a group (as root):**

```bash
adduser <username> <groupname>
```
Group membership changes take effect after the user logs out and back in. A reboot would alos work, but it is not necessary.

**Verify the groups exist and list their members:**

```bash
getent group <groupname1> <groupname2>
```

**Check all groups assigned to a specific user:**

```bash
groups <username>
```
Both `sudo` and `user42` should appear in the output.

**Check all groups assigned to the current user:**

```bash
groups
```

## 4. SSH service
**SSH (Secure Shell)** is a network protocol that allows secure, encrypted remote access to another machine through a terminal.

For Born2beRoot, SSH allows us to access the Debian virtual machine from our host computer without using the VirtualBox console.

### 4.1 Configuring SSH

**Install the OpenSSH server:**

```bash
sudo apt install openssh-server
```
The `openssh-server` package allows the VM to accept incoming SSH connections.


**Check the SSH service status:**

```bash
sudo service ssh status
```
The output should indicate that the service is `active (running)`.


**Change SSH server:**

By default, SSH uses port `22`. Born2beRoot requires:

- SSH to run on port `4242`.
- Root login through SSH to be disabled.

Edit the SSH server configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```
Find and configure the following directives:

```text
Port 4242
PermitRootLogin no
```

- `Port 4242`: Changes the port on which the SSH server listens.
- `PermitRootLogin no`: Prevents the root account from connecting through SSH, regardless of the authentication method.

The modified lines must be uncommented.

**Validate and restart SSH:**

```bash
sudo sshd -t
sudo service ssh restart
sudo service ssh status
```

The first command checks the configuration for errors. No output indicates that the syntax validation succeeded.

**Verify the effective SSH configuration:**

```bash
sudo sshd -T | grep -E '^(port|permitrootlogin) '
```

Expected output:

```text
port 4242
permitrootlogin no
```


### 4.2 Connecting via SSH

To connect to the VM from the host computer, we need to configure **port forwarding** in VirtualBox.

With the VM using NAT networking, go to:

**VirtualBox → Settings → Network → Adapter → Advanced → Port Forwarding**

Create the following rule:

| Setting | Value |
|---|---|
| Name | SSH |
| Protocol | TCP |
| Host IP | Leave empty |
| Host Port | 4243 |
| Guest IP | Leave empty |
| Guest Port | 4242 |

**Understanding port forwarding:**

Port forwarding allows connections received on a specific port of the host machine to be redirected to a port inside the virtual machine.

- **Host:** Our physical computer, where VirtualBox runs.
- **Guest:** Our Debian virtual machine.
- **Host Port (`4243`):** The port used to initiate the SSH connection from the host.
- **Guest Port (`4242`):** The port where the SSH server listens inside the VM.

**Host IP and Guest IP:**

Both fields are optional in the standard VirtualBox NAT configuration.

| Setting | Empty | Specified |
|---|---|---|
| Host IP | Accepts connections on all available host network interfaces. | Restricts connections to the specified host IP address. |
| Guest IP | VirtualBox uses the VM's guest address automatically. | Forwards connections to the specified guest IP address. |

For example, setting the Host IP to `127.0.0.1` restricts incoming connections to the host's loopback interface, meaning only the host computer can connect through this forwarding rule.

Leaving the Host IP empty can allow other devices on the network to access the forwarded port, depending on the host's firewall and network configuration.

For this project, both IP fields were left empty. This works because VirtualBox handles the forwarding automatically. However, setting the Host IP to `127.0.0.1` would provide a more restrictive configuration.

**Connect to the virtual machine:**

Open a terminal on the host computer and run:

```bash
ssh <username>@localhost -p <hostport>
```
For this project:

```bash
ssh varaniba@localhost -p 4243
```
Breaking down the command:

- `ssh`: Starts an SSH connection.
- `varaniba`: The user account on the VM.
- `localhost`: Refers to our host computer, where the forwarding rule is configured.
- `-p 4243`: Specifies the host port used for the connection.

VirtualBox forwards the connection from the host's port `4243` to the VM's port `4242`.
On the first connection, SSH may ask us to verify the server's host key. Once accepted, enter the Debian user's password.
After authentication, the terminal prompt should look similar to:

```text
varaniba@varaniba42:~$
```

This confirms that we are connected to the virtual machine through SSH.

To close the VM from the terminal on the host computer:

```bash
sudo shutdown now
```

**Note:** Although the terminal runs on the host computer, commands entered in this SSH session execute inside the VM. To be able to access the VM through the terminal, the VM needs to be running and unlocked. The moment the VM is shut down, the connection to the terminal will be close.

## 5. UFW firewall

**UFW (Uncomplicated Firewall)** is a tool used to manage firewall rules on Linux. A firewall controls incoming and outgoing network traffic based on predefined rules.

Born2beRoot requires UFW to be enabled and configured to allow incoming connections through port `4242`, which is used by the SSH service.

### 5.1. Installing UFW

**Install the UFW package:**

```bash
sudo apt install ufw
```

**Check the firewall status:**

```bash
sudo ufw status
```

Initially, the output will generally show `Status: inactive`, meaning the firewall is not enabled.


### 5.2. Configuring UFW

**Allow connections through port 4242:**

```bash
sudo ufw allow 4242
```

This creates a firewall rule allowing incoming connections through port `4242`, which is used by our SSH server.

**Enable the firewall:**

```bash
sudo ufw enable
```

It is recommended to allow port `4242` **before enabling UFW**, especially when configuring the VM through SSH. Otherwise, enabling the firewall could block our SSH connection.

### 5.3. Verifying UFW Configuration

**Check the firewall status and rules:**

```bash
sudo ufw status verbose
```

The output should indicate that the firewall is active and port `4242` is allowed.


## 6. Sudo Security Configuration

Born2beRoot requires additional security settings for `sudo`, including password attempt limits, custom error messages, command logging, TTY enforcement, and restricted executable paths.

**Sudoers:**

`sudoers` is the configuration system that defines which users can execute commands with elevated privileges and under what conditions. Its main configuration file is `/etc/sudoers`.

The `/etc/sudoers.d/` directory allows us to create additional configuration files that are read by `sudo`, provided the main configuration includes this directory.

Using a separate file is preferable because:
- It keeps our custom settings separate from the system's default configuration.
- It makes configuration easier to maintain and troubleshoot.
- It reduces the need to modify the main configuration file.

For Born2beRoot, instead of modifying `/etc/sudoers` directly, we will create `/etc/sudoers.d/born2beroot`.

### 6.1. Configuring Sudo

**Create a directory for sudo logs (as root):**

```bash
mkdir -p /var/log/sudo
```

**Create a custom sudo configuration file:**

```bash
visudo -f /etc/sudoers.d/born2beroot
```

Using `visudo` instead of directly editing the file checks the configuration for syntax errors before saving.

Add the following configuration:

```text
Defaults passwd_tries=3
Defaults badpass_message="Incorrect password. Please try again."
Defaults logfile="/var/log/sudo/born2beroot.log"
Defaults log_input,log_output
Defaults iolog_dir="/var/log/sudo"
Defaults use_pty
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
When saving, `visudo` creates a temporary file (`born2beroot.tmp`) where our changes are stored. After exiting the editor, it checks the file for syntax errors. If the configuration is valid, the temporary file is renamed to `born2beroot`. If errors are found, `visudo` offers the option to correct them before saving.

**Understanding the configuration:**

| Setting | Description |
|---|---|
| `passwd_tries` | Sets how many times a user can enter an incorrect password when using `sudo`. |
| `badpass_message` | Shows a custom message when the user enters an incorrect password. |
| `logfile` | Defines where to save a record of sudo commands, including who used them and when. |
| `log_input,log_output` | Records what users type and what appears in the terminal when running commands with `sudo`. |
| `iolog_dir` | Defines the folder where these input and output recordings are saved. |
| `use_pty` | Runs sudo commands in a separate virtual terminal to improve security. |
| `secure_path` | Defines which directories sudo looks in to find commands to execute. |

**Terminal security:**

A **TTY** is a terminal that allows us to interact with Linux by typing commands. A **PTY (pseudo-terminal)** is a virtual terminal created by software, like the one we normally use when connecting to our VM through SSH.

The subject requires TTY mode to be enabled but does not specify which sudo setting to use:

- **`requiretty`:** Only allows sudo commands to run when the user has a terminal. For example, `sudo apt update` works when typed in our terminal but may be blocked when executed by an automated script without a terminal.

- **`use_pty`:** Runs sudo commands in a separate virtual terminal. For example, when we execute `sudo apt update`, sudo creates a virtual terminal for `apt` and manages communication with our original terminal. We still see everything in the same window.

Normally, a program running with root privileges can interact directly with our terminal. If the program is malicious, it could attempt to insert commands for our shell to execute.

We chose `use_pty` because running privileged commands in a separate terminal helps prevent this interference, while still allowing automated tasks to use `sudo` without an interactive terminal.

### 6.2. Verifying the Configuration

After configuring `sudo`, we need to verify that the security settings work as expected.

**Check the configuration for syntax errors (as root):**

```bash
visudo -c
```

This checks that our sudo configuration files contain no syntax errors. A successful result indicates that the files were parsed correctly.

**Verify sudo authentication and administrator privileges:**

Return to the regular user:

```bash
exit
```

First, clear any previous sudo authentication:

```bash
sudo -k
```

Normally, `sudo` remembers our authentication for a short period, so we don't need to enter the password for every command. The `-k` option clears this authentication, forcing sudo to request our password again.

Now compare:

```bash
whoami
```

Expected output:

```text
varaniba
```

This shows which user is currently executing commands.

Then:

```bash
sudo whoami
```

Expected output:

```text
root
```

This confirms that our regular user (`varaniba`) can execute commands with root privileges using `sudo`, without permanently switching to the root account.

**Verify the password attempt limit and custom error message:**

```bash
sudo -k
sudo whoami
```

Enter an incorrect password three times.

We should see our custom error message, and sudo should stop requesting passwords after three failed attempts.

**Verify that sudo logs are being created:**

```bash
sudo ls -la /var/log/sudo
```

Breaking down the command:

- `sudo`: Executes the command with root privileges.
- `ls`: Lists files and directories.
- `-l`: Shows details such as permissions, owner, size, and modification date.
- `-a`: Includes hidden files and directory entries.

We should see:

- `born2beroot.log`: Records sudo command events, such as which commands were executed and by whom.
- `00/`: Contains directories with detailed recordings of sudo sessions.
- `seq`: Used by sudo to generate unique session identifiers.

This confirms that sudo has created the log files and directories.

**Verify sudo command logging:**

```bash
sudo cat /var/log/sudo/born2beroot.log
```
This displays a history of commands executed using `sudo`, including who executed them and when.

**Verify sudo input/output logging:**

```bash
sudo sudoreplay -d /var/log/sudo -l
```

This will display a list of the recorded sessions, confirming that sudo is saving input/output recordings, not just recording which commands were executed. Breaking down the command:

- `sudo`: Executes the command with root privileges.
- `sudoreplay`: Allows us to inspect recorded sudo sessions.
- `-d /var/log/sudo`: Specifies where the recordings are stored.
- `-l`: Lists the available recorded sessions.

We can also replay a recorded session using its session ID. Find a session ID (`TSID`) and replay its recording:

```bash
sudo sudoreplay -d /var/log/sudo <session_id>
```

This allows us to verify that sudo records what appears in the terminal during a command, not just which command was executed.

**Verify the active sudo settings:**

```bash
sudo -l
```
This displays the commands our user is allowed to execute with sudo and the sudo defaults that apply to the user.


## 7. Password Policy

Born2beRoot requires a strong password policy to protect user accounts. This includes password expiration rules and password complexity requirements.

### 7.1. Password Expiration

`/etc/login.defs` is a system configuration file that defines default settings for user accounts, including password expiration rules.

**Configure password expiration (as root):**

```bash
su -
nano /etc/login.defs
```

Find and modify the following settings:

```text
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

**Understanding the configuration:**

| Setting | Description |
|---|---|
| `PASS_MAX_DAYS` | A password expires after 30 days and must be changed. |
| `PASS_MIN_DAYS` | Users must wait at least 2 days before changing their password again. |
| `PASS_WARN_AGE` | Users receive a warning during login starting 7 days before their password expires. |

These settings are applied by default when new accounts are created. Existing accounts must be updated separately using `chage`.

**Update existing users:**

Before updating the settings, check their current values:

```bash
chage -l varaniba
chage -l root
```

The `chage -l` command displays the password expiration settings currently applied to an account.

Apply the new rules to both accounts:

```bash
chage -M 30 -m 2 -W 7 varaniba
chage -M 30 -m 2 -W 7 root
```

- `-M 30`: Maximum password age of 30 days.
- `-m 2`: Minimum password age of 2 days.
- `-W 7`: Warning period of 7 days.

### 7.2. Password Complexity

Password complexity defines the requirements a new password must satisfy.

For this, we use **PAM (Pluggable Authentication Modules)**, a system that manages authentication in Linux. We will use the `libpam-pwquality` package to check whether new passwords meet our security requirements.

**Install the package (as root):**

```bash
apt install libpam-pwquality
```

**Configure password complexity:**

`/etc/security/pwquality.conf` is a system configuration file that defines password complexity requirements, including minimum length, required character types, and restrictions on repeated characters.

```bash
nano /etc/security/pwquality.conf
```

Add or modify the following settings:

```text
minlen = 10
ucredit = -1
lcredit = -1
dcredit = -1
maxrepeat = 3
usercheck = 1
difok = 7
enforce_for_root
```

**Understanding the configuration:**

| Setting | Description |
|---|---|
| `minlen = 10` | Requires passwords to contain at least 10 characters. |
| `ucredit = -1` | Requires at least one uppercase letter. |
| `lcredit = -1` | Requires at least one lowercase letter. |
| `dcredit = -1` | Requires at least one number. |
| `maxrepeat = 3` | Prevents a character from appearing more than 3 times consecutively. |
| `usercheck = 1` | Prevents passwords from containing the username. |
| `difok = 7` | Requires at least 7 characters in the new password that are not present in the previous password. |
| `enforce_for_root` | Enforces password quality rules even when root changes a password. |

**Password policy for root:**

The root password must follow the same complexity requirements, except for the comparison with the previous password (`difok`).

When root changes a password, the previous password is not requested. Therefore, checks that compare the old and new passwords cannot be performed.

### 7.3. Updating Existing Passwords

After configuring the password policy, Born2beRoot requires all existing account passwords, including root's, to be changed.

This ensures that the new passwords follow the configured requirements, since modifying the configuration files does not automatically update existing passwords.

To change the password of the current user:

```bash
passwd
```

**Password expiration restrictions:**

The minimum password age of 2 days prevents regular users from changing a recently updated password.

If the password change is blocked, root can reset the user's password using:

```bash
passwd varaniba
```

However, when root resets another user's password, the previous password is not requested, so the `difok = 7` comparison cannot be performed.

All existing accounts with passwords must be updated before completing the configuration.

### 7.4. Verifying the Password Policy

Return to the regular user:

```bash
exit
```

**Verify password expiration settings:**

```bash
sudo chage -l varaniba
sudo chage -l root
```

The output should include:

```text
Minimum number of days between password change : 2
Maximum number of days between password change : 30
Number of days of warning before password expires : 7
```

This confirms that the password expiration rules have been applied to both accounts.

**Verify password complexity settings:**

```bash
grep -E '^[[:space:]]*(minlen|ucredit|lcredit|dcredit|maxrepeat|usercheck|difok|enforce_for_root)' /etc/security/pwquality.conf
```

This displays the password complexity rules defined in the configuration file.

**Verify that PAM uses the password quality module:**

```bash
grep -E '^[[:space:]]*password.*pam_pwquality\.so' /etc/pam.d/common-password
```

An active `pam_pwquality.so` entry confirms that the module is included in PAM's password-changing configuration.

If no active entry appears, check the PAM configuration:

```bash
sudo pam-auth-update
```

Make sure the password-quality profile is enabled.

**Test password complexity enforcement:**

When the account is eligible for a password change, run:

```bash
passwd
```

Try entering a password that does not meet the requirements, such as one that is too short or contains no uppercase letters.

The system should reject the password and display an error explaining why it does not meet the policy.

The minimum password age of 2 days may prevent this test immediately after updating a password.



# REQUIREMENTS

# INSTRUCTIONS

# RESOURCES

## References
To create this project, the references and tools used were:

- Linux man page [manual](https://linux.die.net/man/)
- Stack Overflow [website](https://stackoverflow.com/questions)
- Github [website](https://github.com/)
- Born2BeRoot guide [website](https://noreply.gitbook.io/born2beroot)
- Debian [Recommended Partitioning Scheme](https://www.debian.org/releases/trixie/amd64/apcs03.en.html)
- Debian [Directory Tree](https://www.debian.org/releases/trixie/amd64/apcs02.en.html)
- Debian [Disk Space Needed for Tasks](https://www.debian.org/releases/trixie/amd64/apds02.en.html)
- Debian [Package Management](https://www.debian.org/doc/manuals/debian-reference/ch02)
- Debian Wiki[sudo](https://wiki.debian.org/sudo)
- Debian [sudoers Configuration Manual](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html)
- Debian [visudo Manual](https://manpages.debian.org/trixie/sudo/visudo.8.en.html)
- Debian Wiki[System Groups](https://wiki.debian.org/SystemGroups)
- Debian Wiki [SSH](https://wiki.debian.org/SSH)
- Debian [SSH Server Configuration Manual](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html)
- Debian [SSH Client Manual](https://manpages.debian.org/trixie/openssh-client/ssh.1.en.html)
- VirtualBox [User Manual](https://www.virtualbox.org/manual/)
- VirtualBox [NAT Networking and Port Forwarding](https://www.virtualbox.org/manual/topics/networkingdetails.html#nat)
- Debian [Password Aging Configuration](https://manpages.debian.org/trixie/login.defs/login.defs.5.en.html)
- Debian [Password Quality Configuration](https://manpages.debian.org/trixie/libpwquality-common/pwquality.conf.5.en.html)
- Debian [PAM Password Quality Module](https://manpages.debian.org/trixie/libpam-pwquality/pam_pwquality.8.en.html)
- Debian [PAM Configuration](https://manpages.debian.org/trixie/libpam-runtime/pam-auth-update.8.en.html)



## AI Use:

- ChatGPT and Codex:
  - Understanding programming concepts;
  - Debugging and reasoning about implementation issues;
  - Improving README wording, grammar, and clarity.

All the code, the implementation choices and final project structure were created by the author.

