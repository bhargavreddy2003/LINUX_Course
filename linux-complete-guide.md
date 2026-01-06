# COMPLETE LINUX GUIDE: BASIC TO SENIOR LEVEL
## A Self-Contained Resource for Data Engineers & System Administrators

---

## TABLE OF CONTENTS

1. **FOUNDATIONS & BASICS**
2. **SYSTEM ADMINISTRATION**
3. **FILE SYSTEMS & STORAGE**
4. **NETWORKING & SOCKETS**
5. **PROCESS MANAGEMENT & DEBUGGING**
6. **SHELL SCRIPTING MASTERY**
7. **PERFORMANCE TUNING & OPTIMIZATION**
8. **SECURITY HARDENING**
9. **SYSTEM MONITORING & LOGGING**
10. **CONTAINERIZATION & KERNEL**
11. **REAL-WORLD TRICKS & OPTIMIZATION**

---

# PART 1: FOUNDATIONS & BASICS

## 1.1 Linux Directory Structure

```
/           → Root directory (all files/directories branch from here)
/bin        → Essential user binaries (ls, cat, cp, mv)
/sbin       → System binaries for root (ifconfig, halt, reboot)
/usr        → User programs, libraries, documentation
/usr/bin    → Non-essential user binaries (mostly installed here)
/usr/sbin   → Non-essential system binaries
/etc        → Configuration files for all programs
/etc/passwd → User account information
/etc/shadow → Encrypted user passwords
/etc/group  → Group information
/home       → Home directories for regular users
/root       → Home directory for root user
/tmp        → Temporary files (cleared on reboot typically)
/var        → Variable data (logs, databases, cache)
/var/log    → System and application logs
/boot       → Boot loader files, kernel images
/lib        → Shared libraries needed by /bin and /sbin
/dev        → Device files (hard drives, terminals, printers)
/proc       → Virtual filesystem with process info
/sys        → Virtual filesystem with hardware/kernel info
/opt        → Optional software packages
/srv        → Service-specific data (web servers, etc)
```

## 1.2 Basic Commands You Must Know

### Navigation & File Operations
```bash
pwd                           # Print current working directory
cd ~                          # Go to home directory
cd -                          # Go to previous directory
cd ..                         # Go up one directory
cd /path/to/dir              # Absolute path navigation

ls -la                        # List all files with details (including hidden)
ls -lh                        # List files with human-readable sizes
ls -lrt                       # List by modification time (oldest first)
ls -lS                        # List by size (largest first)
tree -L 2                     # Show directory structure with depth limit

cp -r source dest            # Recursive copy
cp -v file1 file2            # Copy with verbose output
cp -p file1 file2            # Preserve timestamps and permissions

mv old_name new_name         # Move or rename file
mv -i file newloc            # Move with confirmation if exists

rm -f file                   # Remove file forcefully
rm -rf directory/            # Remove directory recursively
shred -vfz -n 5 file        # Securely delete (overwrites 5 times)

mkdir -p dir/sub/deep       # Create nested directories
touch filename              # Create empty file or update timestamp
```

### File Viewing & Manipulation
```bash
cat file.txt                            # Display entire file
less file.txt                           # View file with pagination (q to quit)
head -20 file.txt                       # Show first 20 lines
tail -20 file.txt                       # Show last 20 lines
tail -f /var/log/syslog                 # Follow log updates in real-time

grep "pattern" file.txt                 # Search for text pattern
grep -r "pattern" /directory/           # Recursive search in directory
grep -i "pattern" file.txt              # Case-insensitive search
grep -c "pattern" file.txt              # Count matching lines
grep -E "regex|pattern" file.txt        # Extended regex support

wc -l file.txt                          # Count lines
wc -w file.txt                          # Count words
wc -c file.txt                          # Count bytes

sed 's/old/new/g' file.txt              # Replace all occurrences
sed -n '5,10p' file.txt                 # Print lines 5-10
sed -i.bak 's/old/new/g' file.txt      # In-place edit with backup

awk '{print $1, $3}' file.txt           # Print columns 1 and 3
awk -F: '{print $1}' /etc/passwd        # Use colon as field separator
awk '/pattern/ {count++} END {print count}' file.txt

cut -d: -f1 /etc/passwd                 # Extract field 1 (delimiter :)
sort file.txt                           # Sort file contents
sort -u file.txt                        # Sort and remove duplicates
sort -k2 -n file.txt                    # Sort by column 2 numerically

uniq file.txt                           # Remove duplicate consecutive lines
uniq -c file.txt                        # Count duplicates

diff file1 file2                        # Show differences
diff -u file1 file2                     # Unified diff (better format)

find /path -name "*.log" -type f        # Find files with extension
find /path -type d -name "test"        # Find directories
find /path -mtime -7                   # Modified in last 7 days
find /path -size +100M                 # Larger than 100MB
find /path -exec rm {} \;              # Execute command on results

locate filename                         # Find file by name (uses database)
updatedb                                # Update locate database

file /path/to/file                      # Identify file type
```

### Stream Redirection & Pipes
```bash
command > file.txt              # Redirect stdout to file (overwrite)
command >> file.txt             # Append stdout to file
command 2> error.log            # Redirect stderr to file
command 2>&1 file.txt          # Redirect both stdout and stderr
command < input.txt             # Redirect stdin from file

command1 | command2             # Pipe output of command1 to command2
cat file.txt | wc -l            # Pipe file to word count
ps aux | grep process_name      # Find running process
echo "text" | tee file.txt      # Output to both screen and file

command1 && command2            # Run command2 only if command1 succeeds
command1 || command2            # Run command2 only if command1 fails
command1; command2              # Run both commands sequentially

(command1 && command2) || command3  # Grouping with precedence
```

---

# PART 2: SYSTEM ADMINISTRATION

## 2.1 User & Group Management

### Creating Users & Groups
```bash
# User Management
sudo useradd -m -s /bin/bash -d /home/username username
# -m: Create home directory
# -s: Specify shell
# -d: Specify home directory path
# -G: Add to groups: sudo useradd -m -G sudo,developers username

sudo passwd username            # Set/change password interactively
echo "newpassword" | sudo passwd --stdin username  # Non-interactive

sudo userdel -r username        # Delete user and home directory
sudo usermod -aG groupname username    # Add user to group
sudo usermod -s /bin/zsh username     # Change user shell
sudo usermod -d /new/home username    # Change home directory

# Group Management
sudo groupadd developers               # Create group
sudo groupdel developers               # Delete group
sudo gpasswd -a username groupname    # Add user to group
sudo gpasswd -d username groupname    # Remove user from group

# View User/Group Info
id username                    # Show user ID, group IDs
id -u username                 # Show just user ID
id -g username                 # Show primary group ID
groups username                # Show all groups user belongs to
getent passwd username         # Get user entry from /etc/passwd
getent group groupname         # Get group entry from /etc/group
w                             # Show logged-in users
who                           # Show user login info
last                          # Show login history
lastlog                       # Show last login for each user
```

### Switching Users & Privilege Escalation
```bash
su - username                  # Switch to user (with login shell and env)
su username                    # Switch to user (current environment)
sudo command                   # Run single command as root
sudo -i                        # Switch to root with full login shell
sudo -u username command       # Run command as specific user
sudo -l                        # List user's sudo privileges
sudo -v                        # Refresh sudo timestamp (extend timeout)
sudo -k                        # Clear sudo timestamp (require password again)
sudo !!                        # Re-run previous command as root
sudo -H command               # Set HOME environment variable
```

### Sudoers Configuration (Advanced)
```bash
sudo visudo                    # Safely edit sudoers file (DO NOT use nano/vim directly)

# /etc/sudoers examples:
root ALL=(ALL:ALL) ALL                    # Root user full access
%sudo ALL=(ALL:ALL) ALL                   # All users in sudo group
username ALL=(ALL) NOPASSWD: /bin/ls      # Allow specific command without password
%developers ALL=(ALL) NOPASSWD: /usr/bin/systemctl  # Group with no password for service control
username ALL=(ALL) NOPASSWD: /bin/systemctl start apache2

# Aliases (variables for sudoers):
User_Alias ADMINS = user1, user2
Host_Alias WEBSERVERS = web1, web2
Cmnd_Alias SYSCTL = /bin/systemctl, /usr/sbin/service
ADMINS WEBSERVERS = (ALL) SYSCTL
```

## 2.2 Process Management (Beyond Basics)

