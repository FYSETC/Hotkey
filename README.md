# Hotkey

## Introduction
Elevate your 3D printing experience with our latest innovation, the HotKey Macro Buttons designed exclusively for the discerning 3D printing enthusiast.

**Key Features:**
- Dedicated MCU: Built on the robust and reliable RP2040 platform, our HotKey system guarantees a seamless integration and flawless performance.
- 12 Linear Red Switched Macro Buttons: Customize your 3D printer like never before. With 12 programmable macro buttons, execute your frequently used commands with a single press. Whether you’re toggling between settings, setting up a new print, or any other task, speed and convenience are at your fingertips.
- Designed with Voron in Mind: While our HotKey Macro Buttons can enhance any 3D printing setup, they were meticulously designed with the Voron Trident and Voron 2.4 models as the primary inspiration. This ensures an impeccable fit and intuitive usage for Voron users.
- Seamless Integration: Add these buttons to your printer's skirt without any hassle. Our design philosophy prioritizes both aesthetics and functionality, ensuring your 3D printer not only gets a boost in efficiency but also looks sleeker.


## Firmware
### Katapult (Formally CAN Boot)
Katapult is used to allow easy upgrading of your formware down the line, and is not just soley for CAN based devices. 
#### Setup Katapult
If you already have Katapult installed, you can skip this step. 

1. Connect to your Klipper host computer via ssh.
2. Clone the Katapult firmware to your Klipper Host
```bash
cd ~/
git clone https://github.com/Arksine/katapult
```
3. Within either Fluidd or Mainsail UI, Edit your Moonraker.conf and add the following at the bottom to allow Moonraker to manage your Katapult updates. 
```yaml
[update_manager Katapult]
type: git_repo
path: ~/katapult
origin: https://github.com/Arksine/katapult.git
is_system_service: False
```
#### Compile Katapult Firmware
1. Navigate into the Katapult Directory
```bash
cd ~/katapult
```
2. Run make clean 
```bash
make clean KCONFIG_CONFIG=config.hotkey
```
3. Open menuconfig 
```bash
make menuconfig KCONFIG_CONFIG=config.hotkey
```
4. Set the following settings
    - Micro-controller Architecture (Raspberry Pi RP2040)
    - Flash Chip (W25Q080 with CLKDIV 2)
    - Build Katapult deployment application (Do not build)
    - Communication interface (USB)
    - () GPIO pins to set on bootloader entry
    - [*] Support bootloader entry on rapid double clip of reset button
    - [_] Enable bootloader entry on button (or gpio) state
    - [_] Enable Status LED
  
![image](https://github.com/FYSETC/Hotkey/assets/5789676/60e889fd-c6e1-4ce9-8491-8a0437bb2e50)

5. Quit and save the configuration
6. Run the make command to compile the firmware
```bash
   make KCONFIG_CONFIG=config.hotkey -j4
```
7. You should now have a katapult.uf2 file at ~/katapult/out/

#### Burn Katapult Firmware to the Hotkey Pad
1. Use a USB-C data cable to connect the Hotkey Pad board to the klipper host.
2. Press the boot button while inserting the USB Cable to put the Hotkey Pad into boot mode.
3. Run lsusb to see if the connection is successful.
```bash
  lsusb
```
![image](https://github.com/FYSETC/Hotkey/assets/5789676/e6b0f349-866d-4f64-adb9-d933bc5cfa64)

You should see a device with the ID `2e8a:0003 Raspberry Pi RP2 Boot`, if that doesn't show, please start at step 1 again. 

4. Flash the Hotkey Pad
```bash
sudo make KCONFIG_CONFIG=config.hotkey flash FLASH_DEVICE=2e8a:0003
```
- You will be prompted to enter your password.
- This will flash the Katapult bootloader image and restart the Hotkey Pad
5. Once complete, reboot your host computer and move onto flashing Klipper


### Klipper

#### Compile Klipper firmware for the Hotkey Pad
1. Connect to your Klipper host computer via ssh. 
2. cd to the Klipper directory 
```bash
cd ~/klipper
```
3. Run make clean 
```bash
make clean KCONFIG_CONFIG=config.hotkey
```
4. Open menuconfig 
```bash
make menuconfig KCONFIG_CONFIG=config.hotkey
```
5. Set the following settings
    - [*] Enable extra low-level configuration options
    - Micro-controller Architecture (Raspberry Pi RP2040)
    - Bootloader offset (16KiB bootloader)
    - Communication interface (USB)
    - ( ) GPIO pins to set at micro-controller startup (NEW)
![image](https://github.com/FYSETC/Hotkey/assets/5789676/08a39397-c703-4173-890a-9375d9d1e3e9)

6. Quit (press q) and save the configuration
7. Run Make to compile the firmware
```bash
make KCONFIG_CONFIG=config.hotkey -j4
```
#### Flash Klipper USB firmware with Katapult over USB
1. Find the Serial ID (hint - it will likely say rp2040 in the name. If you have multiple rp2040 based devices, check your printer.cfg and see which ones you are already using).
```bash
ls /dev/serial/by-id
```
![image](https://github.com/FYSETC/Hotkey/assets/5789676/b8880321-a4b2-4746-8df8-d8d213e296f9)

2. Copy the ID containing ‘rp2040’ and make a note of it for use in your printer.cfg.
3. Run the make flash command to flash the firmware
```bash
make KCONFIG_CONFIG=config.hotkey flash FLASH_DEVICE= {Your serial ID here }
```
   **Example**
```bash
make KCONFIG_CONFIG=config.hotkey flash FLASH_DEVICE=/dev/serial/by-id/usb-katapult_rp2040_E6625C4893378533-if00
```
![image](https://github.com/FYSETC/Hotkey/assets/5789676/5edbfa4f-8526-44f4-b4c7-69ed31639149)

 4. Your Hotkey Pad should now have Klipper Firmware installed and be ready to use. You can check it's worked by running another
```bash
ls /dev/serial/by-id
```

## Configuration 

### How to use all this stuff:
1.  Copy this .cfg file into your Klipper config directory and then add `[include Hotkey.cfg]` to the top of your printer.cfg in order to register the MCU, LEDs and macros with Klipper.
2.  If you installed the Stealthburner as well and want to get the "STATUS_ERROR" state transmitted to the hotkey button PCB, you have to replace the "STATUS_ERROR" Macro in your stealthburner_leds.cfg with this:
```bash
[gcode_macro status_error]
gcode:
    STATUS_ERROR_HOTKEY
    _set_sb_leds_by_name leds="logo" color="error" transmit=0
    _set_sb_leds_by_name leds="nozzle" color="error" transmit=1
```
3.  Connect the PCB via USB to your RaspberryPi and SSH to your Pi
4.  Get the Serial ID of the PCB with "ls /dev/serial/by-id/*". Usualy it´s like usb-Klipper_rp2040_xxx
5.  Replace the serial under the [mcu hotkey] section with your id
6.  Save your config and restart Klipper.

