# Ubuntu 22.04

Installation guide Ubuntu 22.04 virtualbox image for C/C++ development behind company firewall and proxy.

## getting image

Preinstalled basic Ubuntu 22.04 LTS from osboxes.org
user + password: root/osboxes osboxes.org

https://www.osboxes.org/ubuntu/#ubuntu-22-04-jammy-vbox

extract the downloaded archive into the virtual machine destination folder e.g.: `C:\virtualmachines\Ubuntu22`

## create vm

Using powershell cli:

Make sure VirtualBox is in the path assuming the default install location:

```
$env:path += ";C:\Program Files\Oracle\VirtualBox"
```

Create the VirtualMachine in VirtualBox using the gui or the scripted lines below (based on VirtualBox v7.0).

The nat-prt-forwarding is specific for the occasion, adjust to your requirements.

```
VBoxManage createvm --name Ubuntu22 --basefolder=C:\virtualmachines\ --ostype=Ubuntu22_LTS_64 --register

VBoxManage modifyvm Ubuntu22 --ioapic=on --rtc-use-utc=on --cpus=4 --pae=on --hwvirtex=on --nested-hw-virt=on --memory 16384 --vram 128 --nic1 nat --nic-type1=virtio --accelerate-3d=on --graphicscontroller=vmsvga --usb-xhci=on --mouse=usbtablet --keyboard=usb

VBoxManage modifyvm Ubuntu22 --nat-pf1="ssh,tcp,,8022,,22" --nat-pf1="https,tcp,,8443,,443" --nat-pf1="plc,tcp,,8740,,11740" --nat-pf1="opcua,tcp,,4840,,4840" --nat-pf1="p2069,tcp,,2069,,2069" --nat-pf1="p2070,tcp,,2070,,2070"

VBoxManage storagectl Ubuntu22 --name "SATA Controller" --add sata --controller IntelAhci --hostiocache=on 

VBoxManage storageattach Ubuntu22 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --nonrotational=on --medium  "C:\virtualmachines\Ubuntu22\Ubuntu 22.04 (64bit).vdi"

VBoxManage storageattach Ubuntu22 --storagectl "SATA Controller" --port 1 --device 0 --medium additions
```

## setting up Ubuntu 22.04

Start the VM and login either using the GUI or by pressing CTRL-ALT-F2 using the console.

### example credentials and company name and domain

company : example.com
Windows domain: XMPL
Windows domain username: brinkman

### create user and add to groups

```bash
$ sudo bash
# usedadd -m -G adm,sudo,dip,plugdev,lpadmin,lxd,sambashare,vboxsf -s /bin/bash brinkman
# passwd brinkman 
```

### update VBoxGuestAdditions

> [!NOTE]
> The osbox image has not yet installed the build-essentials.
> Postpone this directly after the alpaca proxy installation.

```bash
$ sudo apt install build-essential
$ cd /media/$user/VBox*
$ ./autorun.sh
```

### poweroff vm and add shares

```bash
VBoxManage.exe sharedfolder add Ubuntu22 --name=downloads --hostpath=c:\users\brinkman\Downloads --automount --auto-mount-point=/home/brinkman/DownloadsWin
VBoxManage.exe sharedfolder add Ubuntu22 --name=devtools --hostpath=c:\dev\devtools --automount --auto-mount-point=/home/brinkman/Projects/devtools --readonly
```

> [!NOTE]
> The devtools repo could also be cloned in the vm instead of through the Windows shared directory.


### download proxy server