### Process Control
```bash
ps aux                         # Show all running processes
ps auxf                        # Show process tree
ps -ef                         # Alternative format (POSIX)
ps -eo pid,user,%cpu,%mem,cmd --sort=-%cpu  # Sort by CPU usage
ps -p 1234 -o pid,ppid,cmd    # Show specific process details

# Process Priority
nice -n 10 command            # Run with lower priority (0-19, higher = lower priority)
nice -n -5 command            # Run with higher priority (negative requires sudo)
renice -n 5 -p 1234           # Change priority of running process (PID 1234)
renice -n -10 -u username     # Change priority for all processes by user

# Background/Foreground Jobs
command &                      # Run command in background
jobs                          # List background jobs
fg %1                         # Bring job 1 to foreground
bg %1                         # Resume job 1 in background
Ctrl+Z                        # Suspend current foreground process
wait                          # Wait for background jobs to complete
nohup long_command &          # Run command immune to hangups (survives terminal close)

# Process Termination
kill 1234                     # Terminate process gracefully (SIGTERM)
kill -9 1234                  # Force kill process (SIGKILL - no cleanup)
kill -15 1234                 # Send SIGTERM (default, allows graceful shutdown)
kill -STOP 1234               # Pause process
kill -CONT 1234               # Resume process
killall process_name          # Kill all processes by name
pkill -f "pattern"            # Kill by pattern match

# Resource Limits
ulimit -a                     # Show all resource limits
ulimit -n 4096                # Set max open files
ulimit -u 1000                # Set max processes per user
# Permanent limits in /etc/security/limits.conf:
# username soft nofile 2048
# username hard nofile 4096
```

### Process Information Tools
```bash
top                           # Interactive process monitor (q to quit)
top -u username               # Top processes for specific user
top -p 1234 -p 5678          # Monitor specific PIDs
# In top: Shift+M (sort by memory), Shift+P (sort by CPU), k (kill), r (renice)

htop                          # Modern process monitor with tree view
htop -u username              # Show processes for specific user

atop                          # Advanced system monitor (includes I/O, network)
atop -d 2 -c                  # Update every 2 seconds, continuous

ps_mem.py                     # Show actual memory usage (not RSS) - install: pip install ps_mem

cat /proc/[PID]/status        # Detailed process information
cat /proc/[PID]/fd            # Show file descriptors
cat /proc/[PID]/limits        # Show resource limits
cat /proc/[PID]/environ       # Show environment variables
cat /proc/[PID]/maps          # Show memory mapping
lsof -p 1234                  # Show files opened by process
```

---

# PART 3: FILE SYSTEMS & STORAGE

## 3.1 Understanding Inodes & File System Basics

### Inode Structure
```bash
# An inode stores:
# - File type (regular, directory, symlink, etc.)
# - Permissions (rwx for user, group, others)
# - Owner (UID) and group (GID)
# - File size in bytes
# - Timestamps (atime, mtime, ctime)
# - Link count (hard links)
# - Pointers to data blocks

ls -i file.txt                # Show inode number
stat file.txt                 # Show detailed inode information
stat -c "%i %s %y %n" *      # Custom format: inode, size, modify time, name

# Check filesystem for inode usage
df -i                         # Show inode usage per filesystem
df -i /home                   # Inode usage for /home

# Inode limitations
# ext4: max 2^32 inodes
# XFS: dynamic inode allocation (no fixed limit)
```

### Hard Links vs Symbolic Links
```bash
# HARD LINKS: Points directly to the inode
ln file.txt hardlink_file     # Create hard link
# - Multiple names for same inode/data
# - Only works within same filesystem
# - If original deleted, hardlink still accesses data
# - Cannot hard link directories
# - Storage shows same inode number

# SYMBOLIC LINKS: Points to filename (not inode)
ln -s file.txt symlink_file   # Create symbolic link
ln -s /path/to/file symlink   # Create symlink to different location
# - Works across filesystems
# - If original deleted, symlink breaks
# - Can symlink directories
# - Shows different inode number
# - Can detect and follow with: ls -L (follow links)

ls -l                         # 'l' at start indicates symlink, '->' shows target
readlink symlink_file         # Show what symlink points to
find . -type l                # Find all symlinks

# Common symlink patterns
ln -s /usr/bin/python3 /usr/bin/python  # Version abstraction
ln -s /var/log/app/current /var/log/app_latest  # Log rotation pattern
```

## 3.2 File Systems: ext4, XFS, Btrfs

### Choosing the Right Filesystem

| Aspect | ext4 | XFS | Btrfs |
|--------|------|-----|-------|
| **Best For** | General purpose, default choice | Large files, high throughput | Advanced features, flexibility |
| **Max File Size** | 16 TiB | 9 exabytes | 16 exabytes |
| **Max Filesystem Size** | 1 exabyte | 9 exabytes | 16 exabytes |
| **Inode Allocation** | Fixed | Dynamic | Dynamic |
| **Journaling** | Yes (ordered) | Yes (metadata) | Copy-on-write |
| **Snapshots** | No | No | Yes (native) |
| **Compression** | No | No | Yes (native) |
| **RAID** | No (external) | No (external) | Yes (native) |
| **Performance (small files)** | Good | Fair | Slower |
| **Performance (large files)** | Good | Excellent | Good |
| **Stability** | Excellent | Excellent | Good (still maturing) |
| **Recommended Use** | Servers, general systems | HPC, databases, media | Development, specialized needs |

### Creating & Managing Filesystems
```bash
# Create filesystem
sudo mkfs.ext4 /dev/sdX1     # Create ext4 on partition
sudo mkfs.ext4 -L "data" /dev/sdX1  # With label
sudo mkfs.xfs /dev/sdX1       # Create XFS
sudo mkfs.btrfs /dev/sdX1     # Create Btrfs

# Mounting
sudo mount /dev/sdX1 /mnt/data     # Mount filesystem
sudo mount -o remount,ro /mnt/data # Remount as read-only
mount | grep ext4                  # Show mounted filesystems
mount -t ext4 -O ^has_journal /    # Show ext4 without journal

# UUID (Universal Unique Identifier)
sudo blkid                    # Show block device info and UUIDs
lsblk -o NAME,UUID           # Show device UUIDs

# /etc/fstab entries for permanent mounting:
# UUID=abc123... /mnt/data ext4 defaults,nofail 0 2
# /dev/sdX1 /mnt/data ext4 defaults,nofail 0 2
# Arguments: device, mountpoint, type, options, dump, fsck

# Unmounting
sudo umount /mnt/data             # Unmount filesystem
sudo umount -l /mnt/data          # Lazy unmount (unmount when in use)
lsof /mnt/data                    # Check what's using mountpoint
```

### Filesystem Optimization
```bash
# ext4 Tuning
sudo tune2fs -c 40 /dev/sdX1           # Set max mounts before fsck (0=disabled)
sudo tune2fs -i 30d /dev/sdX1          # Set check interval to 30 days
sudo tune2fs -O ^has_journal /dev/sdX1 # Remove journaling (careful!)
sudo tune2fs -l /dev/sdX1              # List filesystem info
sudo e4defrag -v /dev/sdX1             # Defragment (rarely needed on ext4)

# XFS Tuning
sudo xfs_info /mnt/data                # Show XFS information
sudo xfs_db -x /dev/sdX1               # Interactive XFS debugger

# Btrfs Operations
sudo btrfs filesystem df /mnt/data      # Show Btrfs space usage
sudo btrfs filesystem show              # Show Btrfs filesystems
sudo btrfs subvolume create /mnt/sub1   # Create subvolume (like partition)
sudo btrfs subvolume snapshot /mnt/data /mnt/data-backup  # Create snapshot
sudo btrfs filesystem balance /mnt/data # Rebalance data across disks

# TRIM for SSDs (discard unused blocks)
sudo fstrim -v /                # Manual TRIM
# In /etc/fstab, add 'discard' to options for automatic TRIM:
# /dev/sdX1 / ext4 defaults,discard 0 1
```

## 3.3 Disk Management & Partitioning

### LVM (Logical Volume Manager) - Advanced Storage

LVM provides abstraction between physical disks and filesystems, allowing dynamic resizing.

```bash
# Physical Volumes (PV)
sudo pvcreate /dev/sdX1 /dev/sdY1          # Create PVs
sudo pvdisplay                             # Show PV information
sudo pvscan                                # Scan for PVs
sudo pvremove /dev/sdX1                    # Remove PV

# Volume Groups (VG)
sudo vgcreate myvg /dev/sdX1 /dev/sdY1    # Create VG from PVs
sudo vgdisplay                             # Show VG information
sudo vgextend myvg /dev/sdZ1               # Add PV to VG
sudo vgreduce myvg /dev/sdX1               # Remove PV from VG
sudo vgremove myvg                         # Delete VG

# Logical Volumes (LV)
sudo lvcreate -L 50G -n data myvg          # Create 50GB LV
sudo lvcreate -l 50%VG -n data myvg        # Create LV using 50% of VG space
sudo lvdisplay                             # Show LV information
sudo lvresize -L +10G /dev/myvg/data       # Expand LV by 10GB
sudo lvresize -L 100G /dev/myvg/data       # Set LV to 100GB
sudo lvremove /dev/myvg/data               # Delete LV
sudo lvs -o lv_name,lv_size,lv_status      # Custom output format

# Filesystem resize (must be done after LV resize)
sudo resize2fs /dev/myvg/data              # Resize ext4 (online safe)
sudo xfs_growfs /mnt/data                  # Resize XFS (online safe)
sudo btrfs filesystem resize max /mnt/data # Resize Btrfs

# Real-world scenario: Expand a full partition
sudo lvextend -L +50G /dev/myvg/root      # Expand LV
sudo resize2fs /dev/myvg/root              # Expand filesystem (online)
df -h                                      # Verify new size

# Snapshots (for backup, testing)
sudo lvcreate -L 10G -s -n backup /dev/myvg/data
# -s: snapshot flag
# Snapshot uses copy-on-write, takes space only for changes
sudo lvremove /dev/myvg/backup             # Remove snapshot
```

