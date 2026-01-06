# LINUX COMMAND-LINE UTILITIES & TOOLS: COMPREHENSIVE GUIDE
## Complete Reference for wget, curl, sudo, and All Essential Tools

---

## TABLE OF CONTENTS

1. **DOWNLOAD & TRANSFER TOOLS** (wget, curl, scp, rsync)
2. **PRIVILEGE & USER MANAGEMENT** (sudo, su, visudo)
3. **TEXT PROCESSING TOOLS** (grep, sed, awk, cut, paste, sort, uniq)
4. **FILE SEARCHING & FILTERING** (find, locate, which, whereis)
5. **COMPRESSION & ARCHIVING** (tar, gzip, bzip2, xz, zip, unzip)
6. **FILE PERMISSIONS & OWNERSHIP** (chmod, chown, chgrp, umask)
7. **TASK SCHEDULING** (cron, crontab, at, systemd timers)
8. **SERVICE MANAGEMENT** (systemctl, service, systemd)
9. **ADVANCED UTILITIES** (xargs, parallel, jq, curl advanced)
10. **CONCEPTS & PATTERNS** (Combining tools, best practices)

---

# PART 1: DOWNLOAD & TRANSFER TOOLS

## 1.1 wget - Web Grabber

**Purpose**: Download files, recursive website downloads, resume interrupted downloads

### Basic Usage
```bash
wget https://example.com/file.zip                    # Download single file
wget -O custom_name.zip https://example.com/file.zip # Save with custom name
wget -c https://example.com/largefile.zip            # Resume interrupted download
wget --limit-rate=500k https://example.com/file.zip  # Limit download speed (500KB/s)
wget --tries=5 https://example.com/file.zip          # Retry 5 times if failed
```

### Recursive Download (Mirror Website)
```bash
# Download entire website
wget -r https://example.com                          # Recursive (default depth=5)
wget -r -l 10 https://example.com                    # Set maximum depth to 10
wget -r --no-parent https://example.com/folder/      # Don't go to parent directories
wget -r -A "*.pdf,*.doc" https://example.com        # Download only PDF and DOC files
wget -r -R "*.jpg,*.png" https://example.com        # Exclude JPG and PNG files
wget -k https://example.com/page.html               # Convert links to work locally (with -r)
```

### Authentication & Cookies
```bash
# Basic authentication
wget --user=username --password=password https://protected.com/file.zip

# With cookies
wget --save-cookies=cookies.txt --keep-session-cookies https://example.com
wget --load-cookies=cookies.txt https://example.com/file.zip

# HTTP headers
wget --user-agent="Mozilla/5.0" https://example.com/file.zip
wget --referer="https://referrer.com" https://example.com/file.zip
```

### Advanced Features
```bash
# Timeout and DNS timeout
wget --timeout=10 --dns-timeout=5 https://example.com/file.zip
# --timeout: Total timeout for all operations
# --dns-timeout: Timeout for DNS resolution
# --connect-timeout: Timeout for connection establishment
# --read-timeout: Timeout between data reads

# Background download
wget -b https://example.com/largefile.zip           # Run in background
tail -f wget-log                                     # Check progress

# FTP download
wget ftp://ftp.example.com/file.tar.gz             # Download via FTP

# Multi-file download from list
wget -i urls.txt                                    # Download all URLs in file

# Quota
wget -Q 100M https://example.com/files/             # Stop after downloading 100MB
```

## 1.2 curl - Data Transfer Tool

**Purpose**: HTTP requests, API testing, advanced header manipulation, multiple protocols

### Basic Usage
```bash
curl https://example.com                            # Simple GET request
curl -O https://example.com/file.zip                # Download file (keep original name)
curl -o custom_name.zip https://example.com/file.zip # Download with custom name
curl -C - https://example.com/largefile.zip         # Resume download (-C -)
curl --limit-rate 500k https://example.com/file.zip # Limit speed
curl --max-time 30 https://example.com              # Maximum 30 seconds total
curl --connect-timeout 10 https://example.com       # 10 second connection timeout
```

### HTTP Methods & Headers
```bash
# GET request (default)
curl https://example.com

# POST request with data
curl -X POST -d "param1=value1&param2=value2" https://api.example.com/endpoint
curl -X POST -d @data.json https://api.example.com/endpoint  # From file

# JSON POST (common for APIs)
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"key":"value"}' \
  https://api.example.com/endpoint

# Custom headers
curl -H "Authorization: Bearer token123" https://api.example.com/endpoint
curl -H "User-Agent: MyApp/1.0" https://example.com
curl -H "X-Custom-Header: value" https://example.com

# Multiple headers
curl -H "Auth: token" -H "Type: json" https://example.com

# Show response headers
curl -i https://example.com                         # Include headers in output
curl -I https://example.com                         # Show headers only (HEAD request)
curl -D headers.txt https://example.com             # Save headers to file
```

### Authentication
```bash
# Basic authentication
curl -u username:password https://example.com

# Bearer token (API key)
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com

# Digest authentication (more secure than basic)
curl --digest -u username:password https://example.com

# Certificate-based authentication
curl --cert client-cert.pem:password https://example.com
```

### Cookies & Sessions
```bash
# Save cookies
curl -c cookies.txt https://example.com             # Save cookies to file

# Use saved cookies
curl -b cookies.txt https://example.com             # Load cookies from file

# Cookie with name and value
curl -b "session=abc123" https://example.com

# Show cookies
curl -v https://example.com 2>&1 | grep "Cookie"
```

### Advanced Features
```bash
# Parallel requests (multiple downloads)
curl -Z https://example.com/file1.zip https://example.com/file2.zip https://example.com/file3.zip

# Follow redirects
curl -L https://example.com                         # Follow redirect chains

# Proxy support (HTTP, HTTPS, SOCKS)
curl -x proxy.example.com:8080 https://example.com
curl -x "http://proxy:8080" https://example.com
curl -x "socks5://proxy:1080" https://example.com

# User agent spoofing
curl -A "Mozilla/5.0" https://example.com

# Verbose output (debugging)
curl -v https://example.com                         # Verbose (show headers, response)
curl -vv https://example.com                        # Very verbose
curl -w "%{http_code}" https://example.com          # Show HTTP status code

# Save response to file
curl -o filename.html https://example.com
curl -O https://example.com/path/filename.html      # Keep original filename

# Silent mode
curl -s https://example.com                         # Suppress progress bar
curl -s -o /dev/null https://example.com            # Silent, no output

# Retry failed requests
curl --retry 5 --retry-delay 2 https://example.com
```

