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
  
Ansible works by making _plays_, where a play provisions a nominated group of machines (_hosts_) with the software you want to install, enables the OS features, sets up users, starts web services, sets up firewall rules, copies on files etc. etc.. A _play_ is described in a _playbook_, a YAML file that describes a set of _tasks_ that need to run to complete the play. 

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
  
* Login to the virtual machine




