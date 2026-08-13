# Linux Cheatsheet

Complete detailed reference guide for the Linux operating system and command line.

## 🎯 Linux Fundamentals

### System Information & Environment

**System & Kernel Information:**

Display Linux kernel version
```bash
uname -r
uname -a
```

Display system information
```bash
hostnamectl
hostnamectl set-hostname new-hostname
```

Show system uptime
```bash
uptime
```

Display CPU information
```bash
cat /proc/cpuinfo
lscpu
nproc
```

Display memory information
```bash
free -h
cat /proc/meminfo
```

Display disk information
```bash
df -h
du -sh /directory
```

Show system load
```bash
cat /proc/loadavg
```

Display OS information
```bash
cat /etc/os-release
lsb_release -a
```

**User & Group Information:**

Display current user
```bash
whoami
```

Display user and group IDs
```bash
id
id username
```

List all users
```bash
cat /etc/passwd
```

List all groups
```bash
cat /etc/group
```

Display currently logged-in users
```bash
who
w
```

Show user login history
```bash
last
last -10
```

Display sudo version
```bash
sudo -V
```

Check sudoers file
```bash
sudo visudo
```

### Directory Navigation & File Listing

**Navigation Commands:**

Print working directory
```bash
pwd
```

Change directory
```bash
cd /path/to/directory
cd ~               # Home directory
cd -               # Previous directory
cd ..              # Parent directory
cd /               # Root directory
```

List directory contents
```bash
ls
ls -l              # Long format
ls -la             # Including hidden files
ls -lh             # Human-readable sizes
ls -lS             # Sort by size
ls -lt             # Sort by modification time
ls -ltr            # Reverse time sort
ls -R              # Recursive
```

Tree structure
```bash
tree
tree -L 2          # Depth 2
tree -I 'pattern'  # Exclude pattern
```

## 📁 File & Directory Management

### File Operations

**Create & Delete Files:**

Create empty file
```bash
touch filename
touch file1 file2 file3
```

Create file with content
```bash
echo "content" > filename
cat > filename << EOF
multiple lines
of content
EOF
```

Copy file
```bash
cp source destination
cp -r source/ destination/    # Recursive
cp -v source destination      # Verbose
cp -i source destination      # Interactive
```

Move/Rename file
```bash
mv source destination
mv -i source destination      # Interactive
mv -v source destination      # Verbose
```

Delete file
```bash
rm filename
rm -f filename                # Force
rm -r directory/              # Recursive
rm -rf directory/             # Force recursive
rm -i filename                # Interactive
```

Secure delete (wipe)
```bash
shred -vfz -n 3 filename
```

Create temporary file
```bash
mktemp
mktemp -d                     # Temporary directory
```

**Directory Operations:**

Create directory
```bash
mkdir dirname
mkdir -p path/to/nested/dir   # Create parents
mkdir -m 755 dirname          # With permissions
```

Remove directory
```bash
rmdir dirname                 # Empty directory only
rm -r dirname/                # Recursive (with files)
```

Create symbolic link
```bash
ln -s target linkname         # Symbolic link
ln target linkname            # Hard link
ln -s source ../link          # Relative path
```

List links
```bash
ls -l                         # Shows -> target
ls -i                         # Shows inode
```

Change to recent directory
```bash
cd -
pushd /path/to/dir            # Save to stack
popd                          # Return from stack
dirs                          # Show directory stack
```

### File Content & Searching

**View & Edit File Content:**

Display file content
```bash
cat filename
cat -n filename               # With line numbers
cat -E filename               # Show end of lines
cat -A filename               # Show all characters
```

View beginning/end
```bash
head filename                 # First 10 lines
head -n 20 filename           # First 20 lines
tail filename                 # Last 10 lines
tail -n 20 filename           # Last 20 lines
tail -f filename              # Follow (live)
```

View in pages
```bash
less filename
more filename
most filename
```

