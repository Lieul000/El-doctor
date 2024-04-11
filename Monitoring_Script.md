Monitoring_Script

This script appends a line to a file named mycron, which contains the cron job entry to execute the monitoring script every hour. It then installs this crontab file using crontab mycron and removes the temporary mycron file.

Make sure to replace /path/to/monitoring_script.sh with the actual path to your monitoring script. After adding this line, remember to make the script executable with chmod +x monitoring_script.sh.

````
#!/bin/bash

# Define the CSV file path
CSV_FILE="/path/to/metrics.csv"

# Date and Time
echo "## Server Monitoring Report - $(date +"%F %k:%M:%S %Z")" >> "$CSV_FILE"

# OS Information
echo "" >> "$CSV_FILE"
echo "## OS Information" >> "$CSV_FILE"
echo "  Hostname: $(hostname | cut -f 1 -d.)" >> "$CSV_FILE"
echo "  Architecture: $(uname -m)" >> "$CSV_FILE"
echo "  Kernel: $(uname -r)" >> "$CSV_FILE"

# Services
echo "" >> "$CSV_FILE"
echo "## Services" >> "$CSV_FILE"
sudo systemctl list-units --type=service | grep "running" | sed -e 's/loaded.*running.*/running/g' -e 's/.service//g' >> "$CSV_FILE"

# Internet Connectivity
echo "" >> "$CSV_FILE"
echo "## Internet Connectivity" >> "$CSV_FILE"
ping -c 1 google.com &> /dev/null && echo "  Status: Connected" || echo "  Status: Disconnected" >> "$CSV_FILE"

# IP Addresses
echo "" >> "$CSV_FILE"
echo "## IP Addresses" >> "$CSV_FILE"
echo "  Public IP: $(curl -s ipecho.net/plain;echo)" >> "$CSV_FILE"
echo "  Private IP: $(/sbin/ip -o -4 addr list enp0s8 | awk '{print $4}' | cut -d/ -f1)" >> "$CSV_FILE"

# DNS Server
echo "" >> "$CSV_FILE"
echo "## DNS Server" >> "$CSV_FILE"
cat /etc/resolv.conf | grep nameserver | awk '{print $2}' >> "$CSV_FILE"

# Network Services
echo "" >> "$CSV_FILE"
echo "## Network Services" >> "$CSV_FILE"
sudo ss -tlnp | tail -n+2 | tr -s ' ' | cut -d ' ' -f 1,4,7 | column -ts ' ' >> "$CSV_FILE"

# CPU Usage
echo "" >> "$CSV_FILE"
echo "## CPU Usage" >> "$CSV_FILE"
sar -P ALL -h 0 | tail -n +3 | tr -s ' ' | tr '[:lower:]' '[:upper:]' | cut -d ' ' -f 2,3,5,8 | sed -e '1 a \ ' -e 's/ALL/\*/g' | column -tes ' ' >> "$CSV_FILE"

# Memory Usage
echo "" >> "$CSV_FILE"
echo "## Memory Usage" >> "$CSV_FILE"
egrep --color 'Mem|Cache|Swap' /proc/meminfo >> "$CSV_FILE"

# Storage Usage
echo "" >> "$CSV_FILE"
echo "## Storage Usage" >> "$CSV_FILE"
df -h >> "$CSV_FILE"

# Disk I/O
echo "" >> "$CSV_FILE"
echo "## Disk I/O" >> "$CSV_FILE"
iostat -d, >> "$CSV_FILE"

# Schedule execution using cron
echo "0 * * * * /bin/bash /path/to/monitoring_script.sh" >> mycron
crontab mycron
rm mycron

```