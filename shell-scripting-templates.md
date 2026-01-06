# SHELL SCRIPTING TEMPLATES & PATTERNS
## Ready-to-Use Scripts for Common Tasks

---

## TABLE OF CONTENTS

1. **Basic Script Template**
2. **Function-Based Template**
3. **Script with Error Handling**
4. **Logging & Configuration**
5. **Command-Line Arguments**
6. **File Processing**
7. **System Administration Tasks**
8. **Data Processing & Transformation**
9. **Monitoring & Health Checks**
10. **Deployment & DevOps Scripts**

---

# PART 1: BASIC SCRIPT TEMPLATE

## 1.1 Minimal Shell Script

```bash
#!/bin/bash
# Script Name: script_name.sh
# Description: Brief description of what script does
# Author: Your Name
# Date: 2025-01-06
# Version: 1.0

# Colors for output (optional)
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Simple function
hello() {
    echo "Hello, $1!"
}

# Main execution
main() {
    echo "Script started"
    hello "World"
    echo "Script completed"
}

# Run main function
main "$@"
```

## 1.2 Script with Shebang and Permissions

```bash
#!/bin/bash
# Always start with shebang (#!/bin/bash)
# This tells the system to execute using bash

# Make executable:
# chmod +x script.sh

# Run directly:
# ./script.sh

# Or run with explicit bash:
# bash script.sh

echo "Script is running"
```

---

# PART 2: FUNCTION-BASED TEMPLATE

## 2.1 Complete Function Template

```bash
#!/bin/bash

################################################################################
# Script: complete_template.sh
# Description: Template showing best practices for function-based scripts
# Usage: ./complete_template.sh [options]
# Author: Your Name
# Created: 2025-01-06
################################################################################

set -euo pipefail  # Exit on error, undefined vars, pipe failures
IFS=$'\n\t'        # Set safe IFS

# ============================================================================
# VARIABLES & CONFIGURATION
# ============================================================================

# Script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Logging
LOG_DIR="${SCRIPT_DIR}/logs"
LOG_FILE="${LOG_DIR}/$(basename "$0" .sh)_$(date +%Y%m%d_%H%M%S).log"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# ============================================================================
# LOGGING FUNCTIONS
# ============================================================================

# Create log directory
init_logging() {
    mkdir -p "$LOG_DIR"
}

# Log message with timestamp
log() {
    local level="$1"
    shift
    local message="$*"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[${timestamp}] [${level}] ${message}" | tee -a "$LOG_FILE"
}

log_info() {
    echo -e "${GREEN}[INFO]${NC} $*" | tee -a "$LOG_FILE"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $*" | tee -a "$LOG_FILE"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $*" | tee -a "$LOG_FILE"
}

log_debug() {
    if [[ "${DEBUG:-0}" == "1" ]]; then
        echo -e "${BLUE}[DEBUG]${NC} $*" | tee -a "$LOG_FILE"
    fi
}

# ============================================================================
# UTILITY FUNCTIONS
# ============================================================================

# Check if command exists
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Check if file exists
file_exists() {
    [[ -f "$1" ]]
}

# Check if directory exists
dir_exists() {
    [[ -d "$1" ]]
}

# Check if running as root
is_root() {
    [[ "$EUID" -eq 0 ]]
}

# Cleanup function (runs on exit)
cleanup() {
    log_info "Cleaning up..."
    # Add cleanup tasks here
    # Remove temp files, close connections, etc.
}

# Error handling (called on script error)
error_exit() {
    local line_number="$1"
    local error_message="${2:-Unknown error}"
    
    log_error "Script failed at line $line_number: $error_message"
    exit 1
}

# Set up trap for cleanup and errors
trap cleanup EXIT
trap 'error_exit ${LINENO} "$BASH_COMMAND"' ERR

# ============================================================================
# MAIN FUNCTIONS
# ============================================================================

# Example function with parameters and return codes
process_file() {
    local file="$1"
    
    if ! file_exists "$file"; then
        log_error "File not found: $file"
        return 1
    fi
    
    log_info "Processing: $file"
    
    # Do something with file
    wc -l "$file"
    
    return 0
}

# Function to display usage
usage() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS]

OPTIONS:
    -h, --help          Show this help message
    -v, --verbose       Enable verbose output
    -d, --debug         Enable debug output
    -f, --file FILE     Process specific file
    -c, --config CONFIG Use config file

EXAMPLES:
    $(basename "$0") -f myfile.txt
    $(basename "$0") --verbose --file data.csv

EOF
}

# Parse command-line arguments
parse_args() {
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -h|--help)
                usage
                exit 0
                ;;
            -v|--verbose)
                VERBOSE=1
                shift
                ;;
            -d|--debug)
                DEBUG=1
                shift
                ;;
            -f|--file)
                FILE="$2"
                shift 2
                ;;
            -c|--config)
                CONFIG_FILE="$2"
                shift 2
                ;;
            *)
                log_error "Unknown option: $1"
                usage
                exit 1
                ;;
        esac
    done
}

# Main function
main() {
    log_info "Starting script"
    
    if [[ -z "${FILE:-}" ]]; then
        log_error "No file specified"
        usage
        exit 1
    fi
    
    process_file "$FILE"
    
    log_info "Script completed successfully"
}

# ============================================================================
# SCRIPT EXECUTION
# ============================================================================

# Initialize logging
init_logging

# Parse arguments
parse_args "$@"

# Run main function
main
```