On windows download [alpaca proxy](https://github.com/samuong/alpaca/releases) for linux-amd64
 
### create systemd unit file with config

Restart vm login as user bosch and open terminal.

```bash
$ sudo bash
# cd /opt/
# mkdir alpaca
# cd alpaca
# cp /home/brinkman/DownloadsWin/alpaca_v2.*_linux-amd64 .
# chmod 755 *
# ln alpaca_v2.0.2_linux-amd64 alpaca
```

To use the proxy your comapny domain creditials are needed:

```bash
$ /opt/alpaca/alpaca -H -d XMPL -u brinkman
Password (for XMPL\brinkman):
# Add this to your ~/.profile
NTLM_CREDENTIALS="XMPL@brinkman:09817465783637abcdfe............"; export NTLM_CREDENTIALS
```

Using the output above create the file `/home/brinkman/.config/alpaca.environment`:

```
NTLM_CREDENTIALS=XMPL@brinkman:09817465783637abcdfe............
DOMAIN_USER=brinkman
URL_PAC=http://example.com/proxyfile.pac
```

> [!TIP]
> Copy the snipped below to a file in windows Donwload directory, and copy the file using the share.

Create the alpaca unit configuration file `/home/brinkman/.local/share/systemd/user/alpaca.service`:

```
[Unit]
Description=Alpaca Proxy
After=network.target

[Service]
Type=oneshot
RemainAfterExit=true
StandardOutput=journal
EnvironmentFile=%h/.config/alpaca.environment
ExecStart=/opt/alpaca/alpaca -C ${URL_PAC} -u ${DOMAIN_USER}
ExecStop=killall -9 alpaca
Restart=on-failure

[Install]
WantedBy=default.target
```

Enable the alpaca.service on every login:

```bash
$ systemctl --user enable alpaca.service
```

### set proxy environment vars

Make sure the proxy enviroment vars are set on `~/.profile`:

```bash
export http_proxy=http://localhost:3128
export https_proxy=http://localhost:3128
export HTTP_PROXY=http://localhost:3128
export HTTPS_PROXY=http://localhost:3128
```

Also add the proxy to your `~/.gitconfig`:

```
[http]
        proxy = http://localhost:3128

[credential]
        helper = store

[help]
    autocorrect = prompt
```

And make sure the proxy env vars are passed on when using `sudo` by creating the file `/etc/sudoers.d/01-proxy`:

```
Defaults env_keep += "http_proxy HTTP_PROXY"
Defaults env_keep += "https_proxy HTTPS_PROXY"
```

Snapd has also a need for its own proxy settings:

```
$ sudo snap set system proxy.http=http://localhost:3128
$ sudo snap set system proxy.https=http://localhost:3128
```

### set firefox config network tp proxy

`use system proxy settings`

and

`enable DNS over HTTPS`

## ssh access 

Install openssh server and enable the server

```bash
$ sudo apt install openssh-server
$ sudo systemctl enable ssh
```

Make sure ssh-agent gets started at login:

add to `~/.profile`

```bash
# start ssh-agent kill if already running.
pidof -q ssh-agent && ssh-agent -k
eval `ssh-agent`
```

### copy windows pub key into ~/.ssh/authorized_keys

Make sure a ssh private-public key pair is available. Run `ssh-keygen` to create a key-pair if needed.

```
PS C:\> cd $env:HOME
PS C:\Users\brinkman> cat .\.ssh\id_rsa.pub | ssh brinkman@ubuntu> "mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

Add `Host` to `.ssh\config`

```
...
Host Ubuntu22
    HostName localhost
    Port 8022
    User brinkman
...
```

## apt.kitware.com repo

Follow steps outlined by kitware to add the [kitware cmake](https://apt.kitware.com/) repo to Ubuntu.

## ninja-build, cifs-utils, 

```
$ sudu apt install ninja-build
$ sudo apt install cifs-utils
```

## mount script for window shares

To mount the window shared directory read-only:

```bash
$sudo mount -t cifs //<unc-path>/winshare /home/brinkman/winshare/ -o user=brinkman,pass=<Windows Password brinkman>,dom=XMPL,ro
```

## zfs drive (optional)

Poweroff the VM and close all VirtualBox Manager windows.

```powershell
PS C:\virtualmachines\zfs> VBoxManage.exe createmedium disk --filename C:\virtualmachines\zfs\flow.vmdk --size 512000 --format VMDK --variant Split2G
PS C:\virtualmachines\zfs> VBoxManage.exe storageattach Ubuntu22 --storagectl "SATA Controller" --port 2 --device 0 --type hdd --nonrotational=on --medium  "C:\virtualmachines\zfs\flow.vmdk"
```

Start the vm.

```
$ sudo apt-get install zfsutils-linux procinfo
$ lsblk
# The blockdevices are shown including the disk `/dev/sdb` just created.
$ sudo zpool list
# no pools yet
$ sudo zpool create data /dev/sdb
$ sudo zpool list
# data ... ONLINE
$ sudo zfs create data/Projects
$ sudo zfs set mountpoint=/home/Projects data/Projects
$ sudo zfs create data/Projects/<project>
```

Use the ~/Projects diretory for development. 

## /boot fs to small

The default installation uses a separate /boot filesystem of ~256MB. This might soon lead to space issues when the kernel is updated.
Either remove old unused kernels regulary:

```bash
$ dpkg -l |tail -n +6 |grep -E 'linux-image-[0-9]+'
<list installed kernels>
$ sudo dpkg --purge linux-image-5.15.0-25-generic
# or if dplg fails
$ sudo apt purge linux-image-5.15.0-25-generic
$ sudo apt autoremove
``` 
Or better move the boot fs to root fs:

> [!TIP]
> Make a snapshot of the vm in VirtualBox before makeing the changes. If things go wrong, delete current state, (reboot prev state), snapshot again, try again.

```bash
$ sudo umount /boot
$ sudo mount /dev/sda2 /mnt
$ cd /mnt
$ sudo cp -a * /boot
$ sudo umount /mnt
$ sudo update-grub
```

Edit the `/etc/fstab` file and comment out the line containing the `/boot` mountpoint.

Reboot
