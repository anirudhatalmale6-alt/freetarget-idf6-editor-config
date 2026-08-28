Squiggly lines / "include file not found in browse path"
========================================================

That exact wording is the giveaway. "browse path" is the Tag Parser's
vocabulary - it is the fallback symbol database, not the real IntelliSense
engine. The real engine talks about "cannot open source file". So if you are
still being told something is not on the browse path, the Tag Parser is still
the thing answering, and none of the include settings are being read at all.

The compile is a completely separate mechanism and is unaffected by any of
this, which is why it builds fine.


Do these in order
-----------------

STEP 1 - one find and replace in the attached c_cpp_properties.json

The file has your real include paths baked in - all 150 of them, pulled
straight out of the compile_commands.json that your project generates, so
there is no guessing left. But 141 of them sit under the IDF install, and I
do not know where yours is.

Open .vscode/c_cpp_properties.json, find and replace:

        C:/esp/v6.0.2/esp-idf          ->   your actual 6.0.2 path

Forward slashes, and no trailing slash. If your install is at
C:\Users\allan\esp\v6.0.2\esp-idf then write C:/Users/allan/esp/v6.0.2/esp-idf.

While you are in there, check the compilerPath line near the top. I have it as

        C:/Users/allan/.espressif/tools/xtensa-esp-elf/esp-15.2.0_20251204/xtensa-esp-elf/bin/xtensa-esp32s3-elf-gcc.exe

The toolchain moved between 5.3.1 and 6.0.2. It used to be a separate
xtensa-esp32s3-elf toolchain on GCC 11.2 - that is what your old setting
pointed at. 6.0 uses the unified xtensa-esp-elf on GCC 15.2. Look in
%USERPROFILE%\.espressif\tools\xtensa-esp-elf\ and use whatever version folder
is actually there.


STEP 2 - check the setting is not being overridden

The attached settings.json sets "C_Cpp.intelliSenseEngine": "default". A
workspace setting normally wins, but check you do not also have it pinned in
your user settings:

    Ctrl+Shift+P -> Preferences: Open User Settings (JSON)

If "C_Cpp.intelliSenseEngine" appears in there set to "Tag Parser", delete
that line. Same for "C_Cpp.default.compilerPath" if it is pointing anywhere
near the old 5.3.1 toolchain.


STEP 3 - check the active configuration

Bottom right of the VS Code status bar shows the C/C++ configuration in use.
It must say ESP-IDF. If it says Win32, or Linux, or anything else, click it
and pick ESP-IDF. None of these settings apply to a configuration you are not
actually on.


STEP 4 - the file has to exist

    idf.py build

compile_commands.json is written by the build into build/. Two things have to
line up for it to be found:

  - The folder you have open in VS Code has to be the ESP32 folder itself, the
    one with CMakeLists.txt in it. If you have opened a parent folder, then
    ${workspaceFolder}/build is the wrong place and the setting silently does
    nothing.

  - build/config/sdkconfig.h has to exist, which it will after a build. Every
    CONFIG_ symbol in your code comes from that one header. Without it you get
    a squiggle on more or less every line that touches configuration.


STEP 5 - force it to re-read

Nothing above takes effect until you do both of these:

    Ctrl+Shift+P -> C/C++: Reset IntelliSense Database
    Ctrl+Shift+P -> Developer: Reload Window


If it is still not clean
------------------------

Run this and send me the output:

    Ctrl+Shift+P -> C/C++: Log Diagnostics

It prints the configuration actually in effect for the file you have open -
which config was selected, whether compile_commands.json was found and used,
the include paths in force, and the compiler it thinks it is talking to. It
is about a page. That will tell us in one go which of the five steps above is
not landing, instead of us going round again.

One thing to look for yourself in that output: if it names a configuration
provider - something like espressif.esp-idf-extension - then the ESP-IDF
extension is supplying the configuration itself and is overriding this file
entirely. Say so if you see it and I will give you the version for that path
instead.