---

# PART 3: SCRIPT WITH ERROR HANDLING

## 3.1 Robust Error Handling Template

```bash
#!/bin/bash

################################################################################
# COMPREHENSIVE ERROR HANDLING TEMPLATE
################################################################################

# Exit on error
set -e

# Exit on undefined variable
set -u

# Exit if any command in pipeline fails
set -o pipefail

# ============================================================================
# ERROR HANDLING FUNCTIONS
# ============================================================================

# Print error and exit
die() {
    echo "ERROR: $*" >&2
    exit 1
}

# Assert file exists
assert_file_exists() {
    if [[ ! -f "$1" ]]; then
        die "File not found: $1"
    fi
}

# Assert directory exists
assert_dir_exists() {
    if [[ ! -d "$1" ]]; then
        die "Directory not found: $1"
    fi
}

# Assert command exists
assert_command() {
    if ! command -v "$1" >/dev/null 2>&1; then
        die "Command not found: $1"
    fi
}

# Try command with retry logic
try_command() {
    local max_attempts=3
    local timeout=5
    local attempt=1
    
    while [[ $attempt -le $max_attempts ]]; do
        if timeout $timeout "$@"; then
            return 0
        fi
        
        echo "Attempt $attempt failed. Retrying..." >&2
        ((attempt++))
        sleep 2
    done
    
    die "Command failed after $max_attempts attempts: $*"
}

# Wrapped execution with error details
run_or_die() {
    if ! "$@"; then
        die "Failed to execute: $*"
    fi
}

# ============================================================================
# MAIN
# ============================================================================

main() {
    # Check prerequisites
    assert_command "curl"
    assert_command "jq"
    
    # Try command with retries
    try_command curl https://api.example.com/health
    
    # Run command or exit with error
    run_or_die echo "Processing complete"
}

main "$@"
```

---

# PART 4: LOGGING & CONFIGURATION

## 4.1 Advanced Logging Template

