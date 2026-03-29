# `bash_ct` Changelog

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
* `_ct_resolve` optimized for reuse in both trace and conflict modes
  
  * now caches `compgen` results
  * reduces repeated shell calls
  * improved performance for `-c` mode in large environments
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

