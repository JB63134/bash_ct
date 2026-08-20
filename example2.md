## `ct` conflict mode flags `-c` `-j` `-x`

** Normal command conflict report `-c`**

```bash
22:47:17 Wed Aug 19: ~ $ ct -c

Command Resolution & Shadowing Report:
This may take a moment

Keywords 

Aliases 
  Alias: ..           
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: egrep        
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: fgrep        
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: grep         
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: ls           
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins


Functions 
  Function: cd        
     ↳ Also defined as: builtin(shadowed),external(shadowed)
     ↳ Resolution:      function wins

  Function: in4       
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      function wins


Builtins 
  Builtin: .            enabled
     ↳ Also defined as: keyword,external(shadowed)
     ↳ Resolution:      keyword wins

  Builtin: [            enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: cd           enabled
     ↳ Also defined as: function,external(shadowed)
     ↳ Resolution:      function wins

  Builtin: echo         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: false        enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: kill         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: printf       enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: pwd          enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: test         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: true         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins
```

---

Adding the `-x` flag temporarily extends the executable search path used by ct so that commands normally outside the user's $PATH can be detected. The user's original $PATH is restored when the analysis completes.

Pay particular attention to `fstrim` in the alias section of the output.


```bash
22:47:22 Wed Aug 19: ~ $ ct -cx

Command Resolution & Shadowing Report:
This may take a moment

$PATH has been manually extended.

Keywords 

Aliases 
  Alias: ..           
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: egrep        
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: fgrep        
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: fstrim       
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: grep         
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins

  Alias: ls           
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      alias wins


Functions 
  Function: cd        
     ↳ Also defined as: builtin(shadowed),external(shadowed)
     ↳ Resolution:      function wins

  Function: in4       
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      function wins


Builtins 
  Builtin: .            enabled
     ↳ Also defined as: keyword,external(shadowed)
     ↳ Resolution:      keyword wins

  Builtin: [            enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: cd           enabled
     ↳ Also defined as: function,external(shadowed)
     ↳ Resolution:      function wins

  Builtin: echo         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: false        enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: kill         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: printf       enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: pwd          enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: test         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins

  Builtin: true         enabled
     ↳ Also defined as: external(shadowed)
     ↳ Resolution:      builtin wins
```

since admin paths were added, ct was able to detect the shadowed fstrim in a system directory.

---

Use `-j` flag for JSON output. 

```bash
22:47:28 Wed Aug 19: ~ $ ct -cjx

{
  "report_time": "2026-08-19T22:47:45-0500",
  "message": "$PATH has been manually extended.",
  "resolution_and_shadowing_report": {
    "keywords": [

    ],
    "aliases": [
      {
        "name": "..",
        "winner": "alias",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "egrep",
        "winner": "alias",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "fgrep",
        "winner": "alias",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "fstrim",
        "winner": "alias",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "grep",
        "winner": "alias",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "ls",
        "winner": "alias",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      }
    ],
    "functions": [
      {
        "name": "cd",
        "winner": "function",
        "also_defined_as": [{ "type": "builtin", "shadowed": true },
                            { "type": "external", "shadowed": true }]
      },
      {
        "name": "in4",
        "winner": "function",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      }
    ],
    "builtins": [
      {
        "name": ".",
        "state": "enabled",
        "winner": "keyword",
        "also_defined_as": [{ "type": "keyword", "shadowed": false },
                            { "type": "external", "shadowed": true }]
      },
      {
        "name": "[",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "cd",
        "state": "enabled",
        "winner": "function",
        "also_defined_as": [{ "type": "function", "shadowed": false },
                            { "type": "external", "shadowed": true }]
      },
      {
        "name": "echo",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "false",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "kill",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "printf",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "pwd",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "test",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      },
      {
        "name": "true",
        "state": "enabled",
        "winner": "builtin",
        "also_defined_as": [{ "type": "external", "shadowed": true }]
      }
    ]
  }
}

22:47:46 Wed Aug 19: ~ $ 
```
