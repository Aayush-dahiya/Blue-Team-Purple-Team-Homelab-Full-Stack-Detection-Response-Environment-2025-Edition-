# Getting data in

Docs: [https://help.splunk.com/en/splunk-enterprise/get-started/get-data-in/9.3/introduction/get-started-with-getting-data-in](https://help.splunk.com/en/splunk-enterprise/get-started/get-data-in/9.3/introduction/get-started-with-getting-data-in)

Here are some questions that need to be answered before moving forward.

| **Question** | **Your Answer (Based on Your Setup)** |
| --- | --- |
| **1. What kind of data do I want to index?** | Windows Sysmon logs, Network connection logs (via Sysmon + Defender/OpenEDR), Linux system logs, Apache web server logs. |
| **2. Is there an app for that?** | Yes — Sysmon TA, Windows TA, Linux TA, Apache TA.  |
| **3. Where does the data reside? (Local or remote?)** | All logs are stored locally on VMs in the lab environment. |
| **4. Should I use forwarders to access remote data?** | Yes, i will use forwarders. |
| **5. What do I want to do with the indexed data?** | Create dashboards, build alerts, investigate malicious activity, analyze telemetry, build SOC detections. |

Let’s start with sending windows data.

## Windows Setup

### Installing sysmon.

I downloaded the sysmon zip from the sysinternals site (microsoft)

```bash
https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
```

and i got the configuration file from swiftonsecurity github.

```bash
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml
```

then i ran the following command from admin terminal to install

```powershell
./sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

Now let us install the universal forwarder.

Very basic - just put the ip of the Splunk server in deployment and receiving index.  

### Preparing the Sysmon Add-on for Splunk

I downloaded the Splunk Add-on for Sysmon (the TA, not the dashboard app) from Splunkbase. After unzipping it, I verified the folder structure:

```
Splunk_TA_sysmon/
    default/
    metadata/
```

This add-on parses Sysmon logs from Universal Forwarders.

---

### Installing the Sysmon Add-on on the Deployment Server

My Splunk server runs on Debian, so I placed the Sysmon TA into the deployment-apps directory:

```
/opt/splunk/etc/deployment-apps/Splunk_TA_sysmon
```

I made sure it wasn't nested inside another folder, then set the correct permissions:

```
sudo chown -R splunk:splunk /opt/splunk/etc/deployment-apps/Splunk_TA_sysmon
```

---

### Enabling Sysmon Input in the Add-on

Inside the add-on, I created a local directory:

```
/opt/splunk/etc/deployment-apps/Splunk_TA_sysmon/local
```

Then I created an inputs.conf file inside that folder:

```
/opt/splunk/etc/deployment-apps/Splunk_TA_sysmon/local/inputs.conf
```

I added the following content:

```
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

This configuration tells the Universal Forwarder to send Sysmon events to the sysmon index.

---

### Deploying the Sysmon Add-on to the Universal Forwarder

In Splunk Web, I went to Settings → Forwarder Management. I created a server class called windows-sysmon, added the Sysmon TA to it, and added my Windows Universal Forwarder host to that server class.

Then I reloaded the deployment server to propagate the changes:

```
sudo /opt/splunk/bin/splunk reload deploy-server
```

---

### Allowing the Universal Forwarder to Read Sysmon Logs

The Universal Forwarder needs permission to read the Sysmon event log channel. By default, the SplunkForwarder service cannot access this channel when running under a normal user or Network Service account.

To fix this, I opened services.msc on the Windows machine, found the SplunkForwarder service, and opened its Properties. I went to the Log On tab and changed it to run as Local System. After applying the change, I restarted the service.

This step is essential—without this permission, the forwarder cannot read the Sysmon event channel.

---

### Forcing the UF to Check In

To ensure the Universal Forwarder picked up the new configuration, I restarted it:

```
Restart-Service SplunkForwarder
```

Restarting triggers the UF to check in with the deployment server.

---

### Creating the Sysmon Index in Splunk

In Splunk Web, I went to Settings → Indexes and created a new index named sysmon. This index must exist before Sysmon events can be stored, since inputs.conf references it.

---

### Verifying the Forwarder Received the Configuration

On the Windows machine, I checked the forwarder's inputs configuration using btool:

```
"C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" btool inputs list --debug | Select-String Sysmon
```

I confirmed that the Sysmon stanza appeared with disabled set to 0.

---

### Checking That Sysmon Logs Arrive in Splunk

I checked the index using `index=”sysmon”` and there were events present there.

# Linux Server Setup + Apache

## linux logging setup – what i did

I started with Ubuntu Server Minimal, which relies entirely on journald and does not create the usual log files under /var/log. Since Splunk Universal Forwarder cannot read journald logs, I needed traditional text-based log files. To fix that, I installed rsyslog. Once rsyslog was installed and running, Ubuntu immediately began writing standard logs into /var/log, such as syslog, auth.log, kern.log, dpkg logs, and apt logs. These files gave me the Linux visibility I needed.

---

### sysmon for linux installation – what i did

I installed Sysmon for Linux directly from Microsoft’s package repository using my actual commands. These were the exact lines I ran:

```
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get update
sudo apt-get install sysmonforlinux

```

This added the Microsoft apt repository to my system and allowed me to install sysmonforlinux through apt without any manual compilation.

After installing Sysmon, I downloaded the MSTIC Sysmon configuration file from their public GitHub repository. With the config file in place, I initialized Sysmon using:

```
sudo sysmon -i /path/to/sysmon.conf

```

Sysmon began running immediately. In my setup, rsyslog automatically forwarded Sysmon’s output into `/var/log/syslog` without requiring any manual configuration. Because of this, I did not need to add any extra journald rules or rsyslog routing rules. Sysmon events appeared directly in syslog alongside normal system messages.

---

### creating the splunk index – what i did

On the Splunk server, I created a dedicated index called `linux_hosts`. This index was meant to store all Linux operating system logs, including Sysmon events. I created the index in Splunk Web under Settings → Indexes, then specified `linux_hosts` in my Universal Forwarder configuration.

---

### configuring the splunk universal forwarder – what i did

Once the Linux logs and Sysmon logs were available under /var/log, I configured Splunk Universal Forwarder to send them to the `linux_hosts` index. I created my inputs.conf under:

```
/opt/splunkforwarder/etc/system/local/inputs.conf

```

Inside the file, I monitored the logs I wanted and sent them all to the `linux_hosts` index. The configuration matched the log files that existed on my system, which included auth.log, dpkg.log, syslog, kern.log, and apt history.

After saving inputs.conf, I restarted the Splunk Universal Forwarder. The forwarder then began sending all of these logs, including Sysmon events inside syslog, directly into my `linux_hosts` index on the Splunk server.

## 

# DVWA Setup on Ubuntu for My SOC Lab

To add a vulnerable web application to my SOC environment, I set up DVWA (Damn Vulnerable Web App) on Ubuntu using Apache, PHP, and MariaDB. This setup generates predictable, attack-friendly web traffic that I can send to Splunk for monitoring and analysis.

---

### Installing Apache, PHP, and MariaDB

I started by updating the server and installing all required components: Apache as the web server, PHP with its needed modules, and MariaDB as the database backend.

```
sudo apt update
sudo apt upgrade -y

sudo apt install apache2 -y
sudo apt install php php-mysqli php-gd php-xml php-pdo php-zip php-mbstring -y
sudo apt install mariadb-server -y
```

I enabled and started both services:

```
sudo systemctl enable apache2
sudo systemctl enable mariadb
sudo systemctl start apache2
sudo systemctl start mariadb
```

I verified Apache was running by checking its status.

---

### Configuring MariaDB for DVWA

Inside MariaDB, I created the DVWA database and a dedicated user account.

```
sudo mysql

CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

This gave DVWA its own database for Apache/PHP to connect to during installation.

---

### Downloading and Preparing DVWA

I moved to the Apache web root, removed the default index file, and cloned DVWA directly from GitHub.

```
cd /var/www/html
sudo rm index.html
sudo git clone https://github.com/digininja/DVWA.git
```

I renamed the folder to make the path cleaner:

```
sudo mv DVWA dvwa
```

I adjusted ownership and permissions so Apache (www-data) could read and write where needed.

```
sudo chown -R www-data:www-data /var/www/html/dvwa
sudo chmod -R 755 /var/www/html/dvwa
```

DVWA was now located at `/var/www/html/dvwa`.

---

### Configuring DVWA

I switched to the DVWA config directory and copied the template configuration file.

```
cd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
```

Inside `config.inc.php`, I set:

```
$_DVWA['db_user'] = 'dvwa';
$_DVWA['db_password'] = 'password';
$_DVWA['db_database'] = 'dvwa';
$_DVWA['db_server'] = 'localhost';
```

This allowed DVWA to communicate with the MariaDB instance I set up earlier.

---

### Adjusting PHP Settings for DVWA

DVWA requires two PHP settings to be enabled. I edited the main PHP Apache configuration:

```
sudo nano /etc/php/*/apache2/php.ini
```

I set the following to On:

```
allow_url_fopen = On
allow_url_include = On
```

Then I restarted Apache:

```
sudo systemctl restart apache2
```

---

### Setting Insecure Permissions Required by DVWA

DVWA needs write access to specific directories. Since the application is intentionally vulnerable, it requires insecure permissions on some folders.

```
sudo chmod -R 777 /var/www/html/dvwa/hackable/uploads
sudo chmod -R 777 /var/www/html/dvwa/external/phpids/0.6/lib/IDS/tmp
```

These permissions allowed DVWA's upload and IDS components to function properly.

---

### Completing DVWA Setup in the Browser

I accessed DVWA in the browser:

```
http://<server-ip>/dvwa
```

I logged in using:

```
admin / password
```

Inside the DVWA interface, I navigated to Setup and clicked "Create / Reset Database." DVWA connected to the MariaDB database and completed the installation.

---

### Enhancing Apache Logging for SOC Use

To get rich HTTP logs for Splunk, I adjusted Apache's log format. I opened Apache's main configuration file:

```
sudo nano /etc/apache2/apache2.conf
```

I verified or added this LogFormat line:

```
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %{X-Forwarded-For}i %{Host}i" combined

```

This format logs:

- Client IP
- Full request line
- User agent
- Referrer
- Host header
- X-Forwarded-For
- Status code
- Bytes returned

These fields are valuable for web attack analysis in Splunk.

I restarted Apache:

```
sudo systemctl restart apache2
```

Apache now writes detailed logs to:

```
/var/log/apache2/access.log
/var/log/apache2/error.log
```

---

### Forwarding Apache Logs to Splunk

I created a separate index in Splunk called `apache_server` to store all Apache logs.

On the Ubuntu machine running the Splunk Universal Forwarder, I edited the inputs file:

```
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

I added:

```
[monitor:///var/log/apache2/access.log]
index = apache_server

[monitor:///var/log/apache2/error.log]
index = apache_server
```

Then I restarted the Splunk Universal Forwarder:

```
sudo systemctl restart splunkforwarder
```

All Apache logs began flowing into Splunk under the `apache_server` index.

---

### Testing and Verifying Apache Logging with DVWA Attacks

To verify that logs were reaching Splunk, I performed several attacks inside DVWA:

- Brute-force login attempts
- SQL injection (`' OR 1=1 --`)
- XSS payloads
- Directory traversal
- Command injection
- File upload tests

These actions generated the expected entries in `access.log` and `error.log`. I confirmed that the events appeared in the `apache_server` index in Splunk.

## Host Heartbeat Logging for My SOC Lab

To continuously verify that my Linux host is alive and sending data, I implemented a heartbeat logging mechanism. This creates a small log entry every minute, which is forwarded to Splunk. It allows me to detect when a system becomes unresponsive and build alerts around missing heartbeats.

---

### Creating the Heartbeat Script

I created a simple script that writes a timestamped "alive" message to a dedicated log file. This script runs on a schedule and produces a consistent log line that Splunk can ingest.

I created the script at:

```
sudo nano /usr/local/bin/heartbeat.sh
```

The script contains:

```bash
#!/bin/bash
echo "$(date -u +"%Y-%m-%dT%H:%M:%SZ") host=$(hostname) status=alive" >> /var/log/heartbeat.log
```

I made it executable:

```
sudo chmod +x /usr/local/bin/heartbeat.sh
```

I also created the log file and set permissions to allow the script to write to it:

```
sudo touch /var/log/heartbeat.log
sudo chmod 666 /var/log/heartbeat.log
```

---

### Scheduling the Heartbeat

I configured the heartbeat to run every minute. I added a cron job:

```
sudo crontab -e
```

And inserted:

```
* * * * * /usr/local/bin/heartbeat.sh
```

This ensures the script writes a new line to `/var/log/heartbeat.log` once every minute.

---

### Forwarding the Heartbeat Log to Splunk

Since the Splunk Universal Forwarder was already running on the host, I added the heartbeat log to my existing inputs file for automatic forwarding.

I edited:

```
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

And added:

```
[monitor:///var/log/heartbeat.log]
index = linux_hosts
sourcetype = linux_heartbeat
```

I restarted the forwarder:

```
sudo systemctl restart splunkforwarder
```

Splunk immediately began receiving heartbeat events in the `linux_hosts` index.

---

### Verifying Heartbeat Events Inside Splunk

To confirm ingestion, I ran a simple search:

```
index=linux_hosts sourcetype=linux_heartbeat
```

I saw one event per minute, each including a timestamp, hostname, and status field. Switching the Splunk time picker to a real-time window let me watch heartbeats appear live as they were generated.

---

### Alerting on Missing Heartbeats

To detect when a host stops sending heartbeats, I created an alert in Splunk that checks whether the latest heartbeat is older than a threshold. This lets me identify outages, crashes, or forwarder failures quickly.