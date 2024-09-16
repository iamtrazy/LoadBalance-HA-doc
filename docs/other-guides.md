# DHCP Configuration

### Edit DHCP server configuration:
```bash
sudo nano /etc/default/isc-dhcp-server
```

Set the interfaces for DHCPv4:
```bash
INTERFACESv4="enp0s8"
```

### Edit DHCPD configuration:
```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Set the lease times and configure the subnet:
```bash
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.56.0 netmask 255.255.255.0 {
  range 192.168.56.101 192.168.56.120;
  option routers 192.168.56.100;
}
```

### Enable and restart DHCP service:
```bash
sudo systemctl enable isc-dhcp-server
sudo systemctl restart isc-dhcp-server
```

---

# Samba Configuration

### Edit Samba global configuration:
```bash
sudo nano /etc/samba/smb.conf
```

Add the following global settings:
```bash
[global]
server string = File Server
workgroup = WORKGROUP
security = user
map to guest = Bad User
name resolve order = bcast host
include = /etc/samba/shares.conf
```

### Edit shared directory configuration:
```bash
sudo nano /etc/samba/shares.conf
```

Configure public and protected shares:
```bash
[Public Files]
path = /srv/share/public_files
force user = smbuser
force group = smbgroup
create mask = 0664
force create mode = 0664
directory mask = 0775
force directory mode = 0775
public = yes
writable = yes

[Protected Files]
path = /srv/share/private_files
force user = smbuser
force group = smbgroup
create mask = 0664
force create mode = 0664
directory mask = 0775
force directory mode = 0775
public = yes
writable = no
```

### Create the Samba group and user:
```bash
sudo groupadd --system smbgroup
sudo useradd --system --no-create-home --group smbgroup -s /bin/false smbuser
```

### Create shared directories and set permissions:
```bash
sudo mkdir -p /srv/share/private_files
sudo mkdir /srv/share/public_files
sudo chown -R smbuser:smbgroup /srv/share
sudo chmod -R g+w /srv/share
```

### Enable and restart Samba:
```bash
sudo systemctl enable smbd
sudo systemctl restart smbd
```

---

# Disk Usage Configuration

### Enable quota on root disk:
```bash
tune2fs -O quota /dev/mapper/debianBox--vg-root
```

### Verify quota is enabled:
```bash
sudo tune2fs -l /dev/mapper/debianBox--vg-root | grep -i quota
```

### Install quota tools:
```bash
sudo apt -y install quota quotatool
```

### Enable quota:
```bash
sudo quotaon -uv /
```

### Show current quota status:
```bash
sudo quotaon -ap
```

### Set quota for user `debian`:
```bash
sudo edquota -u debian
```

### Show user quota reports:
```bash
sudo repquota -au
```

### Edit `/etc/fstab` to make quota permanent:
```bash
sudo nano /etc/fstab
```

Add the following options:
```bash
errors=remount-ro,usrquota,grpquota
```

### Create a test file for disk usage:
```bash
head -c 100M /dev/zero > file.tmp
```
(Note: Use regular `debian` account to test disk limits, not `sudo`.)

---

# Backup Script Configuration

### Full backup:
```bash
rsync -av "$SRC" "$DEST_FOLDER"
```

### Incremental backup:
```bash
rsync -av --link-dest="$YESTERDAY_DIR_ABS" "$SRC" "$DEST_FOLDER"
```

### Full restore:
```bash
rsync -av --delete "$DEST/" "$SRC/"
```

### Incremental restore:
```bash
rsync -av "$DEST/" "$SRC/"
```

---

# Hardening SSH Server

### Edit SSH configuration:
```bash
sudo nano /etc/ssh/sshd_config
```

Modify the following parameters:
```bash
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
StrictModes yes
MaxAuthTries 6
MaxSessions 10
```

Save and restart the SSH service:
```bash
sudo systemctl restart sshd
```