```bash
#!/bin/bash

################################################################################
# ADVANCED LOGGING SYSTEM
################################################################################

# ============================================================================
# CONFIGURATION
# ============================================================================

# Log levels
readonly LOG_LEVEL_DEBUG=1
readonly LOG_LEVEL_INFO=2
readonly LOG_LEVEL_WARN=3
readonly LOG_LEVEL_ERROR=4

# Current log level (default: INFO)
LOG_LEVEL=${LOG_LEVEL:-$LOG_LEVEL_INFO}

# Log file
LOG_FILE="${LOG_FILE:-.}/script.log"

# Enable/disable features
LOG_TO_FILE=1
LOG_TO_STDOUT=1
LOG_WITH_TIMESTAMP=1
LOG_WITH_COLORS=1

# Colors
if [[ $LOG_WITH_COLORS -eq 1 ]]; then
    COLOR_DEBUG='\033[0;36m'    # Cyan
    COLOR_INFO='\033[0;32m'     # Green
    COLOR_WARN='\033[0;33m'     # Yellow
    COLOR_ERROR='\033[0;31m'    # Red
    COLOR_RESET='\033[0m'
else
    COLOR_DEBUG=''
    COLOR_INFO=''
    COLOR_WARN=''
    COLOR_ERROR=''
    COLOR_RESET=''
fi

# ============================================================================
# LOGGING FUNCTIONS
# ============================================================================

# Initialize logging
init_logging() {
    mkdir -p "$(dirname "$LOG_FILE")"
    touch "$LOG_FILE"
    
    log_info "Logging initialized"
    log_info "Log file: $LOG_FILE"
}

# Core logging function
_log() {
    local level="$1"
    local level_num="$2"
    local color="$3"
    shift 3
    local message="$*"
    
    # Skip if log level too low
    [[ $level_num -lt $LOG_LEVEL ]] && return
    
    # Format timestamp
    local timestamp=""
    if [[ $LOG_WITH_TIMESTAMP -eq 1 ]]; then
        timestamp="$(date '+%Y-%m-%d %H:%M:%S') "
    fi
    
    # Format message
    local formatted="${timestamp}[${level}] ${message}"
    
    # Log to stdout
    if [[ $LOG_TO_STDOUT -eq 1 ]]; then
        echo -e "${color}${formatted}${COLOR_RESET}"
    fi
    
    # Log to file (without colors)
    if [[ $LOG_TO_FILE -eq 1 ]]; then
        echo "${timestamp}[${level}] ${message}" >> "$LOG_FILE"
    fi
}

# Logging functions
log_debug() {
    _log "DEBUG" $LOG_LEVEL_DEBUG "$COLOR_DEBUG" "$@"
}

log_info() {
    _log "INFO" $LOG_LEVEL_INFO "$COLOR_INFO" "$@"
}

log_warn() {
    _log "WARN" $LOG_LEVEL_WARN "$COLOR_WARN" "$@"
}

log_error() {
    _log "ERROR" $LOG_LEVEL_ERROR "$COLOR_ERROR" "$@"
}

# ============================================================================
# CONFIGURATION FILE HANDLING
# ============================================================================

# Load configuration from file
load_config() {
    local config_file="$1"
    
    if [[ ! -f "$config_file" ]]; then
        log_error "Config file not found: $config_file"
        return 1
    fi
    
    log_info "Loading config: $config_file"
    source "$config_file"
    
    return 0
}

# Validate required variables
validate_config() {
    local required_vars=("$@")
    local missing_vars=()
    
    for var in "${required_vars[@]}"; do
        if [[ -z "${!var:-}" ]]; then
            missing_vars+=("$var")
        fi
    done
    
    if [[ ${#missing_vars[@]} -gt 0 ]]; then
        log_error "Missing required variables: ${missing_vars[*]}"
        return 1
    fi
    
    return 0
}

# ============================================================================
# EXAMPLE CONFIG FILE (save as config.sh)
# ============================================================================

# Example config file content:
# # config.sh
# APP_NAME="MyApp"
# APP_VERSION="1.0.0"
# DATABASE_HOST="localhost"
# DATABASE_PORT=5432
# DEBUG_MODE=0

# ============================================================================
# MAIN
# ============================================================================

main() {
    init_logging
    
    # Load and validate configuration
    load_config "./config.sh"
    validate_config "APP_NAME" "DATABASE_HOST" || exit 1
    
    log_info "Starting application: $APP_NAME"
    log_debug "Debug mode enabled"
    log_warn "This is a warning"
    log_error "This is an error (but we continue)"
    
    log_info "Application completed"
}

main "$@"
```

---

# PART 5: COMMAND-LINE ARGUMENTS

## 5.1 Argument Parsing Template

```bash
#!/bin/bash

################################################################################
# COMMAND-LINE ARGUMENT PARSING
################################################################################

# ============================================================================
# VARIABLES
# ============================================================================

VERBOSE=0
DEBUG=0
DRY_RUN=0
CONFIG_FILE=""
OUTPUT_FILE=""
INPUT_FILES=()

# ============================================================================
# ARGUMENT PARSING
# ============================================================================

# Parse short options
parse_short_opts() {
    while getopts "hvdc:o:" opt; do
        case $opt in
            h)
                show_help
                exit 0
                ;;
            v)
                VERBOSE=1
                ;;
            d)
                DEBUG=1
                ;;
            c)
                CONFIG_FILE="$OPTARG"
                ;;
            o)
                OUTPUT_FILE="$OPTARG"
                ;;
            *)
                echo "Unknown option: -$OPTARG" >&2
                show_help
                exit 1
                ;;
        esac
    done
    
    # Shift processed options
    shift $((OPTIND-1))
    
    # Remaining arguments are input files
    INPUT_FILES=("$@")
}

# Parse long options (alternative to getopts)
parse_long_opts() {
    while [[ $# -gt 0 ]]; do
        case "$1" in
            --help)
                show_help
                exit 0
                ;;
            --verbose)
                VERBOSE=1
                shift
                ;;
            --debug)
                DEBUG=1
                shift
                ;;
            --dry-run)
                DRY_RUN=1
                shift
                ;;
            --config)
                CONFIG_FILE="$2"
                shift 2
                ;;
            --output)
                OUTPUT_FILE="$2"
                shift 2
                ;;
            --input)
                INPUT_FILES+=("$2")
                shift 2
                ;;
            --)
                shift
                INPUT_FILES+=("$@")
                break
                ;;
            *)
                echo "Unknown option: $1" >&2
                show_help
                exit 1
                ;;
        esac
    done
}

# Show help message
show_help() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS] [FILES]

OPTIONS:
    -h, --help              Show this help message
    -v, --verbose           Enable verbose output
    -d, --debug             Enable debug output
    -c, --config FILE       Config file
    -o, --output FILE       Output file
    --dry-run               Show what would be done without doing it

EXAMPLES:
    $(basename "$0") -v file1.txt file2.txt
    $(basename "$0") --verbose --config config.ini --output result.txt input.txt
    $(basename "$0") --help

EOF
}

# Validate arguments
validate_args() {
    if [[ ${#INPUT_FILES[@]} -eq 0 ]]; then
        echo "Error: No input files specified" >&2
        show_help
        exit 1
    fi
    
    for file in "${INPUT_FILES[@]}"; do
        if [[ ! -f "$file" ]]; then
            echo "Error: File not found: $file" >&2
            exit 1
        fi
    done
    
    if [[ -n "$CONFIG_FILE" ]] && [[ ! -f "$CONFIG_FILE" ]]; then
        echo "Error: Config file not found: $CONFIG_FILE" >&2
        exit 1
    fi
}

# ============================================================================
# MAIN
# ============================================================================

main() {
    # Parse arguments (choose one method)
    # parse_short_opts "$@"      # For short options only (-h, -v, etc.)
    parse_long_opts "$@"         # For long options (--help, --verbose, etc.)
    
    validate_args
    
    [[ $VERBOSE -eq 1 ]] && echo "Verbose mode enabled"
    [[ $DEBUG -eq 1 ]] && echo "Debug mode enabled"
    [[ $DRY_RUN -eq 1 ]] && echo "Dry-run mode - no changes will be made"
    
    echo "Input files: ${INPUT_FILES[*]}"
    echo "Config file: $CONFIG_FILE"
    echo "Output file: $OUTPUT_FILE"
}

main "$@"
```

