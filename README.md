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
### Embedded bootloader types.
