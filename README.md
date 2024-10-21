#!/bin/bash
# Simple Recon script to build onto , Building to work along with 'Netdiscover' for inhouse scanning to determine oddities  

# Input file containing list of IP addresses
IP_FILE="ips.txt"

# Check if file exists
if [[ ! -f "$IP_FILE" ]]; then
    echo "Input file not found!"
    exit 1
fi

# Function to perform WHOIS lookup
whois_lookup() {
    local ip=$1
    echo "=== WHOIS Information for IP: $ip ==="
    whois $ip | grep -E "OrgName|OrgId|Country|NetRange|CIDR|TechEmail|AbuseEmail"
}

# Function to get Geolocation info
geolocation_lookup() {
    local ip=$1
    echo "=== Geolocation Information for IP: $ip ==="
    
    # Using ipinfo.io API for Geolocation
    response=$(curl -s http://ipinfo.io/$ip/json)

    if [[ "$response" == *"\"bogon\""* ]]; then
        echo "IP address $ip is a bogon (invalid or reserved IP)"
    else
        echo $response | jq '.ip, .hostname, .city, .region, .country, .org, .loc'
    fi
}

# Iterate through each IP in the input file
while read -r ip; do
    echo "--------------------------------------------"
    echo "Processing IP: $ip"
    
    # Perform WHOIS and Geolocation lookups
    whois_lookup $ip
    geolocation_lookup $ip
    
    echo "--------------------------------------------"
    echo ""
done < "$IP_FILE"

echo "Incident Response Reconnaissance completed."