View specific lines
```bash
sed -n '5,10p' filename       # Lines 5-10
sed -n '10p' filename         # Line 10
```

Word count
```bash
wc filename
wc -l filename                # Line count
wc -w filename                # Word count
wc -c filename                # Byte count
```

Text manipulation
```bash
cat filename | head -n 5
sed 's/old/new/g' filename    # Replace text
```

**Search & Find:**

Search in files
```bash
grep "pattern" filename
grep -i "pattern" filename    # Case insensitive
grep -v "pattern" filename    # Invert match
grep -c "pattern" filename    # Count matches
grep -n "pattern" filename    # Line numbers
grep -r "pattern" directory/  # Recursive
```

Extended grep
```bash
egrep "pattern1|pattern2" file
grep -E "^[0-9]" filename     # Regex
```

Find files
```bash
find /path -name "filename"
find / -name "*.txt"          # By extension
find / -name "*log*"          # Partial name
find / -type f                # Files only
find / -type d                # Directories only
find / -size +100M            # Size > 100MB
find / -mtime -7              # Modified < 7 days
find / -user username         # By owner
find / -perm 644              # By permissions
```

Find and execute
```bash
find / -name "*.tmp" -delete
find / -name "*.log" -exec rm {} \;
```

Locate
```bash
locate filename
updatedb                      # Update database
```

## 🔐 File Permissions & Ownership

### Permissions Management

**Change Permissions:**

View permissions
```bash
ls -l filename
stat filename
```

Change mode (symbolic)
```bash
chmod u+x filename            # User execute
chmod g+w filename            # Group write
chmod o+r filename            # Other read
chmod a+x filename            # All execute
chmod u-x filename            # Remove user execute
```

Change mode (numeric)
```bash
chmod 755 filename            # rwxr-xr-x
chmod 644 filename            # rw-r--r--
chmod 700 filename            # rwx------
chmod 777 filename            # rwxrwxrwx (dangerous!)
```

Recursive
```bash
chmod -R 755 directory/
chmod -R g+w directory/
```

Change ownership
```bash
chown owner filename
chown owner:group filename
chown -R owner:group directory/
```

Change group only
```bash
chgrp group filename
chgrp -R group directory/
```

Umask (default permissions)
```bash
umask                         # Show current
umask 022                     # Set default
```

**Special Permissions:**

Setuid bit (execute as owner)
```bash
chmod u+s filename            # 4755
```

Setgid bit (execute as group)
```bash
chmod g+s filename            # 2755
```

Sticky bit (only owner/root can delete)
```bash
chmod +t directory/           # 1777
```

Immutable (cannot be modified)
```bash
chattr +i filename            # Immutable
lsattr filename                # List attributes
chattr -i filename            # Remove immutable
```

Append only
```bash
chattr +a filename
```

## 👥 User & Group Management

### User Administration

**User Operations:**

Create user
```bash
useradd username
useradd -m -s /bin/bash username    # With home & shell
useradd -d /home/custom username    # Custom home
useradd -g group username           # With group
useradd -G group1,group2 username   # Multiple groups
```

Create user with password
```bash
useradd -m -p $(openssl passwd -1 password) username
```

Modify user
```bash
usermod -s /bin/zsh username        # Change shell
usermod -d /new/home username       # Change home
usermod -l newname username         # Rename user
usermod -a -G group username        # Add to group
usermod -L username                 # Lock account
usermod -U username                 # Unlock account
```

Delete user
```bash
userdel username                    # User only
userdel -r username                 # User & home
```

Password management
```bash
passwd username                     # Set password
passwd -d username                  # Remove password
passwd -l username                  # Lock
passwd -u username                  # Unlock
passwd -e username                  # Force change
```

Change password (current user)
```bash
passwd
```

**Group Operations:**

Create group
```bash
groupadd groupname
groupadd -g 1001 groupname          # Specific GID
```

