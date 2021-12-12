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
Note: If updatedb and locate do not work, you can install it with `sudo apt install locate`

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

Change a user's password
```bash
sudo passwd user
```

Make a user's password expire
```bash
sudo passwd --expire user
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

Make file editable
```bash
sudo chmod a+rwx <file>
sudo chattr -i <file>
```

# GUI-Based Settings

### Automatic Updates

- Search for the Software and Updates application
- On the Ubuntu Software tab, check the boxes for 
    - Canonical-supported free and open-source software
    - Community-maintained free and open-source software
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
sudo apt remove -y nmap zenmap wireshark john ophcrack && sudo apt install -y ufw libpam-cracklib aide aide-common fail2ban clamav auditd
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
sudo ufw default deny routed
sudo ufw reload
sudo ufw enable
```

Set password policies
```bash
sudo sed -ri 's/^(password\s*requisite\s*pam_cracklib\.so.*)$/\1 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1/' /etc/pam.d/common-password
sudo sed -ri 's/^(password.*pam_unix\.so.*)$/\1 remember=5 minlen=8 rounds=3/' /etc/pam.d/common-password
sudo sed -ri 's/PASS_MAX_DAYS\s*[0-9]*/PASS_MAX_DAYS 90/' /etc/login.defs
sudo sed -ri 's/PASS_MIN_DAYS\s*[0-9]*/PASS_MIN_DAYS 10/' /etc/login.defs
sudo sed -ri 's/PASS_WARN_AGE\s*[0-9]*/PASS_WARN_AGE 7/' /etc/login.defs
echo "auth required pam_tally2.so deny=5 onerr=fail unlock_time=30" | sudo tee -a /etc/pam.d/common-auth
```

Secure permissions for important files
```bash
sudo chmod o-rwx,g-wx /etc/shadow
sudo chmod o-rwx,g-wx /etc/gshadow
sudo chmod o-rwx,g-wx /etc/shadow-
sudo chmod o-rwx,g-wx /etc/gshadow-
sudo chmod u=rw,go= /etc/sudoers
sudo chmod u=rw,go=r /etc/passwd
sudo chmod u=rw,go=r /etc/group

sudo chown root:root /boot/grub/grub.cfg
sudo chmod og-rwx /boot/grub/grub.cfg

sudo chmod u=rwx,go= /home/*
```

Secure root account
```
sudo passwd -l root
```

Memory Security
```bash
# Prevent core dumps
echo -e "\n* hard core 0" | sudo tee -a /etc/security/limits.conf
echo -e "\nfs.suid_dumpable = 0 | sudo tee -a /etc/sysctl.conf"
sudo sysctl -w fs.suid_dumpable=0
# ASLR
echo "kernel.randomize_va_space = 2" | sudo tee -a /etc/sysctl.conf
sudo sysctl -w kernel.randomize_va_space=2
```

Boot security
```bash
sudo grub-mkpasswd-pbkdf2
```

