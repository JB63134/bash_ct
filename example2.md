```bash

15:02:35 Sun Mar 29: ~ $ ct -c

Command Resolution & Shadowing Report:
This may take a moment

Keywords 
  Keyword: then       
     ↳ Also defined as: external(shadowed)


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



15:02:41 Sun Mar 29: ~ $ ct -cj

{
  "report_time": "2026-03-29 15:02:46",
  "resolution_&_shadowing_report": {
    "keywords": [
      {
        "name": "then",
        "winner": "keyword",
        "also_defined_as": ["external"]
      }
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

15:02:46 Sun Mar 29: ~ $ ct -cjx

{
  "report_time": "2026-03-29 15:02:54",
  "message": "$PATH has been manually extended."
  "resolution_&_shadowing_report": {
    "keywords": [
      {
        "name": "then",
        "winner": "keyword",
        "also_defined_as": ["external"]
      }
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

15:02:54 Sun Mar 29: ~ $ 


```
