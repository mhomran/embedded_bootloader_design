## Acknowledgment
<a href="https://youtube.com/playlist?list=PLl3yF2kjT6AzLxhOuCEBY-8RzOIB1sfnN">Embedded bootloader design YouTube playlist</a>

## Bootloader fundementals

### What the bootloader.
- Bootloader is a code provides a method to program and update MCU flash without using specific hardware programming tool like JTAG.
- Bootloader may update the MCU flash remotely (most common) or locally. 
- Bootloader code stored in ROM or Protected flash to avoid accidental overwriting.

### Why the bootloader.
- Bootloader needed to upgrade product's system when bug are found.
- To upgrade the product's system when the new feature is added.
- Assuming that the product is deployed and we published 30 thousand devices.

### Booting conditions.
- There exist two modes: the application and the boot modes.
- To switch from the application mode to boot mode:
  - The manual conditions: push button signal for 3 seconds, or USB signal etc.
  - The automatic conditions: receiving specific packet for update new version or updating using communication interfaces.


### System components without bootloader.
<figure>
  <img src="img/system_without_bootloader.png">
  <figcaption>System components without bootloader</figcaption>
</figure>


### System components with bootloader.
<figure>
  <img src="img/system_with_bootloader.png">
  <figcaption>System components with bootloader</figcaption>
</figure>

- Branching code (boot manager)
    - decides to go for bootloader code or application code. 
    - The code can have a seperate section in memory or share the same section of the bootloader.
    - It checks on non-volatile memory location. If location x has 1, then jump to the bootloader code, otherwise jump to application code.
    - Checking for timeout (checking for the non-volatile memory).
- Bootloader code verify and update.
- Application code
    - Receiving new image
    - Perform main functionality.

### Bootloader general sequence.
<figure>
  <img src="img/bootloader_sequence.png">
  <figcaption>bootloader sequence</figcaption>
</figure>

- Note: it's not prefered to use flash memory in branching code to store the check flag, because you can't write just one word, you can write an entire sector which will lead to wasted space. Instead, you can use EEPROM.

- Application checking
    1. CRC checking
    1. Flash erasing checking
    1. Image verification flag
    1. etc.

### Embedded bootloader types (providing)

- vendor bootloader
- custom bootloader
- both vendor and custom bootloader

- Note: In case of existing vendor bootloader and you want to add your custom bootloader, it will be added in flash. And you should adjust the <code>Vector Table Offset Register</code> to point to the new application program which starts with the vector table.

- There's pins for determining the booting mode (Vendor, flash, SRAM), typically they are two pins. The MCU remap the physical address of the chosen memory to the right place, for example if the flash mode is used the MCU will remap the start of the custom bootloader in the flash memory to address 0x0 for arm-cortex-m CPUs.

## bootloader design considerations.

### Why application code receives the image and not the bootloader ?!

- Problem: Suppose your bootloader receives the image using Wireless communication. So, there should be a wireless stack in the bootloader section. At the same time, the application code uses wireless, so it also needs wireless stack. In case of receiving the image in the bootloader section, there will be <strong>duplication</strong>.
- Solution: make the application code receive the image, so just one wireless stack exist in the flash memory. At this case the bootloader is called bootstrap.

<hr>

- Problem: what if the application code is corrupted for some reason ?
- Solution: make the communication stack (e.g. wireless stack) a shared library within a specific section.

<hr>

- Problem: what if the application code doesn't exist yet ?
- Solution: the bootloader is the one who's responsible for receiving the image using the shared communication stack.

### building image strcture
1. Image header
    - Image CRC
    - Image size
    - Image status (Active-Inactive-Backup)
    - Image ID
    - Image entry point address 
1. Image content
    - Image interrupt vector table
    - Image functionality

### Image verification
- Bootloader should has image verification to prevent untrusted program to run.
- Verification on new image may happen for just first time or with each reset cycle, but this is overhead on system.
- Instead, we can verify image for first time then set image verification flag.
- New image should be verified before the programming is attempted.
- HW/SW Encryption and CRC modules is highly recommended for image verification.

### Security
- <strong>System</strong> should offer security ways to prevent untrusted server to update firmware code.
- <strong>Password</strong> can be used as authentication for locking bootloader itself from updating and flash operation.
- HW/SW <strong>encryption</strong> is highly recommended for update firmware.
- <strong>Bootloader</strong> should be abstracted like Z-Area in TI vendor to keep it hidden from app side.
- <strong>lock JTAG</strong> pins before deployment to prevent any attacks that may happen on bootloader code.

### Image storing
- Image can be saved in free space of internal flash or in external memory.
- Images in external flash should be kept encrypted.
- Images in internal should be decrypted.
- Image can be located at absolute or relevant location.
- Relevant location means independent position code (IPC).

### Failure receiving and recovery

- In case of external memory supported.
- Active, inactive slots used for recovery operation.
- Server host should be indicated for either failure or recovery operations.

### Invocation rules before jumping from the bootloader code to application code
- disable (maskable) interrupts
- Deinit HW modules 
- Disable systick timer
- Relocate interrupt vector table to avoid undefined behavior in runtime.
- Set shared non-volatile flag
- Initialize MSP stack pointer 

### Invocation rules after jumping from the bootloader code to application code
- Clear RAM.
- Init (.data) sections 
- Reinitialize HW modules.
- Enable interrupts.

### ARM architecture hints
<figure>
  <img src="img/arm_system_modes.png">
  <figcaption>ARM System Modes</figcaption>
</figure>

### Interrupt vector table

- It's prefered that the application and the bootloader has their own interrupt vector table.

## Simple bootloader

### Technical requirements
- Bootloader should update FW Manually using "push button holding signal"
- By default, system should be in boot mode if there's no application or corruption cases.
- Push button holding signal in boot mode should do nothing.
- when system in boot mode, "boot led" should be blinking.
- When system is in App mode, "App led" should be blinking.
- Switching from boot mode to app mode should be protected if application exist.
- Each program should have its ows vector table and bootloader vector table is located at 0 address by default.
- Boot manager should be as a part from bootloader memory.

### System design
<figure>
  <img src="img/simple_bootloader_system_design.png">
  <figcaption>System Design</figcaption>
</figure>
<figure>
  <img src="img/simple_bootloader_memory_design.png">
  <figcaption>Memory Design</figcaption>
</figure>

### Hints:
- Sometimes the watchdog timer can't be disabled except by a reset. So, you may need to initialize it after the bootmanager, so you already determined whether you'll enter the bootloader or the application. 

- To debug the application through debugging the bootloader, you can use this gdb command to load the symbols from app .elf file
  ```
  add-symbol-file app.elf
  ```


