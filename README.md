# IAR + VSCode + CMake on Windows

This mini-guide provides the essentials for quickly setting up a CMake project targetting an Arm Cortex-M4, built with IAR, in Visual Studio Code.

> __Warning__ 
> The information provided in this repository is subject to change without notice and does not represent a commitment on any part of IAR. While the information contained herein might be useful as reference, IAR assumes no responsibility for any errors or omissions.

## Required software
Below you will find the software and their versions used in this guide. Newer versions might work with little or no modification(s).

| Software | Version | Link |
| - | - | - |
| IAR Embedded Workbench | 9.40.1 | [link](https://www.iar.com/products/architectures/arm/iar-embedded-workbench-for-arm/iar-embedded-workbench-for-arm---free-trial-version/)
| Microsoft Visual Studio Code | 1.82.0 | [link](https://code.visualstudio.com)
| [IAR Debug Extension](https://github.com/IARSystems/iar-vsc-debug) | 1.30.2 | [link](https://marketplace.visualstudio.com/items?itemName=iarsystems.iar-debug)
| Microsoft C/C++ Extension Pack | 1.3.0 | [link](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)
| Microsoft CMake Tools Extension | 1.15.31 | [link](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
| CMake | 3.27.4 | [link](https://github.com/Kitware/CMake/releases/download/v3.27.4/cmake-3.27.4-windows-x86_64.msi)

## Procedure
This guide assumes all the required software installed with their defaults and ready to use.

- In Visual Studio Code, select: `File` → `Open Folder...` (<kbd>Ctrl</kbd>+<kbd>K</kbd>, <kbd>Ctrl</kbd>+<kbd>O</kbd>).
> __Note__
> VSCode will ask you if you _trust the authors_ of the files in the opened folder.

- Create a new empty folder. (e.g., "`hello`"). This folder will be referred to as `<proj-dir>` from now on.
  
- Create a `<proj-dir>/main.c` source file with the following content:
```c
#include <intrinsics.h>
#include <stdio.h>
#include <stdint.h>

__root uint_fast8_t counter = 0;

void main() {
   while (counter < 10) {
     printf("Hello world! %u\n", counter);
     ++counter;
   }
   for(;;) {
      ++counter;
   }
}
```

### Creating the CMakeLists
- Create a `<proj-dir>/CMakeLists.txt` file with the following content:
```cmake
cmake_minimum_required(VERSION 3.27)

# Set the project name and its required languages (ASM, C and/or CXX)
project(example LANGUAGES C)

# Add an executable named "hello"
add_executable(hello)

# Add "hello" source file(s)
target_sources(hello PRIVATE main.c)

# Set the compiler options for "hello"
target_compile_options(hello PRIVATE
  --cpu Cortex-M4
  --dlib_config normal
  --debug
  -On
  -e
)

# Set the linker options for "hello"
target_link_options(hello PRIVATE
  --semihosting
  --map .
  --config "${TOOLKIT_DIR}/config/linker/ST/stm32f407xG.icf"
)
```

> __Note__
> For more information on using CMake with the IAR compiler, refer to the [cmake-tutorial](https://github.com/iarsystems/cmake-tutorial).

### Configuring the CMake Tools extension
* Create a `<proj-dir>/.vscode/cmake-kits.json` file with the following content (adjusting the paths as needed):
```json
[
  {
    "name": "IAR",
    "compilers": {
      "C":   "C:/Program Files/IAR Systems/Embedded Workbench 9.2/arm/bin/iccarm.exe"
    },
    "cmakeSettings": {
      "CMAKE_BUILD_TYPE": "Debug",
      "TOOLKIT_DIR": "C:/Program Files/IAR Systems/Embedded Workbench 9.2/arm",
      "CMAKE_MAKE_PROGRAM": "C:/Program Files/IAR Systems/Embedded Workbench 9.2/common/bin/ninja.exe"
    }
  }
]
```
> __Note__
> * CMake Tools will prefer [Ninja](https://ninja-build.org/) if it is present, unless configured otherwise.
> * If you change the active kit while a project is configured, the project configuration will be re-generated with the chosen kit.

### Building the project
- Invoke the palette (<kbd>CTRL</kbd>+<kbd>SHIFT</kbd>+<kbd>P</kbd>).
   - Perform __CMake: Configure__.
   - Select __IAR__ from the drop-down list.
- Invoke the palette 
   - Perform __CMake: Build__.

__Output:__
```
[main] Building folder: hello 
[build] Starting build
[proc] Executing command: "C:\Program Files\CMake\bin\cmake.EXE" --build c:/path/to/hello/build --config Debug --target all --
[build] [1/2  50% :: 0.047] Building C object CMakeFiles\hello.dir\main.o
[build] [2/2 100% :: 0.096] Linking C executable hello.exe
[driver] Build completed: 00:00:00.154
[build] Build finished with exit code 0
```

Happy building!


## Enabling Intellisense for the IAR C/C++ Compiler for Arm in Visual Studio Code 
In order to get accurate results, Intellisense needs: 
- the compiler's internal macros
- the compiler's keywords

For consistency, we will create two header files containing such information.
### `%APPDATA%/Code/IAR/iccarm_predef.h`
This header file can be generated with the following command:
```
mkdir %APPDATA%/Code/IAR
"C:/Program Files/IAR Systems/Embedded Workbench 9.2/arm/bin/iccarm.exe" --NCG . --c++ --predef_macros  %APPDATA%/Code/IAR/iccarm_predef.h
```

### `%APPDATA%/Code/IAR/iccarm_keywords.h`
Create this header file with the following contents:
```cpp
#define __absolute
#define __arm
#define __big_endian 
#define __cmse_nonsecure_call
#define __cmse_nonsecure_entry
#define __exception
#define __fiq
#define __interwork
#define __intrinsic
#define __irq
#define __little_endian
#define __naked
#define __no_alloc
#define __no_alloc16
#define __no_alloc_str
#define __no_alloc_str16
#define __nested
#define __no_init
#define __noreturn
#define __nounwind
#define __packed
#define __pcrel
#define __ramfunc
#define __root
#define __ro_placement
#define __sbrel
#define __stackless
#define __svc
#define __swi
#define __task
#define __thumb
#define __weak
```
> __Note__ Keywords available as per IAR C/C++ Compiler for Arm 9.40.1

### `%APPDATA%/Code/User/settings.json`
The following JSON file configures Microsoft Intellisense to work with the IAR C/C++ Compiler for Arm:
```json
{
    // Intellisense settings
    "C_Cpp.default.compilerPath": "",
    "C_Cpp.default.cStandard": "c17",
    "C_Cpp.default.cppStandard": "c++17",
    "C_Cpp.default.intelliSenseMode": "clang-arm",
    "C_Cpp.default.configurationProvider": "ms-vscode.cmake-tools",
    "C_Cpp.default.mergeConfigurations": true,
    "C_Cpp.default.systemIncludePath": [
        "C:/Program Files/IAR Systems/Embedded Workbench 9.2/arm/inc/c",
        "C:/Program Files/IAR Systems/Embedded Workbench 9.2/arm/inc/c/aarch32"
    ],
    "C_Cpp.default.forcedInclude": [
        "${env:APPDATA}/Code/IAR/iccarm_predef.h",
        "${env:APPDATA}/Code/IAR/iccarm_keywords.h"
    ],
    // CMake settings
    "cmake.enabledOutputParsers": [
        "iar",
        "cmake"
    ]
}
```

## Debugging the project
Now we need to setup the launch configuration for the IAR C-SPY Debugger at `<proj-dir>/.vscode/launch.json`:
```json
{
    "version": "0.2.0",
    "configurations": [
      {
        "type": "cspy",
        "name": "C-SPY: Simulator",
        "request": "launch",
        "program": "${command:cmake.launchTargetPath}",
        "stopOnEntry": true,
        "workbenchPath": "C:/Program Files/IAR Systems/Embedded Workbench 9.2",
        "target": "arm",
        "driver": "Simulator",
        "driverOptions": [
          "--endian=little",
          "--cpu=Cortex-M4",
          "--fpu=None",
          "--semihosting"
        ]
      }
    ]
  }
```

Once the executable was built:
- In `main.c`, click on the left curb of the line which increments the counter (`++counter;`) to set a breakpoint.
- Go to __Run__ → __Start Debugging__ (<kbd>F5</kbd>) to start the debugging session.

- ![image](https://github.com/IARSystems/iar-vscode-cmake/assets/54443595/20c0eb3b-cf83-4a86-9ed0-695b6a8728c5)


Happy debugging!
