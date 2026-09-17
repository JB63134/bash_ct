# `bash_ct` Changelog

## V4.4.50

### Improved

* Added detection and reporting of empty `$PATH` components.

  * Empty components are represented explicitly in the PATH trace.
  * The human-readable report identifies them as `'' (current directory)`.
  * A PATH note explains that Bash treats an empty component as the current directory.
* Improved `$PATH` candidate validation by requiring entries to be both regular files and executable.
* Improved command resolution and shadowing detection across aliases, functions, keywords, builtins, and `$PATH`.
* Expanded filesystem and kernel-level tracing, including symlinks, `/etc/alternatives`, ELF interpreters, shebangs, and usr-merged paths.
* Improved JSON output and escaping, including explicit handling of empty strings versus `null`.
* Improved POSIX-mode handling and special-builtin precedence.
* Fixed numerous edge cases in PATH resolution, conflict reporting, JSON output, and shell-state restoration.

## V4.4.05

### documentation

* Fixed usage dialog to make -x clear

## V4.4.0

### Improved

#### Code cleanup
* fix json.  on autoextended path the message needed a comma. remove dependency on date
* fixed some json output path and external were both showing up depending on edge case.
* ELF header code, shebang detection and builtin enabled/disabled detection, are now native bash. 
* conflict reports switched to compgen to remove dependency on awk.
* removed dependencies for grep head awk sed - switch to native bash 
* tried to clean up posix mode behaviour.  
* passed shellcheck.


## V4.3.0
### Added

#### Conflict Analysis Mode (`-c`)

* Introduced `ct -c` for environment-wide command name collision analysis
* Scans:

  * aliases
  * functions
  * builtins
  * keywords
  * external `$PATH` commands (presence-only detection)
* Reports resolution hierarchy (what wins in Bash precedence order)
* Detects shadowing relationships across all categories

#### External command integration in conflict mode

* External binaries are reported as:

  * `external(shadowed)`
* No full `$PATH` resolution is performed in `-c` mode (intentional simplification)
* Reduces:

  * output noise
  * false positives from PATH ordering ambiguity
  * runtime overhead in large environments

---

### Changed

#### Output semantics

* Conflict mode reports **name collisions only**
* Trace mode remains responsible for full resolution + filesystem inspection

---

### Improved

#### Code cleanup

* fixed `nameref` usage for cleaner internal references
* Reduced reliance on global variables
* Improved function encapsulation and readability
* `_ct_resolve` optimized for reuse in both trace and conflict modes
  
  * now caches `compgen` results
  * reduces repeated shell calls
  * improved performance in `-c` mode on large environments
---

## V4.2.15
- Json output-fixed boolean output for posix detection

## V4.2.10
- Posix mode detection and rule set added

## V4.2.5
- Public release

## V4.0.0...
- refinements... multiple sections rewritten

## V3.0.0
- major refactor

## V2.0.0
- moved to its own script...

## V1.0.0
- Original was a flaged mode in another script
- only detected commands and displayed $PATH shadowing