### RAID Configuration

```bash
# RAID 0 (Striping): Speed, no redundancy
# RAID 1 (Mirroring): Redundancy, 50% capacity loss
# RAID 5 (Striping with parity): Balance, 1 disk failure tolerance
# RAID 6 (Striping with dual parity): 2 disk failure tolerance
# RAID 10 (Mirrored stripes): Best performance + redundancy

# Using mdadm (software RAID)
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
# Create RAID1 array from 2 disks

sudo cat /proc/mdstat                      # Check RAID status
sudo mdadm --detail /dev/md0               # Detailed RAID info
sudo mdadm /dev/md0 --add /dev/sdc1       # Add disk to array
sudo mdadm /dev/md0 --remove /dev/sda1    # Remove failed disk

# Format RAID array
sudo mkfs.ext4 /dev/md0

# Monitor RAID
sudo watch -n 2 'cat /proc/mdstat'        # Watch RAID status
```

---

# PART 4: NETWORKING & SOCKETS

## 4.1 Network Configuration & Diagnostics

### IP & Interface Management
```bash
# View network interfaces
ip link show                   # All interfaces (layer 2)
ip addr show                   # All interfaces with IPs
ip -4 addr show                # Only IPv4
ip -6 addr show                # Only IPv6
ifconfig                       # Legacy tool (less used now)

# Configure IP addresses
sudo ip addr add 192.168.1.100/24 dev eth0     # Add IP temporarily
sudo ip addr del 192.168.1.100/24 dev eth0     # Remove IP
sudo ip link set eth0 up                        # Enable interface
sudo ip link set eth0 down                      # Disable interface
sudo ip link set eth0 name newname              # Rename interface

# Routes
ip route show                  # View routing table
sudo ip route add 10.0.0.0/8 via 192.168.1.1  # Add route
sudo ip route del 10.0.0.0/8                   # Delete route
sudo ip route add default via 192.168.1.1      # Set default gateway
route -n                       # Legacy routing view

# Persistent network configuration (/etc/netplan/01-netcfg.yaml on Ubuntu):
# version: 2
# ethernets:
#   eth0:
#     dhcp4: true
#   eth1:
#     addresses:
#       - 192.168.1.100/24
#     gateway4: 192.168.1.1
#     nameservers:
#       addresses: [8.8.8.8, 8.8.4.4]

sudo netplan apply             # Apply netplan changes
sudo systemctl restart networking  # Restart networking service
```

### DNS & Name Resolution
```bash
# Check DNS servers
cat /etc/resolv.conf           # Current DNS configuration
resolvectl status              # systemd-resolved status
systemd-resolve --status       # Detailed resolver status

# DNS queries
nslookup google.com           # Translate name to IP (older tool)
dig google.com                # Detailed DNS query
dig +short google.com         # Short format
dig @8.8.8.8 google.com       # Query specific DNS server
host google.com               # Simple lookup
host 8.8.8.8                  # Reverse lookup

# Configure DNS (/etc/netplan/01-netcfg.yaml or /etc/resolv.conf)
# nameserver 8.8.8.8
# nameserver 8.8.4.4

# Test DNS resolution
getent hosts google.com        # Show what system resolves to
```

## 4.2 Advanced Network Tools

### Socket Statistics (ss) - Modern Network Tool

```bash
# ss is the modern replacement for netstat (faster, more features)

# List all listening ports
ss -ltn                       # Listening TCP sockets (numeric)
ss -lun                       # Listening UDP sockets
ss -ltun                      # All listening TCP and UDP

# Show all established connections
ss -tn state established      # Established TCP connections
ss -tn state time-wait        # TIME_WAIT state connections
ss -tn state listen           # Listening sockets

# Show process information
sudo ss -ltnp                 # Include process PID and name
sudo ss -tnp | grep :3306    # Find MySQL process on port 3306

# Filter by source/destination
ss -tn src 192.168.1.100     # Connections from specific IP
ss -tn dst 8.8.8.8           # Connections to specific IP
ss -tn src :80 dst :5000     # From port 80 to port 5000

# Memory statistics
ss -s                        # Summary of socket statistics

# Unix domain sockets
ss -x                        # Show Unix domain sockets
ss -xp                       # With process information

# Real-time monitoring
watch -n 1 'ss -tn state established | wc -l'  # Count connections
watch 'ss -tn | grep ESTABLISHED'
```

### netstat (Legacy but Still Used)

```bash
# netstat is deprecated but still on many systems

netstat -tuln               # TCP, UDP, Listening, Numeric
netstat -tuln | grep :80    # Find service on port 80
sudo netstat -tulnp         # Include process information
netstat -s                  # Protocol statistics
netstat -an | grep ESTABLISHED | wc -l  # Count established connections

# Check for listening ports and who's using them
sudo lsof -i :8080          # Which process is using port 8080
sudo lsof -i :22            # Which process is using SSH port
sudo lsof -u username       # All files opened by user
sudo lsof -p 1234          # All files opened by process
```

### Packet Capture & Analysis

```bash
# tcpdump - packet sniffer
sudo tcpdump -i eth0                    # Capture packets on eth0
sudo tcpdump -i eth0 -w traffic.pcap    # Write to file
sudo tcpdump -r traffic.pcap            # Read from file
sudo tcpdump -i eth0 'tcp port 80'      # Only HTTP traffic
sudo tcpdump -i eth0 'src 192.168.1.100'  # From specific IP
sudo tcpdump -n -i eth0 -c 10          # Capture 10 packets, no DNS resolution

# wireshark - GUI packet analyzer (if GUI available)
sudo wireshark &            # Launch Wireshark
tshark -i eth0              # CLI version of Wireshark

# Monitor bandwidth usage
iftop -i eth0               # Real-time bandwidth per connection
nethogs -i eth0             # Bandwidth usage by process
bmon -i eth0                # Bandwidth monitor
```

## 4.3 TCP/IP Tuning for Data Pipelines

Critical for Kafka, Spark, and large data transfers:

```bash
# View current TCP parameters
sysctl -a | grep net.ipv4.tcp

# Key TCP tuning parameters
# /etc/sysctl.conf entries:

# Increase max connections
net.core.somaxconn = 4096           # Listen socket backlog
net.ipv4.tcp_max_syn_backlog = 4096 # SYN queue size

# Increase file descriptors
net.core.netdev_max_backlog = 5000

# TCP window size (important for high-bandwidth/high-latency networks)
net.ipv4.tcp_rmem = 4096 87380 67108864  # Min, default, max receive buffer
net.ipv4.tcp_wmem = 4096 65536 67108864  # Min, default, max send buffer

# TCP timeout settings
net.ipv4.tcp_fin_timeout = 15           # Wait time after FIN
net.ipv4.tcp_keepalive_time = 600       # Keepalive interval (seconds)
net.ipv4.tcp_keepalive_probes = 5       # Keepalive probes before drop
net.ipv4.tcp_keepalive_intvl = 15       # Keepalive probe interval

# Enable TCP fast open
net.ipv4.tcp_fastopen = 3               # Enable for both client and server

# BBR congestion control (better than Reno for high throughput)
net.ipv4.tcp_congestion_control = bbr

# Increase TIME_WAIT buckets
net.ipv4.tcp_max_tw_buckets = 2000000

# Apply changes
sudo sysctl -p /etc/sysctl.conf         # Apply sysctl changes
sudo sysctl -w net.ipv4.tcp_rmem="4096 87380 67108864"  # Immediate change

# Check if BBR is available
cat /proc/sys/net/ipv4/tcp_available_congestion_control

# For data pipelines connecting to Kafka/Spark
# Recommended settings:
# net.core.rmem_max = 134217728         # 128MB
# net.core.wmem_max = 134217728
# net.ipv4.tcp_rmem = 4096 131072 67108864
# net.ipv4.tcp_wmem = 4096 131072 67108864
```

---

# PART 5: PROCESS MANAGEMENT & DEBUGGING

## 5.1 Advanced Debugging with strace, ltrace, and gdb

### strace - System Call Tracer

strace traces all system calls made by a process—essential for debugging "Permission denied" and "File not found" errors.

