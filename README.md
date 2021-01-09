# CPXIII

This README contains a checklist and scripts for the Ubuntu and Debian VMs in the CyberPatriots competition.

This is not permitted for use outside of McKinley High School; using it is a violation of the CyberPatriots rules.

# Table of Contents

[Tools for Forensics Questions](#tools-for-forensics-questions)

[GUI-Based Settings](#gui-based-settings)

[Scripts](#scripts)

[Check next](#check-next)

[Troubleshooting](#troubleshooting)

# Debian setup

Install sudo
```bash
sudo apt install sudo
gpasswd -a user sudo
```
Restart the machine to apply the group change

# Tools for Forensics Questions
### Also useful for simple tasks found in the README

Find files (replace \*.mp3 with the appropriate file extension)
```bash
sudo updatedb
locate "**/*.mp3"
```

List all programs on the system (check for any potential hacking tools)
```bash
for app in /usr/share/applications/*.desktop ~/.local/share/applications/*.desktop; do app="${app##/*/}"; echo "${app::-8}"; done
```

Find which users are in a group
```bash
# Option 1:
getent group groupname
# Option 2:
cat /etc/groups | grep "groupname"
```

Print a list of users
```bash
sudo cut -d: -f1 /etc/passwd | grep -v -E '(root|daemon|bin|sys|sync|games|man|lp|mail|news|uucp|proxy|www-data|backup|list|irc|gnats|nobody|libuuid|syslog|messagebus|colord|lightdm|whoopsie|avahi-autoipd|avahi|usbmux|kernoops|pulse|rtkit|speech-dispatcher|dispatcher|hplip|saned|ubuntu|_apt|uuidd|dnsmasq|geoclue|gnome-initial-setup|gdm|vboxadd|sshd)'
```

Create/remove a user
```bash
sudo useradd user
sudo userdel user
```

Create/remove a group
```bash
sudo groupadd group
sudo groupdel group
```

Add/remove a user to/from a group
```bash
sudo gpasswd -a user group
sudo gpasswd -d user group
```
# GUI-Based Settings

### Automatic Updates
GNOME Desktop
- Search for the Software and Updates application
- Choose the Updates tab
- Check for updates daily and automatically download & install security updates
Other
- If there is no Software and Updates application, check the system settings

### Mozilla Firefox
- Go to the settings (via the menu on the top right)
- Go to Privacy and Security settings
- Ensure Dangerous and Deceptive Content is blocked
- Ensure pop-up windows are blocked

# Scripts

To paste into the terminal, use `Ctrl-Shift-V`

To copy from the terminal, use `Ctrl-Shift-C`

Log in to sudo
```
sudo echo "Logged in as sudo"
```

Run these to create the directory /cp (this is where all scripts will go)
```bash
sudo mkdir /cp
sudo chmod a+rwx /cp
```

Program install and remove (**Warning: this will not remove all hacking tools, you must check for those manually**)
```bash
sudo apt remove -y nmap zenmap wireshark john ophcrack

sudo apt install -y ufw libpam-cracklib
```

Disable guest access
```bash
sudo sh -c 'printf "[SeatDefaults]\nallow-guest=false\n" > /etc/lightdm/lightdm.conf'
```

Prevent root login with SSH
```bash
sudo sed -i 's/PermitRootLogin .*/PermitRootLogin no/' /etc/ssh/sshd_config
```

Enable firewall
```bash
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw reload
sudo ufw enable
```

Set password policies
```bash
sudo sed -ri 's/^(password\s*requisite\s*pam_cracklib\.so.*)$/\1 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1/' /etc/pam.d/common-password
sudo sed -ri 's/^(password.*pam_unix\.so.*)$/\1 remember=5 minlen=8/' /etc/pam.d/common-password
sudo sed -ri 's/PASS_MAX_DAYS\s*[0-9]*/PASS_MAX_DAYS 90/' /etc/login.defs
sudo sed -ri 's/PASS_MIN_DAYS\s*[0-9]*/PASS_MIN_DAYS 10/' /etc/login.defs
sudo sed -ri 's/PASS_WARN_AGE\s*[0-9]*/PASS_WARN_AGE 7/' /etc/login.defs
sudo echo "auth required pam_tally2.so deny=5 onerr=fail unlock_time=30" >> /etc/pam.d/common-auth
```

Secure permissions for important files
```bash
sudo chmod u=rw,go= /etc/shadow
sudo chmod u=rw,go= /etc/sudoers
sudo chmod u=rw,go=r /etc/passwd

sudo passwd -l root
```

# Check next

### Services
List services
```bash
# Ubuntu
sudo service --status-all
# Debian
sudo systemctl list-unit-files
```
Disable service
`sudo systemctl disable <service>`

### Network connections
```bash


### Cron Jobs
```bash
ls -a "/etc/cron*"
```

# Updates

**Automatic updates must be enabled in the GUI before running the terminal commands.**

```bash
sudo apt update
sudo apt upgrade
```

# Troubleshooting

Could not lock /var/lib/dpkg/lock-frontend
```bash
sudo killall apt apt-get

sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock*
sudo dpkg --configure -a
sudo apt update
```

more stuff at [sumwonyuno.github.io/cp-lockdown](https://sumwonyuno.github.io/cp-lockdown)
