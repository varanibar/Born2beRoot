This project has been created as part of the 42 curriculum by *varaniba*

## Table of contents

- [DESCRIPTION](#description)
  - [Debian vs Rocky Linux](#debian-vs-rocky-linux)
  - [AppArmor vs SELinux](#apparmor-vs-selinux)
  - [UFW vs firewalld](#ufw-vs-firewalld)
  - [VirtualBox vs UTM](#virtualbox-vs-utm)
- [INSTRUCTIONS](#instructions)
  - [Command reference](#command-reference)
  - [Setup](#setup)
    - [Preparation](#preparation)
    - [1. Create the virtual machine](#1-create-the-virtual-machine)
    - [2. Operating System installation](#2-operating-system-installation)
    - [3. Virtual Machine configuration](#3-virtual-machine-configuration)
      - [3.1 Sudo](#31-sudo)
      - [3.2 Users](#32-users)
      - [3.3 Groups](#33-groups)
    - [4. SSH service](#4-ssh-service)
      - [4.1 Configuring SSH](#41-configuring-ssh)
      - [4.2 Connecting via SSH](#42-connecting-via-ssh)
    - [5. UFW firewall](#5-ufw-firewall)
      - [5.1. Installing UFW](#51-installing-ufw)
      - [5.2. Configuring UFW](#52-configuring-ufw)
      - [5.3. Verifying UFW Configuration](#53-verifying-ufw-configuration)
    - [6. Sudo Security Configuration](#6-sudo-security-configuration)
      - [6.1. Configuring Sudo](#61-configuring-sudo)
      - [6.2. Verifying the Configuration](#62-verifying-the-configuration)
    - [7. Password Policy](#7-password-policy)
      - [7.1. Password Expiration](#71-password-expiration)
      - [7.2. Password Complexity](#72-password-complexity)
      - [7.3. Updating Existing Passwords](#73-updating-existing-passwords)
      - [7.4. Verifying the Password Policy](#74-verifying-the-password-policy)
    - [8. Monitoring Script](#8-monitoring-script)
      - [8.1. Creating the Script](#81-creating-the-script)
      - [8.2. Scheduling with Cron](#82-scheduling-with-cron)
      - [8.3. Verifying the Configuration](#83-verifying-the-configuration)
      - [8.4. Disabling the Monitoring Script](#84-disabling-the-monitoring-script)
- [RESOURCES](#resources)
  - [References](#references)
  - [AI Use](#ai-use)

# DESCRIPTION

This project consists of setting up a server by following specific rules. A minimum amount of services will be installed, therefore there will be no graphical interface.

## Debian vs Rocky Linux

Both are community-maintained Linux distributions suitable for running a server without a graphical interface. Their main differences for this project are the package management tools and security configuration.

| Debian | Rocky Linux |
|---|---|
| Independent distribution with a broad range of desktop and server software. | Enterprise distribution designed for compatibility with Red Hat Enterprise Linux (RHEL). |
| Uses `.deb` packages, managed with tools such as `apt` and `dpkg`. | Uses RPM packages, managed with tools such as `dnf` and `rpm`. |
| Uses AppArmor by default. | Commonly uses SELinux for mandatory access control. |

Debian 13 was selected for this project because its documentation and package management tools provide an accessible starting point for learning system administration. Rocky Linux provides an alternative for learning the RHEL ecosystem. Sources: [Debian](https://www.debian.org/intro/why_debian.html), [Rocky Linux](https://rockylinux.org/), and [DNF documentation](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/).

## AppArmor vs SELinux

Both provide **mandatory access control (MAC)**: security policies restrict what a process can access in addition to standard Linux ownership and permission checks. Their purpose is to limit the damage a compromised application can cause.

| AppArmor | SELinux |
|---|---|
| Uses application profiles, with file access rules primarily based on paths. | Uses security labels assigned to processes and resources, with policies defining permitted interactions. |
| Profiles describe permitted access to files and other resources for a confined application. | Policies control access according to security contexts, including process and file types. |
| Enforce mode blocks policy violations; complain mode primarily logs them. | Enforcing mode blocks policy violations; permissive mode logs them without blocking them. |

This project uses AppArmor, which is enabled by default on Debian. Protection depends on the profiles loaded and their modes; enabling AppArmor does not mean every application is confined. Sources: [AppArmor documentation](https://apparmor.net/), [AppArmor on Debian](https://wiki.debian.org/AppArmor/HowToUse), and [Rocky Linux SELinux documentation](https://docs.rockylinux.org/guides/security/learning_selinux/).

## UFW vs firewalld

Both are tools for managing the Linux kernel's firewall rules. They control network traffic, while AppArmor and SELinux control application access to system resources.

| UFW | firewalld |
|---|---|
| Provides a simple interface for allowing or denying traffic by port, protocol, or address. | Organizes rules using zones that represent different levels of network trust. |
| Managed with the `ufw` command. | Managed with tools such as `firewall-cmd`. |
| Suits a small set of host firewall rules. | Supports zone-based configuration and separates runtime changes from permanent settings. |

This project uses UFW to configure the required SSH access on port `4242`. Its command syntax suits the small number of rules needed for this VM. Sources: [Ubuntu firewall documentation](https://ubuntu.com/server/docs/firewalls/) and [firewalld documentation](https://firewalld.org/documentation/index.html).

## VirtualBox vs UTM

Both can host a virtual machine, allowing Debian to run with its own virtual disk, memory allocation, processors, and network configuration.

| VirtualBox | UTM |
|---|---|
| Provides desktop virtualization on supported Windows, Linux, and macOS hosts. | Provides virtualization and emulation on Apple platforms, including macOS. |
| Includes VM resource settings and NAT port forwarding used in this setup. | Supports QEMU-based virtualization and emulation, with an Apple Virtualization backend also available on macOS. |
| Host and guest architecture support depends on the VirtualBox version and platform. | Can virtualize a matching CPU architecture or emulate a different one through QEMU. |

Virtualization runs a compatible guest using hardware acceleration. Emulation can reproduce another CPU architecture but generally has greater overhead. The guest ISO must therefore match the chosen platform and execution mode.

This project uses VirtualBox, following the referenced installation guide. UTM is an alternative on macOS, particularly when working with Apple Silicon and ARM guest images. Sources: [VirtualBox documentation](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/Introduction.html), [UTM documentation](https://docs.getutm.app/), and [UTM architecture settings](https://docs.getutm.app/settings-qemu/system/).



# INSTRUCTIONS

## Command reference

A list of commands grouped by setup section, with detailed explanations in the sections below.

Commands marked with `sudo` are run as the regular VM user. Administrative commands without `sudo` are run in a root shell, as indicated in the linked sections. Values in angle brackets are placeholders.

**[3. Virtual Machine configuration](#3-virtual-machine-configuration)**

- `su -` → Switch to root and load root's login environment.
- `exit` → Leave the current shell; return from root to the previous user, or log out of a login session.
- `apt update` → Update the available package lists.
- `apt install sudo` → Install sudo.
- `sudo -V` → Display the sudo version and configuration information.
- `adduser <username>` → Create a user account.
- `id <username>` → Display a user's ID and group memberships.
- `addgroup <groupname>` → Create a group.
- `getent group <groupname>` → Display a group's entry and listed members.
- `adduser <username> <groupname>` → Add a user to a group.
- `getent group <groupname1> <groupname2>` → Display entries for multiple groups.
- `groups <username>` → Display the groups assigned to a specified user.
- `groups` → Display the current user's active groups.

**[4. SSH service](#4-ssh-service)**

- `sudo apt install openssh-server` → Install the SSH server.
- `sudo service ssh status` → Check the SSH service status.
- `sudo nano /etc/ssh/sshd_config` → Edit the SSH server configuration.
- `sudo sshd -t` → Check the SSH configuration for errors.
- `sudo service ssh restart` → Restart the SSH service to apply changes.
- `sudo sshd -T | grep -E '^(port|permitrootlogin) '` → Display the effective SSH port and root-login settings.
- `ssh <username>@localhost -p <hostport>` → Connect from the host computer through the forwarded SSH port; this project uses `ssh varaniba@localhost -p 4243`.
- `sudo shutdown now` → Shut down the VM when run inside its console or SSH session.

**[5. UFW firewall](#5-ufw-firewall)**

- `sudo apt install ufw` → Install UFW.
- `sudo ufw status` → Display the firewall status and active rules.
- `sudo ufw allow 4242` → Allow incoming traffic on port 4242.
- `sudo ufw enable` → Enable the firewall.

**[6. Sudo Security Configuration](#6-sudo-security-configuration)**

- `mkdir -p /var/log/sudo` → Create the sudo log directory if needed.
- `visudo -f /etc/sudoers.d/born2beroot` → Edit the project's sudo configuration with syntax checking.
- `visudo -c` → Check the sudoers configuration for syntax errors.
- `sudo -k` → Invalidate cached sudo authentication so the next password-protected command requests authentication again.
- `whoami` → Display the current effective username.
- `sudo whoami` → Verify that sudo executes the command as root.
- `sudo ls -la /var/log/sudo` → List sudo log files and recording directories, including hidden entries and file details.
- `sudo cat /var/log/sudo/born2beroot.log` → Display the sudo command log.
- `sudo sudoreplay -d /var/log/sudo -l` → List recorded sudo sessions.
- `sudo sudoreplay -d /var/log/sudo <session_id>` → Replay a recorded sudo session.
- `sudo -l` → Display the user's permitted sudo commands and applicable defaults.

**[7. Password Policy](#7-password-policy)**

- `nano /etc/login.defs` → Edit default password aging settings for new accounts.
- `chage -l <username>` → Display an account's password aging settings; use `sudo` when checking another account as the regular user.
- `chage -M 30 -m 2 -W 7 <username>` → Set the maximum password age to 30 days, minimum age to 2 days, and warning period to 7 days.
- `apt install libpam-pwquality` → Install the PAM password-quality module.
- `nano /etc/security/pwquality.conf` → Edit password complexity requirements.
- `passwd` → Change the current user's password.
- `passwd varaniba` → Reset `varaniba`'s password as root.
- `grep -E '^[[:space:]]*(minlen|ucredit|lcredit|dcredit|maxrepeat|usercheck|difok|enforce_for_root)' /etc/security/pwquality.conf` → Display the configured password-quality settings.
- `grep -E '^[[:space:]]*password.*pam_pwquality\.so' /etc/pam.d/common-password` → Check for an active PAM password-quality entry.
- `sudo pam-auth-update` → Open the PAM profile configuration tool.

**[8. Monitoring Script](#8-monitoring-script)**

- `nano /usr/local/bin/monitoring.sh` → Create or edit the monitoring script.
- `chmod +x /usr/local/bin/monitoring.sh` → Make the script executable.
- `/usr/local/bin/monitoring.sh` → Run the script manually.
- `apt install cron` → Install cron.
- `systemctl enable --now cron` → Enable cron at startup and start it immediately.
- `crontab -e` → Edit the current user's crontab; run as root to manage the monitoring schedule.
- `crontab -l` → Display the current user's crontab.
- `systemctl status cron` → Check the cron service status.

**System information used by the monitoring script**

- `uname -a` → Display kernel and system information.
- `lscpu` → Display CPU information, including processor sockets visible to the VM.
- `nproc --all` → Display the number of installed logical processors.
- `free -m` → Display memory and swap usage in MiB.
- `df` → Display filesystem space usage.
- `/proc/stat` → Source of CPU counters used to calculate usage; this is a file, not a command.
- `uptime -s` → Display the system startup time.
- `lsblk` → Display block devices, including LVM logical volumes.
- `ss` → Display socket information used to count established TCP connections.
- `who` → List logged-in terminal sessions.
- `ip` → Display network information, including IP and MAC addresses.
- `grep` → Match log entries used to count sudo commands.
- `wall` → Broadcast the monitoring output to logged-in terminals.

## Setup

The initial steps, [1. Create the virtual machine](#1-create-the-virtual-machine) and [2. Operating System installation](#2-operating-system-installation), follow the [Born2beRoot guide](https://noreply.gitbook.io/born2beroot). Refer to that guide for detailed installation and partitioning steps. Starting with [3. Virtual Machine configuration](#3-virtual-machine-configuration), the sections document the configuration used for this project, explain the choices made, and describe the commands used to configure and verify the system.

### Preparation:

- **Hypervisor:** [VirtualBox](https://www.virtualbox.org/)
- **Operating system:** [Debian 13 (Trixie)](https://www.debian.org/download)
- **External storage:** Not needed if the VM is stored in the sgoinfre directory.

### 1. Create the virtual machine:
Using VirtualBox, create a new virtual machine and configure the following resources:

| Resource | Configuration | Justification |
|---|---|---|
| Base Memory (RAM) | 2048 MB | Sufficient for Debian without a graphical interface, while limiting host memory usage. |
| Processors (vCPUs) | 1 | Provides enough processing capacity for the OS and additional services without unnecessarily allocating host resources. |
| Virtual Disk | 30 GB | Provides space for the operating system, partitions, logs, and additional services. |

### 2. Operating System installation:
Born2beRoot allows either Debian or Rocky Linux. For this project, Debian 13 (Trixie) was selected because it is suitable for beginners and has extensive documentation.

The installation must follow these requirements:
- **No graphical interface:** Install Debian without a desktop environment, as required by the subject.
- **Hostname:** varaniba42 — student login followed by 42.
- **Username:** varaniba — student login.
- **Disk partitioning:** Use manual partitioning to create the required partitions and mount points. Partition sizes must be chosen to provide sufficient space for the operating system and services while avoiding unnecessary disk allocation.

The following table lists the partition sizes chosen for this VM and explains their purpose. For the steps to configure encrypted storage and LVM, refer to the guide linked above.

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

### 3. Virtual Machine configuration:
Launch the virtual machine and log in with the user created during installation.

#### 3.1 Sudo:

**sudo (superuser do)** allows authorized users to execute commands with root privileges without logging in as root.
- **Root:** Has full administrative privileges. Commands can be executed without sudo.
- **Regular user:** Has limited permissions and must use sudo for administrative tasks, provided the user has been granted permission.

**Install sudo:**

Switch to root user `su`, the `-` loads root's login environment:
```bash
su -
```

Remain in this root shell for the rest of section 3 to install sudo and manage users and groups.

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

#### 3.2 Users:

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

#### 3.3 Groups:
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
Group membership changes take effect after the user logs out and back in (use `exit` to log out). A reboot would also work, but it is not necessary.

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

### 4. SSH service
**SSH (Secure Shell)** is a network protocol that allows secure, encrypted remote access to another machine through a terminal.
For Born2beRoot, SSH provides access to the Debian virtual machine from the host computer without using the VirtualBox console.

This section is performed as the regular user, using `sudo` to give individual administrative commands root privileges. If the root shell from section 3 is still active, return to the regular user:

```bash
exit
```

If the user has just been added to the `sudo` group, log out of the regular user's VM session using the same command and log back in so the new group membership takes effect.

#### 4.1 Configuring SSH

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

#### 4.2 Connecting via SSH

To connect to the VM from the host computer, **port forwarding** must be configured in VirtualBox.

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

- **Host:** The physical computer where VirtualBox runs.
- **Guest:** The Debian virtual machine.
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
- `localhost`: Refers to the host computer, where the forwarding rule is configured.
- `-p 4243`: Specifies the host port used for the connection.

VirtualBox forwards the connection from the host's port `4243` to the VM's port `4242`.
On the first connection, SSH may request verification of the server's host key. Once accepted, enter the Debian user's password.
After authentication, the terminal prompt should look similar to:

```text
varaniba@varaniba42:~$
```

This confirms that the SSH connection to the virtual machine has been established.

To close the VM from the terminal on the host computer:

```bash
sudo shutdown now
```

**Note:** Although the terminal runs on the host computer, commands entered in this SSH session execute inside the VM. To be able to access the VM through the terminal, the VM needs to be running and unlocked. The moment the VM is shut down, the connection to the terminal will be closed.

### 5. UFW firewall

**UFW (Uncomplicated Firewall)** is a tool used to manage firewall rules on Linux. A firewall controls incoming and outgoing network traffic based on predefined rules.

Born2beRoot requires UFW to be enabled and configured to allow incoming connections through port `4242`, which is used by the SSH service.

This section continues as the regular VM user, using `sudo` because installing UFW and managing firewall rules require root privileges.

#### 5.1. Installing UFW

**Install the UFW package:**

```bash
sudo apt install ufw
```

**Check the firewall status:**

```bash
sudo ufw status
```

Initially, the output will generally show `Status: inactive`, meaning the firewall is not enabled.


#### 5.2. Configuring UFW

**Allow connections through port 4242:**

```bash
sudo ufw allow 4242
```

This creates a firewall rule allowing incoming connections through port `4242`, which is used by the SSH server.

**Enable the firewall:**

```bash
sudo ufw enable
```

It is recommended to allow port `4242` **before enabling UFW**, especially when configuring the VM through SSH. Otherwise, enabling the firewall could block the SSH connection.

#### 5.3. Verifying UFW Configuration

**Check the firewall status and rules:**

```bash
sudo ufw status
```

The output should indicate that the firewall is active and port `4242` is allowed.


### 6. Sudo Security Configuration

Born2beRoot requires additional security settings for `sudo`, including password attempt limits, custom error messages, command logging, TTY enforcement, and restricted executable paths.

The configuration commands below are run as root because they modify sudo's system configuration. Switch from the regular VM user to root:

```bash
su -
```

Remain in this root shell until the instruction to return to the regular user in section 6.2.

**Sudoers:**

`sudoers` is the configuration system that defines which users can execute commands with elevated privileges and under what conditions. Its main configuration file is `/etc/sudoers`.

The `/etc/sudoers.d/` directory can contain additional configuration files that are read by `sudo`, provided the main configuration includes this directory.

Using a separate file is preferable because:
- It keeps the custom settings separate from the system's default configuration.
- It makes configuration easier to maintain and troubleshoot.
- It reduces the need to modify the main configuration file.

For Born2beRoot, instead of modifying `/etc/sudoers` directly, the configuration will be stored in `/etc/sudoers.d/born2beroot`.

#### 6.1. Configuring Sudo

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
When saving, `visudo` creates a temporary file (`born2beroot.tmp`) where the changes are stored. After exiting the editor, it checks the file for syntax errors. If the configuration is valid, the temporary file is renamed to `born2beroot`. If errors are found, `visudo` offers the option to correct them before saving.

**Check the configuration for syntax errors:**

```bash
visudo -c
```

This checks that the sudo configuration files contain no syntax errors. A successful result indicates that the files were parsed correctly.

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

A **TTY** is a terminal that allows interaction with Linux by typing commands. A **PTY (pseudo-terminal)** is a virtual terminal created by software, such as the terminal used when connecting to the VM through SSH.

The subject requires TTY mode to be enabled but does not specify which sudo setting to use:

- **`requiretty`:** Only allows sudo commands to run when the user has a terminal. For example, `sudo apt update` works when typed in a terminal but may be blocked when executed by an automated script without a terminal.

- **`use_pty`:** Runs sudo commands in a separate virtual terminal. For example, when `sudo apt update` is executed, sudo creates a virtual terminal for `apt` and manages communication with the original terminal. All output remains visible in the same window.

Normally, a program running with root privileges can interact directly with the terminal. If the program is malicious, it could attempt to insert commands for the shell to execute.

The `use_pty` setting was chosen because running privileged commands in a separate terminal helps prevent this interference, while still allowing automated tasks to use `sudo` without an interactive terminal.

#### 6.2. Verifying the Configuration

After configuring `sudo`, the security settings must be verified. These checks are performed as the regular user to verify sudo permissions, password prompts, and logging for that account.

**Verify sudo authentication and administrator privileges (as regular user):**


```bash
exit
```

First, clear any previous sudo authentication:

```bash
sudo -k
```

Normally, `sudo` remembers authentication for a short period, so a password is not required for every command. The `-k` option clears this authentication, forcing sudo to request the user's password again.

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

This confirms that the regular user (`varaniba`) can execute commands with root privileges using `sudo`, without permanently switching to the root account.

**Verify the password attempt limit and custom error message:**

```bash
sudo -k
sudo whoami
```

Enter an incorrect password three times.

The custom error message should appear, and sudo should stop requesting passwords after three failed attempts.

**Verify that sudo logs are being created:**

```bash
sudo ls -la /var/log/sudo
```

Breaking down the command:

- `sudo`: Executes the command with root privileges.
- `ls`: Lists files and directories.
- `-l`: Shows details such as permissions, owner, size, and modification date.
- `-a`: Includes hidden files and directory entries.

The expected entries are:

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
- `sudoreplay`: Allows inspection of recorded sudo sessions.
- `-d /var/log/sudo`: Specifies where the recordings are stored.
- `-l`: Lists the available recorded sessions.

A recorded session can also be replayed using its session ID. Find a session ID (`TSID`) and replay its recording:

```bash
sudo sudoreplay -d /var/log/sudo <session_id>
```

This verifies that sudo records what appears in the terminal during a command, not just which command was executed.

**Verify the active sudo settings:**

```bash
sudo -l
```
This displays the commands the user is allowed to execute with sudo and the sudo defaults that apply to the user.


### 7. Password Policy

Born2beRoot requires a strong password policy to protect user accounts. This includes password expiration rules and password complexity requirements.

Sections 7.1–7.3 are performed as root to modify system configuration files and account password settings. In section 7.4, the checks are performed as the regular user, using `sudo` where access to account password settings requires root privileges.

#### 7.1. Password Expiration

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

#### 7.2. Password Complexity

Password complexity defines the requirements a new password must satisfy.

Password complexity is enforced through **PAM (Pluggable Authentication Modules)**, a system that manages authentication in Linux. The `libpam-pwquality` package checks whether new passwords meet the configured security requirements.

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

#### 7.3. Updating Existing Passwords

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

#### 7.4. Verifying the Password Policy

These checks are performed as the regular user, using `sudo` where root privileges are required.

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


### 8. Monitoring Script

Born2beRoot requires a Bash script that displays system information on all logged-in terminals at startup and every 10 minutes.

This section is performed as root to create the script `monitoring.sh` in `/usr/local/bin`, access sudo logs, and manage root's crontab. From the regular VM user, switch to root:

```bash
su -
```

The implementation uses three components:

- **Bash:** Collects and formats system information.
- **`wall`:** Displays messages on the terminals of logged-in users.
- **`cron`:** Executes the script automatically according to a schedule.

#### 8.1. Creating the Script

Create the script as root:

```bash
nano /usr/local/bin/monitoring.sh
```

The script collects the following information:

| Information | Command or source | Description |
|---|---|---|
| Architecture | `uname -a` | Displays the operating system and kernel information. |
| Physical processors | `lscpu` | Counts processor sockets visible to the VM. |
| Virtual processors | `nproc --all` | Counts logical processors available to the VM. |
| RAM usage | `free -m` | Displays memory usage and available memory in MB. |
| Disk usage | `df` | Displays disk space and utilization. |
| CPU usage | `/proc/stat` | Calculates CPU usage by comparing two measurements. |
| Last reboot | `uptime -s` | Displays when the system last started. |
| LVM | `lsblk` | Checks whether LVM logical volumes are present. |
| Connections | `ss` | Counts established TCP connections. |
| Users | `who` | Counts distinct users currently logged into terminals. |
| Network | `ip` | Retrieves the VM's IPv4 and MAC addresses. |
| Sudo | `grep` | Counts command entries in `born2beroot.log`. |

The information is collected each time the script runs, ensuring that the displayed values reflect the current system state.

The `wall` command broadcasts the results to logged-in terminals.

**Make the script executable:**

```bash
chmod +x /usr/local/bin/monitoring.sh
```

**Test the script:**

```bash
/usr/local/bin/monitoring.sh
```

#### 8.2. Scheduling with Cron

`cron` is a Linux service that automatically executes commands at scheduled times. Its schedules are stored in crontabs.

**Install and enable cron (as root):**

```bash
apt install cron
systemctl enable --now cron
```

**Edit root's crontab:**

```bash
crontab -e
```

Add:

```cron
@reboot /usr/local/bin/monitoring.sh
*/10 * * * * /usr/local/bin/monitoring.sh
```

- `@reboot`: Executes the script when cron starts, normally at system startup.
- `*/10 * * * *`: Executes the script every 10 minutes.

#### 8.3. Verifying the Configuration

**Verify the scheduled tasks:**

```bash
crontab -l
```

**Verify that cron is active:**

```bash
systemctl status cron
```

**Execute the script manually:**

```bash
/usr/local/bin/monitoring.sh
```

The monitoring information should appear on the terminals of logged-in users.

#### 8.4. Disabling the Monitoring Script

The subject requires the script to be stopped during evaluation without modifying the script itself.

**Temporarily disable its scheduled execution:**

```bash
crontab -e
```

Comment out both entries:

```cron
# @reboot /usr/local/bin/monitoring.sh
# */10 * * * * /usr/local/bin/monitoring.sh
```

This prevents cron from launching the script again while keeping the script unchanged.

The entries can be reactivated by removing the `#` characters.



# RESOURCES

## References
To create this project, the references and tools used were:

- Linux man page [manual](https://linux.die.net/man/)
- Github [website](https://github.com/)
- Born2BeRoot guide [website](https://noreply.gitbook.io/born2beroot)
- Debian [Recommended Partitioning Scheme](https://www.debian.org/releases/trixie/amd64/apcs03.en.html)
- Debian [Directory Tree](https://www.debian.org/releases/trixie/amd64/apcs02.en.html)
- Debian [Disk Space Needed for Tasks](https://www.debian.org/releases/trixie/amd64/apds02.en.html)
- Debian [Package Management](https://www.debian.org/doc/manuals/debian-reference/ch02)
- Debian Wiki [sudo](https://wiki.debian.org/sudo)
- Debian [sudoers Configuration Manual](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html)
- Debian [visudo Manual](https://manpages.debian.org/trixie/sudo/visudo.8.en.html)
- Debian Wiki [System Groups](https://wiki.debian.org/SystemGroups)
- Debian Wiki [SSH](https://wiki.debian.org/SSH)
- Debian [SSH Server Configuration Manual](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html)
- Debian [SSH Client Manual](https://manpages.debian.org/trixie/openssh-client/ssh.1.en.html)
- VirtualBox [User Manual](https://www.virtualbox.org/manual/)
- VirtualBox [NAT Networking and Port Forwarding](https://www.virtualbox.org/manual/topics/networkingdetails.html#nat)
- Debian [Password Aging Configuration](https://manpages.debian.org/trixie/login.defs/login.defs.5.en.html)
- Debian [Password Quality Configuration](https://manpages.debian.org/trixie/libpwquality-common/pwquality.conf.5.en.html)
- Debian [PAM Password Quality Module](https://manpages.debian.org/trixie/libpam-pwquality/pam_pwquality.8.en.html)
- Debian [PAM Configuration](https://manpages.debian.org/trixie/libpam-runtime/pam-auth-update.8.en.html)
- Debian [Cron Manual](https://manpages.debian.org/trixie/cron/crontab.5.en.html)
- Debian [Crontab Manual](https://manpages.debian.org/trixie/cron/crontab.1.en.html)
- Debian [Wall Manual](https://manpages.debian.org/trixie/bsdutils/wall.1.en.html)
- Debian [Bash Manual](https://manpages.debian.org/trixie/bash/bash.1.en.html)

- Debian [Reasons to Use Debian](https://www.debian.org/intro/why_debian.html)
- Rocky Linux [Project Overview](https://rockylinux.org/)
- Rocky Linux [DNF Package Manager](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/)
- AppArmor [Documentation](https://apparmor.net/)
- Debian Wiki [Using AppArmor](https://wiki.debian.org/AppArmor/HowToUse)
- Rocky Linux [SELinux Security](https://docs.rockylinux.org/guides/security/learning_selinux/)
- Ubuntu Server [Firewall Documentation](https://ubuntu.com/server/docs/firewalls/)
- firewalld [Documentation](https://firewalld.org/documentation/index.html)
- Oracle VirtualBox [Introduction and Supported Platforms](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/Introduction.html)
- UTM [Documentation](https://docs.getutm.app/)
- UTM [Architecture and System Settings](https://docs.getutm.app/settings-qemu/system/)
- UTM [QEMU Backend](https://docs.getutm.app/settings-qemu/settings-qemu/)



## AI Use:

- ChatGPT and Codex:
  - Understanding programming concepts;
  - Debugging and reasoning about implementation issues;
  - Improving README structure, wording, grammar, and clarity.

All the code, the implementation choices and final project structure were created by the author.