```bash
# Basic tracing
strace ls -la                          # Trace list command
strace -p 1234                         # Attach to running process
strace -o trace.log ./myapp           # Save output to file
strace -e trace=open,read ./myapp     # Trace specific syscalls

# Trace categories
strace -e trace=file ./app            # File-related syscalls
strace -e trace=process ./app         # Process-related syscalls
strace -e trace=network ./app         # Network syscalls
strace -e trace=signal ./app          # Signal handling
strace -e trace=ipc ./app             # Inter-process communication

# Useful filters
strace -e openat,stat,access ./app    # Multiple specific calls
strace -e 'open*' ./app               # Wildcard (all open variants)
strace -c ./app                       # Count syscalls (summary)
strace -r ./app                       # Show relative time between calls
strace -T ./app                       # Show time spent in each syscall

# Follow child processes
strace -f ./app                       # Trace child processes
strace -f -e trace=file ./app         # Trace files in child processes

# Finding library issues
strace -e openat ./app 2>&1 | grep "libXXX"

# Debugging example: Why isn't my config file being read?
strace -e openat,stat ./myapp 2>&1 | grep config
# Shows exactly which paths the program is trying to access
```

### ltrace - Library Call Tracer

ltrace traces library calls (like strace but for shared libraries):

```bash
ltrace ./app                    # Trace library calls
ltrace -c ./app                 # Count library calls (summary)
ltrace -C ./app                 # Decode C++ symbols
ltrace -e malloc ./app          # Trace memory allocation
ltrace -p 1234                  # Attach to running process
ltrace -o trace.log ./app       # Save to file
ltrace -s 200 ./app             # Show 200 bytes of string args
```

### gdb - GNU Debugger

gdb is for interactive debugging with breakpoints, stepping, and memory inspection:

```bash
gdb ./myapp                          # Start debugging
gdb -p 1234                          # Attach to running process
gdb --args ./myapp arg1 arg2        # Pass command-line arguments

# GDB commands (inside gdb):
(gdb) run                           # Start program
(gdb) run arg1 arg2                 # Start with arguments
(gdb) break main                    # Set breakpoint at main function
(gdb) break file.c:42               # Set breakpoint at line 42 in file.c
(gdb) break myfunction              # Break at function
(gdb) info break                    # List all breakpoints
(gdb) delete 1                      # Delete breakpoint 1
(gdb) continue                      # Continue execution
(gdb) step                          # Execute next line (step into functions)
(gdb) next                          # Execute next line (skip functions)
(gdb) finish                        # Run until function returns
(gdb) print variable                # Print variable value
(gdb) print array[5]                # Print array element
(gdb) print *pointer                # Dereference pointer
(gdb) set variable = value          # Change variable value
(gdb) watch variable                # Stop when variable changes
(gdb) backtrace                     # Show call stack
(gdb) frame 2                       # Switch to frame 2
(gdb) info locals                   # Show local variables
(gdb) info args                     # Show function arguments
(gdb) disassemble main              # Show assembly code
(gdb) x/10x &variable               # Examine memory (10 hex values)
(gdb) quit                          # Exit GDB

# Debugging core dumps (post-mortem)
ulimit -c unlimited                 # Enable core dumps
./myapp                             # Crashes and creates core dump
gdb ./myapp core                    # Debug the core dump
(gdb) backtrace                     # See where it crashed
```

### valgrind - Memory Debugging

Detects memory leaks, buffer overflows, and memory errors:

```bash
valgrind ./myapp                    # Run with memory checking
valgrind --leak-check=full ./app    # Full leak detection
valgrind --leak-check=full --show-leak-kinds=all ./app  # All leak types
valgrind --log-file=valgrind.log ./app  # Save output to file
valgrind --tool=cachegrind ./app    # Cache profiling
valgrind --tool=callgrind ./app     # Call graph profiling
```

## 5.2 Performance Profiling

### perf - Linux Performance Analyzer

```bash
sudo perf record ./myapp            # Record performance data
sudo perf record -g ./myapp         # Record with call graph
sudo perf record -F 1000 ./myapp    # Sample at 1000 Hz
sudo perf report                    # Analyze recorded data
sudo perf stat ./myapp              # Show performance statistics
sudo perf stat -d ./myapp           # Detailed statistics
sudo perf top                       # Real-time performance monitoring
sudo perf list                      # List available events
```

---

# PART 6: SHELL SCRIPTING MASTERY

## 6.1 Variables, Arrays, and Functions

### Variables
```bash
#!/bin/bash

# Variable assignment (no spaces around =)
name="John"
age=30
path="/home/user"
PI=3.14159

# Variable expansion
echo "Name: $name"              # Simple expansion
echo "Age is ${age} years"      # Braced expansion (safer with concatenation)
echo "Path: $path"

# String operations
str="Hello World"
echo ${str:0:5}                 # Substring (Hello)
echo ${str#*o}                  # Remove prefix (o World)
echo ${str%o*}                  # Remove suffix (Hell)
echo ${str^^}                   # Convert to uppercase (HELLO WORLD)
echo ${str,,}                   # Convert to lowercase
echo ${#str}                    # Length of string

# Default values
echo ${undefined_var:-"default"}  # Use default if not set
echo ${undefined_var:="default"}  # Set and use default
echo ${undefined_var:+alternate}  # Use alternate if set

# Arithmetic
count=5
count=$((count + 1))            # Arithmetic expansion
count=$((count * 2))
result=$((10 / 3))              # Integer division
result=$((10 % 3))              # Modulo

# Declare special variable types
declare -i count=10             # Integer variable
declare -r PI=3.14              # Read-only variable
declare -x VAR="export"         # Export to environment
declare -a array=()             # Array variable
declare -A assoc=()             # Associative array

# Local variables in functions
function myfunc() {
    local var="local"           # Only visible in function
    global="global"             # Visible everywhere
}
```

### Arrays
```bash
#!/bin/bash

# Array declaration and assignment
arr=(apple banana cherry)
arr[3]="date"
arr[10]="fig"                   # Sparse array (11 elements, mostly empty)

# Indexed array operations
echo ${arr[0]}                  # First element
echo ${arr[-1]}                 # Last element (in bash 4.3+)
echo ${arr[@]}                  # All elements
echo ${arr[*]}                  # All elements (as single string)
echo ${#arr[@]}                 # Array length
echo ${!arr[@]}                 # Array indices

# Array slicing
echo ${arr[@]:1:2}              # Elements from index 1, length 2
echo ${arr[@]:2}                # From index 2 to end

# Modifying arrays
arr+=(grape)                    # Append element
arr[2]="blueberry"              # Replace element
unset arr[1]                    # Remove element

# Array iteration
for item in "${arr[@]}"; do
    echo "$item"
done

# Associative arrays (hash maps)
declare -A person
person[name]="Alice"
person[age]=25
person[city]="NYC"

echo ${person[name]}            # Access by key
echo ${person[@]}               # All values
echo ${!person[@]}              # All keys

for key in "${!person[@]}"; do
    echo "$key: ${person[$key]}"
done

# Useful array operations
# Remove duplicates from array
arr=(apple banana apple cherry banana)
arr=($(printf '%s\n' "${arr[@]}" | sort -u))

# Join array with delimiter
IFS=,
joined="${arr[*]}"              # Uses IFS
echo $joined                    # apple,banana,cherry

# Split string into array
IFS=: read -ra parts <<< "user:home:shell"
echo ${parts[0]}                # user
```

### Functions
```bash
#!/bin/bash

# Simple function
function greet() {
    echo "Hello, $1"            # $1 is first argument
}
greet "Alice"

# Alternative syntax (POSIX)
greet2() {
    echo "Hi, $1"
}
greet2 "Bob"

# Function with multiple parameters
add() {
    local a=$1
    local b=$2
    local sum=$((a + b))
    echo $sum                   # Return via echo
}
result=$(add 5 3)
echo "Result: $result"

# Return codes (exit status)
check_file() {
    [[ -f "$1" ]] && return 0 || return 1
}
if check_file "/etc/passwd"; then
    echo "File exists"
else
    echo "File not found"
fi

# Function returning multiple values
get_user_info() {
    echo "Alice" "engineer"     # Two outputs
}
read name job <<< $(get_user_info)
echo "Name: $name, Job: $job"

# Recursive function (factorial)
factorial() {
    local n=$1
    [[ $n -le 1 ]] && echo 1 || echo $((n * $(factorial $((n-1)))))
}
echo $(factorial 5)             # 120

# Function with default parameters
greet_with_default() {
    echo "Hello, ${1:-Stranger}"  # Use "Stranger" if no argument
}
greet_with_default
greet_with_default "Alice"
```

## 6.2 Control Flow & Error Handling

