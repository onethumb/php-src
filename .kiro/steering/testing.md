---
inclusion: always
---

# Testing Guidelines

## No regressions!

This codebase builds and tests both `ext/hash` and `ext/standard/tests/strings/crc32.phpt` completely, so ensure there are never any regessions before moving onto a new task. If there are, ALWAYS fix them first.

**ALWAYS** run the `ext/hash` test suite entirely, ensuring no regressions, using `x64\Release_TS\php.exe run-tests.php ext/hash`.

**ALWAYS** run the `ext/standard/strings/crc32` tests, ensuring no regressions, using `x64\Release_TS\php.exe run-tests.php ext/standard/tests/strings/crc32.phpt`.

## One-off tests

**ALWAYS** create a test `.php` file and run tests that way, such as `x64\Release_TS\php.exe test_thing.php`.

**NEVER** run tests using `php.exe -r`.

**ALWAYS** clean up one-off tests and any other debug or testing artifacts.