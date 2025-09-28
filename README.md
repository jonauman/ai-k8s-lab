# ai-k8s-lab

Codebase to bootstrap a kubernetes lab running an AI workload on commodity hardware.

This repository:
- installs CoreOS with GPU libraries (ucore) via ignition configuration
- installs Kubernetes with GPU operator via ansible

# Usage

## CoreOs

To install coreos:

1. Create a bootable coreOs USB drive from the [Bare Metal : Fedora CoreOS : LiveDVD : iso](https://fedoraproject.org/coreos/download?stream=stable). There are numerous instructions (google) on how to create a bootable USB drive, but I used [balenaEtcher](https://etcher.balena.io/) on my MacOS laptop.

1. update the coreos/config.ign with your own password hash and public ssh key. Password hash should be in sha256 format and can be generated with the mkpasswd command on linux. Alternatively, you can use the current password hash for the coreadmin user (password is Ch@ng3m3!

1. Copy the coreos/config.ign to a webserver which your lab machine has access. For ease of use, you can paste the updated config to a public github gist. Make sure to use the "raw" url of the gist as the install parameter. You can use tinyurl (or similar) to shorten the URL since the URL will need to be typed into the coreos-installer command.

1. Place the USB drive in the lab machine and change the boot order for this one time. Usually hitting F12 key while the machine is booting will work. Select the USB drive.

1. Once booted, type in the following command:

    `coreos-installer install /dev/nvme0n1 --ignition-file -u [link to raw config.ign url]`

    Use the storage device on which you want to install CoreOS if not the first nvme drive.

1. With any luck, the machine will reboot several times while it patches the OS with the ucore OCI image. Ucore is advertised as a CoreOS image with "batteries included", providing broader support for additional hardware like GPUs and Wifi cards.

### Kubernetes

The goal here is to install kubernetes with GPU support so we can run AI workloads against it. Since CoreOS is mostly a read-only operation system, it requires a reboot to make changes such as package installs and system configuration changes. [Ansible](https://docs.ansible.com/) is a good tool for automating multiple changes to a system and continuing after a reboot, so that's what we'll use.

1. Install ansible on an admin machine (e.g., laptop) that can ssh into the lab machine. The admin key should load the private key paired with the ssh key provided in config.ign 

    `ssh-add ~/.ssh/id_rsa(or similar)`

1. Ensure you can ssh into the lab machine
    `ssh coreadmin@lab-ip-address`

1. Edit the k8s/inventory.ini file and replace the existing IP with the IP address of your lab machine

1. Check that ansible can connect to the lab machine
    `ansible lab -m setup -i inventory.ini`

1. If all good there, continue to run the playbook
    `ansible-playbook -i inventory.ini playbook.yml`


