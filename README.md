# Homelab Automation with Autoinstall and Ansible.

The following sections detail my WIP setup for automating the installation and configuration of my homelab servers. The goal is to have a fully automated setup that can be used to quickly provision new servers and virtual machines as well as other applications.

## Ubuntu Server Autoinstall Setup

### Prerequisites

- A machine to install Ubuntu Server on
- A USB drive to create a bootable Ubuntu Server installer
- Ubuntu Server ISO image

### Steps

#### 1. Create Bootable USB

On a Linux machine, use `dd` to create a bootable USB drive with the Ubuntu Server ISO. Replace `/path/to/ubuntu.iso` with the path to your ISO file and `/dev/sdX` with your USB drive.

```bash
sudo dd if=/path/to/ubuntu.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

#### 2. Generate Encrypted Password

Generate an encrypted password to use in the autoinstall file.

#### 3. Set Environment Variables

Set environment variables for the hostname, realname, username, and encrypted password. Replace the placeholders with your actual details.

```bash
export HOSTNAME="<your_hostname>"
export REALNAME="<your_realname>"
export USERNAME="<your_username>"
export AUTOINSTALL_PASS="<your_encrypted_password>"
export PUB_SSH_KEY="<your_public_ssh_key>"
```

#### 4. Substitute Environment Variables

Substitute the environment variables into the autoinstall configuration file. This assumes the configuration file is named `user-data.template` and located in the current directory.

```bash
envsubst < user-data.template > user-data
```

#### 5. Move `user-data` to USB Drive

Move the `user-data` file to the root of the bootable USB drive. The autoinstall process will look for this file during installation.

#### 6. Install Ubuntu Server

Insert the USB drive into your server machine and boot from it. Follow the prompts to start the installation.

Please be careful with your `user-data` file as it contains your sensitive information. Delete it or move it to a secure location after you're done with the installation. Also, be sure to double-check the `dd` command and make sure you're writing to the correct device as it will overwrite all data on the target device.


## Ubuntu Server Dotfiles and VM Creation via Ansible

This repo contains a collection of Ansible playbooks for automating system 
configuration and management tasks on ubuntu server as well as adding virtual machines.
It is not yet fully automated.

## Prerequisites

Before running the playbooks, ensure you have met the following requirements:

- Install Ansible: If Ansible is not installed on your client machine, use the appropriate package manager to install it. For example, on macOS, you can use Homebrew: `brew install ansible`.
- Verify virtualization support: Run the `kvm-ok` command on the target system to check if virtualization support is enabled. Refer to the [Ubuntu Virtualization documentation](https://ubuntu.com/server/docs/virtualization-libvirt) for more details.
  - If no, enable virtualization in BIOS: Access the system BIOS settings and ensure that virtualization is enabled. The exact location of the option may vary depending on your motherboard. For example, on the Designaire x399, it can typically be found under "MIT frequency setting," "Advanced Core," or "SVM mode."
 
### Usage

To run the available playbooks, follow the instructions below:

**Note:** You'll need to create an `inventory.ini` file if not done already.

1. **host-setup.yaml**: This playbook sets up minimal workflow on ubuntu server.

Run the playbook using the following command:
```
ansible-playbook -i <inventory-file> ansible/host-setup.yaml -e "target_hosts=homelab,create_env=genpur"
```
2. **vm-dependencies.yaml**: Sets up the dependencies needed for virtual machines.

Run the playbook using the following command:
```
ansible-playbook -i <inventory-file> vm-dependencies.yaml
```

**Note:** Before you move on you have to set up a couple items. 

- Set the multipass password on your host machine with `multipass set local.passphrase`.
- Generate an ssh key at the location `~/.ssh/multipass-ssh-key`.

3. **create-vm.yaml**: This playbook creates a virtual machine on ubuntu server with the same configuration.

Run the playbook using the following command:
```
ansible-playbook -i <inventory-file> ansible/create_vm.yaml -e "vm_name=myvm vm_cpus=2 vm_mem=10G vm_disk=10G vm_os=22.04"
```

4. From there you can either run the included `ansible/vm-setup.yaml` to run the set up from the client, or you can run the `setup-host` from the host and specify the vm name.

- From the client:
```
ansible-playbook -i inventory.ini ansible/vm-setup.yaml -e "vm_name=myvmff"
```

- From the host:
```
ansible-playbook -vv ~/vm_configs/host-setup.yaml -i ~/vm_configs/vm-inventory.ini -e "target_hosts=myvmf"
```


Files that set up the dotfiles/python environments.
1. **dotfiles.yaml**: This playbook sets up various development tools and configurations related to dotfiles. It installs Python, Node.js, FZF, Vim plugins, and more.
2. **python-env.yaml**: This playbook creates a Python virtual environment and installs necessary packages. The `env` variable can be customized to specify the environment name.


Please ensure that you modify the playbooks to suit your specific requirements and verify that you have the necessary privileges to execute the tasks on the target systems.

Remember to replace the `<inventory-file>` placeholder with the path to your Ansible inventory file.

6. **gpu-dependencies.yaml**: The playbook installs the necessary drivers for my 3070 and reboots.
7. **host-dependencies.yaml**: Installing docker and container with nvidia support.
 
**Note**: These were not fully tested, but the source is here:

See [this article](https://linuxconfig.org/how-to-install-the-nvidia-drivers-on-ubuntu-22-04) for more info.
[This article](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#setting-up-docker) as well.

## Additional Playbooks (TODO):

1. - **ssh.yaml**: Playbook to change ports.
2. - **mount-drives.yaml**: Playbook to mount drives.
3. - **mount-encrypted-drive.yaml**: Playbook to mount and unmount encrypted drives.
4. - **firewall.yaml**: Playbook to set up a firewall.
5. - Setup vincuna LLM to serve.
6. - Setup task server.
7. - Zoneminder??
8. - Nextcloud

