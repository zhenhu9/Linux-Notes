## Process management

### Utilities about process




------

**chroot**

chroot [OPTION] NEWROOT [COMMAND [ARG]...]

run command or interactive shell with special root directory.

Each process/command on Linux and Unix-like system has current working
directory called root directory of a process/command. You can change
the root directory of a command using chroot command, which ends up
changing the root directory for both current running process and its
children.

A process/command that is run in such a modified environment cannot
access files outside the root directory. This modified environment is
commonly known as “jailed directory” or “chroot jail”. Only a
privileged process and root user can use chroot command. This is
useful to:


1. Privilege separation for unprivileged process such as Web-server or
   DNS server.

2. Setting up a test environment.

3. Run old programs or ABI in-compatibility programs without crashing
   application or system.

4. System recovery.

5. Reinstall the bootloader such as Grub or Lilo.

6. Password recovery – Reset a forgotten password and more.

The term “jail” shouldn’t be confused with FreeBSD’s jail command,
which creates a chroot environment that is more secure than the usual
chroot environment.

Some Linux distributions have dedicated tools to set up chroot
environments, such as debootstrap for Ubuntu, but we’re being
distro-agnostic here.

When Should You Use a chroot?

A chroot environment provides functionality similar to that of a
virtual machine, but it is a lighter solution. The captive system
doesn’t need a hypervisor to be installed and configured, such as
VirtualBox or Virtual Machine Manager. Nor does it need to have a
kernel installed in the captive system. The captive system shares your
existing kernel.

In some senses, chroot environments are closer to containers such as
LXC than to virtual machines. They’re lightweight, quick to deploy,
and creating and firing one up can be automated. Like containers, one
convenient way to configure them is to install just enough of the
operating system for you to accomplish what is required. The “what is
required” question is answered by looking at how you’re going to use
your chroot environment.

```
--groups=G_LIST
       specify supplementary groups as g1,g2,..,gN

--userspec=USER:GROUP
       specify user and group (ID or name) to use

--skip-chdir
       do not change working directory to '/'

If no command is given, run '"$SHELL" -i' (default: '/bin/sh -i').
```

Examples:

A minimal chroot environment or jail.

1. Creating the new chroot directory and filesystem structure of the
   chroot environment.
 
```
$ chr=/home/your_current_user_account/testroot
$ mkdir -p $chr
$ mkdir -p $chr/{bin,lib,lib64}
```

2. Copy the binaries that we need in our minimalist Linux environment.

```
$ cp -v /bin/{bash,touch,ls,rm} $chr/bin
```

3. Copy those binaries's dependencies respectively.

```
$ ldd /bin/bash

linux-vdso.so.1 (0x00007ffd69439000)
libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007f3eb8ac2000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f3eb8abc000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3eb88f7000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3eb8c42000)

$ list="$(ldd /bin/bash | egrep -o '/lib.*\.[0-9]')"
$ for i in $list; do cp -v --parents "$i" "${chr}"; done

$ list="$(ldd /bin/touch | egrep -o '/lib.*\.[0-9]')"
$ for i in $list; do cp -v --parents "$i" "${chr}"; done

$ list="$(ldd /bin/ls | egrep -o '/lib.*\.[0-9]')"
$ for i in $list; do cp -v --parents "$i" "${chr}"; done

$ list="$(ldd /bin/rm | egrep -o '/lib.*\.[0-9]')"
$ for i in $list; do cp -v --parents "$i" "${chr}"; done
```

4. Setting the roof of the chroot environment and specifying which
   application to run as the shell.

```
$ sudo chroot $chr /bin/bash

bash-5.1#
```

5. Testing in chroot jail.

```
bash-5.1# touch sample.txt

bash-5.1# ls -al

bash-5.1# rm sample.txt
```

6. Use exit to leave the chroot environment.

```
bash-5.1# exit
```

------

https://www.cyberciti.biz/faq/unix-linux-chroot-command-examples-usage-syntax/#:~:text=chroot%20command%20details%20%20%20%20Description%20,%20%2030m%20%201%20more%20rows%20

https://www.howtogeek.com/441534/how-to-use-the-chroot-command-on-linux/