---

# PART 6: FILE PROCESSING

## 6.1 File Processing Template

```bash
#!/bin/bash

################################################################################
# FILE PROCESSING TEMPLATE
################################################################################

# ============================================================================
# SAFE FILE READING
# ============================================================================

# Read file line by line (preserves spaces)
process_file_line_by_line() {
    local file="$1"
    local line_num=0
    
    while IFS= read -r line; do
        ((line_num++))
        
        # Skip empty lines
        [[ -z "$line" ]] && continue
        
        # Skip comment lines
        [[ "$line" =~ ^# ]] && continue
        
        echo "Line $line_num: $line"
        
        # Process line (example)
        # echo "$line" | some_command
    done < "$file"
}

# Read file with field separator
process_csv_file() {
    local file="$1"
    
    while IFS=',' read -r field1 field2 field3; do
        echo "Field1: $field1, Field2: $field2, Field3: $field3"
    done < "$file"
}

# Read file into array
read_file_into_array() {
    local file="$1"
    local -n array_ref="$2"  # nameref to pass array by reference
    
    mapfile -t array_ref < "$file"
    
    echo "Read ${#array_ref[@]} lines"
}

# Process multiple files
process_multiple_files() {
    local -a files=("$@")
    
    for file in "${files[@]}"; do
        [[ ! -f "$file" ]] && continue
        
        echo "Processing: $file"
        process_file_line_by_line "$file"
    done
}

# ============================================================================
# FILE MANIPULATION
# ============================================================================

# Append to file safely
append_to_file() {
    local file="$1"
    local content="$2"
    
    # Create backup
    cp "$file" "${file}.bak"
    
    # Append
    echo "$content" >> "$file"
    
    echo "Appended to $file"
}

# Replace in file
replace_in_file() {
    local file="$1"
    local search="$2"
    local replace="$3"
    
    # Backup
    cp "$file" "${file}.bak"
    
    # Replace (sed in-place)
    sed -i "s|${search}|${replace}|g" "$file"
    
    echo "Replaced '$search' with '$replace' in $file"
}

# Count lines in file
count_lines() {
    local file="$1"
    wc -l < "$file"
}

# Get file size in human-readable format
get_file_size() {
    local file="$1"
    du -h "$file" | cut -f1
}

# ============================================================================
# BATCH FILE PROCESSING
# ============================================================================

# Process all files matching pattern
process_all_matching() {
    local pattern="$1"
    local action="$2"
    
    while IFS= read -r file; do
        echo "Processing: $file"
        $action "$file"
    done < <(find . -name "$pattern" -type f)
}

# Example actions
convert_file() {
    local file="$1"
    # Example: convert markdown to HTML
    pandoc "$file" -o "${file%.md}.html"
}

# ============================================================================
# MAIN
# ============================================================================

main() {
    local test_file="test.txt"
    
    # Create test file
    cat > "$test_file" << 'EOF'
# Comment line
field1,field2,field3
value1,value2,value3
value4,value5,value6
EOF
    
    echo "=== Line by line processing ==="
    process_file_line_by_line "$test_file"
    
    echo -e "\n=== CSV processing ==="
    process_csv_file "$test_file"
    
    echo -e "\n=== File statistics ==="
    echo "Lines: $(count_lines "$test_file")"
    echo "Size: $(get_file_size "$test_file")"
    
    # Cleanup
    rm "$test_file" "${test_file}.bak" 2>/dev/null || true
}

main "$@"
```

