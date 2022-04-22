# fedora-packaging

A quick documentation how to setup a Fedora Packager env to manage fedora packages.

## Setting Fedora

Before creating a Vagrant Virtual Machine with all tools required for Fedora packaging lets install the software needed in the HOST machine

- 1 Required Packages
```
sudo dnf install @vagrant
sudo dnf install nfs-utils && sudo systemctl enable nfs-server
vagrant plugin install vagrant-ssh
```

- 2 Enable nfs, rpc-bind and mountd services for firewalld
```
sudo firewall-cmd --permanent --zone=libvirt --add-service=nfs3 \
    && sudo firewall-cmd --permanent --zone=libvirt --add-service=nfs \
    && sudo firewall-cmd --permanent --zone=libvirt --add-service=rpc-bind \
    && sudo firewall-cmd --permanent --zone=libvirt --add-service=mountd \
    && sudo firewall-cmd --reload
```


- 3 Using NFS shares from Vagrant without password prompts
```
sudo visudo

# Allow Vagrant to manage /etc/exports
Cmnd_Alias VAGRANT_EXPORTS_ADD = /usr/bin/tee -a /etc/exports
Cmnd_Alias VAGRANT_NFSD_CHECK = /usr/bin/systemctl status --no-pager nfs-server.service
Cmnd_Alias VAGRANT_NFSD_START = /usr/bin/systemctl start nfs-server.service
Cmnd_Alias VAGRANT_NFSD_APPLY = /usr/sbin/exportfs -ar
Cmnd_Alias VAGRANT_EXPORTS_REMOVE = /bin/sed -r -e * d -ibak /*/exports
Cmnd_Alias VAGRANT_EXPORTS_REMOVE_2 = /bin/cp /*/exports /etc/exports
%vagrant ALL=(root) NOPASSWD: VAGRANT_EXPORTS_ADD, VAGRANT_NFSD_CHECK, VAGRANT_NFSD_START, VAGRANT_NFSD_APPLY, VAGRANT_EXPORTS_REMOVE, VAGRANT_EXPORTS_REMOVE_2
```

- 4 Add yourself to the vagrant group:

```
$ sudo getent group vagrant >/dev/null || sudo groupadd -r vagrant
$ sudo gpasswd -a ${USER} vagrant
$ sudo -u ${USER} ${SHELL}
```

## Creating the Virtual Machine
Before creating the `Fedora Packager VM` check the `Global Settings` [here](https://github.com/dougsland/fedora-packaging/blob/37f1119c0af123f6d9ad2ab99d1cd0802acc9e29/Vagrantfile#L1) for setting [Fedora Account (FAS)](https://github.com/dougsland/fedora-packaging/blob/680cab8b6cb7c406c6897f792befe752d5270fd1/Vagrantfile#L4), [private key for FAS account](https://github.com/dougsland/fedora-packaging/blob/680cab8b6cb7c406c6897f792befe752d5270fd1/Vagrantfile#L12), [Git username/email](https://github.com/dougsland/fedora-packaging/blob/680cab8b6cb7c406c6897f792befe752d5270fd1/Vagrantfile#L5), [non root user for the development](https://github.com/dougsland/fedora-packaging/blob/680cab8b6cb7c406c6897f792befe752d5270fd1/Vagrantfile#L8), [password](https://github.com/dougsland/fedora-packaging/blob/680cab8b6cb7c406c6897f792befe752d5270fd1/Vagrantfile#L8), etc.  
If all settings are good, run the below command to create the VM:
```
vagrant up
```

After the VM is up, user can access it via `vagrant ssh fedora-packager` (PASSWORD SET IN VAGRANT FILE) and execute the following commands:
- `fkinit -u FAS_ACCOUNT` to start using fedpkg command  
- `fedpkg clone PACKAGE` to clone a pakcage. Example: `fedpkg clone ovn && cd ovn`
- List all branches: `git branch -a`
- Switch branches: `fedpkg switch-branch f35`

## Resources
https://developer.fedoraproject.org/tools/vagrant/vagrant-libvirt.html
https://developer.fedoraproject.org/tools/vagrant/vagrant-nfs.html
