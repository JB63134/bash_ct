
### Example: `$PATH` Auto-Extension and `-x` Manual Extension

**On a typical system, fstrim will not be found in the user's `$PATH`.**

``` bash

21:54:17 Wed Aug 19: ~ $ ct fstrim
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
```

Since `fstrim` wasn't found in the user's `$PATH`, `ct` auto-extended the search to include administrative system directories.

---

Let's add a custom user script called `fstrim`.

```bash
21:54:36 Wed Aug 19: ~ $ mv ~/bin/dummyfile ~/bin/fstrim
```

Now `fstrim` will be found in the user's `$PATH`.

---

```bash
21:54:47 Wed Aug 19: ~ $ ct fstrim
Command Trace of fstrim

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  not found

$PATH in order:
  ↳ /home/jb/bin             - fstrim found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found

Bash Resolution Target:
  ↳ Resolved to: Filesystem → /home/jb/bin/fstrim

Kernel Execution Target:
  ↳ Executable → /home/jb/bin/fstrim
  ↳ Shebang: #!/bin/bash

```


Since `fstrim` was found in the user's `$PATH` there was no need to extend the search.

---

By manually extending the search with `-x` you can see if any system commands are being shadowed.

```bash
21:55:18 Wed Aug 19: ~ $ ct -x fstrim
Command Trace of fstrim

Keyword:  -  not found
Alias:    -  not found
Function: -  not found
Builtin:  -  not found

$PATH in order:
  ↳ /home/jb/bin             - fstrim found
  ↳ /home/jb/bin/scripts     - not found
  ↳ /home/jb/.local/bin      - not found
  ↳ /usr/local/bin           - not found
  ↳ /usr/bin                 - not found
  ↳ /bin                     - not found
  ↳ /usr/local/games         - not found
  ↳ /usr/games               - not found
  ↳ /usr/local/sbin          - not found
  ↳ /usr/sbin                - fstrim [shadowed]
  ↳ /sbin                    - fstrim [shadowed]

Bash Resolution Target:
  ↳ Resolved to: Filesystem → /home/jb/bin/fstrim

Kernel Execution Target:
  ↳ Executable → /home/jb/bin/fstrim
  ↳ Shebang: #!/bin/bash


21:55:21 Wed Aug 19: ~ $ 
```

**No user `$PATH` hit:** `ct` auto-extends the search to system directories.  
**With `-x`:** `ct` extends the search even when a user `$PATH` hit exists.