Modify group
```bash
groupmod -n newname oldname         # Rename
groupmod -g 1002 groupname          # Change GID
```

Delete group
```bash
groupdel groupname
```

Add user to group
```bash
usermod -a -G groupname username
gpasswd -a username groupname
```

Remove user from group
```bash
gpasswd -d username groupname
```

Change group members
```bash
gpasswd groupname
```

Set group admin
```bash
gpasswd -A username groupname
```

Group password
```bash
gpasswd groupname                   # Set password
gpasswd -r groupname                # Remove password
```

## 💻 Process Management

### Process Monitoring & Control

**List & Monitor Processes:**

List processes
```bash
ps                              # Simple
ps aux                          # All with details
ps -ef                          # Extended format
ps -ef | grep processname       # Filter
```

Process tree
```bash
pstree
pstree -p                       # With PID
pstree username                 # User processes
```

Real-time monitoring
```bash
top                             # Interactive
top -u username                 # User processes
htop                            # Enhanced (install)
watch -n 1 'ps aux'             # Repeated command
```

Process details
```bash
ps -p PID -o pid,user,comm,cmd
ps -p 1234 -L                   # Show threads
```

List by state
```bash
ps aux | grep defunct           # Zombie processes
ps T                            # Session processes
```

**Control Processes:**

Send signals
```bash
kill PID                        # SIGTERM (15)
kill -9 PID                     # SIGKILL (9)
kill -STOP PID                  # Pause
kill -CONT PID                  # Resume
kill -HUP PID                   # Hangup (reload config)
kill -USR1 PID                  # User signal 1
```

Kill by name
```bash
killall processname
pkill processname
pkill -f "pattern"              # Pattern match
pkill -u username               # Kill user's processes
```

Process priority
```bash
nice -n 10 command               # Start with priority
renice -n 5 -p PID                # Change priority
renice -n -5 -u username          # Change user processes
```

Background/Foreground
```bash
command &                       # Run in background
fg                               # Bring to foreground
bg                               # Resume in background
jobs                             # List jobs
# Ctrl+Z                        # Suspend (pause)
```

## 📦 Package Management

### APT (Debian/Ubuntu)

**APT Commands:**

Update package list
```bash
sudo apt update
sudo apt update -y
```

Install package
```bash
sudo apt install packagename
sudo apt install packagename1 packagename2
sudo apt install -y packagename     # Non-interactive
```

Upgrade packages
```bash
sudo apt upgrade                # Safe upgrade
sudo apt full-upgrade           # Upgrade + remove if needed
sudo apt dist-upgrade           # Distribution upgrade
```

Remove package
```bash
sudo apt remove packagename     # Keep config
sudo apt purge packagename      # Remove all
sudo apt autoremove             # Remove unused
```

Search package
```bash
apt search keyword
apt search --names-only pattern
```

Show package info
```bash
apt show packagename
apt-cache policy packagename
```

List installed
```bash
apt list --installed
dpkg -l                         # Detailed list
```

Install from file
```bash
sudo apt install ./packagename.deb
sudo dpkg -i packagename.deb
```

### YUM/DNF (RHEL/CentOS/Fedora)

**YUM/DNF Commands:**

Check manager version
```bash
yum --version
dnf --version
```

Update package list
```bash
yum update
dnf update
```

Install package
```bash
yum install packagename
dnf install packagename
yum install -y packagename      # Non-interactive
```

Remove package
```bash
yum remove packagename
dnf remove packagename
```

Search package
```bash
yum search keyword
dnf search keyword
```

Show package info
```bash
yum info packagename
dnf info packagename
```

List installed
```bash
yum list installed
dnf list installed
```

Install from file
```bash
yum localinstall packagename.rpm
dnf install ./packagename.rpm
```

## 🔌 Networking

### Network Configuration & Troubleshooting

**Network Information:**

Display interfaces
```bash
ifconfig                        # (deprecated)
ip addr                         # Modern
ip link                         # Link layer
ip route                        # Routing table
```