### Form Data & File Upload
```bash
# URL-encoded form data (application/x-www-form-urlencoded)
curl -X POST -d "field1=value1&field2=value2" https://api.example.com

# Form with file upload (multipart/form-data)
curl -F "file=@/path/to/file.txt" -F "description=My file" https://api.example.com/upload

# Multiple files
curl -F "file1=@file1.txt" -F "file2=@file2.txt" https://api.example.com/upload

# JSON body
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"John","age":30}' \
  https://api.example.com
```

## 1.3 curl vs wget Comparison

| Feature | curl | wget |
|---------|------|------|
| **Recursive downloads** | Limited (manual) | Native support with -r |
| **Parallel downloads** | Yes (with -Z) | No |
| **Protocol support** | HTTP, HTTPS, FTP, SFTP, SMTP, LDAP, more | HTTP, HTTPS, FTP |
| **API testing** | Excellent | Basic |
| **Custom headers** | Easy (-H) | Difficult |
| **Proxies** | HTTP, HTTPS, SOCKS4/5 | HTTP, HTTPS only |
| **Progress display** | Shows URL being downloaded | Simple progress bar |
| **Library support** | libcurl (widely used) | Limited |
| **Best for** | API testing, single requests | Website mirroring, automation |
| **Resume downloads** | -C - (manual) | Automatic with -c |

### When to Use
- **curl**: API testing, custom requests, header manipulation, scripting with granular control
- **wget**: Website mirroring, batch downloads, resume support, automation scripts

## 1.4 scp & rsync - Secure File Transfer

### scp (Secure Copy)
```bash
# Copy file to remote
scp /local/file user@remote.com:/remote/path/

# Copy file from remote
scp user@remote.com:/remote/file /local/path/

# Copy directory recursively
scp -r /local/dir user@remote.com:/remote/path/

# Use specific SSH key
scp -i ~/.ssh/custom_key /local/file user@remote.com:/remote/

# Specify port
scp -P 2222 /local/file user@remote.com:/remote/

# Preserve permissions and timestamps
scp -p /local/file user@remote.com:/remote/
```

### rsync (Advanced Sync)
```bash
# One-way sync to remote
rsync -av /local/dir/ user@remote.com:/remote/path/
# -a: archive mode (preserves permissions, timestamps)
# -v: verbose
# Trailing slash matters: /dir/ syncs contents, /dir syncs directory itself

# One-way sync from remote
rsync -av user@remote.com:/remote/dir/ /local/path/

# Bidirectional (not recommended for most cases)
rsync -av /local/dir/ user@remote.com:/remote/
rsync -av user@remote.com:/remote/dir/ /local/

# Dry run (show what would happen)
rsync -av --dry-run /local/dir/ user@remote.com:/remote/

# Delete files on destination that don't exist on source
rsync -av --delete /local/dir/ user@remote.com:/remote/

# Exclude patterns
rsync -av --exclude='*.log' --exclude='tmp' /local/ user@remote.com:/remote/

# With bandwidth limit
rsync -av --bwlimit=1000 /local/dir/ user@remote.com:/remote/

# Compression during transfer
rsync -avz /local/dir/ user@remote.com:/remote/

# Using specific SSH port
rsync -av -e "ssh -p 2222" /local/dir/ user@remote.com:/remote/
```

---

# PART 2: PRIVILEGE & USER MANAGEMENT

## 2.1 sudo - Substitute User DO

**Purpose**: Execute commands with root/elevated privileges while maintaining audit trail

### Basic Usage
```bash
sudo command                    # Run command as root
sudo -u username command        # Run as specific user
sudo -i                         # Login as root (interactive shell)
sudo -s                         # Shell as root (with current environment)
sudo -l                         # List current user's sudo privileges
sudo -v                         # Refresh sudo timestamp (extend timeout)
sudo -k                         # Clear sudo timestamp (require password again)
sudo !!                         # Re-run previous command with sudo
```

### Practical Examples
```bash
# System administration
sudo apt-get update             # Update package lists
sudo systemctl restart nginx    # Restart service
sudo usermod -aG sudo newuser   # Add user to sudo group
sudo visudo                     # Safely edit sudoers file

# File operations
sudo chown -R owner:group /path # Change ownership
sudo chmod 755 /usr/local/bin/script  # Change permissions
sudo nano /etc/config.conf      # Edit system files

# Network operations
sudo iptables -L               # List firewall rules
sudo ip addr add 192.168.1.100/24 dev eth0  # Configure IP

# Process management
sudo kill -9 1234              # Force kill process
sudo reboot                    # Reboot system
```

### sudoers Configuration

The `/etc/sudoers` file (edited with `visudo`) controls who can run what with sudo:

```bash
# Edit sudoers file (always use visudo!)
sudo visudo

# Examples in /etc/sudoers:

# Allow root to do anything
root ALL=(ALL:ALL) ALL

# Allow user 'alice' to run all commands without password
alice ALL=(ALL) NOPASSWD: ALL

# Allow user 'bob' to run specific command without password
bob ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Allow group 'sudo' to run all commands
%sudo ALL=(ALL:ALL) ALL

# Allow group 'admins' to run specific commands
%admins ALL=(ALL) NOPASSWD: /bin/systemctl *, /usr/sbin/service *

# Allow specific user on specific hosts only
john webserver1,webserver2=(ALL) NOPASSWD: /usr/bin/systemctl

# Aliases for cleaner configuration
User_Alias ADMINS = alice, bob
Cmnd_Alias SERVICES = /usr/bin/systemctl, /usr/sbin/service
Host_Alias WEBSERVERS = web1, web2, web3
ADMINS WEBSERVERS = (ALL) SERVICES

# Environment variables
Defaults env_keep = "HOME USER LOGNAME"
Defaults secure_path = "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"

# Logging all sudo commands
Defaults log_output, log_input
Defaults!/bin/ls !log_output
```

### Security Best Practices
```bash
# Check who can use sudo
sudo -l                                 # Your privileges
sudo -l -u otheruser                   # Other user's privileges (if you have rights)

# Audit sudo usage
sudo journalctl -u sudo                 # Check systemd logs for sudo usage
grep sudo /var/log/auth.log            # Traditional log file

# Restrict sudo to prevent privilege escalation
# In sudoers - dangerous: NOPASSWD on ALL
# Better: NOPASSWD on specific commands only

# Use sudo with environment preservation carefully
sudo -E command                         # Preserve user environment (RISKY)

# Run command without interactive shell (safer in scripts)
sudo /usr/bin/specific_binary          # Full path required in sudoers
```

