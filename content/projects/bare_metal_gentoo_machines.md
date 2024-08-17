+++
title = 'My Baremetal Gentoo Machines'
date = 2024-07-28T16:54:19+02:00
toc = true
+++

## Why?
1. Create an actual, on-prem, Kubernetes cluster service.
2. Improve my linux administration skills by provisioning physical machines.
    * My experience is limited to WSL, and my personal PC.
3. Extend my current Gentoo usage from personal, to a production environment.
    * I believe that Gentoo is very flexible for both use-cases.
4. Practice my scripting/programming skills by automating the process. 
    * Documentation as a side-effect.

## Machines used
1. 4x Minisform UN100D:
    * Intel N100 (4 core) low power amd64 pc.
2. 6x Raspberry Pi 5:
    * arm64 SBC.

## Journey
The plan sounds simple right? Provision my own, on-prem, baremetal Kubernetes cluster.
But as with any personal projects, many of the implementation details are not considered
when starting. Here are a few of the problems.

1. Different bootloaders:
    * UEFI compliance, two amd64 machines look at different directories when finding the boot files.
    * Raspberry Pi5 is not UEFI compliant and has a different boot behavior.
2. Bootstrapping the live-cd environment
    * Doing this was easy for amd64, since my personal machine is amd64.
    * For the Raspberry Pis, I had to bootstrap the first one using [this guide](https://wiki.gentoo.org/wiki/Raspberry_Pi_Install_Guide).
      Then used that to provision all micro-sd cards for the other machines.
2. Solving Gentoo's compilation-from-source requirement:
    * Gentoo is a very flexible OS. But as a downside, users will need to compile from source.
    Using bin hosts partially solves this, not every package has a bin host.
    * My system configuration is also basic for now (no custom flags, etc). So using bin host is
    okay.
    * I consider this a workaround while I figure out how to create my own distcc cross-compiling
    environment.

Since I use it daily at work, I use a mixture of Python and Bash scripts to implement and document
the process. [This Github Repo](https://github.com/martopad/server-setup).

## Findings

1. Python on the Gentoo's live-cd is up-to-date!
2. Using Python to call bash scripts is a very flexible approach.
    * I used it also to create a customizable environment (passing --conf-file)
    since subprocess takes env parameter.
    * [Example of config file](https://github.com/martopad/server-setup/blob/main/src/arm64/rpi/rpi5/config.ini)

3. Extending Python's builtin json, and toml parsers is surprisingly easy!
    * By using `configparser.ExtendedInterpolation` 
    * I used it to inject dynamic data in my configuration file and strip inline comments.

    Example:
``` toml
[INJECTED]
BASE_DIR=${INJECTED_BASE_DIR} # <-- Value is dynamically injected
                              #     during runtime. In addition, by
                              #     default, these comments will be included as
                              #     the key's value.
[ARGPY]
ARGPY_BASE_DIR=${INJECTED:BASE_DIR}
``` 
4. There is a weird behavior when calling the shell command `arch-chroot` directly via subprocess.run.
I don't know why it happens. But to solve it, I created a bash script that just calls
`arch-chroot`. Like this:

``` bash
#exec_as_chroot.sh
#!/bin/bash
set -ex

arch-chroot "${ARGPY_MNT_ROOT}" << EOF
$@
EOF

```

* Then my python script calls the file like this:

``` python
subprocess.run(
        args = [
            "exec_as_chroot.sh",
            "arg1",
            "arg2"
        ]
)
```       

## Result
Linux machines that are tunning Gentoo as OS. Although using defaults,
the installation is minimal. (I'm sure I can optimize it more).

UN100D machine:
``` bash
un100d02 ~ # neofetch
         -/oyddmdhs+:.                root@un100d02 
     -odNMMMMMMMMNNmhy+-`             ------------- 
   -yNMMMMMMMMMMMNNNmmdhy+-           OS: Gentoo Linux x86_64 
 `omMMMMMMMMMMMMNmdmmmmddhhy/`        Host: Venus Series 
 omMMMMMMMMMMMNhhyyyohmdddhhhdo`      Kernel: 6.6.35-gentoo-dist 
.ydMMMMMMMMMMdhs++so/smdddhhhhdm+`    Uptime: 22 days, 1 hour, 33 mins 
 oyhdmNMMMMMMMNdyooydmddddhhhhyhNd.   Packages: 372 (emerge) 
  :oyhhdNNMMMMMMMNNNmmdddhhhhhyymMh   Shell: bash 5.2.26 
    .:+sydNMMMMMNNNmmmdddhhhhhhmMmy   Terminal: /dev/pts/0 
       /mMMMMMMNNNmmmdddhhhhhmMNhs:   CPU: Intel N100 (4) @ 3.400GHz 
    `oNMMMMMMMNNNmmmddddhhdmMNhs+`    GPU: Intel Alder Lake-N [UHD Graphics] 
  `sNMMMMMMMMNNNmmmdddddmNMmhs/.      Memory: 450MiB / 15672MiB 
 /NMMMMMMMMNNNNmmmdddmNMNdso:`
+MMMMMMMNNNNNmmmmdmNMNdso/-                                   
yMMNNNNNNNmmmmmNNMmhs+/-`                                     
/hMMNNNNNNNNMNdhs++/-`
`/ohdmmddhys+++/:.`
  `-//////:--.
```

Rasperry Pi 5 Machine:
``` bash
rpi504 ~ # neofetch
         -/oyddmdhs+:.                root@rpi504 
     -odNMMMMMMMMNNmhy+-`             ----------- 
   -yNMMMMMMMMMMMNNNmmdhy+-           OS: Gentoo Linux aarch64 
 `omMMMMMMMMMMMMNmdmmmmddhhy/`        Host: Raspberry Pi 5 Model B Rev 1.0 
 omMMMMMMMMMMMNhhyyyohmdddhhhdo`      Kernel: 6.6.36-v8-16k+ 
.ydMMMMMMMMMMdhs++so/smdddhhhhdm+`    Uptime: 23 days, 23 hours, 43 mins 
 oyhdmNMMMMMMMNdyooydmddddhhhhyhNd.   Packages: 359 (emerge) 
  :oyhhdNNMMMMMMMNNNmmdddhhhhhyymMh   Shell: bash 5.2.26 
    .:+sydNMMMMMNNNmmmdddhhhhhhmMmy   Terminal: /dev/pts/0 
       /mMMMMMMNNNmmmdddhhhhhmMNhs:   CPU: ARM Cortex-A76 (4) @ 2.400GHz 
    `oNMMMMMMMNNNmmmddddhhdmMNhs+`    Memory: 283MiB / 8052MiB 
  `sNMMMMMMMMNNNmmmdddddmNMmhs/.
 /NMMMMMMMMNNNNmmmdddmNMNdso:`                                
+MMMMMMMNNNNNmmmmdmNMNdso/-                                   
yMMNNNNNNNmmmmmNNMmhs+/-`
/hMMNNNNNNNNMNdhs++/-`
`/ohdmmddhys+++/:.`
  `-//////:--.
```