Network configuration file
```bash
cat /etc/network/interfaces     # Debian/Ubuntu
cat /etc/sysconfig/network-scripts/ifcfg-eth0  # CentOS
```

Hostname & DNS
```bash
hostname
hostnamectl set-hostname name   # Set hostname
cat /etc/hostname
cat /etc/hosts
cat /etc/resolv.conf            # DNS config
```

Check DNS
```bash
nslookup example.com
dig example.com
host example.com
```

**Network Connectivity:**

Ping
```bash
ping example.com
ping -c 4 example.com           # Linux (4 packets)
ping -n 4 example.com           # Count limit
```

Traceroute
```bash
traceroute example.com
mtr example.com                 # Real-time trace
```

Port scanning
```bash
netstat -tuln                   # Show listening
netstat -anp | grep :8080       # Find process
ss -tuln                        # Modern alternative
ss -anp | grep :8080
```

Check port
```bash
nc -zv example.com 22           # Check connectivity
lsof -i :8080                   # List open ports
```

Connection info
```bash
netstat -an | grep ESTABLISHED
ss -an | grep ESTABLISHED
```

DNS query
```bash
dig @8.8.8.8 example.com        # Query specific DNS
```

**Configure Networking:**

Temporary IP assignment
```bash
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0
```

Enable/disable interface
```bash
ip link set eth0 up
ip link set eth0 down
ifup eth0                       # Legacy
ifdown eth0                     # Legacy
```

Add route
```bash
ip route add 192.168.2.0/24 via 192.168.1.1
ip route add default via 192.168.1.1
ip route del 192.168.2.0/24
```

Configure in Debian/Ubuntu
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Apply netplan
```bash
sudo netplan apply
sudo netplan validate
```

## 📝 Text Processing

### Powerful Text Tools

**sed (Stream Editor):**

Substitute (replace)
```bash
sed 's/old/new/' filename       # First occurrence
sed 's/old/new/g' filename      # All occurrences
sed 's/old/new/2' filename      # Second occurrence
sed 's/old/new/gi' filename     # Case insensitive
```

Delete lines
```bash
sed '5d' filename               # Delete line 5
sed '5,10d' filename            # Delete lines 5-10
sed '/pattern/d' filename       # Delete matching lines
sed '$d' filename               # Delete last line
```

Print specific lines
```bash
sed -n '5,10p' filename         # Print lines 5-10
sed -n '10p' filename           # Print line 10
```

Insert/Append
```bash
sed '5i\NEW LINE' filename      # Insert before line 5
sed '5a\NEW LINE' filename      # Append after line 5
```

In-place editing
```bash
sed -i 's/old/new/g' filename   # Modify file
sed -i.bak 's/old/new/g' file   # Backup original
```

**awk (Text Processing):**

Field separation
```bash
awk '{print $1}' filename       # First field
awk '{print $1,$3}' filename    # Fields 1 and 3
awk -F: '{print $1}' filename   # Custom delimiter (:)
```

Pattern matching
```bash
awk '/pattern/ {print}' filename
awk '$3 > 100 {print}' filename
awk '$1 == "apple"' filename
```

Variables & calculations
```bash
awk '{sum += $2} END {print sum}' filename
awk '{count++} END {print count}' filename
awk '{total += $3} END {print total/NR}' filename  # Average
```

NF and NR
```bash
awk '{print NF}' filename       # Fields per line
awk 'END {print NR}' filename   # Total lines
awk '{print NR, $0}' filename   # Line numbers
```

Conditions
```bash
awk '$2 > 50 {print $1}' filename
awk 'NR > 5 && NR < 10' filename
```

## 🗂️ System Services & Daemons

### Systemd Management

**Service Operations:**

Start/Stop service
```bash
systemctl start servicename
systemctl stop servicename
systemctl restart servicename
systemctl reload servicename        # Reload config
```

