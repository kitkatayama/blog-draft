# RancherOSをインストールする

全体的に冗長な構成になっている。

## tl;dr

## モチベーション

* #ふくばねてす でつよつよ取り上げられていた
* 前々から[@_inductor_](https://twitter.com/_inductor_/)さんが推してて気になってた
* RancherOS環境に移行することで**嫌でもDockerで作業をしなければならない**状態を強制できる
* 軽そう

## 環境

* インストール先
  * PRIMERGY TX1310 M1
    * CPU: Celeron G1820
    * MEM: 16GB
    * HDD: 250GB(SSD)
* インストール準備環境
  * Windows 10

## 前準備

この辺りを先に読んでおくと良い。

* https://rancher.com/rancher-os/
* https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/
* https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/workstation/boot-from-iso/
* https://rancher.com/docs/os/v1.x/en/installation/running-rancheros/server/install-to-disk/

### ファイルの準備

[https://github.com/rancher/os/releases/] よりISOファイルをダウンロード

* `rancheros.iso`を選ぶこと
  * 今回は2019/04/09時点で最新のv1.5.1を使用する

### rufusで起動ディスク作成

RancherOSの手順には載っていないが、 https://rufus.ie/ を使用してISOからブータブルUSBを作成する。

## ベアメタルサーバーへインストール

### USBディスクブート

rufusで作成したブータブルUSBを刺して、サーバーを起動する。USBディスクが正常に読み込まれ、Linuxのカーネルに対応したサーバーであれば、rancherユーザで自動的にログインされる。

* USBを刺す
* BIOS設定によって内蔵ディスクのほうが起動優先度が高い場合がある。その際はBIOSから設定変更
* 以下はPRIMERGY TX1310 M1での例

### SSHでのログイン

コンソール上のtty1では作業が面倒なので、一旦SSHで入れるようにする（この作業は、ブータブルUSBで起動している現在のセッションにのみ設定していることに留意する）。

`sudo passwd rancher`

#### Tips

shd_configには以下の追記があるようにうかがえる。`UseDNS no`は、sshdの名前解決動作の関係で[ログインが遅くなることがあるのが知られている](https://www.google.com/search?q=usedns+no)。

```sshd_config
ClientAliveInterval 180

UseDNS no
PermitRootLogin no
```

### cloud-config.ymlの生成

手順によれば、[GitHubの手順を参考](https://help.github.com/en/articles/connecting-to-github-with-ssh
)に公開鍵認証用のキーペアを作成することが求められる。既に用意しているものがあればそれを使用しても良い。今回は、RancherOS用に新たなキーペアを生成することとする。

ここで`cloud-config.yml`に記載した公開鍵は、`rancher`ユーザの`~/.ssh/authorized_keys`に登録される。

キーペアの生成後、以下のコマンドでcloud-config.ymlを作成する。

`echo -n -e "#cloud-config\nssh_authorized_keys:\n  - " > cloud-config.yml`
`echo "<生成した公開鍵>" >> cloud-config.yml`

### ディスクへの書き込み

念のため、ディスク一覧を確認後実行。

```bash
sudo ls /dev/sd*
sudo ros install -c cloud-config.yml -d /dev/sda
```

再起動確認されるので、再起動。

#### Tips: rancher.password パラメータについて

`rancher.password` は、一時期、無効なパラメータとするように検討されていたようだ。

* https://github.com/rancher/os/issues/1467
* https://github.com/rancher/os/issues/1563

とはいえ、現在でも使用はデフォルトで可能である。セキュリティポリシー等でパスワードログインを無効にしたい場合は、こちらのドキュメントを参照して、カーネルから無効にすることができる。

* https://rancher.com/docs/os/v1.x/en/installation/configuration/disable-access-to-system/

### 再起動後、SSHログイン

公開鍵認証で`rancher`にログインできることを確認する。下記はTera Termの例。

## その他

### ネットワーク設定について

ここに書いてある。

https://rancher.com/docs/os/v1.2/en/networking/interfaces/

```
sudo ros config set rancher.network.interfaces.eth0.address 192.168.*.*/24
sudo ros config set rancher.network.interfaces.eth0.gateway 192.168.*.*
sudo ros config set rancher.network.interfaces.eth0.mtu 1500
sudo ros config set rancher.network.interfaces.eth0.dhcp false
```

設定が即時反映されるか不明だったため、念のため `&&` で実行したが、これらの設定はreboot後に反映されるようだ。

### RancherOS上で実行可能コマンドについて

RancherOSでは、全てDocker上でごにょごにょする思想となっている。とはいえ、ベースはLinuxなので、それなりのコマンドは使用できるようになっている。

```
$ echo $PATH
/bin:/sbin:/usr/bin:/usr/sbin
```

ところで、`/bin`と`/sbin`を見ると、`/usr/bin`と`/usr/sbin`にシンボリックリンクされている。

```
[rancher@rancher ~]$ ls -al /bin /sbin
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 /bin -> usr/bin
lrwxrwxrwx    1 root     root             8 Oct 25 18:43 /sbin -> usr/sbin
```

つまり、RancherOS v1.5.1で使えるコマンドは以下のようになっている。参考になれば幸い。

<details><summary>$ ls -1 /usr/bin /usr/sbin</summary><div>

```
[rancher@rancher ~]$ ls -1 /usr/bin /usr/sbin
/usr/bin:
[
[[
acpi_listen
ar
ash
autologin
awk
basename
bash
blkdiscard
bunzip2
busybox
bzcat
cat
chattr
chgrp
chmod
chown
chrt
chvt
cksum
clear
cloud-init-execute
cloud-init-save
cmp
cp
cpio
crontab
cut
date
dc
dd
deallocvt
df
diff
dirname
dmesg
dnsdomainname
docker
docker-containerd
docker-containerd-ctr
docker-containerd-shim
docker-init
docker-proxy
docker-runc
dockerd
dockerlaunch
dos2unix
du
dumpkmap
dumpsexp
echo
egrep
eject
env
envsubst
expr
factor
fallocate
false
fatattr
fdflush
fgrep
fincore
find
fold
free
fuser
getopt
gettext
gettext.sh
gpg-error
gpgrt-config
grep
growpart
gunzip
gzip
head
hexdump
hmac256
hostid
hostname
id
install
ipcrm
ipcs
iptables-xml
jq
kill
killall
last
less
link
linux32
linux64
ln
logger
login
logname
ls
lsattr
lsof
lspci
lsscsi
lsusb
lzcat
lzcmp
lzdiff
lzegrep
lzfgrep
lzgrep
lzless
lzma
lzmadec
lzmainfo
lzmore
md5sum
mesg
microcom
mkdir
mkfifo
mknod
mkpasswd
mktemp
more
mount
mountpoint
mpicalc
mt
mv
netstat
ngettext
nice
nl
nohup
nproc
nsenter
nslookup
od
openvt
os-subscriber
passwd
paste
patch
pidof
ping
ping6
pipe_progress
printenv
printf
ps
pwd
readlink
realpath
recovery
renice
reset
resize
respawn
rm
rmdir
rngtest
ros
rsync
run-parts
scp
sed
seq
setarch
setkeycodes
setpriv
setserial
setsid
sftp
sh
sha1sum
sha256sum
sha3sum
sha512sum
shred
shuf
sleep
sort
ssh
ssh-add
ssh-agent
ssh-copy-id
ssh-keygen
ssh-keyscan
ssl_client
start_ntp.sh
stdlogctl
strings
stty
su
sudo
sudoedit
sudoreplay
svc
sync
system-docker
system-docker-runc
tail
tar
tee
telnet
test
tftp
time
top
touch
tr
traceroute
true
truncate
tty
udevadm
umount
uname
uniq
unix2dos
unlink
unlzma
unshare
unxz
unzip
uptime
usleep
uudecode
uuencode
uuidgen
uuidparse
vi
vlock
w
watch
wc
wget
which
who
whoami
xargs
xxd
xz
xzcat
xzcmp
xzdec
xzdiff
xzegrep
xzfgrep
xzgrep
xzless
xzmore
yes
zcat

/usr/sbin:
acpid
addgroup
addpart
adduser
agetty
arping
badblocks
blkdeactivate
blkid
blkzone
bridge
chpasswd
chroot
crond
cryptsetup
cryptsetup-reencrypt
ctstat
delgroup
delpart
deluser
devmem
dhcpcd
dmeventd
dmsetup
dmstats
dnsd
dumpe2fs
e2freefrag
e2fsck
e2label
e2undo
e4crypt
ether-wake
fbset
fdformat
fdisk
filefrag
freeramdisk
fsadm
fsck
fsck.ext2
fsck.ext3
fsck.ext4
fsck.xfs
fstrim
genl
getty
halt
hdparm
hwclock
i2cdetect
i2cdump
i2cget
i2cset
ifcfg
ifconfig
ifdown
ifstat
ifup
inetd
init
insmod
integritysetup
ip
ip6tables
ip6tables-restore
ip6tables-save
ipaddr
iplink
ipneigh
iproute
iprule
iptables
iptables-restore
iptables-save
iptunnel
iwconfig
iwgetid
iwlist
iwpriv
iwspy
kacpimon
kdump
kexec
killall5
klogd
lnstat
loadfont
loadkmap
logrotate
logsave
losetup
lsmod
lvchange
lvconvert
lvcreate
lvdisplay
lvextend
lvm
lvmconf
lvmconfig
lvmdiskscan
lvmdump
lvmsadc
lvmsar
lvreduce
lvremove
lvrename
lvresize
lvs
lvscan
makedevs
mdadm
mdev
mkdosfs
mke2fs
mkfs.ext2
mkfs.ext3
mkfs.ext4
mkfs.xfs
mklost+found
mkswap
modprobe
nameif
netconf
nstat
ntpd
parted
partprobe
partx
pivot_root
poweroff
pvchange
pvck
pvcreate
pvdisplay
pvmove
pvremove
pvresize
pvs
pvscan
rdate
readprofile
reboot
resize2fs
resizepart
rmmod
rngd
route
routef
routel
rsyslogd
rtacct
rtmon
rtpr
rtstat
runlevel
setconsole
setlogcons
sfdisk
shutdown
ss
sshd
start-stop-daemon
sulogin
swapoff
swapon
switch_root
sysctl
syslogd
tc
tune2fs
ubirename
udevadm
udevd
uevent
vconfig
veritysetup
vgcfgbackup
vgcfgrestore
vgchange
vgck
vgconvert
vgcreate
vgdisplay
vgexport
vgextend
vgimport
vgimportclone
vgmerge
vgmknodes
vgreduce
vgremove
vgrename
vgs
vgscan
vgsplit
visudo
vmcore-dmesg
wait-for-docker
watchdog
wpa_cli
wpa_passphrase
wpa_supplicant
xfs_admin
xfs_bmap
xfs_copy
xfs_db
xfs_estimate
xfs_freeze
xfs_fsr
xfs_growfs
xfs_info
xfs_io
xfs_logprint
xfs_mdrestore
xfs_metadump
xfs_mkfile
xfs_ncheck
xfs_quota
xfs_repair
xfs_rtcp
xfs_spaceman
```

</div></details>

具体的には`busybox`を使っていたり、`ros`というバイナリにまとめられていたりするので、興味がある人はこちらも参考にしてほしい。

<details><summary>$ ls -al /usr/bin</summary><div>

```
[rancher@rancher ~]$ ls -al /usr/bin/
total 36064
drwxr-xr-x    1 root     root          4096 Apr  9 08:16 .
drwxr-xr-x    1 root     root          4096 Feb 11 17:00 ..
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 [ -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 [[ -> ../../bin/busybox
-rwxr-xr-x    1 root     root         10648 Oct 25 18:43 acpi_listen
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 ar -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 ash -> busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 autologin -> /usr/bin/ros
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 awk -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 basename -> ../../bin/busybox
-rwxr-xr-x    1 root     root        749600 Oct 25 18:43 bash
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 blkdiscard -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 bunzip2 -> ../../bin/busybox
-rwsr-xr-x    1 root     root        721992 Oct 25 18:43 busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 bzcat -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 cat -> busybox
-rwxr-xr-x    1 root     root         10280 Oct 25 18:43 chattr
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 chgrp -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 chmod -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 chown -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 chrt -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 chvt -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 cksum -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 clear -> ../../bin/busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 cloud-init-execute -> /usr/bin/ros
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 cloud-init-save -> /usr/bin/ros
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 cmp -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 cp -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 cpio -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 crontab -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 cut -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 date -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 dc -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 dd -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 deallocvt -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 df -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 diff -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 dirname -> ../../bin/busybox
-rwxr-xr-x    1 root     root         64512 Oct 25 18:43 dmesg
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 dnsdomainname -> busybox
lrwxrwxrwx    1 root     root            30 Apr  9 08:16 docker -> /var/lib/rancher/engine/docker
lrwxrwxrwx    1 root     root            41 Apr  9 08:16 docker-containerd -> /var/lib/rancher/engine/docker-containerd
lrwxrwxrwx    1 root     root            45 Apr  9 08:16 docker-containerd-ctr -> /var/lib/rancher/engine/docker-containerd-ctr
lrwxrwxrwx    1 root     root            46 Apr  9 08:16 docker-containerd-shim -> /var/lib/rancher/engine/docker-containerd-shim
lrwxrwxrwx    1 root     root            35 Apr  9 08:16 docker-init -> /var/lib/rancher/engine/docker-init
lrwxrwxrwx    1 root     root            36 Apr  9 08:16 docker-proxy -> /var/lib/rancher/engine/docker-proxy
lrwxrwxrwx    1 root     root            35 Apr  9 08:16 docker-runc -> /var/lib/rancher/engine/docker-runc
lrwxrwxrwx    1 root     root            31 Apr  9 08:16 dockerd -> /var/lib/rancher/engine/dockerd
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 dockerlaunch -> /usr/bin/ros
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 dos2unix -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 du -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 dumpkmap -> busybox
-rwxr-xr-x    1 root     root         14328 Oct 25 18:43 dumpsexp
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 echo -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 egrep -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 eject -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 env -> ../../bin/busybox
-rwxr-xr-x    1 root     root         26832 Oct 25 18:43 envsubst
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 expr -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 factor -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 fallocate -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 false -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 fatattr -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 fdflush -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 fgrep -> busybox
-rwxr-xr-x    1 root     root         27032 Oct 25 18:43 fincore
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 find -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 fold -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 free -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 fuser -> ../../bin/busybox
-rwxr-xr-x    1 root     root         14448 Oct 25 18:43 getopt
-rwxr-xr-x    1 root     root         22712 Oct 25 18:43 gettext
-rwxr-xr-x    1 root     root          4629 Oct 25 18:43 gettext.sh
-rwxr-xr-x    1 root     root         26896 Oct 25 18:43 gpg-error
-rwxr-xr-x    1 root     root          2355 Oct 25 18:43 gpgrt-config
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 grep -> busybox
-rwxr-xr-x    1 root     root         21431 Feb 11 17:06 growpart
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 gunzip -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 gzip -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 head -> ../../bin/busybox
-rwxr-xr-x    1 root     root         43520 Oct 25 18:43 hexdump
-rwxr-xr-x    1 root     root         14776 Oct 25 18:43 hmac256
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 hostid -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 hostname -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 id -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 install -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 ipcrm -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 ipcs -> ../../bin/busybox
lrwxrwxrwx    1 root     root            18 Apr  9 08:16 iptables-xml -> /usr/sbin/iptables
-rwxr-xr-x    1 root     root         22896 Oct 25 18:43 jq
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 kill -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 killall -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 last -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 less -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 link -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 linux32 -> setarch
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 linux64 -> setarch
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 ln -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 logger -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 login -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 logname -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 ls -> busybox
-rwxr-xr-x    1 root     root         10272 Oct 25 18:43 lsattr
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 lsof -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 lspci -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 lsscsi -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 lsusb -> ../../bin/busybox
lrwxrwxrwx    1 root     root             2 Oct 25 18:43 lzcat -> xz
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzcmp -> xzdiff
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzdiff -> xzdiff
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzegrep -> xzgrep
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzfgrep -> xzgrep
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzgrep -> xzgrep
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzless -> xzless
lrwxrwxrwx    1 root     root             2 Oct 25 18:43 lzma -> xz
-rwxr-xr-x    1 root     root         10264 Oct 25 18:43 lzmadec
-rwxr-xr-x    1 root     root         10240 Oct 25 18:43 lzmainfo
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 lzmore -> xzmore
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 md5sum -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 mesg -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 microcom -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mkdir -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 mkfifo -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mknod -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 mkpasswd -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mktemp -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 more -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mount -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mountpoint -> busybox
-rwxr-xr-x    1 root     root         14496 Oct 25 18:43 mpicalc
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mt -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 mv -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 netstat -> busybox
-rwxr-xr-x    1 root     root         22720 Oct 25 18:43 ngettext
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 nice -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 nl -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 nohup -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 nproc -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 nsenter -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 nslookup -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 od -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 openvt -> ../../bin/busybox
-rwxr-xr-x    1 root     root          1501 Feb 11 17:00 os-subscriber
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 passwd -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 paste -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 patch -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 pidof -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 ping -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 ping6 -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 pipe_progress -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 printenv -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 printf -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 ps -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 pwd -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 readlink -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 realpath -> ../../bin/busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 recovery -> /usr/bin/ros
-rwxr-xr-x    1 root     root         10264 Oct 25 18:43 renice
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 reset -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 resize -> ../../bin/busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 respawn -> /usr/bin/ros
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 rm -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 rmdir -> busybox
-rwxr-xr-x    1 root     root         15064 Oct 25 18:43 rngtest
-rwxr-xr-x    1 root     root      13633768 Feb 11 17:08 ros
-rwxr-xr-x    1 root     root        349520 Oct 25 18:43 rsync
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 run-parts -> busybox
-rwxr-xr-x    1 root     root         84024 Oct 25 18:43 scp
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 sed -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 seq -> ../../bin/busybox
-rwxr-xr-x    1 root     root         14400 Oct 25 18:43 setarch
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 setkeycodes -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 setpriv -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 setserial -> busybox
-rwxr-xr-x    1 root     root         10296 Oct 25 18:43 setsid
-rwxr-xr-x    1 root     root        133400 Oct 25 18:43 sftp
lrwxrwxrwx    1 root     root             4 Oct 25 18:43 sh -> bash
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 sha1sum -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 sha256sum -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 sha3sum -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 sha512sum -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 shred -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 shuf -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 sleep -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 sort -> ../../bin/busybox
-rwxr-xr-x    1 root     root        649880 Oct 25 18:43 ssh
-rwxr-xr-x    1 root     root        313400 Oct 25 18:43 ssh-add
-rwxr-xr-x    1 root     root        297016 Oct 25 18:43 ssh-agent
-rwxr-xr-x    1 root     root         10658 Oct 25 18:43 ssh-copy-id
-rwxr-xr-x    1 root     root        383192 Oct 25 18:43 ssh-keygen
-rwxr-xr-x    1 root     root        395512 Oct 25 18:43 ssh-keyscan
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 ssl_client -> ../../bin/busybox
-rwxr-xr-x    1 root     root           146 Feb 11 17:00 start_ntp.sh
-rwxr-xr-x    1 root     root          6064 Oct 25 18:43 stdlogctl
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 strings -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 stty -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 su -> busybox
-rwsr-xr-x    1 root     root        113520 Oct 25 18:43 sudo
lrwxrwxrwx    1 root     root             4 Oct 25 18:43 sudoedit -> sudo
-rwxr-xr-x    1 root     root         48536 Oct 25 18:43 sudoreplay
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 svc -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 sync -> busybox
-rwxr-xr-x    1 root     root      10352240 Feb 11 16:43 system-docker
-rwxr-xr-x    1 root     root       7690728 Feb 11 16:43 system-docker-runc
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 tail -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 tar -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 tee -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 telnet -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 test -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 tftp -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 time -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 top -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 touch -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 tr -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 traceroute -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 true -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 truncate -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 tty -> ../../bin/busybox
-rwxr-xr-x    1 root     root        298792 Oct 25 18:43 udevadm
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 umount -> busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 uname -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 uniq -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 unix2dos -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 unlink -> ../../bin/busybox
lrwxrwxrwx    1 root     root             2 Oct 25 18:43 unlzma -> xz
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 unshare -> ../../bin/busybox
lrwxrwxrwx    1 root     root             2 Oct 25 18:43 unxz -> xz
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 unzip -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 uptime -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 usleep -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 uudecode -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 uuencode -> ../../bin/busybox
-rwxr-xr-x    1 root     root         10320 Oct 25 18:43 uuidgen
-rwxr-xr-x    1 root     root         31208 Oct 25 18:43 uuidparse
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 vi -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 vlock -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 w -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 watch -> busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 wc -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 wget -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 which -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 who -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 whoami -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 xargs -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 xxd -> ../../bin/busybox
-rwxr-xr-x    1 root     root         68504 Oct 25 18:43 xz
lrwxrwxrwx    1 root     root             2 Oct 25 18:43 xzcat -> xz
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 xzcmp -> xzdiff
-rwxr-xr-x    1 root     root         10264 Oct 25 18:43 xzdec
-rwxr-xr-x    1 root     root          6634 Oct 25 18:43 xzdiff
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 xzegrep -> xzgrep
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 xzfgrep -> xzgrep
-rwxr-xr-x    1 root     root          5630 Oct 25 18:43 xzgrep
-rwxr-xr-x    1 root     root          1804 Oct 25 18:43 xzless
-rwxr-xr-x    1 root     root          2163 Oct 25 18:43 xzmore
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 yes -> ../../bin/busybox
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 zcat -> busybox
```

</div></details>

<details><summary>$ ls -al /usr/sbin</summary><div>

```
[rancher@rancher ~]$ ls -al /usr/sbin/
total 15788
drwxr-xr-x    1 root     root          4096 Apr  9 08:16 .
drwxr-xr-x    1 root     root          4096 Feb 11 17:00 ..
-rwxr-xr-x    1 root     root         43664 Oct 25 18:43 acpid
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 addgroup -> ../../bin/busybox
-rwxr-xr-x    1 root     root         18568 Oct 25 18:43 addpart
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 adduser -> ../../bin/busybox
-rwxr-xr-x    1 root     root         47976 Oct 25 18:43 agetty
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 arping -> ../../bin/busybox
-rwxr-xr-x    1 root     root         22800 Oct 25 18:43 badblocks
-rwxr-xr-x    1 root     root         13906 Oct 25 18:43 blkdeactivate
-rwxr-xr-x    1 root     root         81032 Oct 25 18:43 blkid
-rwxr-xr-x    1 root     root         39488 Oct 25 18:43 blkzone
-rwxr-xr-x    1 root     root         72280 Oct 25 18:43 bridge
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 chpasswd -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 chroot -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 crond -> ../../bin/busybox
-rwxr-xr-x    1 root     root         76936 Oct 25 18:43 cryptsetup
-rwxr-xr-x    1 root     root         79608 Oct 25 18:43 cryptsetup-reencrypt
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 ctstat -> lnstat
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 delgroup -> ../../bin/busybox
-rwxr-xr-x    1 root     root         18568 Oct 25 18:43 delpart
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 deluser -> ../../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 devmem -> ../bin/busybox
-r-xr-xr-x    1 root     root        249144 Oct 25 18:43 dhcpcd
-rwxr-xr-x    1 root     root         38824 Oct 25 18:43 dmeventd
-rwxr-xr-x    1 root     root        133544 Oct 25 18:43 dmsetup
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 dmstats -> dmsetup
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 dnsd -> ../../bin/busybox
-rwxr-xr-x    1 root     root         23056 Oct 25 18:43 dumpe2fs
-rwxr-xr-x    1 root     root         10312 Oct 25 18:43 e2freefrag
-rwxr-xr-x    1 root     root        267560 Oct 25 18:43 e2fsck
lrwxrwxrwx    1 root     root             7 Oct 25 18:43 e2label -> tune2fs
-rwxr-xr-x    1 root     root         14552 Oct 25 18:43 e2undo
-rwxr-xr-x    1 root     root         18632 Oct 25 18:43 e4crypt
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 ether-wake -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 fbset -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 fdformat -> ../../bin/busybox
-rwxr-xr-x    1 root     root        106784 Oct 25 18:43 fdisk
-rwxr-xr-x    1 root     root         14376 Oct 25 18:43 filefrag
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 freeramdisk -> ../bin/busybox
-rwxr-xr-x    1 root     root         19691 Oct 25 18:43 fsadm
-rwxr-xr-x    1 root     root         23032 Oct 25 18:43 fsck
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 fsck.ext2 -> e2fsck
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 fsck.ext3 -> e2fsck
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 fsck.ext4 -> e2fsck
-rwxr-xr-x    1 root     root           433 Oct 25 18:43 fsck.xfs
-rwxr-xr-x    1 root     root         39584 Oct 25 18:43 fstrim
-rwxr-xr-x    1 root     root         51712 Oct 25 18:43 genl
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 getty -> ../bin/busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 halt -> /usr/bin/ros
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 hdparm -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 hwclock -> ../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 i2cdetect -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 i2cdump -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 i2cget -> ../../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 i2cset -> ../../bin/busybox
-rwxr-xr-x    1 root     root          3058 Oct 25 18:43 ifcfg
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 ifconfig -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 ifdown -> ../bin/busybox
-rwxr-xr-x    1 root     root         39352 Oct 25 18:43 ifstat
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 ifup -> ../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 inetd -> ../../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 init -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 insmod -> ../bin/busybox
-rwxr-xr-x    1 root     root         32904 Oct 25 18:43 integritysetup
-rwxr-xr-x    1 root     root        448320 Oct 25 18:43 ip
lrwxrwxrwx    1 root     root            18 Apr  9 08:16 ip6tables -> /usr/sbin/iptables
lrwxrwxrwx    1 root     root            18 Apr  9 08:16 ip6tables-restore -> /usr/sbin/iptables
lrwxrwxrwx    1 root     root            18 Apr  9 08:16 ip6tables-save -> /usr/sbin/iptables
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 ipaddr -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 iplink -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 ipneigh -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 iproute -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 iprule -> ../bin/busybox
-rwxr-xr-x    1 root     root        465280 Nov  5 01:54 iptables
lrwxrwxrwx    1 root     root            18 Apr  9 08:16 iptables-restore -> /usr/sbin/iptables
lrwxrwxrwx    1 root     root            18 Apr  9 08:16 iptables-save -> /usr/sbin/iptables
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 iptunnel -> ../bin/busybox
-rwxr-xr-x    1 root     root         76184 Oct 25 18:43 iwconfig
lrwxrwxrwx    1 root     root             8 Oct 25 18:43 iwgetid -> iwconfig
lrwxrwxrwx    1 root     root             8 Oct 25 18:43 iwlist -> iwconfig
lrwxrwxrwx    1 root     root             8 Oct 25 18:43 iwpriv -> iwconfig
lrwxrwxrwx    1 root     root             8 Oct 25 18:43 iwspy -> iwconfig
-rwxr-xr-x    1 root     root         18528 Oct 25 18:43 kacpimon
-rwxr-xr-x    1 root     root         10232 Oct 25 18:43 kdump
-rwxr-xr-x    1 root     root        154576 Oct 25 18:43 kexec
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 killall5 -> ../../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 klogd -> ../bin/busybox
-rwxr-xr-x    1 root     root         18928 Oct 25 18:43 lnstat
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 loadfont -> ../../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 loadkmap -> ../bin/busybox
-rwxr-xr-x    1 root     root         64320 Oct 25 18:43 logrotate
-rwxr-xr-x    1 root     root         10320 Oct 25 18:43 logsave
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 losetup -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 lsmod -> ../bin/busybox
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvchange -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvconvert -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvcreate -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvdisplay -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvextend -> lvm
-rwxr-xr-x    1 root     root       1911328 Oct 25 18:43 lvm
-rwxr-xr-x    1 root     root         12847 Oct 25 18:43 lvmconf
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvmconfig -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvmdiskscan -> lvm
-rwxr-xr-x    1 root     root         10312 Oct 25 18:43 lvmdump
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvmsadc -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvmsar -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvreduce -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvremove -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvrename -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvresize -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvs -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 lvscan -> lvm
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 makedevs -> ../bin/busybox
-rwxr-xr-x    1 root     root        472576 Oct 25 18:43 mdadm
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 mdev -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 mkdosfs -> ../bin/busybox
-rwxr-xr-x    1 root     root        110504 Oct 25 18:43 mke2fs
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 mkfs.ext2 -> mke2fs
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 mkfs.ext3 -> mke2fs
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 mkfs.ext4 -> mke2fs
-rwxr-xr-x    1 root     root        463280 Oct 25 18:43 mkfs.xfs
-rwxr-xr-x    1 root     root          6104 Oct 25 18:43 mklost+found
-rwxr-xr-x    1 root     root         72576 Oct 25 18:43 mkswap
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 modprobe -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 nameif -> ../bin/busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 netconf -> /usr/bin/ros
-rwxr-xr-x    1 root     root         22864 Oct 25 18:43 nstat
-rwxr-xr-x    1 root     root        676584 Oct 25 18:43 ntpd
-rwxr-xr-x    1 root     root         68632 Oct 25 18:43 parted
-rwxr-xr-x    1 root     root         10280 Oct 25 18:43 partprobe
-rwxr-xr-x    1 root     root         76792 Oct 25 18:43 partx
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 pivot_root -> ../bin/busybox
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 poweroff -> /usr/bin/ros
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvchange -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvck -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvcreate -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvdisplay -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvmove -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvremove -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvresize -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvs -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 pvscan -> lvm
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 rdate -> ../../bin/busybox
-rwxr-xr-x    1 root     root         14528 Oct 25 18:43 readprofile
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 reboot -> /usr/bin/ros
-rwxr-xr-x    1 root     root         52272 Oct 25 18:43 resize2fs
-rwxr-xr-x    1 root     root         35384 Oct 25 18:43 resizepart
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 rmmod -> ../bin/busybox
-rwxr-xr-x    1 root     root         23768 Oct 25 18:43 rngd
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 route -> ../bin/busybox
-rwxr-xr-x    1 root     root           173 Oct 25 18:43 routef
-rwxr-xr-x    1 root     root          1627 Oct 25 18:43 routel
-rwxr-xr-x    1 root     root        583776 Oct 25 18:43 rsyslogd
-rwxr-xr-x    1 root     root         37296 Oct 25 18:43 rtacct
-rwxr-xr-x    1 root     root         47528 Oct 25 18:43 rtmon
-rwxr-xr-x    1 root     root            37 Oct 25 18:43 rtpr
lrwxrwxrwx    1 root     root             6 Oct 25 18:43 rtstat -> lnstat
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 runlevel -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 setconsole -> ../bin/busybox
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 setlogcons -> ../../bin/busybox
-rwxr-xr-x    1 root     root         98184 Oct 25 18:43 sfdisk
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 shutdown -> /usr/bin/ros
-rwxr-xr-x    1 root     root        115560 Oct 25 18:43 ss
-rwxr-xr-x    1 root     root        687944 Oct 25 18:43 sshd
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 start-stop-daemon -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 sulogin -> ../bin/busybox
-rwxr-xr-x    1 root     root         18704 Oct 25 18:43 swapoff
-rwxr-xr-x    1 root     root         43712 Oct 25 18:43 swapon
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 switch_root -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 sysctl -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 syslogd -> ../bin/busybox
-rwxr-xr-x    1 root     root        358736 Oct 25 18:43 tc
-rwxr-xr-x    1 root     root         93768 Oct 25 18:43 tune2fs
lrwxrwxrwx    1 root     root            17 Oct 25 18:43 ubirename -> ../../bin/busybox
lrwxrwxrwx    1 root     root            16 Oct 25 18:43 udevadm -> /usr/bin/udevadm
-rwxr-xr-x    1 root     root        286536 Oct 25 18:43 udevd
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 uevent -> ../bin/busybox
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 vconfig -> ../bin/busybox
-rwxr-xr-x    1 root     root         32704 Oct 25 18:43 veritysetup
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgcfgbackup -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgcfgrestore -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgchange -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgck -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgconvert -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgcreate -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgdisplay -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgexport -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgextend -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgimport -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgimportclone -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgmerge -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgmknodes -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgreduce -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgremove -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgrename -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgs -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgscan -> lvm
lrwxrwxrwx    1 root     root             3 Oct 25 18:43 vgsplit -> lvm
-rwxr-xr-x    1 root     root        177856 Oct 25 18:43 visudo
-rwxr-xr-x    1 root     root         14376 Oct 25 18:43 vmcore-dmesg
lrwxrwxrwx    1 root     root            12 Apr  9 08:16 wait-for-docker -> /usr/bin/ros
lrwxrwxrwx    1 root     root            14 Oct 25 18:43 watchdog -> ../bin/busybox
-rwxr-xr-x    1 root     root         96992 Oct 25 18:43 wpa_cli
-rwxr-xr-x    1 root     root         52088 Oct 25 18:43 wpa_passphrase
-rwxr-xr-x    1 root     root        648808 Oct 25 18:43 wpa_supplicant
-rwxr-xr-x    1 root     root          1380 Oct 25 18:43 xfs_admin
-rwxr-xr-x    1 root     root           638 Oct 25 18:43 xfs_bmap
-rwxr-xr-x    1 root     root        436968 Oct 25 18:43 xfs_copy
-rwxr-xr-x    1 root     root        713384 Oct 25 18:43 xfs_db
-rwxr-xr-x    1 root     root         10224 Oct 25 18:43 xfs_estimate
-rwxr-xr-x    1 root     root           767 Oct 25 18:43 xfs_freeze
-rwxr-xr-x    1 root     root         31136 Oct 25 18:43 xfs_fsr
-rwxr-xr-x    1 root     root        428752 Oct 25 18:43 xfs_growfs
-rwxr-xr-x    1 root     root           472 Oct 25 18:43 xfs_info
-rwxr-xr-x    1 root     root        147152 Oct 25 18:43 xfs_io
-rwxr-xr-x    1 root     root        457296 Oct 25 18:43 xfs_logprint
-rwxr-xr-x    1 root     root        412296 Oct 25 18:43 xfs_mdrestore
-rwxr-xr-x    1 root     root           747 Oct 25 18:43 xfs_metadump
-rwxr-xr-x    1 root     root          1007 Oct 25 18:43 xfs_mkfile
-rwxr-xr-x    1 root     root           650 Oct 25 18:43 xfs_ncheck
-rwxr-xr-x    1 root     root        894072 Oct 25 18:43 xfs_quota
-rwxr-xr-x    1 root     root        704296 Oct 25 18:43 xfs_repair
-rwxr-xr-x    1 root     root        635952 Oct 25 18:43 xfs_rtcp
-rwxr-xr-x    1 root     root        791600 Oct 25 18:43 xfs_spaceman
```

</div></details>