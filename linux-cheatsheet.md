# Linux Command Cheat Sheet: Quick Reference

## File & Directory Operations

```bash
# Navigation
pwd                          # Print working directory
cd /path                     # Change directory
cd ~                        # Go to home
cd ..                       # Go up one level
cd -                        # Go to previous directory

# Listing
ls                          # List files
ls -l                       # Long format (permissions, owner, size, date)
ls -a                       # Show hidden files (start with .)
ls -lh                      # Long format with human-readable sizes
ls -R                       # Recursive listing
ls -lhS                     # Sort by size

# Creating/Removing
mkdir dir                   # Create directory
mkdir -p a/b/c              # Create nested directories
touch file.txt              # Create empty file / update timestamp
rm file                     # Remove file
rm -r dir                   # Remove directory and contents
rm -rf dir                  # Force remove (dangerous!)

# Copying & Moving
cp src dest                 # Copy file
cp -r dir1 dir2             # Copy directory recursively
mv old new                  # Move/rename file
mv dir1 dir2/               # Move directory

# File Info
file filename               # Determine file type
du -sh /path                # Disk usage of directory
wc -l file                  # Count lines
stat file                   # Detailed file information
```

---

## Text Processing & Viewing

```bash
# Viewing
cat file                    # Print entire file
less file                   # Scrollable viewer (q=quit, /=search)
more file                   # Older pager
head -n 10 file             # First 10 lines
tail -n 10 file             # Last 10 lines
tail -f file                # Follow live (logs)

# Searching & Filtering
grep pattern file           # Search for pattern in file
grep -i pattern file        # Case-insensitive search
grep -v pattern file        # Invert (show non-matching)
grep -n pattern file        # Show line numbers
grep -r pattern dir         # Recursive search in directory
find dir -name "*.log"      # Find files by name
find dir -type f -size +1M  # Find files larger than 1MB

# Text Manipulation
cut -d',' -f2 file          # Extract column 2 (CSV, delimiter=comma)
cut -c1-5 file              # Extract characters 1-5
sort file                   # Sort lines
sort -rn file               # Reverse numeric sort
sort -t',' -k2 file         # Sort CSV by column 2
uniq file                   # Remove adjacent duplicates
uniq -c file                # Count occurrences
sed 's/old/new/g' file      # Replace all old with new
sed -n '5,10p' file         # Print lines 5-10
awk -F',' '{print $2}' file # Print column 2 (CSV)

# Combining (pipes)
cat file | grep error       # Find lines with "error"
tail -n +2 file | sort -rn  # Sort all but first line
ps aux | grep python        # Find python processes
journalctl -u service | tail -n 50  # Last 50 log lines
```

---

## curl: HTTP Client

```bash
# Basic
curl https://example.com                    # Fetch URL (print to terminal)

# Output/Input Options
curl -o file.html https://example.com       # Save to file (-o)
curl -O https://example.com/file.zip        # Save with remote filename
curl -m 10 https://example.com              # Timeout after 10 seconds (-m)
curl -s https://example.com                 # Silent (no progress) (-s)
curl -S https://example.com                 # Silent but show errors (-S)
curl -sS https://example.com                # Combine both

# Redirection & Headers
curl -L https://example.com                 # Follow redirects (-L)
curl -I https://example.com                 # Headers only (-I)
curl -H "Accept: application/json" https://api.example.com   # Add header (-H)

# HTTP Methods & Data
curl -X POST https://api.example.com        # Use POST (-X)
curl -d "param=value" https://api.example.com               # Send data (-d)
curl -d '{"key":"value"}' \
     -H "Content-Type: application/json" \
     https://api.example.com                # POST JSON

# Authentication
curl -u username:password https://api.example.com           # Basic auth (-u)
curl -H "Authorization: Bearer TOKEN" https://api.example.com

# Combinations
curl -o data.csv -m 30 -sS -L https://example.com/data.csv
```

---

## apt: Package Management (Debian/Ubuntu)

```bash
# Requires sudo
sudo apt update              # Update package lists
sudo apt upgrade             # Upgrade all packages
sudo apt install package     # Install package
sudo apt install p1 p2 p3    # Install multiple
sudo apt remove package      # Remove package (keep config)
sudo apt purge package       # Remove package and config
sudo apt autoremove          # Remove unused dependencies
sudo apt clean               # Clear package cache

# Searching & Info
apt search pattern           # Search for package
apt show package             # Show package details
apt list --installed         # List installed packages
apt list --upgradable        # Show packages to upgrade
```

---

## I/O Redirection & Pipes

```bash
# Redirection
command > file               # Write stdout to file (overwrite)
command >> file              # Append stdout to file
command 2> errors.log        # Redirect stderr to file
command > out.txt 2>&1       # Redirect both to file
command < input.txt          # Read stdin from file

# Pipes
command1 | command2          # Pipe stdout of cmd1 to stdin of cmd2
command1 | command2 | command3  # Chain multiple

# Splitting
command | tee file           # Print to terminal AND save to file
command | tee -a file        # Append to file instead

# Examples
curl https://example.com | grep title         # Pipe curl to grep
ps aux | grep python                          # Find python processes
tail -n +2 data.csv | sort -rn | head -n 10  # Complex pipeline
cat file | tee output.txt | less              # View and save simultaneously
```

---

## Permissions & Ownership