### Conditionals
```bash
#!/bin/bash

# If-else statements
if [[ $age -gt 18 ]]; then
    echo "Adult"
elif [[ $age -eq 18 ]]; then
    echo "Just turned 18"
else
    echo "Minor"
fi

# Conditional operators
[[ -f file ]]           # File exists
[[ -d dir ]]            # Directory exists
[[ -r file ]]           # Readable
[[ -w file ]]           # Writable
[[ -x file ]]           # Executable
[[ -z $var ]]           # String is empty
[[ -n $var ]]           # String is not empty
[[ $a -eq $b ]]         # Equal (numbers)
[[ $a -ne $b ]]         # Not equal
[[ $a -lt $b ]]         # Less than
[[ $a -le $b ]]         # Less or equal
[[ $a -gt $b ]]         # Greater than
[[ $a -ge $b ]]         # Greater or equal
[[ $str1 = $str2 ]]     # String equality
[[ $str1 != $str2 ]]    # String inequality
[[ $str1 =~ regex ]]    # Regex match

# Logical operators
[[ $a -gt 5 && $b -lt 10 ]]  # AND
[[ $a -eq 5 || $a -eq 10 ]]  # OR
[[ ! -f file ]]               # NOT

# Case statement
case $option in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    restart)
        echo "Restarting..."
        ;;
    *)
        echo "Unknown option"
        ;;
esac

# Ternary-like operator
result=$([[ $a -gt $b ]] && echo "a is bigger" || echo "b is bigger")
```

### Loops
```bash
#!/bin/bash

# For loop
for i in {1..5}; do
    echo "Iteration $i"
done

# For loop with C-style syntax
for ((i=0; i<5; i++)); do
    echo "i = $i"
done

# While loop
counter=0
while [[ $counter -lt 5 ]]; do
    echo "Count: $counter"
    ((counter++))
done

# Until loop (opposite of while)
counter=5
until [[ $counter -eq 0 ]]; do
    echo "Countdown: $counter"
    ((counter--))
done

# Loop over files
for file in *.log; do
    [[ -f "$file" ]] && echo "Processing $file"
done

# Break and continue
for i in {1..10}; do
    [[ $i -eq 3 ]] && continue    # Skip 3
    [[ $i -eq 7 ]] && break       # Exit at 7
    echo $i
done
```

### Error Handling
```bash
#!/bin/bash

set -e              # Exit on any error
set -u              # Error on undefined variable
set -o pipefail     # Pipeline fails if any command fails
set -x              # Print each command (debugging)

# Check command success
if ! command; then
    echo "Command failed"
    exit 1
fi

# Or-else pattern
cd /some/path || exit 1
rm -rf * || { echo "Failed to delete"; exit 1; }

# Error handling with trap
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT    # Run cleanup on script exit
trap cleanup INT     # Run cleanup on Ctrl+C

# Try-catch pattern (not native in bash)
try() {
    [[ $- = *e* ]]; SAVED_OPT_E=$?
    set +e
}

catch() {
    export exception_code=$?
    (( SAVED_OPT_E )) && set +e
    return $exception_code
}

try
(
    false || throw 1
)
catch || {
    echo "Caught error: $exception_code"
}

# Function to exit with error
die() {
    echo "ERROR: $*" >&2
    exit 1
}

# Check if file exists
[[ -f config.file ]] || die "config.file not found"
```

## 6.3 Advanced Scripting Patterns

### Reading and Parsing Files
```bash
#!/bin/bash

# Read file line by line (important: preserves spaces)
while IFS= read -r line; do
    echo "Line: $line"
done < /path/to/file

# Read file with pattern filtering
while IFS= read -r line; do
    [[ $line =~ ^# ]] && continue  # Skip comments
    [[ -z $line ]] && continue      # Skip empty lines
    echo "Processing: $line"
done < config.txt

# Parse CSV file
while IFS=',' read -r id name age; do
    echo "ID: $id, Name: $name, Age: $age"
done < data.csv

# Read command output
while IFS= read -r line; do
    echo "$line"
done < <(ps aux)                    # Process substitution

# Parse /etc/passwd
while IFS=: read -r user pass uid gid comment home shell; do
    [[ $uid -ge 1000 ]] && echo "User: $user"
done < /etc/passwd

# Reading into array
mapfile -t lines < /path/to/file   # Read all lines into array
for line in "${lines[@]}"; do
    echo "$line"
done
```

### Text Processing Utilities
```bash
#!/bin/bash

# Use sed for advanced replacements
sed 's/old/new/' file.txt          # First occurrence per line
sed 's/old/new/g' file.txt         # All occurrences
sed -i.bak 's/old/new/g' file.txt  # In-place with backup
sed -n '5,10p' file.txt            # Print lines 5-10
sed '/pattern/d' file.txt          # Delete matching lines

# Use awk for data extraction
awk -F: '{print $1, $3}' /etc/passwd     # Print user and UID
awk '{sum+=$1} END {print sum}' numbers.txt  # Sum first column
awk 'NR>1 {print $0}' file.txt     # Skip first line
awk 'BEGIN {OFS=","} {print $1, $2}' file.txt  # Change delimiter

# Use grep for searching
grep -r "pattern" /path            # Recursive search
grep -i "pattern" file.txt         # Case-insensitive
grep -v "pattern" file.txt         # Invert match (exclude)
grep -c "pattern" file.txt         # Count matches
grep -l "pattern" *.txt            # List files with matches
grep -n "pattern" file.txt         # Show line numbers

# Combine tools (pipeline)
cat config.txt | grep "^database" | cut -d= -f2 | tr -d ' '
# Get config values: find lines starting with "database", extract value after =, remove spaces
```

### Real-World Script Examples
```bash
#!/bin/bash
# Backup script with error handling and logging

set -e
BACKUP_DIR="/mnt/backup"
SOURCE_DIR="/home/user/data"
LOG_FILE="/var/log/backup.log"
DATE=$(date '+%Y-%m-%d_%H-%M-%S')

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

cleanup() {
    [[ -f /tmp/backup_lock ]] && rm /tmp/backup_lock
    log "Backup process ended"
}
trap cleanup EXIT

# Check if already running
[[ -f /tmp/backup_lock ]] && { log "Backup already running"; exit 1; }
touch /tmp/backup_lock

log "Starting backup of $SOURCE_DIR"

# Create backup
if tar -czf "${BACKUP_DIR}/backup_${DATE}.tar.gz" "$SOURCE_DIR" 2>&1 | tee -a "$LOG_FILE"; then
    log "Backup successful"
    # Keep only last 7 days of backups
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
    log "Old backups cleaned up"
else
    log "Backup failed!"
    exit 1
fi
```

---

# PART 7: PERFORMANCE TUNING & OPTIMIZATION

## 7.1 System Monitoring Tools

### Essential Monitoring Commands

```bash
# Quick overview
uptime                          # Load average and uptime
free -h                         # Memory usage
df -h                          # Disk usage
top                            # Real-time process monitor

# Detailed monitoring
htop                           # Modern top with colors and tree view
atop                           # Includes I/O and network stats
dstat                          # Versatile stats tool

# Load and CPU
w                              # Logged-in users and load
mpstat -P ALL 1               # Per-CPU stats, updated every 1 second
lscpu                         # CPU information

# Memory
cat /proc/meminfo              # Detailed memory info
free -t                        # Show total row
vmstat 1 5                     # 5 samples, 1 second apart
# Output columns: processes, memory, swap, I/O, system, CPU

# Disk I/O
iostat -x 1                   # Extended I/O stats
iotop -o                      # I/O per process
iotop -a                      # Accumulated I/O
blktrace -d /dev/sda          # Detailed block I/O tracing

# Network
iftop -i eth0                 # Bandwidth per connection
nethogs                       # Bandwidth per process
bmon -i eth0                  # Bandwidth monitor
ss -s                         # Socket statistics

# Combined real-time dashboard
nmon                          # IBM's performance monitor
sysstat tools                 # sar, sadc, sa1, sa2

# Collecting stats over time (for trending)
sar -u 1 5                    # CPU usage, 5 samples every 1 sec
sar -B                        # Memory paging statistics
sar -d                        # Disk I/O statistics
sar -n DEV                    # Network interface statistics
```

## 7.2 CPU Optimization

### Understanding CPU Metrics

```bash
# CPU states from top/htop
# us (user): User process time
# sy (system): Kernel time
# ni (nice): Low-priority user process time
# id (idle): CPU idle time
# wa (wait): I/O wait time - CRITICAL: high wa indicates I/O bottleneck
# hi (hard irq): Hardware interrupt time
# si (soft irq): Software interrupt time
# st (steal): Time stolen by hypervisor

# CPU frequency scaling
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# governors: performance, powersave, ondemand, conservative, schedutil

# Set CPU governor
echo "performance" | sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# For all CPUs:
for i in {0..7}; do
    echo "performance" | sudo tee /sys/devices/system/cpu/cpu$i/cpufreq/scaling_governor
done

# CPU frequency
cat /proc/cpuinfo | grep MHz
cpupower frequency-info
sudo cpupower frequency-set -g performance

# Isolate CPUs (for real-time workloads)
# In /etc/default/grub:
# GRUB_CMDLINE_LINUX="isolcpus=1,3,5"
# Then update-grub and reboot

# Thread affinity (pin process to CPU)
taskset -cp 0-2 $$            # Pin current shell to cores 0-2
taskset -p 0,2 1234           # Pin process 1234 to cores 0,2
# In /etc/security/limits.conf for persistent affinity

# Context switch optimization
cat /proc/sched_debug | grep context_switches
sysctl kernel.sched_migration_cost_ns  # Lower = more migration
```

