IVOS_PROJECT_DIR=/mnt/exthdd/user/ivoshopov/projects


all: samba gmediarender minidlna trash-cli dev_tools openvpn default_usb_audio patch_mdadm


define WPA_SUPPLICANT_CONF
network={
    ssid="10"
    psk="_WPA_SUPPLICANT_PASSWORD_SSID_10_"
}
endef
export WPA_SUPPLICANT_CONF

wifi:
	echo "$$WPA_SUPPLICANT_CONF" > tmp
	@echo
	@echo "Please enter the WiFi password"
	@read password && cat tmp | sed "s/_WPA_SUPPLICANT_PASSWORD_SSID_10_/$$password/" >> /etc/wpa_supplicant/wpa_supplicant.conf
	@rm tmp
	@touch $@

mdadm: apt_update
	apt-get install -y mdadm
	mdadm --detail --scan | tee -a /etc/mdadm/mdadm.conf
	@touch $@

mount_exthdd: mdadm
	mkdir -p /mnt/raid1
	ln -s /mnt/raid1 /mnt/exthdd
	echo "/dev/md0  /mnt/raid1/               ext4    defaults,noatime  0       0" >> /etc/fstab
	mount -a
	@touch $@

apt_update: wifi
	apt-get update
	@touch $@

add_users: mount_exthdd
	sed -i "s'# HOME=/home'HOME=/mnt/exthdd/home'" /etc/default/useradd
	useradd ivoshopov
	useradd rosana
	sed -i "s/:pi$$/:pi,ivoshopov,rosana/g" /etc/group
	echo "ivoshopov ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
	@touch $@

samba: apt_update add_users
	apt-get -y install samba
	echo ""
	echo "Please enter the password for user ivoshopov in samba"
	smbpasswd -a ivoshopov
	echo "Please enter the password for user rosana in samba"
	smbpasswd -a rosana
	@touch $@

gmediarender: apt_update
	apt-get -y install gmediarender
	sed -i "s/#\s*ENABLED=1/ENABLED=1/" /etc/default/gmediarender
	#It looks that libgstreamer0.10 is not used only libgstreamer1.0 is enough
	#apt-get install libupnp-dev libgstreamer0.10-dev gstreamer0.10-plugins-base gstreamer0.10-plugins-good gstreamer0.10-plugins-bad gstreamer0.10-plugins-ugly gstreamer0.10-pulseaudio gstreamer0.10-alsa
	apt-get -y install gstreamer1.0-alsa gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-ugly gstreamer1.0-tools
	sed -i "s/INITIAL_VOLUME_DB=-10/INITIAL_VOLUME_DB=0/" /etc/init.d/gmediarender
	@touch $@

minidlna: apt_update mount_exthdd
	apt-get -y install minidlna
	mkdir -p /mnt/exthdd/music
	mkdir -p /mnt/exthdd/photos
	chgrp users /mnt/exthdd/music
	chgrp users /mnt/exthdd/photos
	ln -s /mnt/exthdd/music /var/lib/minidlna/music
	ln -s /mnt/exthdd/photos /var/lib/minidlna/photos
	mkdir -p /mnt/exthdd/minidlna_db
	chmod a+rw /mnt/exthdd/minidlna_db
	sed -i "s'#db_dir=/var/cache/minidlna'db_dir=/mnt/exthdd/minidlna_db'" /etc/minidlna.conf
	@touch $@

#Setup the default audio output to USB card
default_usb_audio:
	cp asound.conf /etc/

trash-cli: apt_update
	apt-get -y install trash-cli
	@touch $@

openvpn: apt_update
	apt-get -y install openvpn
	@touch $@

ivosbash ivosvim: mount_exthdd
	sudo -u ivoshopov make -C $(IVOS_PROJECT_DIR)/$@
	@touch $@

fs_resize:
	raspi-config --expand-rootfs
	@touch $@

dev_tools: apt_update
	apt-get -y install vim git indent ctags cscope screen
	@touch $@

fast_sshd:
	echo "UseDNS no" >> /etc/ssh/sshd_config
	@touch $@

patch_mdadm: mdadm
	patch /lib/udev/rules.d/64-md-raid-assembly.rules 64-md-raid-assembly.rules.patch
	@touch $@

create_raid: mdadm
	mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/disk/by-id/usb-ATA_ST2000LM015-2E81_0123456789ABCDE-0:0-part1 /dev/disk/by-id/ata-Hitachi_HTS545050B9SA00_110201PBN475M7EYXELE-part1
