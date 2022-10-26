# Porting Guide

Following is a step by step guide how to add an additional hardware target to an existing project.

## Step 1: Introduce Board layer for existing target

The project needs to be properly structured in order to be easy retargeted to a different hardware. 
This can be achieved by using a Board layer which contains all target specific components and files.

Required steps:
 - Create a target specific subdirectory used for the Board layer (example: `Board/AVH` for target `AVH`).
 - Identify components used in the project that are target specific and their configuration files 
   which are located in the top-level `RTE` directory. Target specific components are typically part of 
   the components with _Cclass_: `Device`, `Board Support` and `CMSIS Driver`. 
 - Move target specific component configuration files (identified in the previous step) from the top-level 
   `RTE` directory to the Board layer (example: `Board/AVH/RTE`).
 - Identify files used in the project that are target specific and move them to the Board layer. 
   >Note: `main.c` and `main.h` are also target specific.
 - Create Board layer description (example: `Board/AVH/Board.clayer.yml`). It should contain all target 
   specific items (components, files, defines, paths, ...) from the project description (`cproject.yml`).
 - Update the project description (`cproject.yml`):
   - Remove all target specific items which have been moved to the Board layer.
   - Add the Board layer:
   ```
    layers:
      - layer: ./Board/AVH/Board.clayer.yml
        for-type:
          - +AVH
   ```

## Step 2: Add Board layer for additional hardware target

Board layer should provide:
 - Startup and system files with linker script (including stack and heap configuration).
 - `main` function which executes the following:
   - initializes the board and device peripherals and interfaces
   - optionally initializes the Event Recorder
   - initializes the RTOS kernel (CMSIS-RTOS2)
   - initializes the application by calling `app_initialize`
   - starts the RTOS kernel

Required interfaces for this `Hello` demo:
 - STDOUT: Standard Output (typically retargeted to UART using `CMSIS driver:USART`)

>Note: Board layer might provide additional interfaces.

Required steps:
 - Create a target specific subdirectory for the Board layer (example: `Board/HW` for target `HW`).
 - Copy target specific component configuration files to the `RTE` subdirectory (example: `Board/HW/RTE`).
 - Copy other target specific files to the Board layer
 - Create Board layer description (example: `Board/HW/Board.clayer.yml`) which contains all target specific 
   items (components, files, defines, paths, ...).

## Step 3: Register the additional hardware target and its Board layer

Update the solution description (`Hello.csolution.yml`):
 - Add the additional hardware target under `target-types`:
   - `type`: target name (example: `HW`)
   - `device`: target device or
   - `board`: target board
- Add the additional required CMSIS-Packs under `packs:`

Update the project description (`Hello.cproject.yml`):
 - add the Board layer for the additional hardware target under `layers:`
   ```
   - layer: ./Board/HW/Board.clayer.yml
     for-type:
      - +HW
   ```