## 2.2 su - Switch User

**Purpose**: Change user identity (older alternative to sudo)

### Usage
```bash
su - username                   # Switch to user (with login shell and environment)
su username                     # Switch to user (keep current environment)
su -                            # Switch to root with login shell
su -c "command"                 # Run single command as user

# Requires password of target user (major difference from sudo)
```

### Key Differences: su vs sudo

| Feature | su | sudo |
|---------|----|----|
| **Requires password** | Target user's password | Current user's password |
| **Privilege escalation** | Direct (becomes that user) | Controlled (runs specific commands) |
| **Audit trail** | Limited logging | Detailed logging |
| **Security** | Less secure (password sharing) | More secure (permission-based) |
| **Typical use** | Interactive shell as another user | Single command elevation |
| **Best practice** | Avoid for privilege escalation | Use instead of su |

---

# PART 3: TEXT PROCESSING TOOLS

## 3.1 grep - Pattern Matching

**Purpose**: Search text for patterns, filter lines

### Basic Usage
```bash
grep "pattern" file.txt                 # Simple search
grep "pattern" file1.txt file2.txt      # Search multiple files
grep -r "pattern" /directory/           # Recursive search in directory
grep -l "pattern" *.txt                 # List files containing pattern (not content)
grep -c "pattern" file.txt              # Count matching lines
grep -n "pattern" file.txt              # Show line numbers
```

### Pattern Matching Options
```bash
# Case-insensitive search
grep -i "Pattern" file.txt

# Invert match (show lines NOT matching)
grep -v "exclude_this" file.txt

# Match whole words only (not substrings)
grep -w "word" file.txt

# Extended regex (use | + ? { } [ ] for patterns)
grep -E "pattern1|pattern2" file.txt    # OR patterns
grep -E "^[0-9]{3}-[0-9]{4}$" file.txt # Match phone format

# Perl-compatible regex (more powerful)
grep -P "(?P<name>\w+):\s*(?P<value>.+)" file.txt

# Match multiple patterns (AND)
grep "pattern1" file.txt | grep "pattern2"

# Show context around matches
grep -B 2 -A 2 "pattern" file.txt       # 2 lines before and after
grep -B 5 "error" /var/log/syslog       # Show previous 5 lines
```

### Practical Examples
```bash
# Find failed login attempts
grep "Failed password" /var/log/auth.log

# Find configuration errors in logs
grep -i "error" /var/log/application.log | grep -v "Expected error"

# Count occurrences of pattern
grep -o "pattern" file.txt | wc -l      # Count total occurrences

# Find files with errors in logs directory
find /var/log -name "*.log" -exec grep -l "error" {} \;

# Show line numbers with matches
grep -n "TODO" *.py | grep -v "DONE"

# Extract specific information
grep "user:" /etc/passwd | cut -d: -f1  # Get usernames

# Recursive search with file type filtering
grep -r "TODO" --include="*.py" /src/   # Only .py files
grep -r "password" --exclude="*.pyc" /  # Exclude .pyc files
```

## 3.2 sed - Stream Editor

**Purpose**: Stream editing, find-and-replace, text transformation

### Substitution (Most Common)
```bash
# Replace first occurrence on each line
sed 's/old/new/' file.txt

# Replace all occurrences on each line (g = global)
sed 's/old/new/g' file.txt

# Replace on specific line number
sed '5s/old/new/' file.txt              # Line 5 only
sed '1,5s/old/new/' file.txt            # Lines 1-5

# Case-insensitive replacement
sed 's/old/new/gi' file.txt

# In-place file editing (with backup)
sed -i.bak 's/old/new/g' file.txt       # Creates file.txt.bak
sed -i 's/old/new/g' file.txt           # No backup (dangerous!)

# Multiple patterns
sed -e 's/old1/new1/g' -e 's/old2/new2/g' file.txt
sed 's/old1/new1/g; s/old2/new2/g' file.txt
```

### Deletion & Insertion
```bash
# Delete lines
sed '5d' file.txt                        # Delete line 5
sed '1,3d' file.txt                      # Delete lines 1-3
sed '/pattern/d' file.txt                # Delete matching lines
sed '/^$/d' file.txt                     # Delete empty lines
sed '$ d' file.txt                       # Delete last line

# Insert text before line
sed '2i\New line inserted' file.txt      # Insert before line 2
sed '/pattern/i\Insert before pattern' file.txt

# Append text after line
sed '2a\New line appended' file.txt      # Append after line 2
sed '/pattern/a\Append after pattern' file.txt

# Change entire line
sed '5c\New content for line 5' file.txt
```

### Advanced sed Features
```bash
# Print specific lines
sed -n '10,20p' file.txt                 # Print lines 10-20 (suppress default output)
sed -n '1p;$p' file.txt                  # Print first and last line

# Address ranges
sed '1,/pattern/d' file.txt              # Delete from line 1 to first match
sed '/start/,/end/s/old/new/g' file.txt # Replace only between start and end

# Hold space (advanced)
sed -n '1h; 1!H; $!d; x; s/\n/ /g; p' file.txt  # Join all lines

# Backreferences in replacement
sed 's/\(.*\) \(.*\)/\2 \1/g' file.txt  # Swap two fields

# Escape special characters
sed 's|/path/old|/path/new|g' file.txt  # Use | instead of / for paths

# Multiple files with in-place edit
sed -i 's/old/new/g' *.txt               # Edit all .txt files
```

## 3.3 awk - Powerful Text Processing

**Purpose**: Column-based data extraction, calculation, report generation

### Basic Usage
```bash
# Print specific fields (default separator: whitespace)
awk '{print $1}' file.txt                # Print first field
awk '{print $1, $3}' file.txt            # Print fields 1 and 3
awk '{print NF, $NF}' file.txt           # Print number of fields and last field

# With field separator
awk -F: '{print $1}' /etc/passwd         # Use : as separator
awk -F'[,:]' '{print $1}' file.txt       # Multiple separators

# Conditional printing
awk '$3 > 100' file.txt                  # Print lines where field 3 > 100
awk '$1 == "error"' file.txt             # Print lines where field 1 equals "error"
awk '$2 ~ /pattern/' file.txt            # Print lines where field 2 matches pattern

# Skip header row
awk 'NR > 1' file.txt                    # Skip first line
```

