# Linux Hardware Management

There are many commands which can be used to check out the information of our
computer hardwares.

Let's have a look of the cpu with lscpu command.

Some terms about CPU:

- Socket: One socket is one physical CPU package (which occupies one socket on
  the motherboard), represented by the logic socket numbers.

- Core: The physical cpu numbers in a socket, represented by the core numbers.

- Cpu: Each core can run one or more threads, which means each thread is a cpu,
  represented by The logical CPU numbers.

- NODE: “NUMA node” represents the memory architecture; “NUMA” stands for
  “non-uniform memory architecture”. In your system, each socket is attached
  to certain DIMM slots, and each physical CPU package contains a memory
  controller which handles part of the total RAM. As a result, not all
  physical memory is equally accessible from all CPUs: one physical CPU can
  directly access the memory it controls, but has to go through the other
  physical CPU to access the rest of memory.

**Notes**: NUMA can affect some servers, such as a SQL server. There are some
information in `What Is Non-Uniform Memory Access (NUMA)?` of the references.

- Bogomips: Bogomips is a measurement provided in the Linux operating system
  that indicates in a relative way how fast the computer processor runs. The
  program that provides the measurement is called BogoMips, written by Linus
  Torvalds, the main developer of Linux. Bogomips measures how many times the
  processor goes through a particular programming loop in a second.

- Vulnerability: Higher version of lscpu command will display vlunerability
  information.

**Notes**: There are more descriptions about cpu terms which you can find in
references `5 Ways to Check CPU Info in Linux`.

Most of the information lscpu displays can also get from `cat /proc/cpuinfo`.
If you want to get more professional information about cpu, try `cpuid`.

dmidecode is a tool for dumping a computer's DMI (some say SMBIOS) table
contents in a human-readable format. This table contains a description of the
system's hardware components, as well as other useful pieces of information
such as serial numbers and BIOS  revision. Thanks  to this table, you can
retrieve this information without having to probe for the actual hardware.

SMBIOS stands for System Management BIOS, while DMI stands for Desktop
Management Interface. Both standards are tightly related and developed by the
DMTF (Desktop Management Task Force).

biosdecode is a BIOS information decoder.

vpddecode is a VPD structure decoder. VPD stands for "vital product data".

lshw is used to list hardware information.

hardinfo is a gui tool to list all hardware and benchmark information.

lspci, lsusb and lsblk display pci, usb, block devices.

## Related manual pages

- [lscpu](./lscpu.1.manpage)
- [cpuid](./cpuid.1.manpage)
- [dmidecode](./dmidecode.8.manpage)
- [biosdecode](./biosdecode.8.manpage)
- [vpddecode](./vpddecode.8.manpage)
- [lshw](./lshw.1.manpage)
- [hardinfo](./hardinfo.1.manpage)
- [lspci](./lspci.8.manpage)
- [lsusb](./lsusb.8.manpage)
- [lsblk](./lsblk.8.manpage)

## References

- [Understanding output of lscpu](https://unix.stackexchange.com/questions/468766/understanding-output-of-lscpu)
- [What Is Non-Uniform Memory Access (NUMA)?](https://community.idera.com/database-tools/blog/b/community_blog/posts/what-is-non-uniform-memory-access-numa)
- [Physical CPU and Logical CPU](https://www.perfmatrix.com/physical-cpu-and-logical-cpu/#:~:text=In%20above%20example%2C%20the%20number%20of%20Logical%20CPUs,Physical%20cores%20%28CPUs%29%20and%208%20Logical%20cores%20%28CPUs%29.)
- [bogomips](https://searchdatacenter.techtarget.com/definition/bogomips#:~:text=Bogomips%20is%20a%20measurement%20provided%20in%20the%20Linux,program%20that%20provides%20the%20measurement%20is%20called%20BogoMips.)
- [9 Useful Commands to Get CPU Information on Linux](https://www.tecmint.com/check-linux-cpu-information/)
- [5 Ways to Check CPU Info in Linux](https://linuxhandbook.com/check-cpu-info-linux/)
