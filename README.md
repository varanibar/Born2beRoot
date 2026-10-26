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

### 4. Configuring SSH

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

**Note:** Although the terminal runs on the host computer, commands entered in this SSH session execute inside the VM.

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


## 6. Sudo policies

# REQUIREMENTS

# INSTRUCTIONS

# RESOURCES
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
- Debian Wiki[System Groups](https://wiki.debian.org/SystemGroups)
- Debian Wiki [SSH](https://wiki.debian.org/SSH)
- Debian [SSH Server Configuration Manual](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html)
- Debian [SSH Client Manual](https://manpages.debian.org/trixie/openssh-client/ssh.1.en.html)
- VirtualBox [User Manual](https://www.virtualbox.org/manual/)
- VirtualBox [NAT Networking and Port Forwarding](https://www.virtualbox.org/manual/topics/networkingdetails.html#nat)
- Debian Wiki [Uncomplicated Firewall](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)

## AI Use:

- ChatGPT and Codex:
  - Understanding programming concepts;
  - Debugging and reasoning about implementation issues;
  - Improving README wording, grammar, and clarity.

All the code, the implementation choices and final project structure were created by the author.

