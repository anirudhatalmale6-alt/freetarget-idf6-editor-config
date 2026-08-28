# freeETarget - ESP-IDF 6.0 editor config

Same contents as the two zips I sent on Freelancer, in case those did not come
through. Files here match them exactly.

## What is in here

| Path | What it is |
|---|---|
| `.vscode/c_cpp_properties.json` | The IntelliSense fix. 150 include paths written out explicitly. |
| `.vscode/settings.json` | Switches the IntelliSense engine off Tag Parser, clears the stale 5.3.1 compiler path. |
| `main/timer.c` | The 1 ms tick correction from r2 (5000 counts, a true millisecond). |
| `README-intellisense.txt` | Step by step for the squiggly lines. **Read this one first.** |
| `README-r2-update.txt` | What changed between r1 and r2. |

## Two edits you need to make

Both are in `.vscode/c_cpp_properties.json`.

**1.** Find and replace the IDF path. 141 of the include paths sit under your
IDF install and I do not know where yours is:

```
C:/esp/v6.0.2/esp-idf     ->     your actual 6.0.2 path
```

Forward slashes, no trailing slash.

**2.** Check `compilerPath` near the top of the file:

```
C:/Users/allan/.espressif/tools/xtensa-esp-elf/esp-15.2.0_20251204/xtensa-esp-elf/bin/xtensa-esp32s3-elf-gcc.exe
```

The toolchain changed between 5.3.1 and 6.0. The separate per-chip
`xtensa-esp32s3-elf` on GCC 11.2 that your old setting pointed at no longer
exists; 6.0 uses a unified `xtensa-esp-elf` on GCC 15.2. Look in
`%USERPROFILE%\.espressif\tools\xtensa-esp-elf\` and use whichever version
folder is actually there.

## Then

```
idf.py build
```

then in VS Code:

```
Ctrl+Shift+P  ->  C/C++: Reset IntelliSense Database
Ctrl+Shift+P  ->  Developer: Reload Window
```

Both. It will not clear until you do both.

## If it is still not clean

```
Ctrl+Shift+P  ->  C/C++: Log Diagnostics
```

Send me that output. It prints the configuration actually in effect - which
config got selected, whether `compile_commands.json` was found and used, the
include paths in force, and the compiler it thinks it is talking to.
