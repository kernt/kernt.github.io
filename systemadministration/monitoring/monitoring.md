With this script in place, you’ll have one comprehensive solution for monitoring everything on your Linux system.

Linux system administrators often face the challenge of monitoring numerous performance metrics, logs, and system health parameters.

Traditionally, you’d rely on a variety of tools and scripts to gather data on disk usage, CPU performance, memory status, network traffic, and system logs. But what if you could have **one script** to monitor everything on your Linux system?

In this blog post, we’ll show you how to set up a **single monitoring script** that gives you insights into every crucial aspect of your Linux system.

From CPU usage to disk space, memory consumption to network activity,  
In this script will provide a one-stop solution for your system monitoring needs.

Let’s dive into it!
# Warum ist Monitoring Critical

Monitoring deines Linux System ist Kritisch aus mehren gründen:

1. **System Health**: Proactively identifying issues before they become critical can help avoid system downtime.
2. **Performance Optimization**: Monitoring resource usage allows you to adjust your system to ensure peak performance.
3. **Security**: Regular monitoring helps detect abnormal patterns that could indicate security breaches or attacks.
4. **Log Management**: Effective monitoring includes tracking logs to troubleshoot any errors or warnings promptly.

While there are numerous tools like **top**, **htop**, **iftop**, and **df**, managing multiple monitoring solutions can become cumbersome. With a single script, you simplify your workflow and have a centralized point of reference for your system’s health.

# Setting Up a One-Script Solution

Before creating the script, let’s outline what it needs to monitor:

1. **CPU Usage**: Keep an eye on your system’s processor performance.
2. **Memory Usage**: Monitor RAM and swap usage.
3. **Disk Usage**: Ensure that your file systems aren’t getting too full.
4. **Network Activity**: Track incoming and outgoing traffic.
5. **Running Processes**: Watch for any rogue or resource-hogging processes.
6. **System Logs**: Continuously monitor critical log files for errors or warnings.
# Install Necessary Packages:

**für Ubuntu/Debian**

`sudo apt-get install sysstat ifstat -y`  
  
**Für neuere Red Hat versions mit dnf**

`sudo dnf install sysstat ifstat -y`  
  
# For older versions with yum
sudo yum install sysstat ifstat -y

