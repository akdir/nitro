# nitro

Virtual Machine Introspection for KVM.

This is the userland component named `nitro`.
It will receive the events generated by KVM and display them.

# Requirements

- `cmake`
- `python 3`
- `docopt`
- `libvirt`
- `libvmi`

# Setup

- You have to compile `libnitro` :

~~~
cd libnitro
mkdir _build && cd _build
cmake ..
make
~~~

This will produce `libnitro/libnitro.so`.

- Compile and install `libvmi`. See the [install notes](http://libvmi.com/docs/gcode-install.html)

- Configure `libvmi.conf`

Create a configuration in `$HOME/etc/libvmi.conf`

Configure each vm that you want to monitor :

~~~
winxpx64 {
    ostype      = "Windows";
    win_tasks   = 0x88;
    win_pdbase  = 0x18;
    win_pid     = 0x84;
}

win7x64 {
    ostype      = "Windows";
    win_tasks   = 0x188;
    win_pdbase  = 0x28;
    win_pid     = 0x180;
}
~~~

- Install a VM. Make sure to use the `qemu:///system` connection.

(Nitro only supports for now `Windows XP x64` and `Windows 7 x64`, see the `Note` section below)


# Usage

- Make sure that you have loaded the modified kvm modules. 
(`cd kvm-vmi && make modules && make reload`)

- Start the VM that you would like to monitor.

- Wait for the desktop to be available on the VM.

- Start `Nitro` as root

~~~
Usage:
  nitro.py [options] <vm_name> (32 | 64)

Options:
  -h --help     Show this screen.
  --nobackend   Dont analyze events
~~~

The `--nobackend` option will just display the registers :

~~~
SYSCALL cr3 0x101d3000L - rax 0x32L
SYSRET  cr3 0x101d3000L - rax 0x0L
SYSCALL cr3 0x101d3000L - rax 0xcL
SYSRET  cr3 0x101d3000L - rax 0x0L
SYSCALL cr3 0x101d3000L - rax 0x30L
SYSRET  cr3 0x101d3000L - rax 0x0L
SYSCALL cr3 0x101d3000L - rax 0x32L
SYSRET  cr3 0x101d3000L - rax 0x0L
SYSCALL cr3 0x101d3000L - rax 0xcL
SYSRET  cr3 0x101d3000L - rax 0x0L
SYSCALL cr3 0x101d3000L - rax 0x30L
SYSRET  cr3 0x101d3000L - rax 0x0L
~~~

A successful run should give the following output :

~~~
Finding PID of VM win7x64
pid = 11926
Initializing libvmi ...
Taking Physical Memory dump ...
Getting symbols ...
{u'ActiveProcessLinks_off': 392, u'DirectoryTableBase_off': 40, u'PsActiveProcessHead': 272678926175120, u'UniqueProcessId_off': 384, u'ImageFileName_off': 736}
Loading libnitro.so
Initializing KVM
Attaching to the VM
Attaching to VCPUs
Setting Traps
[SYSCALL] explorer.exe -> NtUserPeekMessage
[SYSRET ] explorer.exe -> NtUserPeekMessage
[SYSCALL] explorer.exe -> NtUserGetProp
[SYSRET ] explorer.exe -> NtUserGetProp
[SYSCALL] explorer.exe -> NtUserGetProp
[SYSRET ] explorer.exe -> NtUserGetProp
[SYSCALL] explorer.exe -> NtUserQueryWindow
[SYSRET ] explorer.exe -> NtUserQueryWindow
[SYSCALL] explorer.exe -> NtUserQueryWindow
[SYSRET ] explorer.exe -> NtUserQueryWindow
[SYSCALL] explorer.exe -> NtUserGetProp
[SYSRET ] explorer.exe -> NtUserGetProp
[SYSCALL] explorer.exe -> NtUserQueryWindow
[SYSRET ] explorer.exe -> NtUserQueryWindow
[SYSCALL] explorer.exe -> NtUserGetProp
[SYSRET ] explorer.exe -> NtUserGetProp
[SYSCALL] explorer.exe -> NtCallbackReturn
[SYSRET ] explorer.exe -> NtCallbackReturn
[SYSCALL] explorer.exe -> NtUserGetProp
[SYSRET ] explorer.exe -> NtUserGetProp
[SYSCALL] explorer.exe -> NtUserGetProp
^CUnsetting Traps
Closing KVM
~~~


# Note

We need to use the root user because:
- `libvmi` uses only `qemu:///system` as libvirt connection
- even if you are in the `libvirt` group, and therefore can access `qemu:///system`,
the memory dump produced by `libvirt` is created as `root:root`...