### BEGIN and END Blocks
```bash
# Initialize variables before processing
awk 'BEGIN {sum=0} {sum+=$1} END {print "Total:", sum}' numbers.txt

# Print header and footer
awk 'BEGIN {print "Name\tAge\tSalary"} 
     {print $1 "\t" $2 "\t" $3} 
     END {print "---"; print "Report complete"}' data.txt

# Calculate statistics
awk '{sum+=$1; count++} END {print "Average:", sum/count}' numbers.txt
```

### Built-in Variables
```bash
NR      # Number of Records (line number)
NF      # Number of Fields (number of columns)
FS      # Field Separator (default: whitespace)
OFS     # Output Field Separator (default: space)
RS      # Record Separator (default: newline)
FNR     # File Number Record (line number within current file)
FILENAME # Current filename being processed

# Example usage
awk 'NR==1 {print "File:", FILENAME; print NF " fields"}' file.txt
awk '{print NR ":", $0}' file.txt        # Add line numbers
```

### Practical Examples
```bash
# Parse /etc/passwd (UID >= 1000 = regular users)
awk -F: '$3 >= 1000 {print $1, $3, $6}' /etc/passwd

# Extract columns from CSV
awk -F, '{print $1, $3}' data.csv

# Calculate sums by group
awk -F: '{total[$1]+=$3} END {for (user in total) print user, total[user]}' data.txt

# Replace field and print
awk '{$2 = "REPLACED"; print}' file.txt

# Print lines longer than 80 characters
awk 'length > 80' file.txt

# Combine awk with other tools
ps aux | awk '$3 > 10 {print $1, $3}'    # Show processes using >10% CPU

# String functions in awk
awk '{print toupper($1)}' file.txt       # Convert to uppercase
awk '{print substr($1, 1, 5)}' file.txt  # First 5 characters
awk '{print length($1)}' file.txt        # Length of field

# Pattern matching with regex
awk '/^[A-Z]/ {print}' file.txt          # Lines starting with capital letter
awk '$1 !~ /^#/ {print}' file.txt        # Exclude comment lines
```

## 3.4 Other Text Processing Tools

### cut - Extract Columns
```bash
# Extract specific columns
cut -d: -f1 /etc/passwd                  # Field 1, delimiter :
cut -d, -f2,4 data.csv                   # Fields 2 and 4
cut -c1-5 file.txt                       # Characters 1-5 (byte positions)
cut -c1-5,10-15 file.txt                 # Multiple ranges
```

### paste - Merge Lines
```bash
# Combine files side-by-side
paste file1.txt file2.txt
paste -d, file1.txt file2.txt            # With comma delimiter
paste -d: file1.txt file2.txt file3.txt  # Three files
```

### sort - Sort Lines
```bash
# Basic sorting
sort file.txt                            # Alphabetical
sort -n file.txt                         # Numeric sort
sort -r file.txt                         # Reverse order
sort -k2 file.txt                        # Sort by column 2 (field separator: space)
sort -k2 -t, file.txt                    # Sort by column 2 (field separator: comma)
sort -u file.txt                         # Unique lines only (removes duplicates)
sort -k1,1 -k2,2n file.txt               # Primary sort: col1 (alpha), secondary: col2 (numeric)
```

### uniq - Remove Duplicates
```bash
# Remove duplicate consecutive lines
uniq file.txt

# Count duplicates
uniq -c file.txt

# Show only duplicates
uniq -d file.txt

# Show only unique lines (not duplicated)
uniq -u file.txt

# Case-insensitive duplicates
uniq -i file.txt

# Usually combined with sort first
sort file.txt | uniq
sort file.txt | uniq -c | sort -rn       # Count and sort by frequency
```

---

# PART 4: FILE SEARCHING & FILTERING

## 4.1 find - Comprehensive File Searching

**Purpose**: Find files by name, size, modification time, permissions, and execute actions

### Basic Searching
```bash
find /path -name "filename"               # Exact filename match
find /path -iname "filename"              # Case-insensitive search
find /path -name "*.txt"                  # Wildcard pattern
find /path -name "*.txt" -o -name "*.pdf" # Multiple patterns (OR)
find /path -not -name "*.log"             # Exclude pattern (NOT)
```

### File Type Filtering
```bash
find /path -type f                        # Regular files only
find /path -type d                        # Directories only
find /path -type l                        # Symbolic links only
find /path -type f,d                      # Files OR directories
find /path -type f -path "*/build/*" -prune -o -type f -print # Exclude directory
```

### Size Filtering
```bash
find /path -size +100M                    # Larger than 100MB
find /path -size -1M                      # Smaller than 1MB
find /path -size 100M                     # Exactly 100MB
find /path -size +100M -size -1G          # Between 100MB and 1GB
# Suffixes: c (bytes), k (KB), M (MB), G (GB)
```

### Time-Based Filtering
```bash
find /path -mtime -7                      # Modified within last 7 days
find /path -mtime +30                     # Modified more than 30 days ago
find /path -atime -1                      # Accessed within last 24 hours
find /path -ctime -3                      # Changed within last 3 days
find /path -mmin -60                      # Modified within last 60 minutes
find /path -newer file.txt                # Modified more recently than file.txt
```

### Permission & Ownership
```bash
find /path -perm 644                      # Exact permissions
find /path -perm -644                     # At least these permissions
find /path -perm /644                     # Any of these permission bits
find /path -user username                 # Files owned by user
find /path -group groupname               # Files owned by group
find /path -nouser                        # Files without owner (deleted users)
```

### Depth Control
```bash
find /path -maxdepth 2                    # Maximum 2 levels deep
find /path -mindepth 3                    # Start from level 3
find /path -maxdepth 3 -mindepth 2        # Between levels 2 and 3
find . -maxdepth 1 -type f                # Files in current directory only (not subdirs)
```

### Execute Actions
```bash
# Execute command on each file
find /path -name "*.txt" -exec cat {} \;  # {} = filename, \; = end command
find /path -name "*.txt" -exec rm {} \;   # Delete all .txt files

# Prompt before executing
find /path -name "*.txt" -ok rm {} \;     # Ask before each deletion

# Batch execution (more efficient)
find /path -name "*.txt" -exec cat {} +   # Append all files to single command

# Practical examples
find /tmp -type f -mtime +7 -exec rm {} \;  # Delete files older than 7 days
find . -name "*.pyc" -delete               # Delete .pyc files
find /var/log -name "*.log" -exec gzip {} \;  # Compress old logs

# With conditional execution
find /path -name "*.txt" -exec sh -c 'echo "Found: $1"' _ {} \;
```

