```bash

05:20:31 Thu Jan 22: ~ $ ct fstrim
Command Trace of fstrim

Not in $USER $PATH
Found in system $PATH (auto-extended):

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  not found

$PATH in order:
  ↳ /home/jb/bin             - not found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found
  ↳ /usr/local/sbin          - not found
  ↳ /usr/sbin                - fstrim found
  ↳ /sbin                    - fstrim [shadowed] [usr-merged]

Bash Resolution Target:
  ↳ Resolved to: Filesystem → /usr/sbin/fstrim

Kernel Execution Target:
  ↳ Executable → /usr/sbin/fstrim
  ↳ ELF interpreter: /lib64/ld-linux-x86-64.so.2


05:21:36 Thu Jan 22: ~ $ ct useradd
Command Trace of useradd

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  not found

$PATH in order:
  ↳ /home/jb/bin             - useradd found    # match found in user path
  ↳ /home/jb/bin/scripts     - not found        # system path not auto-extended
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Filesystem → /home/jb/bin/useradd

Kernel Execution Target:
  ↳ Executable → /home/jb/bin/useradd
  ↳ Shebang: /bin/bash


05:22:01 Thu Jan 22: ~ $ ct -x useradd    # path manually extended
Command Trace of useradd

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  not found

$PATH in order:
  ↳ /home/jb/bin             - useradd found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found
  ↳ /usr/local/sbin          - not found
  ↳ /usr/sbin                - useradd [shadowed]    # system path binaries are now found
  ↳ /sbin                    - useradd [shadowed]

Bash Resolution Target:
  ↳ Resolved to: Filesystem → /home/jb/bin/useradd

Kernel Execution Target:
  ↳ Executable → /home/jb/bin/useradd
  ↳ Shebang: /bin/bash



05:22:13 Thu Jan 22: ~ $ type useradd      # system path binaries are not listed
useradd is /home/jb/bin/useradd

05:23:24 Thu Jan 22: ~ $ type -a useradd
useradd is /home/jb/bin/useradd               

05:23:36 Thu Jan 22: ~ $ which useradd        
/home/jb/bin/useradd                          

05:29:51 Thu Jan 22: ~ $ which -a useradd
/home/jb/bin/useradd                          
```
