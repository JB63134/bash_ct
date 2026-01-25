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


05:55:58 Thu Jan 22: ~ $ ct fg
Command Trace of fg

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  fg → found → enabled

$PATH in order:
  ↳ /home/jb/bin             - fg [shadowed]
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Builtin → fg → enabled
  ↳ Note: Filesystem executables named 'fg' are unreachable by bare invocation.

Kernel Execution Target:
  ↳ NONE                  

05:56:06 Thu Jan 22: ~ $ enable -n fg    # disable the builtin

05:56:30 Thu Jan 22: ~ $ ct fg
Command Trace of fg

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  fg → found → disabled

$PATH in order:
  ↳ /home/jb/bin             - fg found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Filesystem → /home/jb/bin/fg

Kernel Execution Target:
  ↳ Executable → /home/jb/bin/fg
  ↳ Shebang: /usr/bin/env bash


05:56:33 Thu Jan 22: ~ $ 
#-------------------------------------------------------------------------------------------------------
Posix mode:
22:58:16 Sat Jan 24: ~ $ foo/bar
Sat Jan 24 10:58:19 PM CST 2026

22:58:19 Sat Jan 24: ~ $ eval
jb       seat0        2026-01-23 08:21
jb       tty2         2026-01-23 08:21

22:58:21 Sat Jan 24: ~ $ ct foo/bar
Command Trace of foo/bar

Keyword:  -  not found
Alias:    -  not found
Function: -  foo/bar → found → Function foo/bar (/home/jb/.bash_functions : line 95)
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

Bash Resolution Target:
  ↳ Resolved to: Function → foo/bar → /home/jb/.bash_functions : line 95

Kernel Execution Target:
  ↳ NONE


22:58:23 Sat Jan 24: ~ $ ct eval 
Command Trace of eval

Keyword:  -  not found
Alias:    -  not found
Function: -  eval → found → Function eval (/home/jb/.bash_functions : line 99)
Builtin:  -  eval → found → enabled [shadowed]

$PATH in order:
  ↳ /home/jb/bin             - not found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Function → eval → /home/jb/.bash_functions : line 99
  ↳ Note: Builtin 'eval' is unreachable by bare invocation.

Kernel Execution Target:
  ↳ NONE


22:58:26 Sat Jan 24: ~ $ set -o posix

22:58:29 Sat Jan 24: ~ $ ct foo/bar
Unknown or invalid command: foo/bar
  ↳ Check your spelling and try again


22:58:33 Sat Jan 24: ~ $ ct eval 
Command Trace of eval

Bash is in POSIX mode:
POSIX special builtins cannot be shadowed by functions;
conflicting function names are disallowed.

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  eval → found → enabled

$PATH in order:
  ↳ /home/jb/bin             - not found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Builtin → eval → enabled

Kernel Execution Target:
  ↳ NONE


22:58:37 Sat Jan 24: ~ $ 

22:59:40 Sat Jan 24: ~ $ ct cd
Command Trace of cd

Bash is in POSIX mode:
POSIX special builtins cannot be shadowed by functions;
conflicting function names are disallowed.

Keyword:  -  not found
Alias:    -  not found
Function: -  cd → found → Function cd (/home/jb/.bash_functions : line 26)
Builtin:  -  cd → found → enabled [shadowed]

$PATH in order:
  ↳ /home/jb/bin             - cd [shadowed]
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Function → cd → /home/jb/.bash_functions : line 26
  ↳ Note: Builtin 'cd' is unreachable by bare invocation.
  ↳ Note: Filesystem executables named 'cd' are unreachable by bare invocation.

Kernel Execution Target:
  ↳ NONE


22:59:43 Sat Jan 24: ~ $ 


```
