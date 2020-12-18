Note: I wasn't able to add the keys used for signing NVIDIA drivers to truested list and hence had to turn off secure boot. Most of this tutorial is about adding the module signing keys and hence not very useful if it can't be added to the list of kernel trusted keys.

1. Check if the nVidia card is supported
	`lscpi | grep -E "VGA|3D"`
   
   The above command should only show one VGA controller. If it shows two VGA controller, it means both Intel and Nvidia controllers are loaded. This tutorial won't work in that case and program that supports Nvidia Optimus technology should be used instead. Or, you can select to boot only the dedicated Nvidia controller.
   
2. Download appropriate driver from nvidia.com

3. Generate signing key to sign the kernel module

	1. Create a secure folder to store signing secret and key
		`sudo mkdir /certs`
	
	2. Create config file for signing secret/cert
		`sudo vi /certs/cert.conf`
		
		Add certificate config to above file. Sample config:
		
		```
		[ req ]
		default_bits = 4096
		distinguished_name = req_distinguished_name
		prompt = no
		string_mask = utf8only
		x509_extensions = cert_extensions

		[ req_distinguished_name ]
		countryName = US
		stateOrProvinceName = New York
		localityName = New York City
		commonName = Secure Boot Signing
		emailAddress = email@site.com

		[ cert_extensions ]
		basicConstraints=critical,CA:FALSE
		keyUsage=digitalSignature
		subjectKeyIdentifier=hash
		authorityKeyIdentifier=keyid
		nsComment = "Self signed certificate for signing kernel modules"
		```
	3. Generate key/secret pair using the above config
		`sudo openssl req -config ./cert.conf -new -nodes -utf8 -sha512 -days 3650 -batch -x509 -outform DER -out signing_key.x509 -keyout signing_key.private`
		
3. Make sure the system is upto date
	In Arch Linux, you can do `sudo pacman -Syu`
	
	After that, `reboot`.
	
4. Disable nouveau driver
	1. `echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf`
	2. Disable Nouveau in grun
		- For Arch Linux, open file `/etc/default/grub` and append `rd.driver.blacklist=nouveau` to the end of `GRUB_CMDLINE_LINUX=""`
		- For Fedora, edit `/etc/sysconfig/grub` instead
	3. Update grub2conf
		- In Arch Linux, run `grub-mkconfig -o /boot/efi/EFI/Manjaro/grubx64.efi` as root user
	4. Generate initramfs
		- In Arch Linux:
			Backup old image: `sudo mv /boot/initramfs-5.6-x86_64.img /boot/initramfs-5.6-x86_64.nouveau.img`
			Generate new one: `sudo mkinitcpio -g /boot/initramfs-5.6-x86_64.img`
		

5. Install required dependecies
	- `sudo pacman -S gcc make dkms libglvnd pkgconf`
	- Also need to install `linux-headers` corresponding to kernel version. You can check /usr/lib/modules/x.y to find out the version number


3. Sign Nvidia module using the key that was just generated
	- I could't figure this out, so I am still running the laptop with secure boot disabled.

4. If the brightness control keys aren't working, add the `Option "RegistryDwords" "EnableBrightnessControl=1"` under `Section "Device"` in `/etc/X11/xorg.conf`

	```
		Section "Device"
			Identifier     "Device0"
			Driver         "nvidia"
			VendorName     "NVIDIA Corporation"
			Option "RegistryDwords" "EnableBrightnessControl=1"
		EndSection
	```

5. If the latop goes on suspend loop after waking up from suspend, add `button.lid_init_state=open` to `GRUB_CMDLINE_LINUX` in file `/etc/default/grub`. Run `sudo update-grub` to update grub and then reboot.