## 7.3 Memory Optimization

### Understanding Memory

```bash
# Memory breakdown
cat /proc/meminfo

# Key metrics:
# MemTotal: Total usable RAM
# MemFree: Completely free RAM
# MemAvailable: Free + Reclaimable (better indicator of available memory)
# Buffers: Cache for I/O
# Cached: Page cache (filesystem cache)
# SwapTotal/SwapFree: Virtual memory
# Slab: Kernel memory

# Swappiness (0-100, 200 in some kernels)
sysctl vm.swappiness                # View current value
sudo sysctl -w vm.swappiness=10     # Lower favors RAM, higher favors swap
# Permanent: add to /etc/sysctl.conf

# Memory pressure
cat /proc/pressure/memory            # PSI (Pressure Stall Information)
# some: percentage of time processes stalled due to memory
# full: percentage of time ALL processes stalled

# Cache pressure
sysctl vm.vfs_cache_pressure        # 100 = default, <100 = keep more cache, >100 = clear faster

# Drop caches (carefully - can cause performance dip)
sync                                # Flush dirty pages first!
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'
# 1 = drop page cache
# 2 = drop slab objects
# 3 = drop both

# Large pages (huge pages)
cat /proc/meminfo | grep Huge
sudo sysctl -w vm.nr_hugepages=100  # Allocate 100x 2MB pages

# Memory limits per process
ulimit -v unlimited                 # Set process memory limit
# Or in /etc/security/limits.conf:
# username hard as 2097152

# Monitor memory by process
ps aux --sort=-%mem | head -20      # Top 20 memory consumers
pmap -x 1234                        # Memory map of process 1234
```

## 7.4 Disk I/O Optimization

### I/O Scheduling

```bash
# Current I/O scheduler
cat /sys/block/sda/queue/scheduler
# Options: noop, deadline, cfq, bfq, kyber (newer kernels)

# Change I/O scheduler
# For SSD: noop or kyber
# For HDD with workload mixing: deadline or bfq
# For single workload: deadline

echo "deadline" | sudo tee /sys/block/sda/queue/scheduler

# Persistent I/O scheduler (in /etc/default/grub):
# GRUB_CMDLINE_LINUX="elevator=deadline"

# I/O stats
iostat -x 1 5                       # Extended stats
# r/s: reads per second
# w/s: writes per second
# rkB/s: read throughput
# wkB/s: write throughput
# svctm: average service time (for operations)
# %util: utilization percentage

# Find slow I/O operations
sudo latency-histogram.sh           # Part of perf-tools

# Disk write coalescing
# In /etc/sysctl.conf:
vm.dirty_ratio = 40                 # Percentage of RAM before sync
vm.dirty_background_ratio = 10      # Start async writeback
vm.dirty_writeback_centisecs = 500  # Writeback interval (centiseconds)
```

## 7.5 Network Optimization for Data Pipelines

```bash
# Already covered in Part 4: TCP/IP Tuning
# Key parameters for Spark, Kafka:

# Increase socket buffers for data transfer
sudo sysctl -w net.core.rmem_max=134217728
sudo sysctl -w net.core.wmem_max=134217728
sudo sysctl -w net.ipv4.tcp_rmem="4096 131072 67108864"
sudo sysctl -w net.ipv4.tcp_wmem="4096 131072 67108864"

# Connection backlog (for many simultaneous connections)
sudo sysctl -w net.core.somaxconn=4096
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=4096

# Congestion control (BBR better for high-throughput WAN)
sudo sysctl -w net.ipv4.tcp_congestion_control=bbr

# Disable TCP timestamps (on local networks, saves CPU)
sudo sysctl -w net.ipv4.tcp_timestamps=0

# Tune NUMA for network (on multi-socket systems)
sudo numactl --localalloc -- ./myapp  # Pin to local memory
```

---

# PART 8: SECURITY HARDENING

## 8.1 SSH Security & Key Management

### SSH Configuration

```bash
# SSH config file locations
~/.ssh/config              # User-specific SSH config
/etc/ssh/ssh_config        # Global SSH client config
/etc/ssh/sshd_config       # SSH server config (requires restart)

# Generate SSH keys (run on client)
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519  # Newer, smaller

# Copy public key to server (initial setup)
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
# Or manually: cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# Client-side SSH optimization (~/.ssh/config):
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 5
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%h-%p-%r
    ControlPersist 600

Host myserver
    HostName 203.0.113.1
    User ubuntu
    IdentityFile ~/.ssh/myserver_rsa
    Port 2222

# Server hardening (/etc/ssh/sshd_config):
Port 2222                           # Non-standard port
PermitRootLogin no                  # Never allow root login
PasswordAuthentication no           # Keys only
PubkeyAuthentication yes            # Enable key auth
X11Forwarding no                    # Disable X11
PrintMotd no                        # Don't show MOTD
AcceptEnv LANG LC_*                 # Restrict environment
PermitTunnel no                     # Disable tunneling
MaxAuthTries 3                      # Limit login attempts
MaxSessions 10                      # Limit concurrent sessions
Protocol 2                          # Only SSH v2

# After modifying sshd_config:
sudo sshd -t                        # Test configuration
sudo systemctl restart sshd         # Apply changes

# SSH key management
ssh-add ~/.ssh/id_rsa              # Add key to SSH agent
ssh-add -l                         # List keys in agent
ssh-agent bash                     # Start new shell with agent
ssh-keygen -p -f ~/.ssh/id_rsa    # Change key passphrase
ssh-keygen -R hostname             # Remove host from known_hosts
```

## 8.2 Firewall Configuration (iptables/nftables)

### iptables (Traditional)

```bash
# View current rules
sudo iptables -L                    # List all rules
sudo iptables -L -n                 # Numeric (no DNS lookup)
sudo iptables -L -v                 # Verbose
sudo iptables -L INPUT -n           # Just INPUT chain
sudo iptables -S                    # Show rules in saved format

# Basic filtering
sudo iptables -A INPUT -i lo -j ACCEPT         # Accept loopback
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # Allow SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT  # Allow HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT # Allow HTTPS
sudo iptables -A INPUT -p tcp --dport 3306 -j DROP  # Block MySQL from outside

# Default policies
sudo iptables -P INPUT DROP        # Drop all incoming by default
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT      # Allow all outgoing

# Advanced rules
sudo iptables -A INPUT -p tcp -m tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
# Rate limiting: allow max 100 burst, then 25/minute

sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT  # Allow subnet
sudo iptables -A INPUT -s 10.0.0.1 -j DROP          # Block specific IP

# Delete rules
sudo iptables -D INPUT 3           # Delete rule 3 from INPUT chain
sudo iptables -F                   # Flush all rules
sudo iptables -X                   # Delete custom chains

# Persist iptables rules
sudo apt-get install iptables-persistent
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6
```

### ufw (Uncomplicated Firewall - Recommended for Beginners)

```bash
# Simple firewall management
sudo ufw status                     # Show firewall status
sudo ufw enable                     # Enable firewall
sudo ufw disable                    # Disable firewall

# Allow/Deny
sudo ufw allow 22/tcp               # Allow SSH
sudo ufw allow http                 # Allow HTTP (port 80)
sudo ufw allow https                # Allow HTTPS (port 443)
sudo ufw allow 3306/tcp             # Allow MySQL from anywhere (DANGEROUS)
sudo ufw allow from 192.168.1.0/24 to any port 22  # Allow SSH from subnet
sudo ufw deny 25/tcp                # Deny SMTP

# Reload and reset
sudo ufw reload                     # Apply changes
sudo ufw reset                      # Reset to defaults

# Default policies
sudo ufw default deny incoming      # Deny all incoming by default
sudo ufw default allow outgoing     # Allow all outgoing by default
```

## 8.3 SELinux & AppArmor (MAC Systems)

### SELinux (Red Hat-based systems)

