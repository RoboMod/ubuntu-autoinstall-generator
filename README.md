# Ubuntu Autoinstall Generator
A script to generate a fully-automated ISO image for installing Ubuntu onto a machine without human interaction. This uses the new autoinstall method
for Ubuntu 24.04 and newer.

**Based on work in**
- https://github.com/covertsh/ubuntu-autoinstall-generator
- https://github.com/linuxrrze/ubuntu-24.04-autoinstall-generator

### Behavior
Check out the usage information below for arguments. The basic idea is to take an unmodified Ubuntu ISO image, extract it, add some kernel command line parameters, then repack the data into a new ISO. This is needed for full automation because the ```autoinstall``` parameter must be present on the kernel command line, otherwise the installer will wait for a human to confirm. This script automates the process of creating an ISO with this built-in.

Autoinstall configuration (disk layout, language etc) can be passed along with cloud-init data to the installer. Some minimal information is needed for
the installer to work - see the Ubuntu documentation for an example, which is also in the ```user-data.example``` file in this repository (password: ubuntu). This data can be passed over the network (not yet supported in this script), via an attached volume, or be baked into the ISO itself.

Furthermore, all files in the "extra-files-dir" will be included in the ```nocloud/extra-files``` directory. This can be used to move some files necessary after system installation like config files which can be deployed automatically afterwards.

To attach via a volume (such as a separate ISO image), see the Ubuntu autoinstall [quick start guide](https://ubuntu.com/server/docs/install/autoinstall-quickstart). It's really very easy! To bake everything into a single ISO instead, you can use the ```-a``` flag with this script and provide a user-data file containing the autoinstall configuration and optionally cloud-init data, plus a meta-data file if you choose. The meta-data file is optional and will be empty if it is not specified. With an 'all-in-one' ISO, you simply boot a machine using the ISO and the installer will do the rest. At the end the machine will reboot into the new OS.

This script can use an existing ISO image or download the latest daily image from the Ubuntu project. Using a fresh ISO speeds things up because there won't be as many packages to update during the installation.

By default, the source ISO image is checked for integrity and authenticity using GPG. This can be disabled with ```-k```.

### Requirements
Tested on a host running Ubuntu 20.04.1.
- Utilities required:
    - ```xorriso```
    - ```7z```
    - ```sed```
    - ```curl```
    - ```gpg```

### Usage
```
Usage: $(basename "${BASH_SOURCE[0]}") [-h] [-v] [-a] [-e] [-u user-data-file] [-m meta-data-file] [-k] [-c] [-r] [-s source-iso-file] [-d destination-iso-file]

💁 This script will create fully-automated Ubuntu ${UBUNTU_VERSION} ${UBUNTU_RELNAME} installation media.

Available options:

-h, --help              Print this help and exit
-v, --verbose           Print script debug info
-a, --all-in-one        Bake user-data and meta-data into the generated ISO. By default you will
                        need to boot systems with a CIDATA volume attached containing your
                        autoinstall user-data and meta-data files.
                        For more information see: https://ubuntu.com/server/docs/install/autoinstall-quickstart
-e, --use-hwe-kernel    Force the generated ISO to boot using the hardware enablement (HWE) kernel. Not supported
                        by early Ubuntu 20.04 release ISOs.
-u, --user-data         Path to user-data file. Required if using -a
-m, --meta-data         Path to meta-data file. Will be an empty file if not specified and using -a
-x, --extra-files-dir   Path to directory with additional files to be included
-k, --no-verify         Disable GPG verification of the source ISO file. By default SHA256SUMS-$today and
                        SHA256SUMS-$today.gpg in ${script_dir} will be used to verify the authenticity and integrity
                        of the source ISO file. If they are not present the latest daily SHA256SUMS will be
                        downloaded and saved in ${script_dir}. The Ubuntu signing key will be downloaded and
                        saved in a new keyring in ${script_dir}
-c, --no-md5            Disable MD5 checksum on boot
-r, --use-release-iso   Use the current release ISO instead of the daily ISO. The file will be used if it already
                        exists.
-s, --source            Source ISO file. By default the latest daily ISO for Ubuntu 20.04 will be downloaded
                        and saved as ${script_dir}/ubuntu-original-$today.iso
                        That file will be used by default if it already exists.
-d, --destination       Destination ISO file. By default ${script_dir}/ubuntu-autoinstall-$today.iso will be
                        created, overwriting any existing file.
```

### Example
```
user@testbox:~$ bash ubuntu-autoinstall-generator.sh -a -u user-data.example -d ubuntu-autoinstall-example.iso
[2020-12-23 14:06:07] 👶 Starting up...
[2020-12-23 14:06:07] 📁 Created temporary working directory /tmp/tmp.jrmlEaDhL3
[2020-12-23 14:06:07] 🔎 Checking for required utilities...
[2020-12-23 14:06:07] 👍 All required utilities are installed.
[2020-12-23 14:06:07] 🌎 Downloading current daily ISO image for Ubuntu 20.04 Focal Fossa...
[2020-12-23 14:08:01] 👍 Downloaded and saved to /home/user/ubuntu-original-2020-12-23.iso
[2020-12-23 14:08:01] 🌎 Downloading SHA256SUMS & SHA256SUMS.gpg files...
[2020-12-23 14:08:02] 🌎 Downloading and saving Ubuntu signing key...
[2020-12-23 14:08:02] 👍 Downloaded and saved to /home/user/843938DF228D22F7B3742BC0D94AA3F0EFE21092.keyring
[2020-12-23 14:08:02] 🔐 Verifying /home/user/ubuntu-original-2020-12-23.iso integrity and authenticity...
[2020-12-23 14:08:09] 👍 Verification succeeded.
[2020-12-23 14:08:09] 🔧 Extracting ISO image...
[2020-12-23 14:08:11] 👍 Extracted to /tmp/tmp.jrmlEaDhL3
[2020-12-23 14:08:11] 🧩 Adding autoinstall parameter to kernel command line...
[2020-12-23 14:08:11] 👍 Added parameter to UEFI and BIOS kernel command lines.
[2020-12-23 14:08:11] 🧩 Adding user-data and meta-data files...
[2020-12-23 14:08:11] 👍 Added data and configured kernel command line.
[2020-12-23 14:08:11] 👷 Updating /tmp/tmp.jrmlEaDhL3/md5sum.txt with hashes of modified files...
[2020-12-23 14:08:11] 👍 Updated hashes.
[2020-12-23 14:08:11] 📦 Repackaging extracted files into an ISO image...
[2020-12-23 14:08:14] 👍 Repackaged into /home/user/ubuntu-autoinstall-example.iso
[2020-12-23 14:08:14] ✅ Completed.
[2020-12-23 14:08:14] 🚽 Deleted temporary working directory /tmp/tmp.jrmlEaDhL3
```

Now you can boot your target machine using ```ubuntu-autoinstall-example.iso``` and it will automatically install Ubuntu using the configuration from ```user-data.example```.


### License
MIT license.
