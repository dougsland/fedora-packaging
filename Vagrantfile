######################## GLOBAL VARIABLES ###########
VAGRANT_COMMAND = ARGV[0]

FAS_ACCOUNT = "dougsland"
PACKAGER_NAME = "Douglas Schilling Landgraf"
PACKAGER_EMAIL = "dougsland@redhat.com"

PACKAGER_USERNAME = "devel"
PACKAGER_PASSWORD = "superpass"
PACKAGER_PACKAGES = "fedora-packager fedora-review vim ctags git krb5-workstation"

PACKAGER_FEDORA_ORIG_PRIV_KEY = "/home/douglas/.ssh/id_rsa_fedora"
PACKAGER_FEDORA_DEST_PRIV_KEY = "/home/" + PACKAGER_USERNAME + "/.ssh/id_rsa"

VM_NAME = "fedora-packager"
VM_HOSTNAME = "fedora-packager"
VM_IMAGE = "fedora/35-cloud-base"

VM_CPUS = 4
VM_CPU_MODE = "host-passthrough"
VM_MEMORY = 25048
VM_NESTED = true
VM_KEYMAP = "pt"

VM_SYNC_FOLDER = "/vagrant"

CMD_GIT_CONFIG_GLOBAL_USERNAME = "sudo -u " + PACKAGER_USERNAME + " git config --global user.name \"" + PACKAGER_NAME + "\""
CMD_GIT_CONFIG_GLOBAL_USEREMAIL = "sudo -u " + PACKAGER_USERNAME + " git config --global user.email \"" + "" + PACKAGER_EMAIL + "\""
######################## GLOBAL VARIABLES ###########

######################## START ######################
Vagrant.configure(2) do |config|
  config.vm.box = VM_IMAGE

  config.ssh.username = PACKAGER_USERNAME
  if VAGRANT_COMMAND == "provision" || VAGRANT_COMMAND == "up"
    config.ssh.username = "root"
  end
  config.vm.provider 'libvirt' do |lv, config|
    lv.memory = VM_MEMORY
    lv.cpus = VM_CPUS
    lv.cpu_mode = VM_CPU_MODE
    lv.nested = VM_NESTED
    lv.keymap = VM_KEYMAP
    config.vm.synced_folder '.', VM_SYNC_FOLDER, type: 'nfs', nfs_udp: false,
				 :linux__nfs_options => ['rw','no_subtree_check','no_root_squash']
  end

  # Set hostname and vm name
  config.vm.define VM_NAME
  config.vm.hostname = VM_HOSTNAME

  # update the system 
  #config.vm.provision 'shell',
  #		inline: 'sudo dnf update -y'

  # Adding user devel
  config.vm.provision 'shell',
		inline: 'useradd ' + PACKAGER_USERNAME 

  config.vm.provision 'shell',
		inline: 'echo ' + PACKAGER_PASSWORD + ' | passwd ' + PACKAGER_USERNAME + ' --stdin'

  # Installing Fedora packager tools
  config.vm.provision 'shell',
		inline: 'sudo dnf install -y ' + PACKAGER_PACKAGES

  # fedora packager priv key
  config.vm.provision "file",
	source: PACKAGER_FEDORA_ORIG_PRIV_KEY,
	destination: PACKAGER_FEDORA_DEST_PRIV_KEY

  # setting permission
  config.vm.provision 'shell', inline: 'chown -R ' + PACKAGER_USERNAME + ':' + PACKAGER_USERNAME + ' /home/' + PACKAGER_USERNAME + '/.ssh/' 

  # Setting git configs
  config.vm.provision 'shell', inline: CMD_GIT_CONFIG_GLOBAL_USERNAME
  config.vm.provision 'shell', inline: CMD_GIT_CONFIG_GLOBAL_USEREMAIL

  config.vm.provision 'shell', inline: 'echo ' + '==============================================================='
  config.vm.provision 'shell', inline: 'echo ' + 'ALL SET!'
  config.vm.provision 'shell', inline: 'echo ' + '- To login in the VM use: vagrant ssh ' + VM_NAME
  config.vm.provision 'shell', inline: 'echo ' + '- Execute: fkinit -u ' + FAS_ACCOUNT + ' to start using fedpkg command'
  config.vm.provision 'shell', inline: 'echo ' + '- fedpkg clone PACKAGE_NAME for cloning a package'
  config.vm.provision 'shell', inline: 'echo ' + '==============================================================='
end
