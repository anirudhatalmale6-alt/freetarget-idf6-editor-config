ESP32 master - r2 update (3 files only)
=======================================

The ESP32.zip you sent me is the r1 package. The r2 package I sent afterwards
changed exactly three files. This zip contains only those three, so you can
drop them in without disturbing any edits you have made since.

Copy over your tree, keeping the same paths:

    .vscode/c_cpp_properties.json
    .vscode/settings.json
    main/timer.c


1. .vscode/settings.json
------------------------
This is the file that causes the squiggly lines.

  "C_Cpp.intelliSenseEngine"    was "Tag Parser"   -> now "default"

      Tag Parser ignores compile_commands.json completely. It guesses at
      symbols from the browse path, so it flags real code as unknown. The
      default engine reads the flags the compiler actually used.

  "C_Cpp.default.compilerPath"  was the 5.3.1 GCC 11.2 xtensa toolchain
                                under c:\Users\allan\esp\esp-idf\tools\...
                                -> now empty, so it is taken from
                                compile_commands.json instead.

  "idf.currentSetup"            was "C:\Users\allan\esp\v5.3.1\esp-idf"
                                -> now "C:\esp\v6.0.2\esp-idf"

      ** Check this one. ** I guessed at the path. Set it to wherever you
      actually installed 6.0.2. If the ESP-IDF extension is still pointed at
      5.3.1, then building from inside VS Code builds this project with 5.3.1,
      and it will not compile - 6.0 moved the headers and deleted the legacy
      timer driver.


2. .vscode/c_cpp_properties.json
--------------------------------
Adds:

    "compileCommands": "${workspaceFolder}/build/compile_commands.json"

and drops the espAdfPath entries, which pointed at an ADF install you do not
have. The build writes compile_commands.json with the exact include paths and
defines used for every source file, so IntelliSense stops guessing.

After copying both files in:

    idf.py build                          (once, to generate the file)
    Ctrl+Shift+P -> C/C++: Reset IntelliSense Database
    Ctrl+Shift+P -> Developer: Reload Window


3. main/timer.c
---------------
The 1 ms tick correction you asked for. r1 kept the original 4960 counts
(992 us). r2 makes it 5000 counts, a true 1000 us, and derives it from the
counter rate so the two cannot drift apart:

    #define TIMER_RESOLUTION_HZ (5 * 1000 * 1000)
    #define ONE_MS              (TIMER_RESOLUTION_HZ / 1000)

The reasoning for why that is safe on production firmware is in the r2
MIGRATION-IDF6.md, under "The 1 ms tick - ported to gptimer, and corrected".


One note on analog_io.c
-----------------------
The only thing I changed in that file is two include lines:

    #include "gpio_types.h"       ->   #include "hal/gpio_types.h"
    #include "adc_types.h"        ->   #include "hal/adc_types.h"

6.0 split the hal component up, and the unqualified names are no longer on the
include path. If IntelliSense red-underlines those two lines and they get
"corrected" back to the short form, the file stops compiling. Everything else
in analog_io.c is byte for byte your original.