Enable/Disable at boot
```bash
systemctl enable servicename
systemctl disable servicename
systemctl is-enabled servicename
```

Check status
```bash
systemctl status servicename
systemctl is-active servicename
systemctl is-failed servicename
```

List services
```bash
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-units --failed
```

View service file
```bash
systemctl cat servicename
systemctl show servicename
```

Edit service
```bash
sudo systemctl edit servicename
sudo systemctl daemon-reload
```

Service dependencies
```bash
systemctl list-dependencies servicename
```

**Logs & Journal:**

View systemd journal
```bash
journalctl                      # All logs
journalctl -f                   # Follow (live)
journalctl -n 50                # Last 50 lines
journalctl -x                   # With explanations
```

Filter by unit
```bash
journalctl -u servicename       # Service logs
journalctl -u servicename -f    # Follow service
```

Filter by priority
```bash
journalctl -p err               # Errors only
journalctl -p notice            # Notice and above
journalctl -p 0..2              # Critical to error
```

Time-based
```bash
journalctl --since "2024-01-01"
journalctl --since "2 hours ago"
journalctl --until "1 hour ago"
```

Boot logs
```bash
journalctl -b                   # Current boot
journalctl -b 0                 # Current boot
journalctl -b -1                # Previous boot
journalctl --list-boots         # Show boots
```

Disk usage
```bash
journalctl --disk-usage
sudo journalctl --vacuum-size=100M
```

## 🔒 Security & Access

### Security Management

**SSH Configuration:**

Generate SSH key
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
```

SSH login
```bash
ssh user@host
ssh -p 2222 user@host           # Custom port
ssh -i keyfile user@host        # Specific key
```

Copy public key
```bash
ssh-copy-id -i keyfile user@host
ssh-copy-id user@host           # Default key
```

SSH config file
```bash
cat ~/.ssh/config
```

SSH permissions
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/authorized_keys
```

Copy files via SSH
```bash
scp file.txt user@host:/path/
scp -r directory/ user@host:/path/
scp user@host:/path/file.txt ./
```

**Sudo Access Control:**

Edit sudoers (always use visudo!)
```bash
sudo visudo                     # Edit safely
sudo visudo -f /etc/sudoers.d/custom
```

Sudoers syntax
```bash
# username HOST=(RUNUSER) COMMAND
# %group HOST=(ALL) ALL
# user1 ALL=(ALL) NOPASSWD: /bin/command
```

View sudoers
```bash
sudo visudo -c                  # Check syntax
sudo ls /etc/sudoers.d/         # List includes
```

Execute as another user
```bash
sudo -u username command
sudo -l                         # Show permissions
sudo -l -u username             # User's permissions
```

Edit as root
```bash
sudo nano /etc/file
sudo -e /etc/file               # Uses $EDITOR
```

**Firewall (UFW):**

Enable/Disable
```bash
sudo ufw enable
sudo ufw disable
sudo ufw status                 # Check status
sudo ufw status verbose         # Detailed
```

Allow/Deny
```bash
sudo ufw allow 22               # Allow port
sudo ufw allow 22/tcp           # TCP only
sudo ufw deny 80                # Deny port
sudo ufw delete allow 80        # Remove rule
```

By service
```bash
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

By IP
```bash
sudo ufw allow from 192.168.1.0/24
sudo ufw deny from 10.0.0.5
```

Advanced
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw show added             # Show pending rules
```

## 💾 Storage & Mounting

### Disk & Filesystem Management

**Disk Operations:**

List disks
```bash
lsblk                          # Block devices tree
lsblk -a                       # All devices
fdisk -l                       # Partition table
parted -l                      # All partitions
```

Disk usage
```bash
df -h                          # Filesystem usage
df -i                          # Inode usage
du -sh /directory              # Directory size
du -sh /*                      # Top-level sizes
du -sh /path/* | sort -h       # Sorted by size
```

