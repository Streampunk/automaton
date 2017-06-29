# automaton

The automaton provides automatic provisionning of software prerequisites required for running other Streampunk software and related applications. This is a prototype [ansible](https://www.ansible.com/) configuration that can be used to remotely deploy software dependencies on Windows and Linux systems. 

A [vagrant](https://www.vagrantup.com/) box configuration file is included that creates a virtual machine that runs an ansible controller, allowing a Windows host to be used as an ansible controller.

This is a prototype and example configuration only. Some of the details are specific to a PoC activity but are provided as a guide to the kind of ansible plays you might want to configure in your own environment. It will be necessary to change these files to be able to used this configuration. In particular, files containing passwords have been encypted meaning this configuration will not work directly out-of-the-box.

The notes provided here are in addition to the [ansible documentation](http://docs.ansible.com/ansible/index.html) and should be read alongside that. Below you will find what amounts to a quick start description that is specific to the way that Streampunk systems are being deployed in small scale environments.

## Basics

Ansible allows you to declare the state that you want your systems to be in and _make it so_. To do this, you:

1. Nominate __one__ system in your infrastructure to be a _controller_.
2. On all other systems, setup remote access to enable script-based provissioning:
  * For linux-based systems, enable `ssh` access.
  * For windows systems, configure [winrm](https://msdn.microsoft.com/en-us/library/aa384426(v=vs.85).aspx) access to `powershell`.
  
Ansible works by making _plays_, where a play provisions a nominated group of machines (_hosts_) with the software you want to install, enables the OS features, sets up users, starts web services, sets up firewall rules, copies on files etc. etc.. A _play_ is described in a _playbook_, a [YAML](https://en.wikipedia.org/wiki/YAML) file that describes a set of _tasks_ that need to run to complete the play. 

In its simplest form, a task is just a command to run on the remote machine. The real power comes with the fact that what is actually happenning with each task is that ansible is writing a python script, copying it over to the remote machine and executing it. This script gathers _facts_ about the machine that can be used as variables to alter the way the script runs, such as installing 32-bit or 64-bit versions of a package as appropriate to the architecture. The programme also registers the result of running the task. A set of pre-configured modules provide higher level functions, such as only installing software that is not already present or only copying on files that need to be copied by computing checksums.
  
## Controller

### Installation on Windows

To run a controller on Windows, you need to install and configure a virtual machine. This can be achieved with a combination of vagrant and either Hyper-V or Oracle's VirtualBox. The following instructions assume the use of VirtualBox.

* Install vagrant on your system using [their download](https://www.vagrantup.com/downloads.html).
* Install VirtualBox on your systems using [their download](https://www.virtualbox.org/wiki/Downloads). It may be a good idea to update your version of virtualbox.
* Extract this github project to a folder on your computer. You may need to install git or )[download a zip version](https://github.com/Streampunk/automaton/archive/master.zip), for example:
```
cd Documents
git clone https://github.com/Streampunk/automaton.git
cd automaton
```

* Set up, provision and run the controller VM with:
```
vagrant up --provider virtualbox
```
  This does a lot of things, including downloading a base VM if one is not available, configuring network interfaces, installing ansible 
  and other dependencies via Ubuntu package management inside the VM and setting up ssh certificates.
  
* Login to the virtual machine and change to the vagrant files folder
```
vagrant ssh
cd /vagrant
```

* Check that ansible is installed and available with ```ansible --version```.

### Installation on Linux

Follow the [ansible instructions](http://docs.ansible.com/ansible/intro_installation.html) for your platform. 

To add support for controlling Windows hosts:

```
pip install "pywinrm>=0.1.1"
```

### Installation on Mac

Follow the [Mac-specific instructions from ansible](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-on-mac-osx) or the [guide from Vladhaus](https://valdhaus.co/writings/ansible-mac-osx/). As a homebrew user, I find the following to work well:

```
brew install ansible
```

To add support for controlling Windows hosts:

```
pip install "pywinrm>=0.1.1"
```

## Windows hosts

For each host, the machines you want to control including the remote installation and management of consoles, must do two things and a test:

1. Add the user that ansible connections are to be made through to the _Administrators_ group. Use the _User Accounts_ tool from the Control Panel to do this and click on _Change Account Type_, selecting Administrator.
2. Set up the [Windows Remote Management](https://msdn.microsoft.com/en-us/library/aa384291(v=vs.85).aspx) system with certificates that enable external remote control from ansible.
3. Test the connection from the controller to the Windows machine.

### Set up Windows Remote Management

Powershell needs to be configured to allow remote access. This can be achieved using [this script](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1), which is included with this distribution as a [file](./windows_config/ConfigureRemotingForAnsible.ps1). 

To run this file, the simplest way is:

1. Run Powershell as admininstrator. One way to do this is type "Powershell" in Cortana, right click "Windows Powershell" and choose "Run as administrator". 
2. Copy the contents of https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 with Ctrl-C.
3. Paste the result into the Powershell window with Ctrl-V.

An alternative approach is:

1. Run Powershell as admininstrator or a command prompt with Administrator privileges. One way to do this is type "Powershell" in Cortana, right click "Windows Powershell" and choose "Run as administrator". 
2. Copy the [`./windows_config/ConfigureRemotingForAnsible.ps1`](./windows_config/ConfigureRemotingForAnsible.ps1) file into the local folder.
3. Run `powershell.exe -File ConfigureRemotingForAnsible.ps1 -Verbose`. Note that you may need to configure powershell execution policy. 

For further details, see the [Ansible Windows System Prep](http://docs.ansible.com/ansible/intro_windows.html#windows-system-prep) instructions.

### Test the controller to host

To test the connection from the controller to the host, edit the `ping_local` and `ping_admin` entries in the hosts file so that the `ansible_host=xxxx` as `xxxx` replaced with the host name or its IP address. Save the file and run ...

```
ansible ping_admin -k -m win_ping
```

When prompted for an SSH password, use the Administartor password for the host. A successful output looks like this:

```
ping_admin | SUCCESS => {
    "changed" : false,
    "ping": "pong"
}
```

## Running playbooks

### Inventory

The `hosts` file provides an inventory for the systems in a PoC lab. Computers to be automatically configured are listed in the `windows`, `rpi` or `linux` group by their operating system. Common connection parameters are found in the `group_vars` by operating systems. Each host in the inventory also appears in a classification that determines the business purpose of the device. These include:

* _servers_ - Backend devices, typically rack mounted in a server room, intended for intensive compute of storage applications.
* _edges_ - Computers used as edge devices, used to connect to input and output audio and video devices.
* _devs_ - Computers used for development of code, software testing and hands-on triage.
* _clock_ - Computer used as a time server, such as a raspberry pi with a GPS hat.

All of the above roles have apart from _clock_ share a _common_ core of tools, as described by the common role in the next section.

The inventory has been written on the assumption that all the machines to be configured have been pre-assigned a name via DNS and that each system is up and running.

### Roles

Roles equivalent to the machine classifications have been defined, with role dependency as follows:

* _common_ - MS build tool, Node.JS, enable ping, python 2.7

### Playbooks



### Roles