---

# PART 7: SYSTEM ADMINISTRATION TASKS

## 7.1 System Administration Script Template

```bash
#!/bin/bash

################################################################################
# SYSTEM ADMINISTRATION SCRIPT TEMPLATE
################################################################################

set -euo pipefail

# ============================================================================
# SYSTEM CHECKS
# ============================================================================

# Check if running as root
check_root() {
    if [[ $EUID -ne 0 ]]; then
        echo "This script must be run as root"
        exit 1
    fi
}

# Check disk space
check_disk_space() {
    echo "=== Disk Space ==="
    df -h | head -n 1
    df -h | grep -E "/dev/|/mnt"
}

# Check memory usage
check_memory() {
    echo -e "\n=== Memory Usage ==="
    free -h
}

# Check system load
check_system_load() {
    echo -e "\n=== System Load ==="
    uptime
}

# ============================================================================
# USER MANAGEMENT
# ============================================================================

# Create user
create_user() {
    local username="$1"
    local home_dir="/home/$username"
    
    if id "$username" &>/dev/null; then
        echo "User already exists: $username"
        return 1
    fi
    
    useradd -m -s /bin/bash "$username"
    echo "Created user: $username"
}

# Add user to group
add_to_group() {
    local username="$1"
    local group="$2"
    
    usermod -aG "$group" "$username"
    echo "Added $username to group $group"
}

# Delete user
delete_user() {
    local username="$1"
    
    if ! id "$username" &>/dev/null; then
        echo "User not found: $username"
        return 1
    fi
    
    userdel -r "$username"
    echo "Deleted user: $username"
}

# List users
list_users() {
    echo "=== System Users (UID >= 1000) ==="
    awk -F: '$3 >= 1000 {print $1, "(" $3 ")"}' /etc/passwd
}

# ============================================================================
# SERVICE MANAGEMENT
# ============================================================================

# Start service
start_service() {
    local service="$1"
    
    systemctl start "$service"
    echo "Started service: $service"
}

# Stop service
stop_service() {
    local service="$1"
    
    systemctl stop "$service"
    echo "Stopped service: $service"
}

# Restart service
restart_service() {
    local service="$1"
    
    systemctl restart "$service"
    echo "Restarted service: $service"
}

# Enable service on boot
enable_service() {
    local service="$1"
    
    systemctl enable "$service"
    echo "Enabled service: $service"
}

# Check service status
check_service_status() {
    local service="$1"
    
    systemctl status "$service" || echo "Service not running: $service"
}

# ============================================================================
# BACKUP & MAINTENANCE
# ============================================================================

# Backup directory
backup_directory() {
    local source_dir="$1"
    local backup_dir="$2"
    local backup_file="${backup_dir}/backup_$(date +%Y%m%d_%H%M%S).tar.gz"
    
    mkdir -p "$backup_dir"
    tar -czf "$backup_file" -C "$(dirname "$source_dir")" "$(basename "$source_dir")"
    
    echo "Backup created: $backup_file"
}

# Clean old logs
clean_old_logs() {
    local log_dir="$1"
    local days="${2:-30}"
    
    find "$log_dir" -name "*.log" -mtime +$days -delete
    echo "Deleted logs older than $days days"
}

# ============================================================================
# MAIN
# ============================================================================

main() {
    check_root
    
    echo "=== System Status Report ==="
    check_system_load
    check_memory
    check_disk_space
    list_users
    
    echo -e "\n=== Service Status ==="
    check_service_status "nginx" || true
    check_service_status "docker" || true
}

main "$@"
```

---

# PART 8: DATA PROCESSING & TRANSFORMATION

## 8.1 Data Processing Template