```bash
# Check SELinux status
getenforce                          # Show current mode
setstatus                           # Detailed status
sestatus -v                         # With file context listing

# SELinux modes
# Enforcing: Blocks policy violations
# Permissive: Logs violations but allows them
# Disabled: SELinux off (requires reboot)

# Switch modes (temporary - reverts on reboot)
setenforce 0                        # Switch to Permissive
setenforce 1                        # Switch to Enforcing

# Permanent mode change (in /etc/selinux/config)
SELINUX=enforcing                   # or permissive/disabled
# Requires reboot to take effect

# Check policy
getsebool -a                        # List all SELinux booleans
getsebool httpd_can_network_connect # Check specific boolean
sudo setsebool -P httpd_can_network_connect on  # Persistent change

# File contexts
ls -Z                               # Show context
ls -Z /var/www/html                 # Check web content context

# Fix file contexts
sudo restorecon -v /var/www/html    # Restore default context
sudo restorecon -R /var/www/html    # Recursive restore

# Change file context
sudo chcon -t httpd_sys_content_t /var/www/html/file.html
sudo chcon -R -t httpd_sys_content_t /var/www/html/

# Generate custom policies
sudo audit2allow -l                 # Show denials
sudo audit2allow -a -M custommodule # Create policy module
```

### AppArmor (Debian/Ubuntu)

```bash
# Check AppArmor status
aa-status                           # Show loaded profiles
sudo systemctl status apparmor      # Service status

# Profile modes
# enforce: Block violations
# complain: Log violations but allow

# Switch profile mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx

# Edit profile
sudo vi /etc/apparmor.d/usr.sbin.nginx

# Reload profiles after editing
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx

# Generate profile for new application
sudo aa-genprof /usr/bin/myapp     # Interactive profile generation
sudo aa-complain /etc/apparmor.d/usr.bin.myapp  # Start in complain mode
sudo aa-logprof                    # Review denied actions
# After reviewing, move to enforce mode

# Disable profile temporarily
sudo ln -s /etc/apparmor.d/profile /etc/apparmor.d/disable/

# View denials in logs
sudo tail -f /var/log/audit/audit.log | grep apparmor
```

---

# PART 9: SYSTEM MONITORING & LOGGING

## 9.1 Journalctl & Systemd Logging

### journalctl Commands

```bash
# Basic viewing
journalctl                          # Show all journal entries (oldest first)
journalctl -f                       # Follow logs in real-time (tail -f style)
journalctl -n 50                    # Show last 50 entries
journalctl -p err                   # Show errors and above (crit, alert, emerg)
journalctl -p warning               # Warnings and above

# Filter by service
journalctl -u sshd                  # SSH service logs
journalctl -u sshd -f               # Follow SSH logs
journalctl -u sshd -n 100           # Last 100 SSH entries
journalctl -u sshd --since "1 hour ago"

# Filter by time
journalctl --since "2025-01-06"
journalctl --since "2025-01-06 10:00:00"
journalctl --since "1 hour ago"
journalctl --since "2 hours ago" --until "1 hour ago"
journalctl --today                  # Today's logs
journalctl --yesterday              # Yesterday's logs

# Filter by priority
journalctl -p debug                 # Debug level
journalctl -p notice                # Notice and above
journalctl -p crit                  # Critical, alert, emergency only

# Filter by user/process
journalctl _UID=1000                # Logs from UID 1000
journalctl _PID=1234                # Logs from process 1234
journalctl _COMM=apache2            # Logs from apache2 process

# Multiple filters (logical AND)
journalctl -u sshd -p err -n 20     # SSH errors, last 20 entries
journalctl --since "1 hour ago" -u networking
journalctl --since "1 hour ago" -u sshd -u networking  # Two services

# Output formats
journalctl -o json                  # JSON format (for parsing)
journalctl -o short                 # Short format (default)
journalctl -o full                  # Full format with all fields
journalctl -o cat                   # Only message text

# Performance analysis
journalctl -u mysqld -n 1000 | grep "took\|duration"  # Find slow queries
journalctl --disk-usage              # Journal disk usage
journalctl --vacuum-size 500M       # Limit journal to 500MB
journalctl --vacuum-time 30d        # Keep only last 30 days

# Search in logs
journalctl | grep "error"           # Simple grep search
journalctl -x                       # Show related entries
journalctl -S 2025-01-01 -U 2025-02-01 | grep "reboot"
```

## 9.2 Rsyslog Configuration

Traditional syslog daemon for centralized logging:

```bash
# Config file location
/etc/rsyslog.conf                   # Main configuration
/etc/rsyslog.d/                     # Config fragments

# Example rsyslog.conf entries:
# Selector: facility.priority
# Facilities: auth, authpriv, cron, daemon, kern, lpr, mail, mark,
#            news, syslog, user, uucp, local0-local7
# Priorities: debug, info, notice, warning, err, crit, alert, emerg

# Send all auth logs to separate file
auth.*   /var/log/auth.log

# Send all errors to admin
*.err    @admin.example.com          # Remote syslog over UDP

# Send all cron jobs to file
cron.*   /var/log/cron.log

# Stop processing after matching
auth.*   /var/log/auth.log
&  ~                                # Discard auth logs (don't pass further)

# Remote logging
*.* @@syslog.server.com             # TCP (reliable)
*.* @syslog.server.com:514          # UDP (fast)

# Filter by program
:programname, isequal, "nginx"
*.* /var/log/nginx-syslog.log

# Reload rsyslog after changes
sudo systemctl restart rsyslog
sudo systemctl reload rsyslog

# Monitor rsyslog
sudo tail -f /var/log/syslog
sudo tail -f /var/log/auth.log
```

---

# PART 10: CONTAINERIZATION & KERNEL

## 10.1 Linux Namespaces & cgroups (Container Foundations)

```bash
# Namespaces: Isolate system resources
# 1. PID namespace: Process IDs isolated
# 2. Network namespace: Network stack isolated
# 3. Mount namespace: Filesystem isolated
# 4. UTS namespace: Hostname/domainname isolated
# 5. IPC namespace: IPC resources isolated
# 6. User namespace: User/group IDs mapped
# 7. Cgroup namespace: Control group hierarchy

# Create new network namespace
sudo ip netns add mynetns
sudo ip netns list
sudo ip netns exec mynetns ip addr  # Execute command in namespace
sudo ip netns delete mynetns

# View process namespaces
ls -la /proc/1/ns/                  # PID 1 namespaces
cat /proc/1/ns/pid                  # Show PID namespace info

# cgroups: Limit/monitor resources
# CPU, memory, block I/O, network

# Check cgroup version
ls /sys/fs/cgroup/cgroup.controllers # If exists, cgroup v2

# cgroup v1 resources
/sys/fs/cgroup/cpu/                 # CPU control
/sys/fs/cgroup/memory/              # Memory control
/sys/fs/cgroup/blkio/               # Block I/O control
/sys/fs/cgroup/net_cls/             # Network classification

# cgroup v2 unified hierarchy
/sys/fs/cgroup/                     # All resources

# Create cgroup (cgroup v1)
sudo cgcreate -g cpu,memory,cpuacct:/mygroup
sudo cgset -r cpu.shares=512 /mygroup   # 50% CPU
sudo cgset -r memory.limit_in_bytes=1073741824 /mygroup  # 1GB

# Run process in cgroup
sudo cgexec -g cpu,memory:/mygroup /path/to/process

# Monitor cgroup (systemd)
systemd-cgtop                       # Top for cgroups
```

## 10.2 Kernel Tuning & Boot Parameters

### Kernel Boot Parameters

Parameters passed to kernel during boot via GRUB:

```bash
# View current boot parameters
cat /proc/cmdline

# Common parameters:
# debug         - Enable debug output
# quiet         - Suppress most messages
# splash        - Show graphical splash
# nomodeset     - Disable kernel mode setting (for GPU issues)
# acpi=off      - Disable ACPI
# mem=512M      - Override RAM size

# Performance-related parameters:
# elevator=deadline              - I/O scheduler
# transparent_hugepage=madvise   - Huge pages
# intel_idle.max_cstate=1        - Disable CPU sleep states (low latency)
# processor.max_cstate=1         - Disable C-states
# nohz_full=1,3,5,7              - Isolate CPUs from tick
# rcu_nocbs=1,3,5,7              - Isolate CPUs from RCU

# Modify GRUB (temporary - single boot)
# Reboot, interrupt GRUB menu (ESC or SHIFT at boot)
# Press 'e' to edit boot entry
# Append parameters to 'linux' line
# Press Ctrl+X to boot

# Modify GRUB permanently
# /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=deadline"
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"

# Update GRUB after changes
sudo update-grub                    # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/CentOS
```

### Kernel Parameters (/etc/sysctl.conf)

