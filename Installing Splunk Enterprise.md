# Installing Splunk Enterprise

I went to the splunk site and signed up to download the splunk enterprise. I wanted easier setup so i went with the wget link for the .deb package ( Nov 18, 2025). 

```bash
wget -O splunk-10.0.2-e2d18b4767e9-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/10.0.2/linux/splunk-10.0.2-e2d18b4767e9-linux-amd64.deb"

```

Before executing this, i got ssh access the ubuntu server, because i didn’t want to install guest additions and work with shared clipboard.

when executed, this will download the enterprise from the link and save the file with the name **splunk-10.0.2-e2d18b4767e9-linux-amd64.deb**

Before installing i will add my user splunk to the sudoers file for better access management.

### Installing the .deb

### **Installation procedure**

- Run the `dpkg` installer with the Splunk Enterprise Debian package name as an argument.
    
    `dpkg -i splunk_package_name.deb`
    

### **Debian commands for showing installation status**

Splunk package status:

`dpkg --status splunk`

It is as simple as that.

## Next steps

Now that you have installed Splunk Enterprise:

- Start it and create administrator credentials. See [Start Splunk Enterprise for the first time](https://help.splunk.com/splunk-enterprise/get-started/install-and-upgrade/9.1/start-using-splunk-enterprise/start-splunk-enterprise-for-the-first-time#id_73f04e49_0810_4340_891c_e7573f05ef03__Start_Splunk_Enterprise_for_the_first_time).
- Configure it to start at boot time. See [Configure Splunk software to start at boot time](https://help.splunk.com/en/?resourceId=Splunk_Admin_ConfigureSplunktostartatboottime).

## On UNIX

1. Use the Splunk Enterprise command-line interface (CLI):
    
    `cd <Splunk Enterprise installation directory>/bin
    ./splunk start`
    
2. Create the Splunk Enterprise admin username. This is the user that you log into Splunk Enterprise with, not the user that you use to log into your machine or onto splunk.com. You can press Enter to use the default username of `admin`.
    
    This appears to be your first time running this version of Splunk.
    
    Splunk software must create an administrator account during startup. Otherwise, you cannot log in.
    Create credentials for the administrator account.
    Characters do not appear on the screen when you type in credentials.
    
    Please enter an administrator username:
    
3. Create the password for the user that you just created. You use these credentials to log into Splunk Enterprise.
    
    Password must contain at least:
    * 8 total printable ASCII character(s).
    Please enter a new password:
    
4. If the default management and Splunk Web ports are already in use (or are otherwise not available), Splunk Enterprise offers to use the next available ports. You can either accept this option or specify a port to use.
5. You can optionally set the `SPLUNK_HOME` environment variable to the Splunk Enterprise installation directory. Setting the environment variable lets you refer to the installation directory later without having to remember its exact location:
    
    `export SPLUNK_HOME=<Splunk Enterprise installation directory>
    cd $SPLUNK_HOME/bin
    ./splunk start`
    
6. Splunk Enterprise displays the license agreement and prompts you to accept before the startup sequence continues.

The Splunk web interface is at 

```markdown
[http://splunk:8000](http://splunk:8000/)
```

Run the following command to enable boot start:

```bash

sudo $SPLUNK_HOME/bin/splunk enable boot-start
```

References:

Splunk Installation: [https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/9.1/install-splunk-enterprise-on-linux-or-macos/install-on-linux](https://help.splunk.com/en/splunk-enterprise/get-started/install-and-upgrade/9.1/install-splunk-enterprise-on-linux-or-macos/install-on-linux)

Enable Boot start: [https://help.splunk.com/en/?resourceId=Splunk_Admin_ConfigureSplunktostartatboottime](https://help.splunk.com/en/?resourceId=Splunk_Admin_ConfigureSplunktostartatboottime)