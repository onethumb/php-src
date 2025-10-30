---
inclusion: always
---

# PHP Build Process Requirements

## CRITICAL: Command Execution Rules

**NEVER USE COMBINED COMMANDS WITH `;` OR `&&` WHEN IN THE PHP SDK ENVIRONMENT!**

- **ALWAYS** execute each command separately
- **NEVER** use command separators like `;` or `&&` 
- **NEVER** combine `cd` with other commands in a single execution
- Each command must be issued individually using separate tool calls

Examples of what NOT to do:
```
❌ cd "path" ; buildconf
❌ cd "path" && configure
❌ buildconf ; nmake
```

Examples of correct usage:
```
✅ cd "path"
✅ buildconf
✅ configure
✅ nmake
```

## Building

### buildconf

First, you use `buildconf` to create `configure`: 

```
$ buildconf
Rebuilding configure.js
Now run 'configure --help'
```

**ALWAYS** call `buildconf` on its own, **NEVER** call it in combination with another command such as `cd`

### configure

Next, you use `configure` to create the `make` artifacts:

Without `crc-fast`: 
```
$ configure --disable-all --enable-cli
PHP Version: 8.5.0-dev

Saving configure options to config.nice.bat

...
```

`crc-fast` is already installed in this system in `C:\Program Files\crc-fast`.

With `crc-fast` (`libcrc-fast.h` is in `C:\Program Files\crc-fast\include`, `crc-fast.dll` is in `C:\Program Files\crc-fast\bin`, and `crc-fast.dll.lib` is in ``C:\Program Files\crc-fast\lib`):
```
$ configure --disable-all --enable-cli --with-crc-fast=C:\Program Files\crc-fast
PHP Version: 8.5.0-dev

Saving configure options to config.nice.bat

...
```

**ALWAYS** call `configure` on its own, **NEVER** call it in combination with another command such as `cd`

### nmake

Compiling PHP uses `nmake` on Windows:

```
$ nmake

Microsoft (R) Program Maintenance Utility Version 14.44.35215.0
Copyright (C) Microsoft Corporation.  All rights reserved.

        type ext\pcre\php_pcre.def > C:\Users\onethumb\git\php-sdk-binary-tools\phpdev\vs17\x64\php-8.4-src\x64\Release_TS\php8ts.dll.def
php_cli.c

...
```

**ALWAYS** call `nmake` on its own, **NEVER** call it in combination with another command such as `cd`


### Cleaning with `nmake`

**ALWAYS** clean old builds up using `nmake clean` before making changes with `buildconf` and `configure`.

**NEVER** run `nmake clean` just to rebuild unless making changes with `buildconf` and/or `configure`.