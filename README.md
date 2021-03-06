# Retail Windows 10 Templates for Packer

### Introduction

This repository contains Windows templates that can be used to create boxes for
Vagrant using Packer ([Website](https://www.packer.io))
([Github](https://github.com/mitchellh/packer)).

This repo is a modified fork of the popular
[StefanScherer/packer-windows](https://github.com/stefanscherer/packer-windows)
repo.

Some of my changes are:

* Focus only on Windows 10 
* Changes to support use of _retail_ Windows 10 iso

TODO Need like a quick-start / how-to-make-it-go section...

### Packer Version

[Packer](https://github.com/mitchellh/packer/blob/master/CHANGELOG.md) `1.1.3` is recommended.

### Windows Versions

TODO Provide equivalent copy...

### Windows Editions

TODO Provide equivalent copy

### Product Keys

TODO Provide updated copy

### Windows Updates

The scripts in this repo will install all Windows updates – by default – during
Windows Setup. This is a _very_ time consuming process, depending on the age of
the OS and the quantity of updates released since the last service pack. You
might want to do yourself a favor during development and disable this
functionality, by commenting out the `WITH WINDOWS UPDATES` section and
uncommenting the `WITHOUT WINDOWS UPDATES` section in `Autounattend.xml`:

```xml
<!-- WITHOUT WINDOWS UPDATES -->
<SynchronousCommand wcm:action="add">
    <CommandLine>cmd.exe /c C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File a:\openssh.ps1 -AutoStart</CommandLine>
    <Description>Install OpenSSH</Description>
    <Order>99</Order>
    <RequiresUserInput>true</RequiresUserInput>
</SynchronousCommand>
<!-- END WITHOUT WINDOWS UPDATES -->
<!-- WITH WINDOWS UPDATES -->
<!--
<SynchronousCommand wcm:action="add">
    <CommandLine>cmd.exe /c a:\microsoft-updates.bat</CommandLine>
    <Order>98</Order>
    <Description>Enable Microsoft Updates</Description>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <CommandLine>cmd.exe /c C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File a:\openssh.ps1</CommandLine>
    <Description>Install OpenSSH</Description>
    <Order>99</Order>
    <RequiresUserInput>true</RequiresUserInput>
</SynchronousCommand>
<SynchronousCommand wcm:action="add">
    <CommandLine>cmd.exe /c C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File a:\win-updates.ps1</CommandLine>
    <Description>Install Windows Updates</Description>
    <Order>100</Order>
    <RequiresUserInput>true</RequiresUserInput>
</SynchronousCommand>
-->
<!-- END WITH WINDOWS UPDATES -->
```

Doing so will give you hours back in your day, which is a good thing.

### WinRM

These boxes use WinRM. There is no OpenSSH installed.

### Hyper-V Support

If you are running Windows 10, Windows Server 2016 or later, then you can also use these packerfiles to build
a Hyper-V virtual machine. I have the ISO already downloaded to save time, and
only have Hyper-V installed on my laptop, so I run:

```
packer build --only hyperv-iso -var 'hyperv_switchname=Ethernet' -var 'iso_url=./server2016.iso' .\windows_2016_docker.json
```

You then can use this box with Vagrant to spin up a Hyper-V VM.

#### Generation 2 VMs

Some of these images use Hyper-V "Generation 2" VMs to enable the latest features and faster booting. However, an extra manual step is needed to put the needed files into ISOs because Gen2 VMs don't support virtual floppy disks.

* `windows_server_insider.json`
* `windows_server_insider_docker.json`
* `windows_10_insider.json`

Trial versions of Windows 2008 R2 / 2012 / 2012 R2 / 2016 are used by default. These images can be used for 180 days without activation.

Alternatively – if you have access to [MSDN](http://msdn.microsoft.com) or [TechNet](http://technet.microsoft.com/) – you can download retail or volume license ISO images and place them in the `iso` directory. If you do, you should supply appropriate values for `iso_url` (e.g. `./iso/<path to your iso>.iso`) and `iso_checksum` (e.g. `<the md5 of your iso>`) to the Packer command. For example, to use the Windows 2008 R2 (With SP1) retail ISO:

1. Download the Windows Server 2008 R2 with Service Pack 1 (x64) - DVD (English) ISO (`en_windows_server_2008_r2_with_sp1_x64_dvd_617601.iso`)
2. Verify that `en_windows_server_2008_r2_with_sp1_x64_dvd_617601.iso` has an MD5 hash of `8dcde01d0da526100869e2457aafb7ca` (Microsoft lists a SHA1 hash of `d3fd7bf85ee1d5bdd72de5b2c69a7b470733cd0a`, which is equivalent)
3. Clone this repo to a local directory
4. Move `en_windows_server_2008_r2_with_sp1_x64_dvd_617601.iso` to the `iso` directory
5. Run:
    
    ```
    packer build \
        -var iso_url=./iso/en_windows_server_2008_r2_with_sp1_x64_dvd_617601.iso \
        -var iso_checksum=8dcde01d0da526100869e2457aafb7ca windows_2008_r2.json
    ```

### Variables

The Packer templates support the following variables:

| Name                | Description                                                      |
| --------------------|------------------------------------------------------------------|
| `iso_url`           | Path or URL to ISO file                                          |
| `iso_checksum`      | Checksum (see also `iso_checksum_type`) of the ISO file          |
| `iso_checksum_type` | The checksum algorithm to use (out of those supported by Packer) |
| `autounattend`      | Path to the Autounattend.xml file                                |

* Secondary Dvd image does not exist: CreateFile ./iso/windows_server_insider_unattend.iso: The system cannot find the file specified.
```

### Using .box Files With Vagrant

The generated box files include a Vagrantfile template that is suitable for use
with Vagrant 1.7.4+, but the latest version is always recommended.

Example Steps for Hyper-V:

```
vagrant box add windows_2016_docker windows_2016_docker_hyperv.box
vagrant init windows_2016_docker
vagrant up --provider hyperv
```
