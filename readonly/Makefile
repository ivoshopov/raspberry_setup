
all: autoremount

upgrade:
	apt-get update
	apt-get upgrade
	@touch $@

upgrade-reboot: upgrade
	reboot
	@touch $@

remove_useless: upgrade-reboot
	apt-get remove --purge wolfram-engine triggerhappy anacron logrotate dphys-swapfile xserver-common lightdm
	insserv -r x11-common; apt-get autoremove --purge
	@touch $@

add_busybox_log: remove_useless
	apt-get install busybox-syslogd
	dpkg --purge rsyslog
	sed -i 's|-C128|-O /var/log/messages|' /etc/default/busybox-syslogd
	@touch $@

cmdline: add_busybox_log
	sed -i 's/$$/ fastboot noswap ro/' /boot/cmdline.txt
	@touch $@

tmpfs: cmdline
	rm -rf /var/lib/dhcp/ /var/run /var/spool /var/lock
	ln -s /tmp /var/lib/dhcp
	ln -s /tmp /var/run
	ln -s /tmp /var/spool
	ln -s /tmp /var/lock
	@touch $@

ntp: tmpfs
	apt-get install ntp
	sed -i "s/fake-hwclock save/mount -o remount,rw \/\n  fake-hwclock save\n  mount -o remount,ro \//" /etc/cron.hourly/fake-hwclock
	dpkg-reconfigure tzdata
	@touch $@

init: ntp
	insserv -r bootlogs
	insserv -r console-setup
	@touch $@

fstab: init
	sed -i "s/\(\/dev\/mmcblk0p1\s*\/boot\s*vfat\s*defaults\) /\1,ro /" /etc/fstab
	sed -i "s/\(\/dev\/mmcblk0p2\s*\/\s*ext4\s*defaults,noatime\) /\1,ro /" /etc/fstab
	echo "tmpfs           /tmp            tmpfs   nosuid,nodev         0       0" >> /etc/fstab
	echo "tmpfs           /var/log        tmpfs   nosuid,nodev         0       0" >> /etc/fstab
	echo "tmpfs           /var/tmp        tmpfs   nosuid,nodev         0       0" >> /etc/fstab
	@touch $@

autoremount: fstab
	echo "DPkg {" > /etc/apt/apt.conf.d/80autoremount
	echo "    // Auto re-mounting of a readonly /" >> /etc/apt/apt.conf.d/80autoremount
	echo "    Pre-Invoke { \"mount -o remount,rw /\"; };" >> /etc/apt/apt.conf.d/80autoremount
	echo "    Post-Invoke { \"test \$${NO_APT_REMOUNT:-no} = yes || mount -o remount,ro / || true\"; };" >> /etc/apt/apt.conf.d/80autoremount
	echo "};" >> /etc/apt/apt.conf.d/80autoremount
	@touch $@

