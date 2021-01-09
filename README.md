# CPXIII

This README contains a checklist and scripts for the Ubuntu and Debian VMs in the CyberPatriots competition.

This is not permitted for use outside of McKinley High School; using it is a violation of the CyberPatriots rules.

# Table of Contents

[Tools for Forensics Questions](#tools-for-forensics-questions)

[GUI-Based Settings](#gui-based-settings)

[Scripts](#scripts)

[Check next](#check-next)

[Troubleshooting](#troubleshooting)

# Tools for Forensics Questions
## Also useful for simple tasks found in the README

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
**GNOME Desktop:**
- Search for the Software and Updates application
- Choose the Updates tab
- Check for updates daily and automatically download & install security updates
**Other**
- If there is no Software and Updates application, check the system settings

### Mozilla Firefox
- Go to the settings (via the menu on the top right)
- Go to Privacy and Security settings
- Ensure Dangerous and Deceptive Content is blocked
- Ensure pop-up windows are blocked

# Scripts

# Check next

# Troubleshooting