```bash
#!/bin/bash

################################################################################
# DATA PROCESSING & TRANSFORMATION TEMPLATE
################################################################################

# ============================================================================
# DATA PARSING
# ============================================================================

# Parse JSON (using jq)
parse_json() {
    local json_file="$1"
    local key="$2"
    
    jq ".${key}" "$json_file"
}

# Parse CSV and process
parse_and_process_csv() {
    local csv_file="$1"
    
    awk -F, 'NR > 1 {
        name=$1
        age=$2
        salary=$3
        
        printf "Name: %-15s Age: %3d Salary: $%8.2f\n", name, age, salary
    }' "$csv_file"
}

# Extract specific columns
extract_columns() {
    local file="$1"
    local -a columns=("${@:2}")
    
    awk -v cols="${columns[*]}" 'BEGIN {FS=","} {
        for (i in cols) {
            printf "%s ", $cols[i]
        }
        printf "\n"
    }' "$file"
}

# ============================================================================
# DATA TRANSFORMATION
# ============================================================================

# Convert CSV to JSON
csv_to_json() {
    local csv_file="$1"
    
    awk -F, '
    NR==1 {
        for(i=1;i<=NF;i++) header[i]=$i
        next
    }
    {
        printf "{"
        for(i=1;i<=NF;i++) {
            printf "\"%s\":\"%s\"", header[i], $i
            if(i<NF) printf ","
        }
        printf "}\n"
    }' "$csv_file"
}

# Convert JSON to CSV
json_to_csv() {
    local json_file="$1"
    
    jq -r '.[] | [.field1, .field2, .field3] | @csv' "$json_file"
}

# ============================================================================
# DATA AGGREGATION
# ============================================================================

# Sum column from CSV
sum_column() {
    local file="$1"
    local column="$2"
    
    awk -F, -v col=$column 'NR > 1 {sum += $col} END {print sum}' "$file"
}

# Count occurrences
count_occurrences() {
    local file="$1"
    local column="$2"
    
    awk -F, -v col=$column 'NR > 1 {count[$col]++} 
    END {for (key in count) printf "%s: %d\n", key, count[key]}' "$file" | \
    sort -t: -k2 -rn
}

# Get unique values
get_unique_values() {
    local file="$1"
    local column="$2"
    
    awk -F, -v col=$column 'NR > 1 {print $col}' "$file" | sort -u
}

# ============================================================================
# DATA VALIDATION
# ============================================================================

# Validate email format
validate_email() {
    local email="$1"
    
    if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 0
    else
        return 1
    fi
}

# Validate numeric
validate_numeric() {
    local value="$1"
    
    if [[ $value =~ ^[0-9]+$ ]]; then
        return 0
    else
        return 1
    fi
}

# Validate line count
validate_line_count() {
    local file="$1"
    local expected="$2"
    
    local actual=$(wc -l < "$file")
    if [[ $actual -eq $expected ]]; then
        echo "Line count OK: $actual"
        return 0
    else
        echo "Line count mismatch: expected $expected, got $actual"
        return 1
    fi
}

# ============================================================================
# MAIN
# ============================================================================

main() {
    # Create test CSV
    local test_csv="data.csv"
    cat > "$test_csv" << 'EOF'
name,age,salary
John,30,75000
Jane,28,80000
Bob,35,70000
Alice,32,85000
EOF
    
    echo "=== CSV Data ==="
    cat "$test_csv"
    
    echo -e "\n=== Parse and Process ==="
    parse_and_process_csv "$test_csv"
    
    echo -e "\n=== Sum Salaries ==="
    echo "Total: $(sum_column "$test_csv" 3)"
    
    echo -e "\n=== Count by Age ==="
    count_occurrences "$test_csv" 2
    
    echo -e "\n=== Unique Names ==="
    get_unique_values "$test_csv" 1
    
    # Cleanup
    rm "$test_csv"
}

main "$@"
```

---

# PART 9: MONITORING & HEALTH CHECKS

## 9.1 Monitoring Script Template

