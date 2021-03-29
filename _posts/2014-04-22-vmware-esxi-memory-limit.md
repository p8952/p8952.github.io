---
layout: "post"
title: "VMware ESXi MEMORY_SIZE_ERROR"
date: "2014-04-22 00:00:00"
---

VMware's ESXi 5.5 increases the recommend memory requirement from 4GB to 8GB,
their own System Requirements document stating that:

"You have 4GB RAM. This is the minimum required to install ESXi 5.5. Provide at
least 8GB of RAM to take full advantage of ESXi features and run virtual
machines in typical production environments."

However when installing ESXi on a system with 4GB of RAM you will receive an
error along the lines of:

```
<MEMORY_SIZE ERROR: This host has 3.71 GiB of RAM. 3.97 GiB are needed>
```

You'll notice that the people writing the System Requirements document are using
the SI unit of gigabyte (GB) while those writing the ESXi installer are using
the binary unit of gibibyte (GiB). As such ESXi does not require 4,000,000,000
bytes of RAM but 4,294,967,296.

Luckily the fix is easy, we can modify the amount minimum amount of RAM the
installer checks for and it will install without issue.

Switch to the virtual terminal by hitting Alt+F1 and login as 'root' with the
password field blank.

After logging in you need to tweak the permissions on upgrade_precheck.py:

```
$ cd /usr/lib/vmware/weasel/util/
$ rm upgrade_precheck.pyc
$ cp upgrade_precheck.py upgrade_precheck.py.tmp
$ cp upgrade_precheck.py.tmp upgrade_precheck.py
$ chmod 777 upgrade_precheck.py
```

Open up upgrade_precheck.py in vi and replace:

```
MEM_MIN_SIZE = (4 * 1024 - 32) * SIZE_MiB
```

With:

```
MEM_MIN_SIZE = (2 * 1024 - 32) * SIZE_MiB
```

Then restart the ESXi installer by killing the weasel process.

```
$ ps -c | grep weasel
$ kill 12345
```

You will automatically get switched away from the virtual terminal and can
continue the installation.
