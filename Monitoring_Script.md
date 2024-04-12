Monitoring_Script

This script appends a line to a file named mycron, which contains the cron job entry to execute the monitoring script every hour. It then installs this crontab file using crontab mycron and removes the temporary mycron file.

Make sure to replace /path/to/monitoring_script.sh with the actual path to your monitoring script. After adding this line, remember to make the script executable with chmod +x monitoring_script.sh.

For example, if you want to create a CSV file named metrics.csv in the /home/user/data directory, the path to the CSV file would be /home/user/data/metrics.csv.

Here's a summary of the steps:

Choose directory: /home/user/data
Provide file name: metrics.csv
Combine directory path and file name: /home/user/data/metrics.csv
You can use this complete path (/home/user/data/metrics.csv) in your script to specify where the CSV file should be saved. Replace /home/user/data with the actual directory path you want to use.

````
#!/bin/bash

# This script monitors server metrics and sends email notifications for critical conditions.

# Define the CSV file path
CSV_FILE="/path/to/metrics.csv"

# Define your email address
EMAIL="your_email@example.com"

# Function to send email notification
send_notification() {
    local subject="$1"
    local message="$2"
    echo "$message" | mail -s "$subject" "$EMAIL"
}

# Date and Time
# Append server monitoring report timestamp to CSV file
echo "## Server Monitoring Report - $(date +"%F %k:%M:%S %Z")" >> "$CSV_FILE"

# OS Information
# Append OS information to CSV file
echo "" >> "$CSV_FILE"
echo "## OS Information" >> "$CSV_FILE"
echo "  Hostname: $(hostname | cut -f 1 -d.)" >> "$CSV_FILE"
echo "  Architecture: $(uname -m)" >> "$CSV_FILE"
echo "  Kernel: $(uname -r)" >> "$CSV_FILE"

# Internet Connectivity
# Check internet connectivity and append status to CSV file
echo "" >> "$CSV_FILE"
echo "## Internet Connectivity" >> "$CSV_FILE"
ping -c 1 google.com &> /dev/null && echo "  Status: Connected" || echo "  Status: Disconnected" >> "$CSV_FILE"

# CPU Usage
# Check CPU usage and append to CSV file
echo "" >> "$CSV_FILE"
echo "## CPU Usage" >> "$CSV_FILE"
sar -P ALL -h 0 | tail -n +3 | tr -s ' ' | tr '[:lower:]' '[:upper:]' | cut -d ' ' -f 2,3,5,8 | sed -e '1 a \ ' -e 's/ALL/\*/g' | column -tes ' ' >> "$CSV_FILE"
cpu_usage=$(sar -P ALL 1 1 | grep -E '^Average:' | awk '{print 100 - $NF}')
echo "CPU Usage: $cpu_usage%" >> "$CSV_FILE"
if [ $cpu_usage -gt 80 ]; then
    send_notification "CPU Usage Alert" "CPU usage is above 80%."
fi

# Memory Usage
# Check memory usage and append to CSV file
echo "" >> "$CSV_FILE"
echo "## Memory Usage" >> "$CSV_FILE"
mem_total=$(grep MemTotal /proc/meminfo | awk '{print $2}')
mem_used=$(grep MemAvailable /proc/meminfo | awk '{print $2}')
mem_percent=$((($mem_total - $mem_used) * 100 / $mem_total))
echo "Memory Usage: $mem_percent%" >> "$CSV_FILE"
if [ $mem_percent -gt 80 ]; then
    send_notification "Memory Usage Alert" "Memory usage is above 80%."
fi

# Storage Usage
# Check storage usage and append to CSV file
echo "" >> "$CSV_FILE"
echo "## Storage Usage" >> "$CSV_FILE"
df -h >> "$CSV_FILE"
storage_usage=$(df --output=pcent / | tail -n 1 | sed 's/%//')
if [ $storage_usage -gt 80 ]; then
    send_notification "Storage Usage Alert" "Storage usage is above 80%."
fi

# Send weekly report via email
# If it's Sunday, send the weekly server monitoring report via email
if [ $(date +%u) -eq 7 ]; then
    mail -s "Weekly Server Monitoring Report" "$EMAIL" < "$CSV_FILE"
fi

# Schedule execution using cron
# Add cron job to execute this script every hour
echo "0 * * * * /bin/bash /path/to/monitoring_script.sh" >> mycron
crontab mycron
rm mycron

```
