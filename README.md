This project has been created as part of the 42 curriculum by *varaniba*

### Table of contents

- [DESCRIPTION](#description)
  - [Debian vs Rocky Linux](#debian-vs-rocky-linux)
  - [AppArmor vs SELinux](#apparmor-vs-selinux)
  - [UFW vs firewalld](#ufw-vs-firewalld)
  - [VirtualBox vs UTM](#virtualbox-vs-utm)
- [INSTRUCTIONS](#instructions)
  - [Command reference](#command-reference)
  - [Project description](#project-description)
    - [Preparation](#preparation)
    - [1. Create the virtual machine](#1-create-the-virtual-machine)
    - [2. Operating System installation](#2-operating-system-installation)
    - [3. Virtual Machine configuration](#3-virtual-machine-configuration)
      - [3.1 Sudo](#31-sudo)
      - [3.2 Users](#32-users)
      - [3.3 Groups](#33-groups)
    - [4. SSH service](#4-ssh-service)
      - [4.1 Configuring SSH](#41-configuring-ssh)
      - [4.2. Verifying SSH Configuration](#42-verifying-ssh-configuration)
      - [4.3 Connecting via SSH](#43-connecting-via-ssh)
    - [5. UFW firewall](#5-ufw-firewall)
      - [5.1. Installing UFW](#51-installing-ufw)
      - [5.2. Configuring UFW](#52-configuring-ufw)
      - [5.3. Verifying UFW Configuration](#53-verifying-ufw-configuration)
    - [6. Sudo Security Configuration](#6-sudo-security-configuration)
      - [6.1. Configuring Sudo](#61-configuring-sudo)
      - [6.2. Verifying the Configuration (as regular user)](#62-verifying-the-configuration-as-regular-user)
    - [7. Password Policy](#7-password-policy)
      - [7.1. Password Expiration](#71-password-expiration)
      - [7.2. Password Complexity](#72-password-complexity)
      - [7.3. Updating Existing Passwords](#73-updating-existing-passwords)
      - [7.4. Verifying the Password Policy (as regular user)](#74-verifying-the-password-policy-as-regular-user)
    - [8. Monitoring Script](#8-monitoring-script)
      - [8.1. Creating the Script](#81-creating-the-script)
      - [8.2. Scheduling with Cron](#82-scheduling-with-cron)
      - [8.3. Verifying the Configuration](#83-verifying-the-configuration)
      - [8.4. Disabling the Monitoring Script](#84-disabling-the-monitoring-script)
    - [9. Creating Signature](#9-creating-signature)
- [RESOURCES](#resources)
  - [References](#references)
  - [AI Use](#ai-use)

# DESCRIPTION

This project consists of setting up a server by following specific rules. A minimum amount of services will be installed, therefore there will be no graphical interface.

## Debian vs Rocky Linux

Both [Debian](https://www.debian.org/intro/about) and [Rocky Linux](https://rockylinux.org/) are free, community-maintained Linux distributions that can be used to run servers without a graphical interface.

Their main differences for Born2beRoot are their intended use, package management tools, and security systems.

| Debian | Rocky Linux |
|---|---|
| General-purpose distribution used for personal computers, servers, and development. | Distribution mainly focused on servers and enterprise environments, where stability and long-term support are important. |
| Uses [`.deb` packages](https://www.debian.org/doc/manuals/debian-faq/pkg-basics.en.html), managed with tools such as `apt`. | Uses [`.rpm` packages](https://docs.rockylinux.org/guides/package_management/dnf_package_manager/), managed with tools such as `dnf`. |
| Uses AppArmor for additional security. | Uses SELinux for additional security. |

**Debian 13 (Trixie)** was selected for this project because it has extensive [documentation](https://www.debian.org/doc/) and straightforward package management tools, making it a suitable starting point for learning Linux system administration.


## AppArmor vs SELinux

[AppArmor](https://apparmor.net/) and [SELinux](https://docs.rockylinux.org/guides/security/learning_selinux/) are Linux security systems that control what programs are allowed to access or do.

Both use **[Mandatory Access Control (MAC)](https://csrc.nist.gov/glossary/term/mandatory_access_control)**, which adds security rules beyond normal Linux file permissions. These rules help limit the damage if a program is compromised.

| AppArmor | SELinux |
|---|---|
| Uses [profiles](https://wiki.apparmor.net/profiles/profile-types-and-syntax/) that define what each program is allowed to access. | Uses [security labels](https://docs.rockylinux.org/guides/security/learning_selinux/) assigned to programs and files, with rules that determine which actions are allowed. |
| Mainly uses file paths, such as `/etc/passwd`, to control access. | Mainly uses security labels to control access to files and other resources. |
| **Enforce mode:** Blocks actions that violate security rules. **Complain mode:** Generally allows these actions but records the violations. | **Enforcing mode:** Blocks actions that violate security rules. **Permissive mode:** Allows these actions but records the violations. |

**Example:**

Imagine a web server tries to read a protected file.

- **AppArmor:** Checks the web server's profile to see whether it can access the file's path.
- **SELinux:** Checks the security labels of the web server and the file to determine whether access is allowed.

For this project, we use AppArmor, which is [enabled by default on Debian](https://wiki.debian.org/AppArmor/HowToUse).

However, AppArmor only restricts programs covered by active security profiles. Enabling AppArmor does not automatically mean every program is protected.

## UFW vs firewalld

[UFW (Uncomplicated Firewall)](https://manpages.debian.org/trixie/ufw/ufw.8.en.html) and [firewalld](https://firewalld.org/documentation/) are Linux tools used to manage firewall rules. A firewall controls incoming and outgoing network traffic, allowing or blocking connections based on security rules.

Unlike AppArmor and SELinux, which control what programs can access, UFW and firewalld control network connections.

| UFW | firewalld |
|---|---|
| Mainly designed to make firewall configuration simple. | Provides more options for managing firewall rules across different networks. |
| Uses the `ufw` command to allow or block connections based on ports, protocols, or IP addresses. | Uses the [`firewall-cmd`](https://firewalld.org/documentation/utilities/firewall-cmd.html) command to manage firewall rules. |
| Manages rules directly, such as allowing SSH connections through port `4242`. | Uses [zones](https://firewalld.org/documentation/zone/use-of-zones) to apply different rules depending on the network, such as a home network or public Wi-Fi. |


## VirtualBox vs UTM

[VirtualBox](https://www.virtualbox.org/) and [UTM](https://docs.getutm.app/) are programs used to create and run virtual machines (VMs). They allow us to install another operating system, such as Debian, without replacing the operating system of our computer.

Both allow us to configure the VM's memory, processors, storage, and network settings.

| VirtualBox | UTM |
|---|---|
| Available for Windows, Linux, and macOS. | Mainly available for macOS and iOS. |
| Mainly uses virtualization to run operating systems compatible with the host computer's processor architecture. | Supports both virtualization and emulation, allowing it to run operating systems designed for different processor architectures. |


**Virtualization vs emulation:**

- **Virtualization:** Uses the host computer's processor to run a compatible operating system, usually with better performance.
- **Emulation:** Simulates a different processor architecture, allowing software designed for another type of processor to run, usually with lower performance.


For this project, we use **VirtualBox** because it is compatible with our Linux host computer and provides the tools needed to configure our Debian VM.


# INSTRUCTIONS

## Command reference

This section lists the main commands used in Born2beRoot, grouped by setup section. Detailed explanations are provided in the corresponding sections below.

Commands with `sudo` are generally executed as the regular user, while administrative commands without `sudo` are executed from a root shell (`su -`).

Values inside angle brackets (`< >`) are placeholders that must be replaced with the appropriate values.

**[3. Virtual Machine Configuration](#3-virtual-machine-configuration)**

- `su -` → Switch to root and load its login environment.
- `exit` → Exit the current shell and return to the previous user, or log out.
- `apt update` → Refresh the list of available software packages.
- `apt install sudo` → Install sudo.
- `sudo -V` → Display the installed sudo version and configuration.
- `adduser <username>` → Create a new user account.
- `id <username>` → Display a user's ID and group memberships.
- `addgroup <groupname>` → Create a new group.
- `adduser <username> <groupname>` → Add a user to an existing group.
- `getent group <groupname>` → Display a group's information and listed members.
- `getent group <groupname1> <groupname2>` → Display information for multiple groups.
- `groups <username>` → Display the groups assigned to a user.
- `groups` → Display the groups of the current process.

**[4. SSH Service](#4-ssh-service)**

- `sudo apt install openssh-server` → Install the SSH server.
- `sudo service ssh status` → Check whether the SSH service is running.
- `sudo nano /etc/ssh/sshd_config` → Edit the SSH server configuration.
- `sudo sshd -t` → Check the SSH configuration for syntax errors.
- `sudo service ssh restart` → Restart SSH to apply configuration changes.
- `sudo sshd -T | grep -E '^(port|permitrootlogin) '` → Verify the active SSH port and root-login settings.
- `ssh <username>@localhost -p <hostport>` → Connect to the VM from the host computer using the forwarded port. For this project: `ssh varaniba@localhost -p 4243`.

**[5. UFW Firewall](#5-ufw-firewall)**

- `sudo apt install ufw` → Install the UFW firewall tool.
- `sudo ufw status` → Display the firewall status and configured rules.
- `sudo ufw allow 4242` → Allow incoming connections through port 4242.
- `sudo ufw enable` → Activate the firewall.

**[6. Sudo Security Configuration](#6-sudo-security-configuration)**

- `mkdir -p /var/log/sudo` → Create the sudo logging directory if it does not exist.
- `visudo -f /etc/sudoers.d/born2beroot` → Edit the project's sudo configuration with syntax checking.
- `visudo -c` → Check the sudoers configuration for syntax errors.
- `sudo -k` → Clear cached sudo authentication so the next authenticated sudo command requires a password.
- `whoami` → Display the current effective username.
- `sudo whoami` → Confirm that sudo runs commands as root.
- `sudo -l` → Display the current user's sudo permissions and applicable settings.
- `sudo ls -la /var/log/sudo` → List sudo log files and recording directories.
- `sudo cat /var/log/sudo/born2beroot.log` → Read the log of sudo commands.
- `sudo sudoreplay -d /var/log/sudo -l` → List recorded sudo sessions.
- `sudo sudoreplay -d /var/log/sudo <session_id>` → Replay a recorded sudo session.

**[7. Password Policy](#7-password-policy)**

- `nano /etc/login.defs` → Edit the default password expiration settings for new accounts.
- `chage -M 30 -m 2 -W 7 <username>` → Set a password's maximum age to 30 days, minimum age to 2 days, and expiration warning to 7 days.
- `chage -l <username>` → Display an account's password expiration settings.
- `apt install libpam-pwquality` → Install the password-quality checking module.
- `nano /etc/security/pwquality.conf` → Configure password complexity requirements.
- `passwd` → Change the current user's password.
- `passwd varaniba` → Change the password of `varaniba` when executed as root.
- `grep -E '^[[:space:]]*(minlen|ucredit|lcredit|dcredit|maxrepeat|usercheck|difok|enforce_for_root)' /etc/security/pwquality.conf` → Display the configured password-quality rules.
- `grep -E '^[[:space:]]*password.*pam_pwquality\.so' /etc/pam.d/common-password` → Verify that the password-quality module is enabled in PAM.
- `sudo pam-auth-update` → Open the PAM configuration tool.

**[8. Monitoring Script](#8-monitoring-script)**

- `nano /usr/local/sbin/monitoring.sh` → Create or edit the monitoring script.
- `chmod +x /usr/local/sbin/monitoring.sh` → Make the script executable.
- `/usr/local/sbin/monitoring.sh` → Run the monitoring script manually.
- `apt install cron` → Install the cron task scheduler.
- `systemctl enable --now cron` → Start cron immediately and enable it at system startup.
- `crontab -e` → Edit the current user's scheduled tasks. Run as root to configure the monitoring schedule.
- `crontab -l` → Display the current user's scheduled tasks.
- `systemctl status cron` → Check whether the cron service is running.

**System information used by the monitoring script**

- `uname -a` → Display the operating system and kernel information.
- `lscpu` → Display CPU information, including the number of processor sockets.
- `grep -c '^processor' /proc/cpuinfo` → Count the logical processors available to the VM.
- `free --mega` → Display memory usage in megabytes.
- `df -m --total -x tmpfs -x devtmpfs` → Display disk usage in MiB, excluding temporary filesystems.
- `vmstat 1 2` → Collect CPU statistics used to calculate CPU usage.
- `uptime -s` → Display the date and time of the last system startup.
- `lsblk` → Display storage devices and logical volumes.
- `ss -tan state established` → Display established TCP connections.
- `users` → Display usernames associated with logged-in sessions.
- `hostname -I` → Display the VM's IP addresses.
- `ip -4 -o addr show scope global` → Display network interfaces with global IPv4 addresses.
- `grep -c 'COMMAND=' /var/log/sudo/born2beroot.log` → Count recorded sudo commands.
- `wall` → Broadcast the monitoring information to logged-in terminals.

**[9. Creating Signature](#9-creating-signature)**

- `sudo shutdown -h now` → Safely shut down the VM before calculating its signature.
- `sha1sum <virtual_disk_name>.vdi` → Calculate the SHA-1 hash of the virtual disk from the host computer.

## Project description

The initial steps, [1. Create the virtual machine](#1-create-the-virtual-machine) and [2. Operating System installation](#2-operating-system-installation), follow the [Born2beRoot guide](https://noreply.gitbook.io/born2beroot). Refer to that guide for detailed installation and partitioning steps. Starting with [3. Virtual Machine configuration](#3-virtual-machine-configuration), the sections document the configuration used for this project, explain the choices made, and describe the commands used to configure and verify the system.

### Preparation:

- **Hypervisor:** [VirtualBox](https://www.virtualbox.org/)
- **Operating system:** [Debian 13 (Trixie)](https://www.debian.org/download)
- **External storage:** Not needed if the VM is stored in the sgoinfre directory.

### 1. Create the virtual machine:
Using [VirtualBox](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/Introduction.html), create a new virtual machine and configure the following resources:

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
- **Disk partitioning:** Use [manual partitioning](https://www.debian.org/releases/trixie/amd64/apcs03.en.html) to create the required partitions and mount points. Partition sizes must be chosen to provide [sufficient space for the operating system and services](https://www.debian.org/releases/trixie/amd64/apcs01.en.html) while avoiding unnecessary disk allocation.

The following table lists the partition sizes chosen for this VM and explains [their purpose](https://www.debian.org/releases/trixie/amd64/apcs02.en.html). For the steps to configure encrypted storage and LVM, refer to the guide linked above.

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

**[sudo (superuser do)](https://manpages.debian.org/trixie/sudo/sudo.8.en.html)** allows authorized users to execute commands with root privileges without logging in as root.
- **Root:** Has full administrative privileges. Commands can be executed without sudo.
- **Regular user:** Has limited permissions and must use sudo for administrative tasks, provided the user has been granted permission.

**Switch to root:**

Switch to root user [`su`](https://manpages.debian.org/trixie/util-linux/su.1.en.html), the `-` loads root's login environment:
```bash
su -
```
Both `su` and `su -` switch to the root user, but `su -` also loads root's login environment and changes to root's home directory (`/root`).

This includes root's `PATH`, which defines where Bash searches for commands. For example, `ufw` is located in `/usr/sbin/`, a directory that may not be included in a regular user's `PATH`.

Using `su -` ensures that administrative commands can be found without needing to use `sudo` while already logged in as root.

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

Both are [package managers](https://www.debian.org/doc/manuals/debian-reference/ch02) used to install, update, and remove software on Debian. They also manage **dependencies**, which are other packages a program needs to work.

| `apt` | `aptitude` |
|---|---|
| Mainly used by typing commands. | Can be used with commands or through an interactive menu in the terminal. |
| Automatically installs the dependencies a package needs. | Also manages dependencies, but can suggest different solutions when packages conflict. |
| Easier to use for everyday tasks. | Offers more options for managing packages. |

**Example:** If we want to install `sudo`, both commands work:

```bash
apt install sudo
aptitude install sudo
```

The difference becomes more noticeable when packages have conflicting dependencies. While `apt` may report that it cannot complete an installation, `aptitude` can suggest alternatives, such as installing a different package version or removing a conflicting package.

For Born2beRoot, `apt` is sufficient because we only need to install and manage a few packages.


#### 3.2 Users:

Linux uses [user accounts](https://www.debian.org/doc/manuals/debian-reference/ch04.en.html) to manage access to files, programs, and system resources.
Born2beRoot requires a regular user with the student's login as username, in addition to the root account.

**[Create a new user](https://manpages.debian.org/trixie/adduser/adduser.8.en.html):**
```bash
adduser <username>
```

The required user (`varaniba`) was already created during Debian installation, so it does not need to be created again.

**Verify that the user exists:**

```bash
id <username>
```

#### 3.3 Groups:
Linux [groups](https://wiki.debian.org/SystemGroups) allow multiple users to share permissions without configuring each user individually.
Born2beRoot requires a regular user to belong to two groups:

- **[sudo](https://wiki.debian.org/sudo)**:  Allows authorized users to execute commands with root privileges using `sudo`. Membership does not make the user root.

- **user42**: A custom group required by the project. It has no special privileges by default and demonstrates Linux group management.

**Create a new group:**

```bash
addgroup <groupname>
```

**Verify that a group exists:**

```bash
getent group <groupname>
```

**Add a user to a group:**

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

**Check all groups assigned to the current user:**

```bash
groups
```

### 4. SSH service
**[SSH (Secure Shell)](https://wiki.debian.org/SSH)** is a network protocol that allows secure, encrypted remote access to another machine through a terminal.
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

The [openssh-server package](https://packages.debian.org/trixie/openssh-server) installs `sshd`, which allows our VM to receive SSH connections.

**Difference between `ssh` and `sshd`:**

- **`ssh` [(SSH client)](https://manpages.debian.org/trixie/openssh-client/ssh.1.en.html):** The program used to connect to another machine. For example, we use the command `ssh varaniba@localhost -p 4243` from our host computer to connect to the VM.
- **`sshd` [(SSH server)](https://manpages.debian.org/trixie/openssh-server/sshd.8.en.html#DESCRIPTION):** A program that runs in the background on our VM, waiting for incoming SSH connections and handling them.

In short, `ssh` initiates the connection, while `sshd` accepts and manages it.

**Check the SSH service status:**

```bash
sudo service ssh status
```
The output should indicate that the service is `active (running)`.


**Change SSH server:**

By default, SSH uses port `22`. Born2beRoot requires:

- SSH to run on port `4242`.
- Root login through SSH to be disabled.

Edit the [SSH server configuration file](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html):

```bash
sudo nano /etc/ssh/sshd_config
```
Find and configure the following directives:

```text
Port 4242
PermitRootLogin no
```

- [Port](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html#Port) 4242: Changes the port on which the SSH server listens.
- [PermitRootLogin](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html#PermitRootLogin) no: Prevents the root account from connecting through SSH, regardless of the authentication method.

The modified lines must be uncommented.

**Validate the SSH configuration:**

```bash
sudo sshd -t
```

The [`sshd -t`](https://manpages.debian.org/trixie/openssh-server/sshd.8.en.html#t) command checks the SSH server configuration for errors. If the configuration is valid, no output is displayed. Otherwise, an error message appears.

**Restart the SSH service:**

Once the configuration has been validated successfully, restart SSH:

```bash
sudo service ssh restart
```

This restarts the SSH service, which manages the `sshd` program, so the changes to its configuration take effect.

#### 4.2. Verifying SSH Configuration

**Check the SSH status:**

```bash
systemctl is-active ssh
systemctl is-enabled ssh
```

For more details:

```bash
sudo service ufw status
```

**Verify the effective SSH configuration:**

```bash
sudo sshd -T | grep -E '^(port|permitrootlogin) '
```

Expected output:

```text
port 4242
permitrootlogin no
```

#### 4.3 Connecting via SSH

To connect to the VM from the host computer, **[port forwarding](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/networkingdetails.html#natforward)** must be configured in VirtualBox.

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
- [`-p 4243`](https://manpages.debian.org/trixie/openssh-client/ssh.1.en.html#p): Specifies the host port used for the connection.

VirtualBox forwards the connection from the host's port `4243` to the VM's port `4242`.
On the first connection, SSH may request [verification of the server's host key](https://manpages.debian.org/trixie/openssh-client/ssh.1.en.html#VERIFYING_HOST_KEYS). Once accepted, enter the Debian user's password.
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

**[UFW (Uncomplicated Firewall)](https://manpages.debian.org/trixie/ufw/ufw.8.en.html)** is a tool used to manage firewall rules on Linux. A firewall controls incoming and outgoing network traffic based on predefined rules.

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

It is recommended to allow port `4242` **before enabling UFW**, especially when configuring the VM through SSH.
Otherwise, enabling the firewall could block the SSH connection.

#### 5.3. Verifying UFW Configuration

**Check the firewall status:**

```bash
sudo ufw status
```

The output should indicate `Status: active` and that port `4242` is allowed.
For more details:

```bash
sudo service ufw status
```


### 6. Sudo Security Configuration

Born2beRoot requires additional security settings for `sudo`, including password attempt limits, custom error messages, command logging, TTY enforcement, and restricted executable paths.

The configuration commands below are run as root because they modify sudo's system configuration. Switch from the regular VM user to root:

```bash
su -
```

Remain in this root shell until the instruction to return to the regular user in section 6.2.

**Sudoers:**

[`sudoers`](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html) is the configuration system that defines which users can execute commands with elevated privileges and under what conditions. Its main configuration file is `/etc/sudoers`.

The [`/etc/sudoers.d/`](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html#Including_other_files_from_within_sudoers) directory can contain additional configuration files that are read by `sudo`.

Using a separate file is preferable because:
- It keeps the custom settings separate from the system's default configuration.
- It makes configuration easier to maintain and troubleshoot.
- It reduces the need to modify the main configuration file.

For Born2beRoot, instead of modifying `/etc/sudoers` directly, the configuration will be stored in `/etc/sudoers.d/born2beroot`.

#### 6.1. Configuring Sudo


**Create a directory for sudo logs:**

```bash
mkdir -p /var/log/sudo
```

This creates the directory where sudo will store its command logs and input/output recordings, as required by Born2beRoot. This will be used for the configuration file on the next step.

The `-p` option creates any missing parent directories and prevents an error if the directory already exists.

**Create a custom sudo configuration file:**

```bash
visudo -f /etc/sudoers.d/born2beroot
```

Using [`visudo`](https://manpages.debian.org/trixie/sudo/visudo.8.en.html) instead of directly editing the file checks the configuration for syntax errors before saving.

There are different [default parameters](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html#SUDOERS_OPTIONS) that can be used to modify `sudo`s' behavior. For this project, the following configuration was added:

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

**Note:** Copying and pasting into `visudo` may not work directly through the VirtualBox console. Alternatively, connect to the VM through SSH and edit the configuration file there.

**Verify the sudoers configuration:**

```bash
visudo -c
```

Although `visudo` checks for syntax errors when saving, this command allows us to verify the saved sudoers configuration files without modifying them.


**Understanding the configuration:**

| Setting | Description |
|---|---|
| `passwd_tries` | Sets how many times a user can enter an incorrect password when using `sudo`. |
| `badpass_message` | Shows a custom message when the user enters an incorrect password. |
| `logfile` | Defines where to save a record of sudo commands, including who used them and when. |
| `log_input,log_output` | Records what users type and what appears in the terminal when running commands with `sudo`. |
| `iolog_dir` | Defines the directory where these input and output recordings are saved. |
| `use_pty` | Runs sudo commands in a separate virtual terminal to improve security. |
| `secure_path` | Defines which directories sudo looks in to find commands to execute. |

**Terminal security:**

A **TTY** is a terminal that allows interaction with Linux by typing commands. A **[PTY (pseudo-terminal)](https://manpages.debian.org/trixie/manpages/pty.7.en.html)** is a virtual terminal created by software, such as the terminal used when connecting to the VM through SSH.

The subject requires TTY mode to be enabled but does not specify which sudo setting to use:

- **[`requiretty`](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html#:~:text=If%20set%2C%20sudo%20will%20only%20run%20when%20the%20user%20is%20logged%20in%20to%20a%20real%20tty.):** Only allows sudo commands to run when the user has a terminal. For example, `sudo apt update` works when typed in a terminal but may be blocked when executed by an automated script without a terminal.

- **[`use_pty`](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html#:~:text=If%20set%2C%20and%20sudo%20is%20running%20in%20a%20terminal):** Runs sudo commands in a separate virtual terminal. For example, when `sudo apt update` is executed, sudo creates a virtual terminal for `apt` and manages communication with the original terminal. All output remains visible in the same window.

Normally, a program running with root privileges can interact directly with the terminal. If the program is malicious, it could attempt to insert commands for the shell to execute.

The `use_pty` setting was chosen because running privileged commands in a separate terminal helps prevent this interference, while still allowing automated tasks to use `sudo` without an interactive terminal.

#### 6.2. Verifying the Configuration (as regular user)

After configuring `sudo`, the security settings must be verified. These checks are performed as the regular user to verify sudo permissions, password prompts, and logging for that account. Switch from root to the regular VM user:

```bash
exit
```

**Verify sudo authentication and administrator privileges:**

First, clear any previous sudo authentication:

```bash
sudo -k
```

Normally, `sudo` remembers authentication for a short period, so a password is not required for every command. The [`-k` option](https://manpages.debian.org/trixie/sudo/sudo.8.en.html#k) clears this authentication, forcing sudo to request the user's password again.

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
- [`sudoreplay`](https://manpages.debian.org/trixie/sudo/sudoreplay.8.en.html): Allows inspection of recorded sudo sessions.
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

A **password policy** defines rules that passwords must follow to improve account security. It helps prevent unauthorized access by requiring strong passwords and regular password changes.

Born2beRoot requires two types of rules:
- **Password expiration:** Limits how long a password can be used and how frequently it can be changed.
- **Password complexity:** Requires passwords to meet certain conditions, such as minimum length, uppercase and lowercase letters, and numbers, making them harder to guess.

Sections 7.1–7.3 are performed as root to modify system configuration files and account password settings. In section 7.4, the checks are performed as the regular user, using `sudo` where administrator privileges are required.

**Switch to root:**

```bash
su -
```

#### 7.1. Password Expiration

[`/etc/login.defs`](https://manpages.debian.org/trixie/login.defs/login.defs.5.en.html) is a system configuration file that defines default settings for user accounts, including password expiration rules.

**Configure password expiration:**

```bash
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

These settings are applied by default when new accounts are created. Existing accounts must be updated separately using [`chage`](https://manpages.debian.org/trixie/passwd/chage.1.en.html).

**Update existing users:**

Before updating the settings of the existing accounts, check their current values:

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

Check again the values of the settings to confirm that they were correctly updated.

#### 7.2. Password Complexity

Password complexity is enforced through **[PAM (Pluggable Authentication Modules)](https://www.debian.org/doc/manuals/debian-reference/ch04.en.html#_pam_and_nss)**, a system that manages authentication in Linux.
The [`libpam-pwquality`](https://packages.debian.org/trixie/libpam-pwquality) package provides the `pam_pwquality` PAM module, which checks whether new passwords meet the configured security requirements.

**Install the package:**

```bash
apt install libpam-pwquality
```

**Configure password complexity:**

[`/etc/security/pwquality.conf`](https://manpages.debian.org/trixie/libpwquality-common/pwquality.conf.5.en.html) is a system configuration file that defines password complexity requirements, including minimum length, required character types, and restrictions on repeated characters. The [pam_pwquality manual](https://manpages.debian.org/trixie/libpam-pwquality/pam_pwquality.8.en.html) documents the configuration options.

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

**Note:** Copying and pasting into Nano may not work directly through the VirtualBox console. Alternatively, connect to the VM through SSH and edit the system configuration file there.

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

When root changes a password, the [previous password is not requested](https://manpages.debian.org/trixie/libpam-pwquality/pam_pwquality.8.en.html#enforce_for_root). Therefore, checks that compare the old and new passwords cannot be performed.

#### 7.3. Updating Existing Passwords

After configuring the password policy, Born2beRoot requires all existing account passwords, including root's, to be changed.

This ensures that the new passwords follow the configured requirements, since modifying the configuration files does not automatically update existing passwords.

To change the password of the current user:

```bash
passwd
```
Change the passwords of both `root` and `varaniba`.

**Password expiration restrictions:**

The minimum password age of 2 days prevents regular users from changing a recently updated password.

If the password change is blocked, root can [reset the user's password](https://manpages.debian.org/trixie/passwd/passwd.1.en.html) using:

```bash
passwd varaniba
```

However, when root resets another user's password, the previous password is not requested, so the `difok = 7` comparison cannot be performed.

All existing accounts with passwords must be updated before completing the configuration.

#### 7.4. Verifying the Password Policy (as regular user)

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

If no active entry appears, check the [PAM configuration](https://manpages.debian.org/trixie/libpam-runtime/pam-auth-update.8.en.html):

```bash
sudo pam-auth-update
```

Make sure the password-quality profile is enabled.


### 8. Monitoring Script

Born2beRoot requires a Bash script named `monitoring.sh` that displays system information on all logged-in terminals at startup and every 10 minutes.

This section is performed as root because we need administrator privileges to create the monitoring script, access the protected sudo logs, and configure root's crontab.
From the regular VM user, switch to root:

```bash
su -
```

The implementation uses three components:

- **[Bash](https://manpages.debian.org/trixie/bash/bash.1.en.html):** A command-line shell and scripting language used to execute commands. In this project, it collects and formats system information.
- **[`wall`](https://manpages.debian.org/trixie/bsdutils/wall.1.en.html):** A command that displays messages on the terminals of logged-in users.
- **[`cron`](https://manpages.debian.org/trixie/cron/cron.8.en.html):** A service that automatically executes commands or scripts according to a schedule.


#### 8.1. Creating the Script

The [`/usr/local/sbin/`](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/ch04s09.html) directory is intended for locally installed system administration programs and scripts, making it an appropriate location for `monitoring.sh`.

Create the script:

```bash
nano /usr/local/sbin/monitoring.sh
```

Insert the following script:

```text
#!/bin/bash

# ARCHITECTURE
arch=$(uname -a)

# PROCESSORS
cpuf=$(lscpu | awk '/Socket\(s\):/ {print $2}')
cpuv=$(grep -c '^processor' /proc/cpuinfo)

# RAM
ram_total=$(free --mega | awk '/^Mem:/ {print $2}')
ram_use=$(free --mega | awk '/^Mem:/ {print $3}')
ram_percent=$(free --mega | awk '/^Mem:/ {printf "%.1f", $3/$2*100}')

# DISK
disk_total=$(df -m --total -x tmpfs -x devtmpfs | awk '$1=="total" {print $2}')
disk_use=$(df -m --total -x tmpfs -x devtmpfs | awk '$1=="total" {print $3}')
disk_percent=$(awk -v used="$disk_use" -v total="$disk_total" \
    'BEGIN {if (total > 0) printf "%.1f", used/total*100; else print 0}')

# CPU USAGE
cpu_idle=$(vmstat 1 2 | tail -1 | awk '{print $15}')
cpu_usage=$((100 - cpu_idle))

# LAST BOOT
last_boot=$(uptime -s)

# LVM
if lsblk -n -o TYPE | grep -q lvm; then
    lvm="yes"
else
    lvm="no"
fi

# TCP CONNECTIONS
connections=$(ss -tan state established | tail -n +2 | wc -l)

# LOGGED-IN USERS
users_count=$(who | awk '$2 != "seat0" {print $1}' | sort -u | wc -l)

# NETWORK
ip_address=$(hostname -I | awk '{print $1}')
interface=$(ip -4 -o addr show scope global | awk 'NR==1 {print $2}')
mac_address="N/A"

if [ -n "$interface" ]; then
    mac_address=$(cat "/sys/class/net/$interface/address")
fi

# SUDO COMMANDS
sudo_count=$(grep -c 'COMMAND=' /var/log/sudo/born2beroot.log || true)

# DISPLAY
wall "	Architecture: $arch
	CPU physical: $cpuf
	vCPU: $cpuv
	Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	Disk Usage: $disk_use/${disk_total}MB ($disk_percent%)
	CPU load: $cpu_usage%
	Last boot: $last_boot
	LVM use: $lvm
	Connections TCP: $connections ESTABLISHED
	User log: $users_count
	Network: IP $ip_address ($mac_address)
	Sudo: $sudo_count commands"
```

**Note:** Copying and pasting into Nano may not work directly through the VirtualBox console. Alternatively, connect to the VM through SSH and edit the script there.

**Understanding the script:**

The script collects the following information required by the subject:

| Information | Command or source | Description |
|---|---|---|
| Architecture | [`uname -a`](https://manpages.debian.org/trixie/coreutils/uname.1.en.html) | Displays the operating system architecture and kernel version. |
| CPU physical | [`lscpu`](https://manpages.debian.org/trixie/util-linux/lscpu.1.en.html) | Shows the number of physical processors. |
| vCPU | [`/proc/cpuinfo`](https://manpages.debian.org/trixie/manpages/proc_cpuinfo.5.en.html) | Shows the number of virtual processors. |
| Memory Usage | [`free --mega`](https://manpages.debian.org/trixie/procps/free.1.en.html) | Displays total and used RAM, and its usage percentage. |
| Disk Usage | [`df -m`](https://manpages.debian.org/trixie/coreutils/df.1.en.html) | Displays total and used disk space, and its usage percentage. |
| CPU load | [`vmstat`](https://manpages.debian.org/trixie/procps/vmstat.8.en.html) | Calculates the current CPU usage percentage. |
| Last boot | [`uptime -s`](https://manpages.debian.org/trixie/procps/uptime.1.en.html) | Displays the date and time of the last reboot. |
| LVM use | [`lsblk`](https://manpages.debian.org/trixie/util-linux/lsblk.8.en.html) | Checks whether LVM is being used. |
| Connections TCP | [`ss`](https://manpages.debian.org/trixie/iproute2/ss.8.en.html) | Counts active TCP connections, such as SSH connections. |
| User log | [`users`](https://manpages.debian.org/trixie/coreutils/users.1.en.html) | Counts how many user sessions are currently logged into the VM. |
| Network | [`hostname -I`](https://manpages.debian.org/trixie/hostname/hostname.1.en.html), [`ip`](https://manpages.debian.org/trixie/iproute2/ip.8.en.html) | Displays the IPv4 and MAC addresses. |
| Sudo | `grep` | Counts commands executed with sudo using the log file. |

The script stores the collected information in variables and uses `wall` to display it on the virtual machine terminal. Each execution collects updated system information.

**Note:** On Debian 13 (Trixie), `wall` may fail to display messages in SSH terminals because their terminal information is not correctly registered with `systemd-logind`. This issue is documented in [Debian's bug report](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1115259).


**Make the script executable:**

```bash
chmod +x /usr/local/sbin/monitoring.sh
```

**Test the script:**

```bash
/usr/local/sbin/monitoring.sh
```

#### 8.2. Scheduling with Cron

[`cron`](https://manpages.debian.org/trixie/cron/cron.8.en.html) is a Linux service that automatically executes commands at scheduled times. Its schedules are stored in [crontabs](https://manpages.debian.org/trixie/cron/crontab.5.en.html). This section is performed as root.

**Install and enable cron:**

Cron may already be installed on Debian, depending on the packages selected during installation. The following command ensures it is installed:

```bash
apt install cron
```

**Enable and start the cron service:**

```bash
systemctl enable --now cron
```

[`systemctl`](https://manpages.debian.org/trixie/systemd/systemctl.1.en.html) is used to manage system services. This command performs two actions:

- `enable`: Configures cron to start automatically whenever the system boots.
- `--now`: Also starts cron immediately, without requiring a reboot.

Without `--now`, `enable` would only configure automatic startup and would not start the service immediately.

This command is equivalent to running:

```bash
systemctl enable cron
systemctl start cron
```

**Verify that cron is enabled and running:**

```bash
systemctl is-enabled cron
systemctl is-active cron
```

- `enabled`: Confirms that cron is configured to start automatically at boot.
- `active`: Confirms that cron is currently running.

For more detailed information:
```bash
service cron status
```

**Edit root's crontab:**

```bash
crontab -e
```

The first time we run this command, cron may display `no crontab for root - using an empty one`, meaning root has no scheduled tasks yet.

It will also ask us to select a text editor. Choose `1` for Nano.

Add the following lines:

```cron
@reboot /usr/local/sbin/monitoring.sh
*/10 * * * * /usr/local/sbin/monitoring.sh
```

- [`@reboot`](https://manpages.debian.org/trixie/cron/crontab.5.en.html#Extensions): Executes the script when cron starts, normally at system startup.
- [`*/10 * * * *`](https://manpages.debian.org/trixie/cron/crontab.5.en.html#FORMAT): Executes the script every 10 minutes.

**Note:** Copying and pasting into Nano may not work directly through the VirtualBox console. Alternatively, connect to the VM through SSH and edit the crontab there.

#### 8.3. Verifying the Configuration

**Verify the scheduled tasks:**

```bash
crontab -l
```

This displays the [scheduled tasks for the current user](https://manpages.debian.org/trixie/cron/crontab.1.en.html). Since we are working as root, it shows root's crontab.

The output should include the two entries for `monitoring.sh`.

**Verify more detailed information:**

```bash
systemctl status cron
```

**Execute the script manually:**

```bash
/usr/local/sbin/monitoring.sh
```
This executes the script without waiting for cron, allowing us to verify that it collects and displays the system information.

The monitoring information should appear in the VirtualBox terminal.

#### 8.4. Disabling the Monitoring Script

The subject requires the script to be stopped during evaluation without modifying the script itself.

**Temporarily disable its scheduled execution:**

```bash
crontab -e
```

Comment out both entries:

```cron
# @reboot /usr/local/sbin/monitoring.sh
# */10 * * * * /usr/local/sbin/monitoring.sh
```

This prevents cron from launching the script again while keeping the script unchanged.

The entries can be reactivated by removing the `#` characters.


### 9. Creating Signature

A **signature** is a value calculated from the contents of a file using a hashing algorithm. For Born2beRoot, we use [SHA-1](https://www.gnu.org/software/coreutils/manual/html_node/sha1sum-invocation.html) to generate a signature of the VM's virtual disk (`.vdi` file).

The signature allows us to verify that the virtual disk has not changed since it was submitted.

**Shut down the virtual machine:**

Before generating the signature, shut down the VM completely:

```bash
sudo shutdown -h now
```

- [`-h`](https://manpages.debian.org/trixie/systemd-sysv/shutdown.8.en.html#OPTIONS): Halts the system and normally powers it off.
- `now`: Starts the shutdown immediately.

This command is executed inside Debian to safely shut down the VM before calculating its signature.

**Generate the signature:**

From the host computer, navigate to the directory containing the `.vdi` file and run:

```bash
sha1sum <virtual_disk_name>.vdi
```

The output contains a 40-character SHA-1 hash followed by the filename.

For this project:

```bash
sha1sum Born2beRoot.vdi
```

**Create the signature file:**

Create a file named `signature.txt` at the root of the project's Git repository and paste only the SHA-1 hash into it, without the `.vdi` filename.

**Preserving the signature:**

The SHA-1 hash depends on the contents of the virtual disk. If any data on the disk changes, the hash will almost certainly change as well.

For this reason, **the VM must not be started or modified after generating the signature**. Even starting the VM normally can change system files and produce a different hash.

During the evaluation, the evaluator will calculate the signature of the virtual disk again and compare it with the value stored in `signature.txt`.

If the two values match, it confirms that the disk has not changed in a way detected by the hash since the signature was created.

If the VM needs to be modified before submission, a new signature must be generated and `signature.txt` updated.


# RESOURCES

## References
To create this project, the references and tools used were:

- Linux man page [manual](https://linux.die.net/man/)
- Github [website](https://github.com/)
- VirtualBox [User Manual](https://docs.oracle.com/en/virtualization/virtualbox/7.1/user/)

- firewalld [Documentation](https://firewalld.org/documentation/index.html)

## AI Use:

- ChatGPT and Codex:
  - Understanding programming concepts;
  - Debugging and reasoning about implementation issues;
  - Improving README structure, wording, grammar, and clarity.

All the code, the implementation choices and final project structure were created by the author.

