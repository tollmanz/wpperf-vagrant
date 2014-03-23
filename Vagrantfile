# -*- mode: ruby -*-
# vi: set ft=ruby :
vagrant_dir = File.expand_path(File.dirname(__FILE__))

# Ensure that shared dirs are available created
FileUtils.mkdir_p(File.dirname(__FILE__)+'/www')
FileUtils.mkdir_p(File.dirname(__FILE__)+'/srv')

Vagrant.configure("2") do |config|
	# Add the box and settings for virtualbox
	config.vm.box = "wheezy-point-three-vbox"
	config.vm.box_url = "http://dl.dropboxusercontent.com/s/zswmalbfkxmfr1u/wheezy-point-three-vbox.box?dl=1"

	config.vm.provider "virtualbox" do |v|
		v.customize ["modifyvm", :id, "--memory", 512]
	end

	# Add the box and settings for fusion
	config.vm.provider "vmware_fusion" do |v, override|
		v.vmx["memsize"] = "512"
		override.vm.box = "wheezy-point-three-fusion"
		override.vm.box_url = "http://dl.dropboxusercontent.com/s/00cwx8tib0prdv3/wheezy-point-three-fusion.box?dl=1"
	end

	config.vm.hostname = "wpperf"
	config.vm.network :private_network, ip: "192.168.11.44"
	config.hostsupdater.aliases = ["wpperf.dev"]

	# Make SSH key available to the box
	config.ssh.forward_agent = true

	# Mount the local project's www/ directory as /var/www inside the virtual machine. This will
	# be mounted as the 'vagrant' user at first, then unmounted and mounted again as 'www-data'
	# during provisioning.
	config.vm.synced_folder "www", "/var/www", :mount_options => [ "dmode=755", "fmode=644" ], :owner => "www-data", :group => "www-data"

	# Set up the salt folder
	config.vm.synced_folder "srv", "/srv", :mount_options => [ "dmode=755", "fmode=644", "uid=1099", "gid=1099" ]

	# Set up the minion
	config.vm.synced_folder "minions", "/etc/salt", :mount_options => [ "dmode=755", "fmode=644" ], :owner => "root", :group => "root"

	config.vm.provision(:shell, :inline => <<-CMD)
		# exit on error
		set -e

		echo "Adding and authorizing deploy user ..."

		if id deploy $1 >/dev/null 2>&1; then
			echo "...deploy user exists"
		else
			echo "...creating deploy user"
			useradd -G sudo -p $(perl -e'print crypt("temp", "temp")') -m -s /bin/bash deploy -u 1099
			usermod -aG sudo deploy
			mkdir -pm 700 /home/deploy/.ssh
		fi

		if [ -e /home/deploy/.ssh/authorized_keys ]; then
			echo ".../home/deploy/.ssh/authorized_keys exists"
		else
			echo "...creating and populating /home/deploy/.ssh/authorized_keys"
			curl -L 'https://github.com/tollmanz.keys' >> /home/deploy/.ssh/authorized_keys
		fi

		# Verify that permissions are correct
		chmod 0600 /home/deploy/.ssh/authorized_keys
		chown -R deploy:deploy /home/deploy/.ssh
	CMD

	# Provision with Salt
	config.vm.provision :salt do |salt|
		salt.verbose = true
		salt.minion_config = 'minions/minion'
		salt.run_highstate = true
	end
end