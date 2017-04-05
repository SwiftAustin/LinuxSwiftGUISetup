# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_BASE =  'ubuntu/yakkety64'.freeze

SWIFT_PATH = 'https://swift.org/builds/swift-3.1-release/ubuntu1610/swift-3.1-RELEASE'.freeze
SWIFT_DIRECTORY = 'swift-3.1-RELEASE-ubuntu16.10'.freeze
SWIFT_FILE = "#{SWIFT_DIRECTORY}.tar.gz".freeze
SWIFT_HOME = "/home/vagrant/#{SWIFT_DIRECTORY}".freeze

Vagrant.configure(2) do |config|
  config.vm.define :ubuntuyak do |ubuntuyak|
    ubuntuyak.vm.box = BOX_BASE
    ubuntuyak.vm.hostname = "ubuntuyak"
    ubuntuyak.vm.network 'forwarded_port', guest: 8090, host: 8090

  	ubuntuyak.vm.provider "virtualbox" do |v|
  	  v.memory = 1024
  	  v.cpus = 2
  	end

    # Prevents "stdin: is not a tty" on Ubuntu (https://github.com/mitchellh/vagrant/issues/1673)
    ubuntuyak.vm.provision 'fix-no-tty', type: 'shell' do |s|
      s.privileged = false
      s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
    end

    ubuntuyak.vm.provision 'shell', privileged: false, inline: <<-SHELL1

  ### Install packages
  # 0. Update latest package lists
  	sudo apt-get --assume-yes update
  # 1. Install compiler, autotools
      sudo apt-get --assume-yes install clang
      sudo apt-get --assume-yes install autoconf libtool pkg-config libpython2.7
  # 2. Install dtrace (to generate provider.h)
      sudo apt-get --assume-yes install systemtap-sdt-dev
  # 3. Install libdispatch pre-reqs
      sudo apt-get --assume-yes install libblocksruntime-dev libkqueue-dev libpthread-workqueue-dev libbsd-dev
  # 4. Kitura packages
      sudo apt-get --assume-yes install libhttp-parser-dev libcurl4-openssl-dev libhiredis-dev
  # 5. Perfect.org packages
    sudo apt-get --assume-yes install openssl libssl-dev uuid-dev
    sudo apt-get --assume-yes install expect
  # 8. Docker - instructions from https://docs.docker.com/engine/installation/linux/ubuntulinux/
      sudo apt-get --assume-yes install apt-transport-https ca-certificates
      sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
      sudo sh -c "umask 022; echo 'deb https://apt.dockerproject.org/repo ubuntu-wily main' > /etc/apt/sources.list.d/docker.list"
      sudo apt-get --assume-yes update
      sudo apt-get --assume-yes install linux-image-extra-$(uname -r) linux-image-extra-virtual
      sudo apt-get --assume-yes install docker-engine
      sudo groupadd docker
      sudo usermod -aG docker ubuntu
      sudo service docker start
      
      curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
      sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
      # Install repo
      sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
      # Update apt-get
      sudo apt-get --assume-yes update
      # Install
      sudo apt-get --assume-yes install code
      mkdir -p .config/Code/User
      curl -q -s -o .config/Code/User/settings.json https://raw.githubusercontent.com/SwiftAustin/LinuxSwiftGUISetup/master/.config/Code/User/settings.json      
  ### Download Swift binary if not found, install it, and add it to the path
      if [ ! -f "#{SWIFT_FILE}" ]; then
          curl -q -s -O "#{SWIFT_PATH}/#{SWIFT_FILE}"
      fi
      sudo tar -xzf #{SWIFT_FILE} --directory / --strip-components=1
      sudo find /usr/lib/swift -type d -print0 | sudo xargs -0 chmod a+rx
      sudo find /usr/lib/swift -type f -print0 | sudo xargs -0 chmod a+r
      
  ### Add Libraries for XWindows and profiling
      sudo apt-get --assume-yes install libgtk2.0 libgconf-2-4 libasound2 valgrind kcachegrind valkyrie
      
  ### Setup for VSCode Swift Integration
    git clone https://github.com/felix91gr/swift-linuxSetup.git
    git clone https://github.com/jinmingjian/sde-demos.git
      
  ### Export LD_LIBRARY_PATH
      if [ $(grep -c "LD_LIBRARY_PATH=/usr/local/lib" .profile) -eq 0 ]; then
          echo "export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH" >> .profile
          source .profile
      fi
      
    SHELL1
      
    ubuntuyak.vm.provision :reload

    ubuntuyak.vm.provision 'shell', privileged: false, inline: <<-SHELL2
      
      sudo docker pull jinmingjian/docker-sourcekite
      
    SHELL2
  end
  
  config.ssh.forward_x11 = true
end
