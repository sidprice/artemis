# artemis

A Visual Studio Code development environment for [SparkFun Artemis](https://www.sparkfun.com/artemis) based boards.

This project provides a starting template to develop, build, load, and debug SparkFun Artemis based boards using Visual Studio Code in conjunction with Segger J-Link. This project natively supports the Windows 10 platform and does ***not*** rely on Windows Subsystem for Linux (WSL), MinGW, etc. The batch file, build.bat, which supports building and loading from Windows is modeled after the Makefile template provided in the [SparkFun Board Support Package](https://github.com/sparkfun/SparkFun_Apollo3_AmbiqSuite_BSPs) repository.

While the current configuration supports Segger J-Link, it can be configured to support OpenOCD, etc.

The SparkFun Artemis [forum](https://forum.sparkfun.com/viewforum.php?f=167) is highly recommended for information and support.

## Dependencies

The latest version of each tool/extension should be used unless otherwise noted. During installation, if the option is available to add a given tool to the system PATH environment variable, please do so. This is very important for the GNU Arm Embedded Toolchain.

The following tools are required to make use of this repository:

* [git](<https://git-scm.com/>)
* [GNU Arm Embedded Toolchain](<https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm>)
* [Segger J-Link](<https://www.segger.com/downloads/jlink/>)
* [Visual Studio Code](<https://code.visualstudio.com/>)

The following Visual Studio Code extensions are required:

* [C/C++](<https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools>)
* [Cortex-Debug](<https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug>)

The following Visual Studio Code extensions are recommended:

* [ARM](<https://marketplace.visualstudio.com/items?itemName=dan-c-underwood.arm>)
* [LinkerScript](<https://marketplace.visualstudio.com/items?itemName=ZixuanWang.linkerscript>)

## Getting Started

### Clone this repository and initialize its submodules

```shell
git clone https://github.com/0xcart/artemis.git
cd artemis
git submodule update --init --recursive
```

This repository includes the SparkFun copy of the [AmbiqSuite SDK](https://github.com/sparkfun/AmbiqSuiteSDK) as a git submodule. In turn, this copy of the SDK includes the [SparkFun Board Support Package](https://github.com/sparkfun/SparkFun_Apollo3_AmbiqSuite_BSPs) repository as a git submodule. By using the `--init` and `--recursive` flags both submodules will be initialized and updated correctly.

### Open the project

Launch Visual Studio Code.

Click `File`, `Open Folder...` from the main menu. Select the `artemis` folder and click `Select Folder`. You should now see the following in the `Explorer` view within Visual Studio Code:

![Explorer](doc/image/explorer.jpg)

### Configure the Cortex-Debug Visual Studio Code extension

Highlight the Cortex-Debug extension in the `Extensions` view within Visual Studio Code. Click the `Manage` (gear) icon. Click the `Extension Settings` option in the displayed popup menu. A `Settings` document will be loaded for the extension.

Click on `Edit in settings.json` which will open the `settings.json` document. Update the settings with the following information taking care the paths for the J-Link GDB server and ARM toolchain are correct for your system configuration.

```json
{
  "editor.tabSize": 2,
  "editor.wordWrapColumn": 120,
  "cortex-debug.JLinkGDBServerPath": "C:/Program Files (x86)/SEGGER/JLink/JLinkGDBServerCL.exe",
  "cortex-debug.armToolchainPath": "C:/Program Files (x86)/GNU Arm Embedded Toolchain/9 2020-q2-update/bin",
  "window.zoomLevel": 0,
}
```

### Update the c_cpp_properties.json settings in the .vscode folder

Update the `compilerPath` value to match your system configuration.

```json
{
  "version": 4,
  "configurations": [
    {
      "name": "gcc-arm",
      "includePath": [
        "${workspaceFolder}/src",
        "${workspaceFolder}/AmbiqSuiteSDK/boards_sfe/artemis_thing_plus/bsp",
        "${workspaceFolder}/AmbiqSuiteSDK/utils",
        "${workspaceFolder}/AmbiqSuiteSDK/devices",
        "${workspaceFolder}/AmbiqSuiteSDK/mcu/apollo3",
        "${workspaceFolder}/AmbiqSuiteSDK/CMSIS/AmbiqMicro/Include",
        "${workspaceFolder}/AmbiqSuiteSDK/CMSIS/ARM/Include"
      ],
      "defines": [],
      "compilerPath": "C:/Program Files (x86)/GNU Arm Embedded Toolchain/9 2020-q2-update/bin/arm-none-eabi-gcc.exe",
      "intelliSenseMode": "gcc-arm"
    }
  ]
}
```

### Update the launch.json settings in the .vscode folder

Update the `serverpath` value to match your system configuration. Also update the `cpuFrequency`, `swoFrequency`, and `device` values to match your Artemis board. For most SparkFun Artemis boards the default values should be correct.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Cortex Debug",
      "type": "cortex-debug",
      "request": "launch",
      "cwd": "${workspaceRoot}",
      "servertype": "jlink",
      "serverpath": "C:/Program Files (x86)/SEGGER/JLink/JLinkGDBServerCL.exe",
      "interface": "swd",
      "swoConfig": {
        "enabled": true,
        "cpuFrequency": 48000000,
        "swoFrequency": 2000000,
        "source": "probe",
        "decoders": [
          {
            "type": "console",
            "label": "ITM",
            "port": 0,
            "encoding": "ascii"
          }
        ]
      },
      "device": "AMA3B1KK-KBR",
      "svdFile": "${workspaceRoot}/AmbiqSuiteSDK/pack/SVD/apollo3.svd",
      "executable": "${workspaceRoot}/bin/output_svl.axf",
      "runToMain": true,
    }
  ]
}
```

### Update build.bat

Several parameters in build.bat must be verified and/or updated.

The first set of parameters are associated with your Artemis board. Please update the values for these parameters to match your boards attributes. The default values found in build.bat are for the Artemis Thing Plus. If you're using this board then no modifications should be necessary. The value for `BLD_BOARD` should match one of the folders found in `artemis/AmbiqSuiteSDK/boards_sfe`.

* BLD_BOARD=artemis_thing_plus
* BLD_PART=APOLLO3
* BLD_MCU=apollo3
* BLD_CPU=cortex-m4
* BLD_FPU=fpv4-sp-d16
* BLD_FABI=hard

The second set of parameters are associated with communications to your board. Please update these values as necessary. A baud rate of 921600bps is correct for the SparkFun Variable Loader (SVL).

* BLD_BAUD=921600
* BLD_PORT=COM4

In the future, when you're ready to add additional source files, libraries, etc., you'll need to update the following parameters:

* BLD_INCLUDE=
* BLD_SOURCE=
* BLD_LIBRARY=

## Build, Load, Debug, and Clean

### Build

Press `F1`. This will display a popup menu listing all commands. Select `Tasks: Run Build Task` from the list of commands to display the build task. Alternatively you can press `Ctrl + Shift + B` to reach the same build task.

![Build](doc/image/build.jpg)

To build the project select `svl binary`. You should see the following printed to the console:

```shell
Making    'bin' directory
Compiling 'src/main.c'
Compiling 'AmbiqSuiteSDK/boards_sfe/common/tools_sfe/templates/startup_gcc.c'
Compiling 'AmbiqSuiteSDK/devices/am_devices_led.c'
Compiling 'AmbiqSuiteSDK/utils/am_util_delay.c'
Compiling 'AmbiqSuiteSDK/utils/am_util_stdio.c'
Linking   'bin/output_svl.axf' with script 'AmbiqSuiteSDK/boards_sfe/common/tools_sfe/templates/asb_svl_linker.ld'
Creating  'bin/output_svl.lst'
Creating  'bin/output_svl.bin'

=== BUILD SUCCESSFUL ===
```

A `bin` directory is created containing the following list of output files:

* am_devices_led.o
* am_util_delay.o
* am_util_stdio.o
* main.o
* output_svl.axf
* output_svl.bin
* output_svl.lst
* output_svl.map
* startup_gcc.o

### Load

To load `output_svl.bin` onto your Artemis board press `Ctrl + Shift + B` and select `bootload`. This executes the `artemis_svl.exe` process provided by SparkFun and loads the binary via the SparkFun Variable Loader (SVL). You should see the following (or similar) printed to the console:

```shell
Artemis SVL Bootloader

phase:  setup
        cleared startup blip
        Got SVL Bootloader Version: 3
        Sending 'enter bootloader' command

phase:  bootload
        have 4484 bytes to send in 3 frames
        sending frame #1, length: 2048
        sending frame #2, length: 2048
        sending frame #3, length: 388

        Upload complete

        Nominal bootload bps: 36222.77
```

### Debug

To begin debugging press `F5`. This will launch the Segger J-Link GDB server. Your application will stop at a breakpoint in `main()`:

![main()](doc/image/main.jpg)

You should see the following printed to the `DEBUG CONSOLE` tab at the bottom of Visual Studio Code:

```shell
Please check OUTPUT tab (Adapter Output) for output from C:/Program Files (x86)/SEGGER/JLink/JLinkGDBServerCL.exe
Launching server: "C:/Program Files (x86)/SEGGER/JLink/JLinkGDBServerCL.exe" "-if" "swd" "-port" "50000" "-swoport" "50001" "-telnetport" "50002" "-device" "AMA3B1KK-KBR"
Launching GDB: "C:\Program Files (x86)\GNU Arm Embedded Toolchain\9 2020-q2-update\bin\arm-none-eabi-gdb.exe" "-q" "--interpreter=mi2"
undefinedC:\Program Files (x86)\GNU Arm Embedded Toolchain\9 2020-q2-update\bin\arm-none-eabi-gdb.exe: warning: Couldn't determine a path for the index cache directory.
Reading symbols from C:\Solutions\artemis/bin/output_svl.axf...
main () at src/main.c:47
47      am_hal_clkgen_control(AM_HAL_CLKGEN_CONTROL_SYSCLK_MAX, 0);
Not implemented stop reason (assuming exception): undefined
Resetting target
Resetting target
SWO enabled successfully.

Temporary breakpoint 1, main () at src/main.c:47
47      am_hal_clkgen_control(AM_HAL_CLKGEN_CONTROL_SYSCLK_MAX, 0);
```

At the top of Visual Studio Code you'll find a popup dialog with Debug buttons that include `Continue`, `Step Over`, `Step Into`, `Step Out`, etc.:

![Debug](doc/image/debug.jpg)

Click the `Continue` button.

At the bottom of Visual Studio Code you'll find several output consoles:

![Output](doc/image/output.jpg)

Click on the `OUTPUT` tab. By default, `Adapter Output` is selected in the drop-down menu on the right:

![Adapter](doc/image/adapter.jpg)

You should see the following (or similar) printed to the `OUTPUT` tab:

```shell
...
Reading register (d11 = 0x       0)
Reading register (d12 = 0x       0)
Reading register (d13 = 0x       0)
Reading register (d14 = 0x       0)
Reading register (d15 = 0x       0)
```

In the drop down menu on the right (the selection is currently `Adapter Output`) select `SWO: ITM [port: 0, type: console]`:

![SWO](doc/image/swo.jpg)

You should see the following (or similar) printed to the `OUTPUT` tab:

```shell
[2020-08-27T19:45:44.753Z]   Test artemis SWO
```

This verifies functions like `am_util_stdio_printf()` are working properly (see src/main.c:31).

### Clean

To clean the project, again bring up the build task by pressing `Ctrl + Shift + B` and select `clean`. You should see the following printed to the console:

```shell
Removing 'bin' directory
```
