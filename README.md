# automaton

The automaton provides automatic provisionning of software prerequisites required for running other Streampunk software and related applications. This is a prototype [ansible](https://www.ansible.com/) configuration that can be used to remotely deploy software dependencies on Windows and Linux systems. 

A [vagrant](https://www.vagrantup.com/) box configuration file is included that creates a virtual machine that runs an ansible controller, allowing a Windows host to be used as an ansible controller.

This is a prototype and example configuration only. Some of the details are specific to a PoC activity but are provided as a guide to the kind of ansible plays you might want to configure in your own environment. It will be necessary to change these files to be able to used this configuration. In particular, files containing passwords have been encypted meaning this configuration will not work directly out-of-the-box.

The notes provided here are in addition to the [ansible documentation](http://docs.ansible.com/ansible/index.html) and should be read alongside that. Below you will find what amounts to a quick start description that is specific to the way that Streampunk systems are being deployed in small scale environments.

## Basics

Ansible allows you to declare the state that you want your systems to be in and _make it so_. To do this, you:

1. Nominate one system in your infrastructure to be a _controller_.
2. On all other systems, setup remote access to enable script-based provissioning:
  * For linux-based systems, enable ssh access.
  * For windows systems, configure winrm access to powershell.