### Complex Searching
```bash
# Combine conditions (AND by default)
find /path -type f -name "*.txt" -size +10M    # Large text files

# Logical OR
find /path \( -name "*.txt" -o -name "*.doc" \) -type f

# Logical NOT
find /path -type f -not -name "*.log"

# Exclude directories from search
find /path -name node_modules -prune -o -type f -print

# Real-world example: Find unmodified Python files in src
find ./src -name "*.py" -type f -mtime +365 -size -1M
```

## 4.2 locate - Fast File Searching

**Purpose**: Quick file search using pre-built database (faster than find, less comprehensive)

### Usage
```bash
locate filename                           # Search by filename
locate -i filename                        # Case-insensitive
locate "*.pdf"                            # Pattern search
locate -c filename                        # Count matches only
locate -e filename                        # Show only existing files
locate -S                                 # Database statistics

# Update database (runs periodically, can force)
sudo updatedb                             # Update locate database
```

### Characteristics
- **Fast**: Uses pre-built database
- **Limited**: Only searches filename (not content like grep)
- **Delayed**: Database may be outdated between updates

## 4.3 which & whereis - Find Commands

### which - Locate Command Executable
```bash
which python                              # Find Python executable
which -a python                           # All matches (multiple versions)
```

### whereis - Find Command, Source, and Man Pages
```bash
whereis python                            # Executable, source, and man page locations
whereis -b python                         # Executable location only
whereis -s python                         # Source files only
whereis -m python                         # Man pages only
```

---

# PART 5: COMPRESSION & ARCHIVING

## 5.1 tar - Tape Archive

**Purpose**: Create archives preserving file structure, permissions, and metadata

### Basic Operations
```bash
# Create archive (uncompressed)
tar -cvf archive.tar /path/to/files       # Create, verbose, file
tar -cf archive.tar file1 file2 dir1      # Shorter syntax

# Extract archive
tar -xvf archive.tar                      # Extract verbose
tar -xf archive.tar -C /destination/      # Extract to specific directory

# List contents (preview)
tar -tf archive.tar                       # List files in archive
tar -tvf archive.tar                      # List with details

# Append files (only on uncompressed tar)
tar -rf archive.tar new_file              # Add to existing archive
```

## 5.2 Compression with tar

### gzip Compression (Most Common)
```bash
# Create compressed archive
tar -czf archive.tar.gz /path/to/files    # Create, compress with gzip, verbose, file

# Extract
tar -xzf archive.tar.gz
tar -xzf archive.tar.gz -C /destination/

# List contents (preview before extracting)
tar -tzf archive.tar.gz

# Compression level (1-9, default 6)
tar -czf --compress-level=9 archive.tar.gz /path/
```

### bzip2 Compression (Better Compression, Slower)
```bash
tar -cjf archive.tar.bz2 /path/to/files
tar -xjf archive.tar.bz2
tar -tjf archive.tar.bz2
```

### xz Compression (Best Compression, Much Slower)
```bash
tar -cJf archive.tar.xz /path/to/files
tar -xJf archive.tar.xz
tar -tJf archive.tar.xz
```

### zstd Compression (Fast, Good Compression)
```bash
tar --zstd -cf archive.tar.zst /path/to/files
tar --zstd -xf archive.tar.zst
tar --zstd -tf archive.tar.zst
```

## 5.3 Advanced tar Features

### Selective Archiving
```bash
# Exclude patterns
tar -czf archive.tar.gz --exclude='*.log' --exclude='tmp' /path/

# Exclude multiple patterns
tar -czf archive.tar.gz --exclude='*.log' --exclude='*.tmp' --exclude='node_modules' /path/

# Only include certain files
tar -czf archive.tar.gz --include='*.py' -C /src .

# Exclude directory entirely
tar -czf archive.tar.gz --exclude=/path/to/exclude /path/
```

### Incremental Backups
```bash
# Full backup
tar -czf full_backup.tar.gz -g metadata.snar /data/

# Incremental (only changes since last snapshot)
tar -czf incremental_backup.tar.gz -g metadata.snar /data/
```

### Preserve Specific Attributes
```bash
tar -czf archive.tar.gz --preserve-permissions --same-owner /path/
# --preserve-permissions: Keep file permissions
# --same-owner: Preserve owner (important for system backups)
```

### Split Large Archives
```bash
# Create multi-volume archive
tar -czf - /large/dir | split -b 1G - backup.tar.gz.

# Reassemble
cat backup.tar.gz.* | tar -xzf -

# Or native tar multi-volume (older systems)
tar -czf backup.tar.gz.part%d -M /large/dir
```

## 5.4 zip & unzip - Cross-Platform Compression

### zip - Create ZIP Archives
```bash
# Single file
zip archive.zip file.txt

# Multiple files
zip archive.zip file1.txt file2.txt file3.txt

# Directory (recursive)
zip -r archive.zip /path/to/directory

# Compression level (0-9)
zip -9 archive.zip file.txt              # Maximum compression

# Exclude patterns
zip -r archive.zip /path -x "*.log" "*.tmp"

# Update existing archive
zip -u archive.zip newfile.txt

# Delete from archive
zip -d archive.zip oldfile.txt

# List contents
zip -l archive.zip

# Store only (no compression)
zip -0 archive.zip file.txt
```

### unzip - Extract ZIP Archives
```bash
# Basic extraction
unzip archive.zip

# Extract to directory
unzip archive.zip -d /destination/

# List contents (preview)
unzip -l archive.zip

# Extract specific files
unzip archive.zip file1.txt file2.txt

# Overwrite without prompting
unzip -o archive.zip

# Never overwrite (skip existing)
unzip -n archive.zip

# Test integrity
unzip -t archive.zip

# Quiet mode
unzip -q archive.zip

# Extract with verbose output
unzip -v archive.zip
```

## 5.5 Compression Comparison

| Format | Speed | Compression | File Size | Use Case |
|--------|-------|-------------|-----------|----------|
| **tar** | N/A | None | ~100% | Archiving only |
| **gzip** | Fast | Good | ~60% | Most common, balance |
| **bzip2** | Slow | Better | ~50% | High compression |
| **xz** | Very slow | Best | ~30% | Long-term storage |
| **zstd** | Very fast | Good | ~55% | Modern standard |
| **zip** | Fast | Good | ~60% | Windows compatibility |