Check disk health
```bash
sudo smartctl -a /dev/sda      # Smart info
sudo smartctl --test=long /dev/sda  # Long test
```

Partition operations
```bash
sudo parted /dev/sda           # Interactive partitioning
sudo fdisk /dev/sda            # Interactive fdisk
```

**Filesystem & Mounting:**

List mounted filesystems
```bash
mount                          # Show mounted
mount | grep -i ext4           # Filter
cat /etc/fstab                 # Persistent mounts
```

Mount device
```bash
sudo mount /dev/sda1 /mnt/storage
sudo mount -t ext4 /dev/sda1 /mnt/mount
sudo mount -o rw,user /dev/sda1 /mnt
```

Unmount
```bash
sudo umount /mnt/point
sudo umount -l /mnt/point      # Lazy unmount
```

Create filesystem
```bash
sudo mkfs.ext4 /dev/sda1
sudo mkfs.btrfs /dev/sda1
sudo mkfs.xfs /dev/sda1
```

Check filesystem
```bash
sudo fsck /dev/sda1            # File system check
sudo fsck.ext4 -n /dev/sda1    # Dry-run
```

Permissions on mount
```bash
sudo mount -o uid=1000,gid=1000 /dev/sda1 /mnt
```

## 🔧 System Administration

### Advanced Administration

**Scheduled Tasks (Cron):**

Edit crontab
```bash
crontab -e                     # Edit user's cron
sudo crontab -e                # Root's cron
```

View crontab
```bash
crontab -l                     # User's entries
sudo crontab -l                # Root's entries
```

Cron syntax
```bash
# Minute Hour Day Month DayOfWeek Command
# 0 0 * * * /path/to/command   # Daily at midnight
# 0 * * * * /path/to/command   # Hourly
# 0 0 * * 0 /path/to/command   # Weekly (Sunday)
# 0 0 1 * * /path/to/command   # Monthly
```

Common examples
```bash
0 2 * * * /usr/bin/backup.sh   # 2 AM daily
0 */4 * * * /path/to/cmd       # Every 4 hours
30 3 * * 1-5 /path/to/cmd      # Weekdays 3:30 AM
```

System cron
```bash
cat /etc/crontab
ls /etc/cron.d/
ls /etc/cron.daily/
ls /etc/cron.hourly/
```

View cron logs
```bash
grep CRON /var/log/syslog
journalctl -u cron
```

**Environment Variables:**

View variables
```bash
env                            # All variables
printenv                       # All variables
echo $PATH                     # Specific variable
```

Export variable (temporary)
```bash
export VAR=value
export PATH=$PATH:/new/path
```

Create permanent
```bash
echo 'export VAR=value' >> ~/.bashrc
echo 'export VAR=value' >> ~/.bash_profile
echo 'export VAR=value' >> /etc/environment
```

Reload environment
```bash
source ~/.bashrc
source ~/.bash_profile
. ~/.bashrc
```

Unset variable
```bash
unset VAR
```

Check variable
```bash
if [ -z "$VAR" ]; then echo "not set"; fi
```

## 📊 System Monitoring

### Performance & Resource Monitoring

**Resource Usage:**

CPU usage
```bash
top
htop                           # Better top
ps aux --sort=-%cpu            # Sort by CPU
mpstat 1 5                     # CPU stats
```

Memory usage
```bash
free -h
free -m
cat /proc/meminfo
ps aux --sort=-%mem            # Sort by memory
```

Disk I/O
```bash
iostat
iostat -x 1                    # Extended stats
iotop                          # Disk I/O monitoring
```

Network monitoring
```bash
nethogs                        # By process
iftop                          # Bandwidth by IP
vnstat                         # Long-term stats
ss -s                          # Socket stats
```

Load average
```bash
uptime
cat /proc/loadavg
w
```

**Detailed Monitoring:**

Memory info
```bash
cat /proc/meminfo
cat /proc/swaps
swapon -s                      # Swap usage
```