```bash
# View permissions
ls -l file                   # Show current permissions

# Change permissions (octal)
chmod 644 file               # rw- r-- r-- (regular file)
chmod 755 file               # rwx r-x r-x (executable)
chmod 600 file               # rw- --- --- (private)
chmod 777 file               # rwx rwx rwx (all permissions)

# Change permissions (symbolic)
chmod +x file                # Add execute permission
chmod -w file                # Remove write permission
chmod u+rw,g-x,o-r file      # Complex: user+rw, group-x, other-r

# Change ownership
chown user:group file        # Change owner and group
sudo chown user file         # Change owner (needs sudo)
chown -R owner dir           # Recursive change

# Common patterns
chmod 644 *.txt              # All txt files: rw-r--r--
chmod 755 *.sh               # All scripts: rwxr-xr-x
sudo chmod 600 ~/.ssh/config # Private SSH config
```

---

## Process Management

```bash
# View Processes
ps                           # Processes in current shell
ps aux                       # All processes (detailed)
ps -u username               # Processes of specific user
ps -p 1234                   # Info about PID 1234
top                          # Interactive process monitor
htop                         # Better version of top

# Running Processes
command &                    # Run in background
jobs                         # List background jobs
fg                           # Bring last background job to foreground
bg                           # Resume suspended job in background
Ctrl+Z                       # Suspend current job
Ctrl+C                       # Terminate current job

# Managing Processes
kill PID                     # Send SIGTERM (default)
kill -9 PID                  # Force kill SIGKILL
killall name                 # Kill by process name
wait PID                     # Wait for process to finish

# Special Variables
$$                           # Current script/shell PID
$!                           # PID of last background process
$?                           # Exit code of last command (0=success)
```

---

## Bash Scripting Basics

```bash
#!/bin/bash                  # Shebang (must be first line)

# Variables
VAR="value"                  # Set variable (no spaces!)
echo $VAR                    # Use variable
echo ${VAR}                  # Same (safer)
VAR="new"                    # Reassign

# Command substitution
RESULT=$(date)               # Run command, capture output
RESULT=`date`                # Older syntax (still works)

# Quoting
echo "$VAR"                  # Variable expansion
echo '$VAR'                  # Literal dollar sign
echo "result: $VAR"          # Mix text and variables

# Conditionals
if [ $? -eq 0 ]; then
    echo "Success"
fi

if [ -f file.txt ]; then
    echo "File exists"
fi

[ $? -eq 0 ] && echo "Success" || echo "Failed"  # One-liner

# Loops
for i in 1 2 3; do
    echo $i
done

while [ condition ]; do
    # Do something
done

# Functions
my_function() {
    echo "Argument 1: $1"
    echo "Argument 2: $2"
    return 0
}

my_function arg1 arg2        # Call function

# String operations
VAR="hello world"
echo ${VAR:0:5}              # First 5 chars: "hello"
echo ${VAR#hello }           # Remove prefix: "world"
echo ${VAR%world}            # Remove suffix: "hello "
```

---

## System Information

```bash
# System
uname -a                     # System information
hostname                     # Machine name
whoami                       # Current user
date                         # Current date/time
uptime                       # System uptime

# Disk & Memory
df -h                        # Disk usage
du -sh /path                 # Directory size
free -h                      # Memory usage
ps aux | head                # Top processes

# Network
ping google.com              # Test connectivity
curl https://example.com     # Fetch webpage
wget https://example.com/file.zip  # Download file
ss -an                       # Network sockets
netstat -an                  # Network statistics
```

---

## Common Flags Across Tools

| Flag | Common Meaning | Example |
|------|---|---|
| `-h` | Help/Human-readable | `ls -h`, `du -h` |
| `-v` | Verbose | `cp -v` |
| `-q` | Quiet | `wget -q` |
| `-f` | Force | `rm -f`, `curl -f` |
| `-r` | Recursive | `cp -r`, `grep -r` |
| `-n` | Number/Numeric | `head -n 10` |
| `-a` | All/Append | `ls -a`, `curl -a` |
| `-l` | Long format | `ls -l` |
| `-s` | Silent | `curl -s` |
| `-o` | Output | `curl -o file` |
| `-u` | User/Username | `curl -u user:pass` |
| `-m` | Max/Minutes | `curl -m 30` |

---

## Useful Combinations

```bash
# Find and process large files
find . -type f -size +100M -exec ls -lh {} \;

# Count files by extension
find . -type f | cut -d. -f2 | sort | uniq -c | sort -rn

# Search and replace in multiple files
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;

# Download multiple files
for url in $(cat urls.txt); do
    curl -o "file_$(date +%s).txt" "$url"
done

# Monitor file changes live
while true; do
    clear
    date
    tail -n 20 logfile.log
    sleep 2
done

# Process CSV and save results
tail -n +2 data.csv | cut -d',' -f2,3 | sort -rn | head -n 10 > results.txt

# Backup with timestamp
cp -r important_dir important_dir_backup_$(date +%Y%m%d_%H%M%S)

# Find and delete old files
find /tmp -type f -mtime +7 -delete    # Delete files older than 7 days
```

---

## Exit Codes

```bash
# Every command returns an exit code
$?           # Check last exit code
0            # Success
1-255        # Various errors
```

---

## Tips & Tricks

```bash
# History
history              # Show command history
!<number>           # Run command by history number
!!                  # Run last command
!grep               # Run last command starting with "grep"
Ctrl+R              # Search history (reverse)

# Tab Completion
# Press Tab to autocomplete filenames, commands

# Man Pages
man command         # Open manual for command
man -k keyword      # Search manuals
man -K pattern      # Search content

# Keyboard Shortcuts
Ctrl+A              # Move to beginning of line
Ctrl+E              # Move to end of line
Ctrl+L              # Clear screen
Ctrl+U              # Clear line
Ctrl+C              # Cancel command
```

---

## Learning Resources

```bash
# Built-in help
command --help      # Quick help for most commands
man command         # Full manual page
info command        # GNU info pages (alternative to man)

# Online
# https://explainshell.com/ (paste command, get explanation)
# https://man.archlinux.org/ (man pages online)
```