---

# PART 6: FILE PERMISSIONS & OWNERSHIP

## 6.1 chmod - Change Permissions

**Purpose**: Control read, write, execute permissions for user, group, others

### Octal Notation (Most Common)
```bash
chmod 755 script.sh
# 7 (user) = rwx (4+2+1)
# 5 (group) = r-x (4+1)
# 5 (others) = r-x (4+1)

# Common permissions:
# 755: rwxr-xr-x (executable files, default for many)
# 644: rw-r--r-- (readable files, default for many)
# 700: rwx------ (private files, only owner)
# 777: rwxrwxrwx (dangerous - everyone can do everything)
# 600: rw------- (private files, only owner, no execute)

chmod 755 /usr/local/bin/myscript
chmod 644 /etc/config.conf
chmod 700 ~/.ssh                          # SSH directory must be 700
chmod 600 ~/.ssh/id_rsa                   # SSH private key must be 600
```

### Symbolic Notation (More Intuitive)
```bash
# u = user (owner), g = group, o = others, a = all

# Add execute for owner
chmod u+x script.sh

# Remove write for group and others
chmod go-w file.txt

# Set exact permissions
chmod u=rwx,g=rx,o= script.sh

# Remove all permissions
chmod 000 file.txt

# Make readable by all
chmod a+r file.txt

# Make executable by owner only
chmod u+x script.sh

# Examples
chmod u+x,g-w,o-rwx file              # Multiple changes at once
chmod a-x file.txt                     # Remove execute from all
chmod u+w file.txt                     # Add write permission for owner
```

### Recursive Changes
```bash
# Apply to directory and contents
chmod -R 755 /var/www/html/             # Recursively set 755
chmod -R u+w /home/user/                # Recursively add write permission
```

### Special Permission Bits

```bash
# setuid (4xxx): File runs with owner's permissions
chmod 4755 /usr/bin/sudo                # SUID + rwxr-xr-x
# When executed, runs with owner's privileges (dangerous for regular users)

# setgid (2xxx): File runs with group's permissions; new files inherit group
chmod 2755 /shared/dir/                 # SGID + rwxr-sr-x
# Useful for shared directories

# Sticky bit (1xxx): Only owner/root can delete files (for /tmp)
chmod 1777 /tmp                         # Sticky bit + rwxrwxrwx
# Prevents users from deleting others' files in /tmp
```

## 6.2 chown & chgrp - Change Ownership

### chown - Change Owner and Group
```bash
# Change owner only
chown newowner file.txt

# Change owner and group
chown newowner:newgroup file.txt

# Change group only (colon prefix)
chown :newgroup file.txt

# Recursive change
chown -R newowner:newgroup /path/to/dir

# Change based on reference file
chown --reference=file1.txt file2.txt    # Make file2.txt same owner as file1.txt

# Practical examples
sudo chown root:root /usr/local/bin/script  # Change to root ownership
sudo chown www-data:www-data /var/www/html  # Web server files
sudo chown -R user:user /home/user/.ssh     # SSH directory ownership
```

### chgrp - Change Group Only
```bash
# Change group
chgrp newgroup file.txt

# Recursive
chgrp -R projectgroup /project/files/

# Reference
chgrp --reference=file1.txt file2.txt
```

## 6.3 umask - Default Permissions

**Purpose**: Set default permissions for newly created files/directories

```bash
# View current umask
umask

# Example output: 0022
# 0 = special bits
# 022 means: new files get 644 (666-022), new dirs get 755 (777-022)

# Set umask for current session
umask 0077                                # Private: files 600, dirs 700
umask 0022                                # Default: files 644, dirs 755
umask 0027                                # Restricted: files 640, dirs 750

# Permanent umask (in ~/.bashrc or ~/.profile)
echo "umask 0077" >> ~/.bashrc

# System-wide umask (/etc/profile, /etc/bashrc)
echo "umask 0077" | sudo tee -a /etc/profile.d/umask.sh
```

---

# PART 7: TASK SCHEDULING

## 7.1 cron & crontab - Scheduled Jobs

**Purpose**: Run commands/scripts automatically at specified times

### crontab Syntax
```bash
# Crontab format:
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 7) (0 or 7 is Sunday)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * <command>

# Examples:
0 0 * * *     # Midnight every day
0 0 * * 0     # Midnight every Sunday
0 0 1 * *     # Midnight on 1st of each month
0 12 * * 1-5  # Noon on weekdays (Mon-Fri)
*/15 * * * *   # Every 15 minutes
0 */2 * * *    # Every 2 hours
0 0 * * *      # Daily
@daily         # Daily at midnight
@weekly        # Weekly on Sunday
@monthly       # First day of month
@yearly        # January 1st
@reboot        # System startup
```

### Managing crontab
```bash
# Edit crontab
crontab -e                                # Edit current user's crontab

# As specific user (requires sudo)
sudo crontab -u username -e

# List crontab entries
crontab -l                                # List current user's jobs
sudo crontab -u username -l               # List specific user's jobs

# Delete crontab
crontab -r                                # Remove all jobs
crontab -r -u username                    # Remove specific user's jobs

# Edit /etc/crontab (system-wide, needs sudo)
sudo nano /etc/crontab
```

### Crontab Examples
```bash
# Backup every night at 2 AM
0 2 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1

# Run every 30 minutes
*/30 * * * * /usr/local/bin/check_disk.sh

# Daily at 3 AM on weekdays
0 3 * * 1-5 /usr/local/bin/maintenance.sh

# First day of month at noon
0 12 1 * * /usr/local/bin/monthly_report.sh

# Every 15 minutes during business hours (9 AM - 5 PM)
*/15 9-17 * * 1-5 /usr/local/bin/check_queue.sh

# Multiple commands
0 0 * * * /home/user/backup.sh && /home/user/notify.sh

# Redirect output and errors
0 2 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1

# With environment variables
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
0 0 * * * /usr/local/bin/script.sh
```

### Cron Job Logging
```bash
# Check cron logs
sudo tail -f /var/log/syslog | grep CRON

# Or with journalctl (systemd)
sudo journalctl -u cron -f

# Check if job ran
sudo grep CRON /var/log/syslog | grep script.sh

# Email cron output
# Cron automatically emails STDOUT/STDERR to user unless redirected

# Disable email notifications
0 0 * * * /script.sh > /dev/null 2>&1    # Discard output
```

