#!/bin/bash

# Define colors
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Display banner
echo -e "${GREEN}"
echo "   sSSs    sSSs_sSSs     .S_sSSs     .S_sSSs      sSSs   .S S.   "
echo "  d%%SP   d%%SP~YS%%b   .SS~YS%%b   .SS~YS%%b    d%%SP  .SS SS.  "
echo " d%S'    d%S'     S%b   S%S   \`S%  S%S   \`S%b  d%S'    S%S S%S  "
echo "S%S     S%S       S%S  S%S    S%S   S%S    S%S  S%S     S%S S%S  "
echo "S&S     S&S       S&S  S%S    S&S   S%S    S&S  S&S     S%S S%S  "
echo "S&S     S&S       S&S  S&S    S&S   S&S    S&S  S&S_Ss   SS SS   "
echo "S&S     S&S       S&S  S&S    S&S   S&S    S&S  S&S~SP    S_S    "
echo "S&S     S&S       S&S  S&S    S&S   S&S    S&S  S&S      SS~SS   "
echo "S*b     S*b       d*S  S*S    S*S   S*S    S*S  S*b     S*S S*S  "
echo "S*S.    S*S.     .S*S  S*S    S*S   S*S    S*S  S*S.    S*S S*S  "
echo " SSSbs   SSSbs_sdSSS   S*S    S*S   S*S    S*S   SSSbs  S*S S*S  "
echo "  YSSP    YSSP~YSSY    S*S    SSS   S*S    SSS    YSSP  S*S S*S  "
echo "                       SP           SP                  SP       "
echo "                       Y            Y                   Y        "
echo -e "================================================================${NC}"

# Function to display usage information
usage() {
    echo "Usage: $0 [-h] -i <wireless_interface>"
    echo "Options:"
    echo "  -h              Display this help message"
    echo "  -i <interface>  Specify the wireless interface"
    exit 1
}

# Function to log error messages in red
log_error() {
    echo -e "${RED}Error: $1${NC}" >&2
}

# Function to log informational messages in green
log_info() {
    echo -e "${GREEN}$1${NC}"
}

# Function to check if a command is available
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Check if required commands are available
check_dependencies() {
    if ! command_exists "airodump-ng"; then
        log_error "airodump-ng is not installed. Please install airodump-ng."
        exit 1
    fi

    if ! command_exists "nmap"; then
        log_error "nmap is not installed. Please install nmap."
        exit 1
    fi

    if ! command_exists "nc"; then
        log_error "nc (netcat) is not installed. Please install nc (netcat)."
        exit 1
    fi
}

# Parse command-line options
while getopts "hi:" opt; do
    case ${opt} in
        h)
            usage
            ;;
        i)
            wireless_interface=$OPTARG
            ;;
        \?)
            log_error "Invalid option: $OPTARG"
            usage
            ;;
        :)
            log_error "Option -$OPTARG requires an argument."
            usage
            ;;
    esac
done
shift $((OPTIND -1))

# Check if wireless interface is specified
if [ -z "$wireless_interface" ]; then
    log_error "Wireless interface not specified."
    usage
fi

# Check dependencies
check_dependencies

# Set the wireless interface to monitor mode
ifconfig $wireless_interface down
iwconfig $wireless_interface mode monitor
ifconfig $wireless_interface up

# Start capturing wireless packets using airodump-ng
log_info "Starting packet capture with airodump-ng..."
airodump-ng --output-format csv --write output $wireless_interface &> /dev/null &

# Wait for a few seconds to capture some data
sleep 10

# Stop airodump-ng
killall airodump-ng &> /dev/null

# Extract MAC addresses from captured data
grep -o -E '([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})' output*.csv | sort -u > mac_addresses.txt

# Loop through each MAC address and perform nmap scan
while IFS= read -r mac_address; do
    log_info "Scanning device with MAC address: $mac_address"
    nmap -O -p 1-65535 -T4 -Pn $mac_address

    # Attempt to connect to common services
    log_info "Attempting to connect to common services on $mac_address"

    # SSH
    log_info "Trying SSH..."
    nc -z -w3 $mac_address 22 && echo -e "${GREEN}SSH: Connected${NC}" || echo -e "${RED}SSH: Not reachable${NC}"

    # FTP
    log_info "Trying FTP..."
    nc -z -w3 $mac_address 21 && echo -e "${GREEN}FTP: Connected${NC}" || echo -e "${RED}FTP: Not reachable${NC}"

    # SMB (Windows File Sharing)
    log_info "Trying SMB..."
    nc -z -w3 $mac_address 445 && echo -e "${GREEN}SMB: Connected${NC}" || echo -e "${RED}SMB: Not reachable${NC}"

    # Telnet
    log_info "Trying Telnet..."
    nc -z -w3 $mac_address 23 && echo -e "${GREEN}Telnet: Connected${NC}" || echo -e "${RED}Telnet: Not reachable${NC}"

    # Web UI (HTTP)
    log_info "Trying Web UI (HTTP)..."
    nc -z -w3 $mac_address 80 && echo -e "${GREEN}Web UI (HTTP): Connected${NC}" || echo -e "${RED}Web UI (HTTP): Not reachable${NC}"

    # Web UI (HTTPS)
    log_info "Trying Web UI (HTTPS)..."
    nc -z -w3 $mac_address 443 && echo -e "${GREEN}Web UI (HTTPS): Connected${NC}" || echo -e "${RED}Web UI (HTTPS): Not reachable${NC}"

done < mac_addresses.txt

# Cleanup temporary files
rm output*.csv mac_addresses.txt

# Extract MAC addresses from captured data if the file exists
if [ -f "output*.csv" ]; then
    grep -o -E '([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})' output*.csv | sort -u > mac_addresses.txt
else
    log_error "No captured data found. Exiting."
    exit 1
fi