```bash
#!/bin/bash

################################################################################
# MONITORING & HEALTH CHECK TEMPLATE
################################################################################

# ============================================================================
# HEALTH CHECKS
# ============================================================================

# Check service health
check_service_health() {
    local service="$1"
    local expected_status="running"
    
    local status=$(systemctl is-active "$service")
    
    if [[ "$status" == "$expected_status" ]]; then
        echo "✓ $service is $status"
        return 0
    else
        echo "✗ $service is $status (expected: $expected_status)"
        return 1
    fi
}

# Check HTTP endpoint
check_http_endpoint() {
    local url="$1"
    local expected_code="${2:-200}"
    
    local response_code=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    
    if [[ "$response_code" == "$expected_code" ]]; then
        echo "✓ $url returned $response_code"
        return 0
    else
        echo "✗ $url returned $response_code (expected: $expected_code)"
        return 1
    fi
}

# Check disk space
check_disk_usage() {
    local path="${1:-.}"
    local threshold="${2:-80}"  # percentage
    
    local usage=$(df "$path" | awk 'NR==2 {print $5}' | sed 's/%//')
    
    if [[ $usage -gt $threshold ]]; then
        echo "✗ Disk usage at ${usage}% (threshold: ${threshold}%)"
        return 1
    else
        echo "✓ Disk usage at ${usage}%"
        return 0
    fi
}

# Check memory usage
check_memory_usage() {
    local threshold="${1:-80}"  # percentage
    
    local usage=$(free | awk 'NR==2 {printf "%d", ($3/$2)*100}')
    
    if [[ $usage -gt $threshold ]]; then
        echo "✗ Memory usage at ${usage}% (threshold: ${threshold}%)"
        return 1
    else
        echo "✓ Memory usage at ${usage}%"
        return 0
    fi
}

# Check process exists
check_process() {
    local process_name="$1"
    
    if pgrep -x "$process_name" > /dev/null; then
        echo "✓ Process running: $process_name"
        return 0
    else
        echo "✗ Process not running: $process_name"
        return 1
    fi
}

# ============================================================================
# MONITORING LOOPS
# ============================================================================

# Monitor with interval
monitor_continuous() {
    local interval="${1:-60}"  # seconds
    
    while true; do
        clear
        echo "=== System Monitoring ==="
        echo "Last check: $(date)"
        echo ""
        
        check_disk_usage "/" 80
        check_memory_usage 80
        check_process "sshd"
        
        echo ""
        echo "Next check in $interval seconds (Ctrl+C to stop)"
        sleep "$interval"
    done
}

# ============================================================================
# ALERTING
# ============================================================================

# Send alert email
send_alert() {
    local subject="$1"
    local message="$2"
    local recipient="${3:-admin@example.com}"
    
    echo "$message" | mail -s "$subject" "$recipient"
    echo "Alert sent to $recipient"
}

# Log alert
log_alert() {
    local alert_type="$1"
    local message="$2"
    local log_file="${3:-alerts.log}"
    
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$alert_type] $message" >> "$log_file"
}

# ============================================================================
# MAIN
# ============================================================================

main() {
    echo "=== Health Check Report ==="
    echo "Time: $(date)"
    echo ""
    
    local exit_code=0
    
    check_disk_usage "/" 80 || exit_code=1
    check_memory_usage 80 || exit_code=1
    check_service_health "sshd" || exit_code=1
    
    if [[ $exit_code -ne 0 ]]; then
        echo ""
        echo "⚠ Some checks failed!"
    fi
    
    exit $exit_code
}

main "$@"
```

---

# PART 10: DEPLOYMENT & DEVOPS SCRIPTS

## 10.1 Deployment Script Template