### Special Considerations
```bash
# Use full paths in cron commands
0 0 * * * /usr/local/bin/backup.sh        # ✓ Good
0 0 * * * backup.sh                       # ✗ Bad (PATH not searched same)

# Set HOME variable if needed
SHELL=/bin/bash
HOME=/home/user
0 0 * * * /script/using/HOME.sh

# Avoid commands that need stdin
# Use: command < /dev/null
# Or: command --non-interactive

# Never redirect to /dev/null without 2>&1 if you want to see errors
0 0 * * * /backup.sh > /dev/null 2>&1    # Discard all
0 0 * * * /backup.sh >> /var/log/backup.log 2>&1  # Save all
```

## 7.2 at - One-Time Scheduled Job

**Purpose**: Schedule command to run once at specific time (not recurring)

```bash
# Schedule job at specific time
at 3:30 PM                                # Interactive, enter command, Ctrl+D when done
echo "command" | at 3:30 PM               # Non-interactive
at 3:30 PM tomorrow                       # Tomorrow at 3:30 PM
at 3:30 PM + 2 days                       # In 2 days

# List scheduled jobs
atq
at -l

# Remove job
atrm 1                                    # Remove job ID 1

# Show job details
at -c 1                                   # Show commands for job 1
```

## 7.3 systemd Timers - Modern Alternative to cron

**Purpose**: Use systemd units for scheduling (more integrated with system)

```bash
# Create timer unit (/etc/systemd/system/myapp.timer)
[Unit]
Description=Run MyApp Daily
After=network.target

[Timer]
OnBootSec=1min
OnUnitActiveSec=1d
Unit=myapp.service

[Install]
WantedBy=timers.target

# Create corresponding service (/etc/systemd/system/myapp.service)
[Unit]
Description=MyApp Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/myapp
User=user
Group=user

# Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable myapp.timer
sudo systemctl start myapp.timer

# Check timer status
sudo systemctl list-timers
sudo systemctl status myapp.timer

# View timer logs
sudo journalctl -u myapp.service -f
```

---

# PART 8: SERVICE MANAGEMENT

## 8.1 systemctl - Service Control

**Purpose**: Start, stop, enable/disable services managed by systemd

### Basic Service Control
```bash
# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration (without restarting, if supported)
sudo systemctl reload nginx

# Reload or restart (reload if supported, else restart)
sudo systemctl reload-or-restart nginx

# Check status
sudo systemctl status nginx
sudo systemctl is-active nginx             # Just status: active/inactive
sudo systemctl is-enabled nginx            # Just status: enabled/disabled
```

### Enable/Disable Services
```bash
# Enable (start on boot)
sudo systemctl enable nginx

# Disable (don't start on boot)
sudo systemctl disable nginx

# Enable and start immediately
sudo systemctl enable --now nginx

# Check if enabled
sudo systemctl is-enabled nginx
```

### Viewing Services
```bash
# List all services
systemctl list-units --type=service

# List all services with status
systemctl list-units --type=service --all

# List enabled services
systemctl list-unit-files | grep enabled

# Show service dependencies
systemctl list-dependencies nginx

# Show service file location
systemctl cat nginx                       # Show nginx.service content
```

### Advanced systemctl Commands
```bash
# Show detailed service information
systemctl show nginx

# Set property temporarily
sudo systemctl set-property nginx CPUAccounting=yes

# Mask service (prevent starting)
sudo systemctl mask nginx                 # Service cannot be started
sudo systemctl unmask nginx               # Allow starting again

# Isolate target (switch runlevel)
sudo systemctl isolate rescue.target      # Single-user mode
sudo systemctl isolate graphical.target   # GUI mode

# Reboot/shutdown via systemctl
sudo systemctl reboot
sudo systemctl poweroff
sudo systemctl halt
sudo systemctl suspend
sudo systemctl hibernate

# Check system load
systemctl daemon-reload                   # After modifying service files
systemctl daemon-reexec                   # Reload systemd itself
```

## 8.2 Creating Custom systemd Services

### Service File Format
```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom Application
After=network.target                      # Start after networking

[Service]
Type=simple                               # simple|forking|oneshot|dbus|notify|idle
User=myapp                                # Run as this user
Group=myapp                               # Run as this group
WorkingDirectory=/opt/myapp               # Working directory
ExecStart=/usr/local/bin/myapp
ExecStop=/usr/local/bin/myapp-stop       # Optional stop command
ExecReload=/bin/kill -HUP $MAINPID       # Optional reload command
Restart=on-failure                        # Restart policy
RestartSec=5                              # Wait 5 seconds before restart
StartLimitBurst=5                         # Max restarts
StartLimitIntervalSec=60                  # In this timeframe
StandardOutput=journal                    # Log to journalctl
StandardError=journal
SyslogIdentifier=myapp

# Environment variables
Environment="PATH=/custom/path"
EnvironmentFile=/etc/myapp/config        # Load from file

# Resource limits
MemoryMax=512M
CPUQuota=50%
TasksMax=256

[Install]
WantedBy=multi-user.target               # Start in this target
```

### Service Type Explanation
```bash
Type=simple      # Default, process runs in foreground
Type=forking     # Process forks, parent exits
Type=oneshot     # Execute once and exit
Type=dbus        # Wait for D-Bus name
Type=notify      # Sends notification when ready
Type=idle        # Start after other jobs complete
```

### Managing Custom Services
```bash
# Create service file
sudo nano /etc/systemd/system/myapp.service

# Reload systemd daemon
sudo systemctl daemon-reload

# Enable and start
sudo systemctl enable myapp
sudo systemctl start myapp

# Check status
sudo systemctl status myapp
journalctl -u myapp -f                    # Follow logs

# Test service (dry-run)
sudo systemctl status myapp --no-pager
```

## 8.3 Service Troubleshooting

```bash
# Check service errors
systemctl status service_name             # See error messages
journalctl -u service_name -n 50         # Last 50 log lines
journalctl -u service_name -f            # Follow live logs

# Validate service file syntax
systemd-analyze verify myapp.service

# Debug failed startup
systemctl status myapp.service --full
systemctl start myapp.service --verbose

# Check service dependencies
systemctl list-dependencies myapp.service

# Show what would happen (dry run)
systemctl start myapp --dry-run

# Kill service forcefully
sudo systemctl kill -9 myapp
```

---

# PART 9: ADVANCED UTILITIES & PATTERNS

## 9.1 xargs - Execute Commands with Arguments

