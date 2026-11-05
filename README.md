# $REPO_NAME

> **NOTE:** Gitea replaces `$REPO_NAME` above when a new repo
> is created from this template (`README.md` must be listed in `.gitea/template`).

---

## Toolchain

| Item     | Required / Supported                    |
|----------|-------------------------------------|
| Build System  | CMake (>= 3.22) + Ninja|
| Compiler   | arm-none-eabi-gcc |
| Config Generator | STM32CubeMX |
| Default MCU Target | STM32F767ZITx (Nucleo-144) |

---

## 1. Install Prerequisites

### macOS

1. Install CMake, Ninja, ARM-GCC Toolchain (arm-none-eabi-gcc), and OpenOCD/ST-Link utilities via Homebrew:
```bash
brew install cmake, ninja, arm-none-eabi-gcc stlink openocd
```

2. Verify tools are on your path:
```bash
cmake --version
arm-none-eabi-gcc --version
```

3. (Optional for GUI pin/clock config) Install STM32CubeMX & STM32CubeIDE for Visual Studio Code Extension in VS Code / Cursor.

### Windows

1. Install STM32CubeCLT from [here](https://www.st.com/en/development-tools/stm32cubeclt.html);

2. Alternatively, install via WinGet (powershell):
```powershell
winget install Kitware.CMake Ninja-build.Ninja Arm.GnuArmEmbeddedToolchain
```

3. Keep project paths short and without spaces, e.g. 
```powershell
C:\dev\arcs\<repo>
```

### Linux

1. Install toolchain:
```bash
sudo apt update
sudo apt install cmake ninja-build gcc-arm-none-eabi libnewlib-arm-none-eabi stlink-tools openocd
```
---

## 2. Build (CLI or VS Code / Cursor)

### Terminal (CLI)

1. From repository root, configure and build using the included CMake presets:
```bash
cmake --preset Debug
cmake --build build/Debug
```
A successful build links **build/Debug/<project_name>.elf** and prints the RAM and FLASH memory usage.

### VS Code / Cursor
1. Open repository root folder in VS Code or Cursor.
2. Ensure the CMake Tools extension is installed.
3. Open command pallete (cmd+shift+p / ctrl+shift+p) -> **CMake: Select Configuration Preset** -> choose **Debug**.
4. Click **Build** in the bottom status bar (or press f7).
---

## 3. Flash & Debug

### Using OpenOCD (macOS / Linux / Windows)
With the board connected via USB (ST-Link):
```bash
openocd -f interface/stlink.cfg -f target/stm32f7x.cfg -c "program build/Debug/STM32-bringup.elf verify reset exit"
```
Adjust **target/stm32f7x.cfg** to e.g. **target/stm32g4x.cfg** if retargeting another board.

### Using st-flash (stlink-tools)
Convert the .elf to a .bin and flash to **0x08000000**:
```bash
arm-none-eabi-objcopy -O binary build/Debug/STM32-bringup.elf build/Debug/firmware.bin
st-flash write build/Debug/firmware.bin 0x08000000
```

### Using VS Code / STM32Cube Extension
Connect the board and use the **Run and Debug** sidebar with STM32CubeCLT (ST-Link GDB Server)

---

## 4. Changing pins, clocks, or target silicon
Hardware initilization is managed by STM32CubeMX via **STM32-bringup.ioc**

1. Open **STM32-bringup.ioc** in STM32CubeMX.
2. Modify pinout, peripheral settings, or clock config tree.
3. Keep Project Manager -> Toolchain / IDE set to CMake.
4. Click Generate Code
5. User Code placed strictly between /* USER CODE BEGIN ... */ and
/* USER CODE END ... */ comments in Core/ is preserved across code generations.
Never write custom code outside of these blocks in generated files.

---

## Project layout

```
CMakeLists.txt                  Project definition
CMakePresets.json               Standard Debug/Release CMake presets
cmake/
    gcc-arm-none-eabi.cmake     ARM cross-compiler toolchain definition
    stm32cubemx/CMakeLists.txt  CubeMX autogen source & HAL linkage
Core/
    Inc/                        Application & HAL Config Headers
    Src/main.c                  Entry point, SystemClock_Config(), and super loop
    Src/stm32f7xx_it.c          Interrupt service routines
    Src/stm32f7xx_hal_msp.c     HAL MSP initilization
Drivers/
    CMSIS/                      ARM Cortex-M & device header files
    STM32F7xx_HAL_Driver/       ST Hardware Abstraction Layer (HAL) drivers
startup_stm32f767xx.s           MCU assembly startup & interrupt vector table
STM32F767XX_FLASH.ld            Linker script defining Flash/RAM layout
STM32-bringup.ioc               STM32CubeMX config file
.gitea/template                 Files gitea fills in when generating a new repo
```

### Config rules

- Commit `.ioc`, `CMakeLists.txt`, `CMakePresets.json`, `cmake/`, `Core/`, `Drivers/`, the `.s` startup file and the `.ld` linker script.
- Never commit `/build`, `.vscode/`, or compiled binaries (`.elf`, `.bin`, `.hex`, `.map`).
- Always keep custom C code inside of /* USER CODE BEGIN ... */ and 
/* USER CODE END ... */ blocks so CubeMX never overwrites your work.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
|  | |
---

## Origin

Template from STM32CubeIDE for Visual Studio Code wizard, selecting board selector,
selecting the STM32F767ZI, creating as a CMake project, and allowing default initilization
of peripherals.

- 