![](https://miro.medium.com/v2/resize:fit:700/1*tHBvunGn5SjQHX2x9JfO9w.png)
# The Complete Monitoring Script

Here’s a comprehensive script that combines all the above metrics into one output. You can schedule this to run at intervals or execute it manually when needed.

## Installation Instrcutions

<<<<<<< HEAD:Systemadministartion/systemadministration/monitoring/monitoring.md
**Create directory for script**

`sudo mkdir -p /opt/script/`
  
**Create the new file, and copy the below script**

`sudo vim /opt/script/monitoring.sh`
  
**Permission for Execute**

`sudo chmod +x /opt/script/monitoring.sh` 
  
# Excute the scritp manually

/opt/script/monitoring.sh 

oder
# Create directory for script

sudo mkdir -p /opt/script/  
# Create the new file, and copy the below script

sudo vim /opt/script/monitoring.sh
  
# Permission for Execute

sudo chmod +x /opt/script/monitoring.sh
  
# Excute the scritp manually

/opt/script/monitoring.sh
(or)

cd /opt/script && ./monitoring.sh
# cat /opt/script/monitoring.sh
  
```sh
#!/bin/bash
# Colors for readability
GREEN='\033[0;32m'  
YELLOW='\033[1;33m'  
RED='\033[0;31m'  
NC='\033[0m' # No Color  
echo -e "${GREEN}===== System Monitoring Script =====${NC}"  
# 1. CPU Usage  
echo -e "${YELLOW}\n>> CPU Usage: ${NC}"  
mpstat | awk '/all/ {print "CPU Load: " $3 "% idle"}'  
# 2. Memory Usage  
echo -e "${YELLOW}\n>> Memory Usage: ${NC}"  
free -h | awk '/Mem/ {print "Total Memory: " $2 "\nUsed: " $3 "\nFree: " $4}'  
echo -e "Swap:\n"$(free -h | awk '/Swap/ {print "Total: " $2 ", Used: " $3 ", Free: " $4}')  
# 3. Disk Usage  
echo -e "${YELLOW}\n>> Disk Usage: ${NC}"  
df -h | grep '^/dev' | awk '{print $1 ": " $5 " used, " $4 " available"}'  
# 4. Network Traffic  
echo -e "${YELLOW}\n>> Network Traffic: ${NC}"  
ifstat -i eth0 1 1 | awk 'NR==3 {print "RX: " $1 " KB/s, TX: " $2 " KB/s"}'  
# 5. Top 5 Memory Consuming Processes  
echo -e "${YELLOW}\n>> Top 5 Memory Consuming Processes: ${NC}"  
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 6  
# 6. Top 5 CPU Consuming Processes  
echo -e "${YELLOW}\n>> Top 5 CPU Consuming Processes: ${NC}"  
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6  
# 7. System Logs Monitoring  
echo -e "${YELLOW}\n>> Recent Errors in System Logs: ${NC}"  
journalctl -p 3 -xb | tail -n 10  
echo -e "${GREEN}===== Monitoring Completed =====${NC}"
```


# Breakdown of the Script

1. **CPU Usage**: We use `mpstat` to get CPU load, particularly focusing on how much idle time the CPU has. If this value is low, your system is under heavy load.
2. **Memory Usage**: The `free -h` command gives a human-readable summary of memory and swap usage. High memory usage can indicate an application using excessive resources.
3. **Disk Usage**: `df -h` shows disk space usage for each partition. Disk space running low can cause performance degradation and system crashes, so this is crucial.
4. **Network Traffic**: Using `ifstat`, the script monitors the incoming and outgoing network traffic on a specific interface (in this case, `eth0`).
5. **Top 5 Memory and CPU Consuming Processes**: With `ps`, the script lists the top 5 processes that are using the most memory and CPU, helping you pinpoint resource-heavy tasks.
6. **System Logs Monitoring**: The `journalctl` the command displays recent errors from system logs, helping you identify issues that might not yet be affecting system performance but are critical to investigate.
# Automating the Monitoring Script

While manually running this script is useful, it becomes even more powerful when set to run automatically at intervals. You can do this using **cron**, the task scheduler built into Linux. For more about [crontab check out here.](https://medium.com/devsecops-community/day-10-scheduling-tasks-with-cron-jobs-in-linux-e298d17fc0e9)
# Method 1: Setting Up a Cron Job

To schedule the script to run, say every hour, follow these steps:

Open your crontab file:

`crontab -e`

Add the following line to schedule the script to run hourly:

`0 * * * * /path/to/your_script.sh >> /var/log/system_monitor.log`

This will execute the script every hour on the hour and log the output to a file.

_Note: You can the crontab time as per your requirement._

# Send logs to Email via the Command line

Now, the server has the logs, but we need to log in and check the files every time, which is not a feasible solution.

Let’s automate via Email.

## Requirements:

1. Postfix — ( [Email configure for linux system check here](https://medium.com/devsecops-community/configure-office-365-email-on-centos-using-postfix-d682ffff6efc))
2. mailutils

**For ubuntu**

`sudo apt-get install mailutils -y`
  
**für Redhat/Centos**

`sudo yum install mailx -y`
## Method 1:

The command below will send our monitoring logs to email. You can change the interval to meet your requirements.

0 * * * tail -n 25 /var/log/system_monitor.log | mail -s "Daily Report" -a /var/log/system_monitor.log recipient@example.com

## Method 2: Re-write the script directly Send to Email

Here’s is alternative script, to send logs directly to Email. Same as the previous one.

```sh
#!/bin/bash
# Output file in HTML format
OUTPUT_FILE="/opt/script/monitoring_report.html"
# HTML Header
echo "<html>  
<head>  
    <title>System Monitoring Report</title>  
    <style>  
        body { font-family: Arial, sans-serif; }  
        h2 { color: #2E8B57; }  
        .section { margin-top: 20px; padding: 10px; }  
        .cpu { color: #FFA500; }  
        .memory, .disk, .network, .processes, .logs { color: #4682B4; }  
        .error { color: #B22222; font-weight: bold; }  
        pre { background-color: #f8f9fa; padding: 10px; border-radius: 5px; }  
    </style>  
</head>  
<body>  
    <h1 style='color: #2E8B57;'>System Monitoring Report</h1>  
    <h2>Date: $(date)</h2>" > "$OUTPUT_FILE"  
# CPU Usage  
echo "<div class='section cpu'><h2>CPU Usage:</h2><pre>" >> "$OUTPUT_FILE"  
mpstat | awk '/all/ {print "CPU Load: " $3 "% idle"}' >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# Memory Usage  
echo "<div class='section memory'><h2>Memory Usage:</h2><pre>" >> "$OUTPUT_FILE"  
free -h | awk '/Mem/ {print "Total Memory: " $2 "\nUsed: " $3 "\nFree: " $4}' >> "$OUTPUT_FILE"  
echo -e "\nSwap:\n$(free -h | awk '/Swap/ {print "Total: " $2 ", Used: " $3 ", Free: " $4}')" >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# Disk Usage  
echo "<div class='section disk'><h2>Disk Usage:</h2><pre>" >> "$OUTPUT_FILE"  
df -h | grep '^/dev' | awk '{print $1 ": " $5 " used, " $4 " available"}' >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# Network Traffic  
echo "<div class='section network'><h2>Network Traffic:</h2><pre>" >> "$OUTPUT_FILE"  
ifstat -i eth0 1 1 | awk 'NR==3 {print "RX: " $1 " KB/s, TX: " $2 " KB/s"}' >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# Top 5 Memory Consuming Processes  
echo "<div class='section processes'><h2>Top 5 Memory Consuming Processes:</h2><pre>" >> "$OUTPUT_FILE"  
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 6 >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# Top 5 CPU Consuming Processes  
echo "<div class='section processes'><h2>Top 5 CPU Consuming Processes:</h2><pre>" >> "$OUTPUT_FILE"  
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6 >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# System Logs Monitoring  
echo "<div class='section logs'><h2>Recent Errors in System Logs:</h2><pre class='error'>" >> "$OUTPUT_FILE"  
journalctl -p 3 -xb | tail -n 10 >> "$OUTPUT_FILE"  
echo "</pre></div>" >> "$OUTPUT_FILE"  
# HTML Footer  
echo "</body></html>" >> "$OUTPUT_FILE"  
```
# Send the email

mail -s "System Monitoring Report" -a "Content-Type: text/html" recipient@example.com < "$OUTPUT_FILE"
# Visualizing Your Data

While the script outputs information to your terminal or logs, you may want to visualize the data, especially if you’re managing a production server. For this, you can integrate the script with tools like **Grafana** and **Prometheus**. These tools collect and graph data, giving you an interactive, visual representation of your system’s performance over time.
# Enhancing Security with Monitoring

System monitoring isn’t just about performance; it also plays a vital role in **security**. For example:

- **Log Monitoring**: Keeping an eye on system logs helps you detect suspicious activities like repeated login attempts or unauthorized access.
- **Process Monitoring**: Watching the processes on your system can reveal malware or unnecessary background tasks.
- **Network Traffic**: Monitoring incoming and outgoing traffic may reveal abnormal patterns or potential security breaches.

Having a single script to monitor everything on your Linux system saves you time and makes the monitoring process more efficient. This approach combines multiple critical system checks into one easy-to-manage script, giving you a quick overview of your system’s health at any given moment.

By automating this process with cron jobs and visualizing the data with tools like Grafana, you can ensure that your system is running smoothly and identify issues before they escalate into critical problems.

**Helpful Tip**: Customize this script based on your system’s unique needs. You can add other checks, such as temperature monitoring for hardware or I/O statistics for high-performance environments.

# Mastering Log Monitoring on Linux: Automate with Shell Scripts for Real-Time Alerts

Learn how to efficiently monitor logs on Linux systems using shell scripting. Detect critical events, automate alerts, and keep your infrastructure running smoothly.

![](https://miro.medium.com/v2/resize:fit:700/1*rsxxyHnO7vXahjWjNIjpcQ.png)

Hey Mate! Welcome to my 92nd blog post. In this post, we are going to discuss monitoring using shell scrip for Linux machines

Here’s a **Linux shell script** that monitors key logs — `journalctl`, `secure.log`, and `auth.log`—for critical events or errors. It scans for **new suspicious entries** and sends an **email alert** to the IT admin if anything concerning is found.

e’s write a script!
# The Monitoring Script (alert_monitor.sh)

```sh
#!/bin/bash  
# Set variables  
ADMIN_EMAIL="admin@example.com"  # Replace with your IT admin's email  
TEMP_LOG="/tmp/critical_log_monitor.txt"  # Temporary log file to store results  
DATE=$(date +"%Y-%m-%d %H:%M:%S")  # Current timestamp  
# Function to search for errors and critical events in logs  
function check_logs() {  
    echo "Critical Log Monitoring Report - $DATE" > "$TEMP_LOG"  
    echo "----------------------------------------" >> "$TEMP_LOG"  
    # Check journalctl for critical errors and alerts  
    echo "[journalctl - Critical Errors]" >> "$TEMP_LOG"  
    journalctl -p 3 -n 50 --no-pager >> "$TEMP_LOG"  # Logs with priority 3 (Error)  
    echo "" >> "$TEMP_LOG"  
    # Check for failed login attempts or suspicious activities in secure.log  
    echo "[/var/log/secure - Failed Logins & Suspicious Activity]" >> "$TEMP_LOG"  
    grep -Ei 'failed|invalid|unauthorized|error' /var/log/secure | tail -n 50 >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
    # Check for failed authentications or privilege escalations in auth.log  
    echo "[/var/log/auth.log - Auth Issues & Privilege Escalation]" >> "$TEMP_LOG"  
    grep -Ei 'failure|authentication|sudo:.*(error|failure)' /var/log/auth.log | tail -n 50 >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
    # Display critical syslog events (replace with your log path if needed)  
    echo "[/var/log/syslog - Critical Events]" >> "$TEMP_LOG"  
    grep -Ei 'kernel:.*error|panic|segfault|critical' /var/log/syslog | tail -n 50 >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
}  
# Function to send email alerts  
function send_alert() {  
    if [[ -s $TEMP_LOG ]]; then  # Check if log file is not empty  
        mail -s "⚠️ Critical Log Alert - $DATE" "$ADMIN_EMAIL" < "$TEMP_LOG"  
        echo "Alert email sent to $ADMIN_EMAIL."  
    else  
        echo "No critical events found. No email sent."  
    fi  
}  
# Run log checks and send alerts  
check_logs  
send_alert  
# Clean up temporary file  
rm -f "$TEMP_LOG"
```

# Explanation of the Script:

**Variables:**

- `ADMIN_EMAIL`: Replace this with the email address of your IT admin.
- `TEMP_LOG`: A temporary file to store log results.

**Log Analysis:**

- `**journalctl**`: Scans for logs with priority `3` (Error).
- `**secure.log**`: Searches for failed login attempts or suspicious activity.
- `**auth.log**`: Checks for failed authentication attempts or privilege escalations.
- `**syslog**`: Looks for critical system events like kernel panics or segmentation faults.

**Sending the Alert:**

- If any critical log entry is found, it sends an **email alert** using the `mail` command.
- If no critical entries are found, it simply exits without sending an email.

**Cleanup:**

- After sending the alert, the temporary log file is deleted.

# Setup Instructions:

**Install** `**mailutils**` (if not already installed):

sudo apt update && sudo apt install mailutils

**Grant Execution Permission:**

chmod +x alert_monitor.sh

**Schedule the Script with Cron (Optional):**  
Run the script periodically using **cron**. For example, to run every 10 minutes:

crontab -e

- Add the following line:

*/10 * * * * /path/to/alert_monitor.sh

## Here is the guide for sending Email from the command Line

# esting the Script:

Simulate a failed login attempt to generate a log entry:

ssh invaliduser@localhost

Run the script manually to ensure it captures the error:

./alert_monitor.sh

# Final Notes:

This script provides a simple way to **monitor key logs** for issues and send **email alerts** to the admin in case of critical events. You can extend it to monitor additional logs or customize the grep patterns to match your environment.

Happy monitoring! 🚀

Here’s an improved version of the script that **only filters critical events** rather than displaying the last 50 lines. The filtering logic has been updated to extract **critical logs** dynamically, ensuring you focus only on relevant incidents.

```sh
#!/bin/bash  
  
# Set variables  
ADMIN_EMAIL="admin@example.com"  # Replace with your IT admin's email  
TEMP_LOG="/tmp/critical_log_monitor.txt"  # Temporary log file to store results  
DATE=$(date +"%Y-%m-%d %H:%M:%S")  # Current timestamp  
  
# Function to check logs for critical events  
function check_logs() {  
    echo "Critical Log Monitoring Report - $DATE" > "$TEMP_LOG"  
    echo "----------------------------------------" >> "$TEMP_LOG"  
  
    # Check journalctl for critical (priority 2) and error (priority 3) events  
    echo "[journalctl - Critical & Error Events]" >> "$TEMP_LOG"  
    journalctl -p 0..3 --no-pager --since "10 minutes ago" >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
  
    # Check secure.log for failed or suspicious logins  
    echo "[/var/log/secure - Critical Login Events]" >> "$TEMP_LOG"  
    grep -Ei 'critical|failed|unauthorized|error|invalid' /var/log/secure | grep "$(date '+%b %d')" >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
  
    # Check auth.log for authentication failures and privilege issues  
    echo "[/var/log/auth.log - Critical Auth Issues]" >> "$TEMP_LOG"  
    grep -Ei 'critical|failure|authentication error|sudo: .*error' /var/log/auth.log | grep "$(date '+%b %d')" >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
  
    # Check syslog for kernel panic, segmentation faults, or critical system errors  
    echo "[/var/log/syslog - Critical System Events]" >> "$TEMP_LOG"  
    grep -Ei 'kernel: .*error|panic|segfault|critical' /var/log/syslog | grep "$(date '+%b %d')" >> "$TEMP_LOG"  
    echo "" >> "$TEMP_LOG"  
}  
  
# Function to send email alerts  
function send_alert() {  
    if [[ -s $TEMP_LOG ]]; then  # Check if the log file contains any content  
        mail -s "⚠️ Critical Log Alert - $DATE" "$ADMIN_EMAIL" < "$TEMP_LOG"  
        echo "Alert email sent to $ADMIN_EMAIL."  
    else  
        echo "No critical events found. No email sent."  
    fi  
}  
  
# Run log checks and send alerts  
check_logs  
send_alert  
  
# Clean up temporary file  
rm -f "$TEMP_LOG"
```

# Explanation of Changes:

`**journalctl**` **Filtering:**

- `journalctl -p 0..3`: Filters **priority levels** from 0 (Emergency) to 3 (Error).
- **Time filter**: `--since "10 minutes ago"` ensures only recent critical events are captured.

**Secure and Auth Log Filters:**

- The script only searches for **today’s logs** by matching the current date (`$(date '+%b %d')`).
- **Error keywords** such as “critical”, “failed”, “unauthorized”, and “invalid” are used to capture significant events.

**Syslog Filter:**

- Searches for **critical events** like kernel errors, segmentation faults, or panic events.

**Efficient Email Alerting:**

- `**if [[ -s $TEMP_LOG ]]**`: Sends the email only if the log file is not empty.
