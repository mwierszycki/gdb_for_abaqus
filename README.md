# Debugger for Abaqus user subroutines

 User subroutines are extremely powerful and flexible tools to extend the functionality of Abaqus solvers, commonly used by advanced Abaqus users. Debugging is an integral part of coding any procedures. For Abaqus user subroutines developing on Windows usually Microsoft Visual Studio is used to write and to debug user subroutines, although Notepad is theoretically enough to write a simple, several-line code. On Linux, the choice of development environment is not so obvious, and depends largely on the user's preferences and the programming project scale. Usually nobody uses e.g. Eclipse to write a short, simple code. The case is similar when choosing a debugger. Just as Vim or Emacs are good choices to write simple user subroutines, the GDB is a very common and powerful debugger. One of its main advantages is the access to it on any Linux distribution - it is installed by default with GNU compilers. So even if for many people its text interface is not very user friendly, for debugging relatively small Abaqus user subroutines it is really an excellent tool - as powerful on the one hand and as simple on the other as debugger should be.
```
$ abaqus debug
```
It is developed for internal use by Abaqus developers but [available to everyone](https://kb.dsxclient.3ds.com/mashup-ui/page/document?q=docid:QA00000007986). The default debugger on Linux is TotalView:
```
$ abaqus debug

DEBUGGER   : totalview
```
There is the -db option which can be used to choose another debugger. The list of recognized debuggers is quite long:
```
devenv,msdev,msdev10,windbg,totalview,tvbeta,tvold,gdb,gdb64,dbx,cvd,wdb,ddd,idb
```
 and we can find the gdb on it as well. Unfortunately the -db option doesn't work correctly for gdb.

First problem is that the arguments required by Abaqus to run a job are not passed to the gdb execution command at all. The reason for that is the restriction for auto-loading files with commands and settings for gdb. This feature is used by abaqus debug command to pass arguments, a name of a subroutine where breakpoint is set and run abaqus in gdb. However by default gdb loads files with commands and settings automatically only from selected locations. The local .gdbinit file created by abaqus debug command cannot be loaded due to this restriction. To enable loading of that file add line:
```
set auto-load safe-path /
```
 to the gdb configuration file ~/.gdbinit in home directory.

Second problem is setting a breakpoint at the subroutine name. It's done when the subroutine is not loaded yet in gdb. Default setting of gdb in that case is to skip breakpoint definition at all. The change it add line:
```
set breakpoint pending on
```
to the configuration file ~/.gdbinit in home directory.

To debug a subroutine, it must be compiled with -g argument to produce debugging information. In the case of GDB you can use -ggdb to produce debugging information for use by GDB specifically. It can be done by modifying Abaqus environment variable compile_fortran in Abaqus environment file: 
```
$ cat abaqus_v6.env

import os
compile_fortran=compile_fortran + ['-ggdb']
```
Please note that for debugging all code optimization options should be removed as well. If the set of arguments for Intel compilers specific for Abaqus is used, then the best option is to copy the complete definition of environment variable compile_fortran (from lnx86_64.env file) into the local Abaqus environment file, and remove all optimization options.

To run Abaqus in gdb use the following command:
```
$ abaqus debug -standard [or -explicit] -db gdb -j job_name -user user.f -stop subroutine_name -int
…
Breakpoint 1, subroutine_name (…) at user.f:line_nb

(gdb)
```
When the execution is stopped, user subroutine debugging can be started.

To execute a single statement and step into the function, use command:
```
(gdb) step
```
To execute a single statement but step over the function, use command:
```
(gdb) next
```
To execute the rest of the current function and step out of the function, use command:
```
(gdb) finish
```
To list the code in the vicinity of where the program is presently stopped, use command:
```
(gdb) list
```
To set the next breakpoints e.g. at a particular line of the code, use command:
```
(gdb) break line_number
```
To print a variable, use command:
```
(gdb) print variable_name
```
Please note that for your convenience most GDB commands can be abbreviated to one (or a few) letter versions e.g.:
```
break -> b
 
run -> r
```
Simplicity of use with the command line should not obscure how powerful gdb really is. Hundreds of other commands can be found in the GDB manual. For example, GDB has also some commands to support Fortran-specific features, e.g. displaying common blocks and its variables:
```
(gdb) info common [common-name]
```
This command prints the values contained in the Fortran COMMON block, whose name is common-name. With no argument, the names of all COMMON blocks visible at the current program location are printed.

More about GDB can be found in the [documentation](https://www.sourceware.org/gdb/documentation/) and tons of [tutorials](https://www.google.com/search?q=gdb+tutorial) on the Internet. You can try also a bit more graphical UI with:
```
$ gdb --tui
```
or [cgdb](https://cgdb.github.io/).

Happy debugging!

Note! To suppress "Missing separate debuginfo for …" messages in gdb use:
```
$ echo 'set build-id-verbose 0' >> ~/.gdbinit
```