```bash
#!/bin/bash

################################################################################
# DEPLOYMENT SCRIPT TEMPLATE
################################################################################

set -euo pipefail

# ============================================================================
# CONFIGURATION
# ============================================================================

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly APP_NAME="${APP_NAME:-myapp}"
readonly APP_DIR="${APP_DIR:-/opt/myapp}"
readonly BACKUP_DIR="${BACKUP_DIR:-/backups}"
readonly LOG_DIR="${LOG_DIR:-/var/log/deployments}"

# ============================================================================
# LOGGING
# ============================================================================

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_DIR/deploy.log"
}

log_success() {
    echo "✓ $*" | tee -a "$LOG_DIR/deploy.log"
}

log_error() {
    echo "✗ $*" | tee -a "$LOG_DIR/deploy.log"
}

# ============================================================================
# PRE-DEPLOYMENT CHECKS
# ============================================================================

check_prerequisites() {
    log "Checking prerequisites..."
    
    # Check required commands
    for cmd in git docker systemctl; do
        if ! command -v "$cmd" &>/dev/null; then
            log_error "Required command not found: $cmd"
            return 1
        fi
    done
    
    # Check disk space
    local available=$(df "$APP_DIR" | awk 'NR==2 {print $4}')
    if [[ $available -lt 1048576 ]]; then  # 1GB
        log_error "Insufficient disk space (need 1GB, have $((available/1024))MB)"
        return 1
    fi
    
    log_success "Prerequisites check passed"
    return 0
}

# ============================================================================
# BACKUP & ROLLBACK
# ============================================================================

create_backup() {
    log "Creating backup..."
    
    mkdir -p "$BACKUP_DIR"
    local backup_file="${BACKUP_DIR}/${APP_NAME}_backup_$(date +%Y%m%d_%H%M%S).tar.gz"
    
    tar -czf "$backup_file" -C "$APP_DIR" .
    
    log_success "Backup created: $backup_file"
    echo "$backup_file"
}

rollback() {
    local backup_file="$1"
    
    if [[ ! -f "$backup_file" ]]; then
        log_error "Backup file not found: $backup_file"
        return 1
    fi
    
    log "Rolling back from: $backup_file"
    
    rm -rf "$APP_DIR"
    mkdir -p "$APP_DIR"
    tar -xzf "$backup_file" -C "$APP_DIR"
    
    log_success "Rollback completed"
}

# ============================================================================
# DEPLOYMENT STEPS
# ============================================================================

stop_service() {
    log "Stopping service..."
    systemctl stop "$APP_NAME" || true
    sleep 2
    log_success "Service stopped"
}

pull_latest_code() {
    log "Pulling latest code..."
    
    cd "$APP_DIR"
    git fetch origin
    git checkout main
    git pull origin main
    
    log_success "Code updated"
}

build_application() {
    log "Building application..."
    
    cd "$APP_DIR"
    
    # Example: Node.js app
    if [[ -f "package.json" ]]; then
        npm ci
        npm run build
    fi
    
    # Example: Docker
    if [[ -f "Dockerfile" ]]; then
        docker build -t "$APP_NAME:latest" .
    fi
    
    log_success "Build completed"
}

run_tests() {
    log "Running tests..."
    
    cd "$APP_DIR"
    
    if [[ -f "package.json" ]]; then
        npm test
    fi
    
    log_success "Tests passed"
}

start_service() {
    log "Starting service..."
    systemctl start "$APP_NAME"
    sleep 2
    
    if systemctl is-active "$APP_NAME" &>/dev/null; then
        log_success "Service started"
        return 0
    else
        log_error "Failed to start service"
        return 1
    fi
}

verify_deployment() {
    log "Verifying deployment..."
    
    # Check service status
    if ! systemctl is-active "$APP_NAME" &>/dev/null; then
        log_error "Service is not running"
        return 1
    fi
    
    # Check health endpoint
    if ! curl -sf "http://localhost:8080/health" > /dev/null; then
        log_error "Health check failed"
        return 1
    fi
    
    log_success "Deployment verified"
}

# ============================================================================
# MAIN DEPLOYMENT FLOW
# ============================================================================

main() {
    mkdir -p "$LOG_DIR"
    
    log "Starting deployment of $APP_NAME"
    
    # Pre-deployment checks
    if ! check_prerequisites; then
        log_error "Prerequisites check failed"
        exit 1
    fi
    
    # Create backup
    local backup_file
    backup_file=$(create_backup)
    
    # Deployment
    if ! (
        stop_service &&
        pull_latest_code &&
        build_application &&
        run_tests &&
        start_service &&
        verify_deployment
    ); then
        log_error "Deployment failed, rolling back..."
        rollback "$backup_file"
        exit 1
    fi
    
    log_success "Deployment completed successfully"
}

# Error handler
trap 'log_error "Deployment failed"; exit 1' ERR

main "$@"
```

---

# QUICK REFERENCE: COMMON PATTERNS

## String Operations
```bash
# Length
${#string}

# Substring
${string:position:length}

# Remove prefix
${string#prefix}

# Remove suffix
${string%suffix}

# Replace
${string/search/replace}

# Replace all
${string//search/replace}

# Default value
${variable:-default}

# Assign default
${variable:=default}

# Alternate
${variable:+alternate}
```

## Array Operations
```bash
# Declare array
declare -a array=()

# Add element
array+=("element")

# Access element
${array[0]}

# All elements
${array[@]}

# Array length
${#array[@]}

# Iterate
for item in "${array[@]}"; do echo "$item"; done
```

## Conditional Tests
```bash
# File tests
[[ -f file ]]       # File exists
[[ -d dir ]]        # Directory exists
[[ -r file ]]       # Readable
[[ -w file ]]       # Writable
[[ -x file ]]       # Executable

# String tests
[[ -z $var ]]       # Empty
[[ -n $var ]]       # Not empty
[[ $str1 = $str2 ]] # Equal
[[ $str1 != $str2 ]] # Not equal

# Numeric tests
[[ $num -eq $num2 ]] # Equal
[[ $num -ne $num2 ]] # Not equal
[[ $num -lt $num2 ]] # Less than
[[ $num -le $num2 ]] # Less or equal
[[ $num -gt $num2 ]] # Greater than
[[ $num -ge $num2 ]] # Greater or equal
```

## Function Definition
```bash
# Standard function
function_name() {
    local var="$1"
    
    echo "Processing: $var"
    
    return 0
}

# With return value
get_sum() {
    echo $((${1} + ${2}))
}

result=$(get_sum 5 3)
echo $result  # 8
```

---

## CONCLUSION

This template collection provides ready-to-use shells for:
- ✅ Basic scripts
- ✅ Function-based scripts
- ✅ Error handling
- ✅ Logging & configuration
- ✅ Command-line argument parsing
- ✅ File processing
- ✅ System administration
- ✅ Data processing
- ✅ Monitoring & health checks
- ✅ Deployment & DevOps

**Pick a template, customize for your needs, and start scripting!**