**Purpose**: Build and execute commands from input, handle multiple arguments efficiently

### Basic Usage
```bash
# Simple conversion from stdin to arguments
echo "file1 file2 file3" | xargs rm      # Remove all files
echo "file1 file2 file3" | xargs -I {} cp {} {}.bak  # Copy with backup

# With find (process many files)
find . -name "*.txt" | xargs cat         # Concatenate all txt files
find . -name "*.log" | xargs rm          # Delete all log files
find . -name "*.py" | xargs grep "TODO"  # Search in all python files

# Handle spaces and special characters
find . -print0 | xargs -0 rm             # Handles filenames with spaces
ls | tr '\n' '\0' | xargs -0 rm          # Safe removal with null separator
```

### Advanced xargs Options
```bash
# Limit number of arguments per command invocation
echo "1 2 3 4 5 6" | xargs -n 2 echo     # Echo 2 at a time
find . -name "*.txt" | xargs -n 1 grep "pattern"

# Use item in multiple places with -I
find . -name "*.txt" | xargs -I {} sh -c 'echo {}; wc -l {}' # Show name and count

# Parallel execution
find . -name "*.gz" | xargs -P 4 -I {} gunzip {}  # 4 parallel processes
ls *.tar | xargs -P 2 -I {} tar -xf {}  # Extract 2 at a time

# Show what would be executed
find . -name "*.txt" | xargs -p rm       # Prompt before each command
find . -name "*.txt" | xargs --verbose rm  # Show commands being executed

# With null separator (safer)
find . -name "*.txt" -print0 | xargs -0 wc -l

# Multiple input patterns
find . -type f | xargs -I {} ls -l {}
```

## 9.2 Combining Tools - Real-World Examples

### Text Processing Pipeline
```bash
# Find error logs, count by error type, show most common
grep -r "ERROR" /var/log/ | \
  cut -d: -f3- | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -10

# Extract IP addresses, count connections
netstat -an | \
  grep "ESTABLISHED" | \
  awk '{print $5}' | \
  cut -d: -f1 | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -10

# Find large files in directories, show summary
find . -type f -size +1M | \
  xargs ls -lh | \
  awk '{print $5, $9}' | \
  sort -hr | \
  head -20
```

### File Management
```bash
# Find and archive old log files
find /var/log -name "*.log" -mtime +30 | \
  xargs tar -czf /backup/old_logs_$(date +%Y%m%d).tar.gz

# Backup modified Python files
find . -name "*.py" -mtime -1 | \
  xargs tar -czf backup_$(date +%Y%m%d_%H%M%S).tar.gz

# Find duplicate files and remove
find . -type f -exec md5sum {} \; | \
  sort | \
  uniq -d -w 32 | \
  awk '{print $2}' | \
  xargs rm
```

### System Administration
```bash
# Find and fix permissions on all web files
find /var/www -type f | xargs chmod 644
find /var/www -type d | xargs chmod 755

# Kill processes matching pattern
ps aux | grep "python" | grep -v grep | awk '{print $2}' | xargs kill -9

# Monitor disk usage and alert
du -sh * | sort -hr | head -10 | \
  awk '$1 > "100M" {print $0}' | \
  while read size dir; do echo "Alert: $dir is $size"; done
```

## 9.3 Best Practices for Combining Tools

### Safe File Deletion
```bash
# ❌ Dangerous - stops on first error, leaves files
find . -name "*.tmp" | xargs rm

# ✓ Better - continues on errors
find . -name "*.tmp" -print0 | xargs -0 rm -f

# ✓ Best - ask before deleting
find . -name "*.tmp" -print0 | xargs -0 rm -i

# ✓ Safest - use delete action
find . -name "*.tmp" -delete
```

### Error Handling in Pipelines
```bash
# Check each step
file list | \
  step1 && \
  step2 && \
  step3

# Use set -e in scripts to exit on first error
set -e
cat file1 | \
  process1 | \
  process2 | \
  save_output
```

---

# COMPREHENSIVE COMMAND REFERENCE TABLE

## Download & Transfer
| Command | Purpose | Example |
|---------|---------|---------|
| **wget** | Web download, recursive | `wget -r -l 5 https://site.com` |
| **curl** | HTTP requests, API testing | `curl -X POST -d '{}' https://api.com` |
| **scp** | Secure copy over SSH | `scp user@host:/file ./` |
| **rsync** | Efficient file sync | `rsync -av source/ dest/` |

## Text Processing
| Command | Purpose | Example |
|---------|---------|---------|
| **grep** | Pattern matching | `grep "error" *.log` |
| **sed** | Stream editing | `sed 's/old/new/g' file.txt` |
| **awk** | Column-based processing | `awk '{print $1, $3}' file.txt` |
| **cut** | Extract fields | `cut -d: -f1 /etc/passwd` |
| **sort** | Sort lines | `sort -k2 -n file.txt` |
| **uniq** | Remove duplicates | `uniq -c file.txt` |

## File Operations
| Command | Purpose | Example |
|---------|---------|---------|
| **find** | Search files | `find . -name "*.py" -type f` |
| **locate** | Fast search (indexed) | `locate filename` |
| **tar** | Archive files | `tar -czf archive.tar.gz /path` |
| **zip/unzip** | ZIP archives | `zip -r archive.zip /path` |
| **chmod** | Change permissions | `chmod 755 script.sh` |
| **chown** | Change owner | `chown user:group file` |

## Task Scheduling
| Command | Purpose | Example |
|---------|---------|---------|
| **crontab** | Recurring jobs | `crontab -e` (edit) |
| **at** | One-time job | `at 3:30 PM` |
| **systemctl** | Service management | `systemctl start service` |

---

## CONCLUSION

This guide covers the essential Linux command-line utilities used daily by sysadmins, data engineers, and DevOps professionals. Master these tools and you'll be able to:

✓ Download and transfer files reliably  
✓ Process and transform text data  
✓ Find and manage files efficiently  
✓ Automate tasks with scheduling  
✓ Manage services and permissions  
✓ Combine tools for complex operations  

The key to expertise is **combining these tools** in creative ways to solve problems efficiently. Practice using pipes, redirection, and tool composition to become proficient.

**Next Steps**:
1. Try each command with `--help` and `man`
2. Build scripts combining multiple tools
3. Master one tool deeply before moving to the next
4. Practice with real data and files
5. Create reusable templates for common tasks

You now have a complete, self-contained reference for Linux command-line utilities!