Check users
Copy-pasting the Users code will run the script for you
Tip: copy the user list in the readme and paste it into an online sorting app (such as https://pinetools.com/sort-list)

Users
```bash
echo '
for username in $(cut -d: -f1 /etc/passwd | grep -v -E "root|daemon|bin|sys|sync|games|man|lp|mail|news|uucp|proxy|www-data|backup|list|irc|gnats|nobody|libuuid|syslog|messagebus|colord|lightdm|whoopsie|avahi-autoipd|avahi|usbmux|kernoops|pulse|rtkit|speech-dispatcher|dispatcher|hplip|saned|ubuntu|_apt|uuidd|dnsmasq|geoclue|gnome-initial-setup|gdm|vboxadd|sshd|mysql)"); do
    echo "Is $username a permitted user?"
    select keepuser in "Yes" "No"; do
        case $keepuser in
            Yes ) 
                echo "Is $username an administrator?"
                select makeadmin in "Yes" "No"; do
                    case $makeadmin in 
                        Yes ) gpasswd -a $username sudo; break;;
                        No ) gpasswd -d $username sudo; break;;
                    esac
                done
                break;;
            No ) userdel $username && echo "Deleted $username"; break;;
        esac
    done
done
' > /cp/userchecks.sh; chmod a+x /cp/userchecks.sh; sudo bash /cp/userchecks.sh;
                
```


### Unsafe services
Disable list of services (recommended to copy in a script file first and remove lines with needed services)
```bash
sudo systemctl disable avahi-daemon
sudo systemctl disable cups
sudo systemctl disable ics-dhcp-server
sudo systemctl disable ics-dhcp-server6
sudo systemctl disable slapd
sudo systemctl disable nfs-server
sudo systemctl disable rpcbind
sudo systemctl disable bind9
sudo systemctl disable vsftpd
sudo systemctl disable apache2
sudo systemctl disable smbd
sudo systemctl disable squid
sudo systemctl disable snmpd
sudo systemctl disable rsync
sudo systemctl disable nis
```

Uninstall list of services (recommended to copy in a script file first and remove lines with needed services)
```bash
sudo apt remove -y nis rsh-client rsh-redone-client talk telnet ldap-utilsyp-tools tftpd atftpd tftpd-hpa telnetd rsh-server rsh-redone-server
```

Note: sometimes using systemctl is insufficient and it is necessary to `sudo apt remove` the service

# Check next

### Check home directories for unsafe files
Examples of unsafe files: passwords or credit card information in plain text
```
ls -aR /home
```

### Updates
Package repositories
```apt-cache policy```
GPG keys
```apt-key list```

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
Ubuntu:
```bash
sudo netstat -ntulp
```
Debian:
```bash
sudo ss -ntulp
```

### Nopasswdlogin group
No user should be in the `nopasswdlogin` group.
Check for users with
```bash
sudo cat /etc/groups | grep "nopasswdlogin
```
remove with 
```bash
sudo gpasswd -d <user> nopasswdlogin
```

### Cron Jobs
Run each line separately
```bash
ls -a /etc/cron*

sudo crontab -e

sudo vim /etc/crontab

sudo vim /etc/anacrontab
```
Add regular AIDE check
```bash
(crontab -l 2>/dev/null; echo "0 5 * * * /usr/bin/aide.wrapper --config /etc/aide/aide.conf --check") | crontab -
```

### File Differences

Ensure diffutils is installed
`sudo apt install diffutils`

General command
`diff <file1> <file2>`

Difference without comments
`diff -d -I '^#' -I '^ #' <file1> <file2>`

Use git to download files to compare with (files are located in Diffs directory)
`git clone https://github.com/neonLian/CPXIII`

###



# Updates

**Automatic updates must be enabled in the GUI before running the terminal commands.**

```bash
sudo apt update
sudo apt upgrade
```

# Lynis Auditing

Install lynis
```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C80E383C3DE9F082E01391A0366C67DE91CA5D5F
sudo apt install apt-transport-https
echo 'Acquire::Languages "none";' | sudo tee /etc/apt/apt.conf.d/99disable-translations
echo "deb https://packages.cisofy.com/community/lynis/deb/ stable main" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list
sudo apt update
sudo apt install lynis
```

Run lynis
```bash
sudo lynis audit system -Q
```

# CIS Assessor tool

Install Java: `sudo apt install openjdk-11-jre`

Download CIS-CAT from https://learn.cisecurity.org/e/799323/l-799323-2019-11-15-3v7x/2mnnf/122157240?h=QZh8uSmWBBGchDMmePPmKujg2GFjSLbKRnpZCSk8Eog

Extract the ZIP, navigate to the directory

Make the script executable: `chmod a+x ./Assessor-CLI.sh`

Run the script: `sudo ./Assessor-CLI.sh -i`

Open the HTML file in the reports folder and review results

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

# Additional Resources

https://www.debian.org/doc/manuals/securing-debian-manual/

https://github.com/Resistor52/SampleLinuxScripts

https://sumwonyuno.github.io/cp-lockdown