```bash
# Apply changes
sudo sysctl -p /etc/sysctl.conf
sudo sysctl -w parameter=value      # Temporary change

# Process/task tuning
kernel.sched_migration_cost_ns = 5000000  # Context switch cost
kernel.sched_min_granularity_ns = 10000000
kernel.sched_wakeup_granularity_ns = 15000000
kernel.sched_latency_ns = 20000000
kernel.sched_rt_period_us = 1000000
kernel.sched_rt_runtime_us = 950000

# Core dumps
kernel.core_pattern = /tmp/core.%e.%p  # Core dump location
fs.suid_dumpable = 0                    # Don't dump SUID processes

# IPC (Inter-process communication)
kernel.shmmax = 68719476736             # Shared memory max (64GB)
kernel.shmall = 16777216                # Shared memory pages
kernel.sem = 250 32000 32 128           # Semaphores

# File system
fs.file-max = 2097152                   # Max open files system-wide
fs.inode-max = 1048576                  # Max inodes
fs.dentry-state = 10,45,45,1024,8192,42949673  # Dentry cache

# Panic behavior
kernel.panic = 10                       # Reboot after 10 sec panic
kernel.panic_on_oops = 1                # Panic on oops (kernel error)

# Networking
net.ipv4.ip_forward = 1                 # Enable routing
net.bridge.bridge-nf-call-iptables = 1  # Bridge firewall
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 1         # Reverse path filtering
```

---

# PART 11: REAL-WORLD TRICKS & OPTIMIZATION PATTERNS

## 11.1 One-Liners & Productivity Hacks

```bash
# Find largest files in directory
find /path -type f -exec ls -lh {} \; | sort -k5 -h | tail -10

# Find files modified in last N days
find /path -type f -mtime -7

# Delete files older than 30 days
find /path -type f -mtime +30 -delete

# Count lines of code in project
find . -name "*.py" -o -name "*.java" | xargs wc -l | tail -1

# Find and replace in multiple files
find . -name "*.py" -exec sed -i 's/old_text/new_text/g' {} \;

# Watch a file for changes (with tail)
watch 'tail -20 logfile.log'

# Monitor disk usage over time
watch 'df -h | grep /home'

# Find which process is using most CPU
ps aux --sort=-%cpu | head -2

# Find which process is using most memory
ps aux --sort=-%mem | head -2

# Kill all processes matching pattern
pkill -f "pattern_in_process_name"

# Monitor network connections in real-time
watch 'ss state established | wc -l'

# Find files with specific permissions
find . -perm 777

# Check if port is open (localhost)
nc -zv localhost 8080

# Check if port is open (remote)
nc -zv example.com 443

# Benchmark disk speed
dd if=/dev/zero of=test.img bs=1G count=1 oflag=dsync

# Compress and send directory
tar czf - /path/to/dir | ssh user@host 'tar xzf - -C /dest'

# Parallel execution
parallel -j 4 process_file {} ::: file1 file2 file3 file4

# Generate SSH key with no passphrase
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519

# Test SSH connection without login
ssh -o ConnectTimeout=5 user@host "exit" && echo "OK" || echo "Failed"

# Append to multiple files
echo "text" | tee -a file1 file2 file3

# Get your public IP
curl ifconfig.me

# Simple HTTP server (Python)
python3 -m http.server 8000

# Simple HTTP server (Ruby)
ruby -run -ehttpd . -p8000

# Quick API test
curl -H "Content-Type: application/json" -X POST -d '{"key":"value"}' http://localhost:8000/api
```

## 11.2 Advanced Debugging Patterns

```bash
# Debug a running process without stopping it
sudo strace -p $(pgrep -f myapp) 2>&1 | head -100

# Find what a process is trying to read/write
sudo strace -e openat -p 1234 2>&1 | grep -v "ENOENT"

# Profile a process
sudo perf record -p 1234 sleep 10 && perf report

# Find memory leaks in C/C++ program
valgrind --leak-check=full --show-leak-kinds=all ./myapp

# Capture network traffic to file then analyze
sudo tcpdump -i eth0 -w traffic.pcap 'tcp port 80'
wireshark traffic.pcap

# Debug systemd service
systemctl status myservice
journalctl -u myservice -f
sudo systemctl status myservice --verbose

# Check why service won't start
sudo systemd-analyze verify myservice

# Debug SSH connection
ssh -vvv user@host           # Verbose debug

# Check what binary a script is using
which python
ls -la $(which python)       # Show if symlink

# Trace library loading
LD_DEBUG=files ./myapp 2>&1 | head -50

# Find dependencies of binary
ldd ./myapp
objdump -p ./myapp | grep NEEDED

# Examine system calls during specific timeframe
sudo strace -c -p 1234 sleep 5 && wait
```

## 11.3 Data Engineering Optimizations

```bash
# For Spark/Hadoop distributed processing:

# Monitor network for data transfer
iftop -i eth0
watch -n 1 'ss -s | grep TCP'

# Monitor disk I/O (important for shuffle)
iotop -o
iostat -x 1

# Set up local SSD for Spark shuffle
# In spark-defaults.conf or submission:
spark.local.dir=/mnt/fast-ssd/spark-shuffle

# Monitor executor memory and GC
# Add to spark-env.sh:
export SPARK_EXECUTOR_MEMORY_OPTIONS="-Xmx4g -Xms4g -XX:+UseG1GC -XX:InitiatingHeapOccupancyPercent=35"

# Optimize for data pipelines:
# 1. Check network saturation
iftop -i eth0
ss state established | wc -l

# 2. Check I/O bottleneck
iostat -x 1 | awk '{print $4, $5}'  # Read/write operations per sec

# 3. Check memory pressure
cat /proc/pressure/memory

# 4. Tune system for continuous loads
# In /etc/sysctl.conf:
net.core.rmem_max=134217728
net.ipv4.tcp_rmem="4096 131072 67108864"
vm.swappiness=5
vm.dirty_ratio=30
vm.dirty_background_ratio=5

# 5. Monitor with custom script
while true; do
    echo "=== $(date) ==="
    free -h | grep Mem
    iostat -x 1 1 | tail -2
    ss -s | grep TCP
    sleep 5
done

# 6. Log performance for analysis
{
    vmstat 1 100000 &
    iostat -x 1 100000 &
    mpstat -P ALL 1 100000 &
    wait
} > performance_data.txt 2>&1 &
```

## 11.4 Disaster Recovery & Backup

```bash
# Full system backup (with compression)
sudo tar --one-file-system -czf /mnt/backup/system_$(date +%Y%m%d).tar.gz \
  --exclude=/proc \
  --exclude=/sys \
  --exclude=/tmp \
  --exclude=/mnt \
  --exclude=/media \
  /

# Backup specific directories
tar -czf backup_$(date +%Y%m%d_%H%M%S).tar.gz /home /etc /var/lib

# Incremental backup (only changes since last full backup)
tar -czf incremental_$(date +%Y%m%d).tar.gz \
  -N "2025-01-01" \
  /home /etc /var

# Backup with progress
pv -tpreb /dev/sda1 | tar czf backup.tar.gz -

# SSH-based backup (compress on remote)
tar -czf - /important/data | ssh user@backup.server 'cat > backup.tar.gz'

# Restore from backup
tar -xzf backup.tar.gz -C /restore/location

# Verify backup integrity
tar -tzf backup.tar.gz > /dev/null && echo "OK" || echo "Corrupt"

# Schedule daily backup with cron
# Add to crontab -e:
# 0 2 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1

# Database backup (MySQL)
mysqldump -u root -p --all-databases > db_backup.sql
mysqldump -u root -p --single-transaction --quick db_name table_name > table_backup.sql

# PostgreSQL backup
pg_dump dbname > dbname.sql
pg_dump -U postgres --all-databases > all_databases.sql

# Snapshot backup (for filesystems supporting it)
sudo lvcreate -L 10G -s -n backup_snapshot /dev/vg0/data
mount /dev/vg0/backup_snapshot /mnt/snapshot
tar -czf backup.tar.gz /mnt/snapshot
umount /mnt/snapshot
sudo lvremove /dev/vg0/backup_snapshot
```

---

## CONCLUSION & CONTINUED LEARNING

This guide covers Linux from basics to senior-level optimization. Key areas to master:

1. **Practice**: Use these commands daily
2. **Reference**: Bookmark man pages (man command)
3. **Experiment**: Set up VMs and test configurations
4. **Monitor**: Always monitor before and after changes
5. **Automate**: Write scripts for repetitive tasks
6. **Document**: Keep notes on what works in your environment

## Additional Resources You Might Need

- `man page_name` - Built-in documentation
- `info command` - More detailed info than man
- `command --help` - Quick help
- `/usr/share/doc/` - System documentation
- `tldr command` - Community-driven simplified manpages (install with apt/yum)

## Advanced Topics to Explore Next

- Kubernetes administration
- Custom kernel compilation
- Real-time Linux (PREEMPT_RT)
- eBPF (extended Berkeley Packet Filter)
- System call interception and modification
- Advanced storage: Ceph, NVMe optimization
- Network virtualization
- Systemd deep-dive
- OOM killer prevention and tuning

Remember: **Always test changes in non-production environments first, especially at the kernel and network level.**

---

**This guide is now complete and self-contained. You will never need to refer to another resource for basic-to-senior Linux knowledge.**