Process info
```bash
cat /proc/[PID]/status
cat /proc/[PID]/maps
cat /proc/[PID]/io
```

CPU info
```bash
cat /proc/cpuinfo
lscpu
nproc
```

Context switches
```bash
vmstat 1 5
vmstat -s
```

Interrupt info
```bash
cat /proc/interrupts
```

## 🔄 Compression & Archives

### Compression Tools

**tar Archives:**

Create tar
```bash
tar -cvf archive.tar file1 file2
tar -cvf archive.tar directory/
```

Extract tar
```bash
tar -xvf archive.tar
tar -xvf archive.tar -C /path/
```

Tar with gzip (most common)
```bash
tar -czvf archive.tar.gz directory/
tar -xzvf archive.tar.gz
tar -tzvf archive.tar.gz       # List contents
```

Tar with bzip2
```bash
tar -cjvf archive.tar.bz2 directory/
tar -xjvf archive.tar.bz2
```

Tar with xz
```bash
tar -cJvf archive.tar.xz directory/
tar -xJvf archive.tar.xz
```

Extract specific file
```bash
tar -xzf archive.tar.gz file.txt
```

**Other Compression:**

gzip
```bash
gzip filename                  # Compress
gunzip filename.gz             # Decompress
gzip -d filename.gz            # Decompress
```

bzip2
```bash
bzip2 filename                 # Compress
bunzip2 filename.bz2           # Decompress
bzip2 -d filename.bz2          # Decompress
```

zip
```bash
zip archive.zip file1 file2
zip -r archive.zip directory/
unzip archive.zip
unzip -l archive.zip           # List contents
```

7zip
```bash
7z a archive.7z file1 file2
7z x archive.7z                # Extract
```

## 📋 Quick Reference Commands

| Command | Purpose | Example |
|---|---|---|
| `ls` | List files | `ls -la /path` |
| `cd` | Change directory | `cd /home/user` |
| `pwd` | Print working dir | `pwd` |
| `mkdir` | Create directory | `mkdir dirname` |
| `rm` | Remove file/dir | `rm -rf dirname` |
| `cp` | Copy file | `cp source dest` |
| `mv` | Move/rename | `mv old new` |
| `grep` | Search text | `grep pattern file` |
| `find` | Find files | `find / -name file` |
| `chmod` | Change perms | `chmod 755 file` |
| `sudo` | As superuser | `sudo command` |
| `ps` | List processes | `ps aux` |

## 🎓 Common Patterns & Workflows

**File Operations**
- Create/Edit
- Search/Find
- Compare/Diff
- Archive/Compress
- Transfer/Copy

**System Admin**
- User Management
- Permission Control
- Package Install
- Service Control
- Network Config

**Performance**
- CPU Monitoring
- Memory Usage
- Disk I/O
- Process Analysis
- Network Stats

## ✅ Best Practices & Tips

**✓ File Management**
- Organize files in logical directories
- Use meaningful file names
- Regular backups of important data
- Proper permissions on sensitive files
- Use version control for code

**✓ User & Security**
- Never use root for daily tasks
- Regular password updates
- SSH key authentication preferred
- Proper sudoers configuration
- Regular security audits

**✓ System Administration**
- Monitor system resources regularly
- Keep logs organized and rotated
- Regular system updates
- Maintain backup strategy
- Document configuration changes

**✓ Performance**
- Clean up temporary files
- Monitor disk usage
- Manage running processes
- Optimize mount options
- Regular maintenance

**💡 Pro Tips:**
- Use aliases for common commands
- Master tab completion
- Use command history effectively
- Create helpful shell scripts
- Learn keyboard shortcuts

**⚠️ Never:**
- Run unnecessary commands as root
- Ignore security warnings
- Delete system directories
- Disable firewalls without reason
- Neglect backups

---
*Source: adapted from the Linux cheatsheet on [engidock.com](https://www.engidock.com/cheatsheets).*
