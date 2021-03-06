LaiNES
======

Compact, cycle-accurate NES emulator in ~1000 lines of C++.

![File Browser](./img/files.png)
![Super Mario Bros. 3](./img/super_mario_3.png)
![Kirby's Adventure](./img/kirby.png)

![Controller Settings](./img/ff_settings.png)
![Super Mario Bros.](./img/super_mario.png)
![The Legend of Zelda](./img/zelda.png)

## How are lines counted?
There have been some discussions about how the number of lines has been calculated. The claim of ~1k lines doesn't include the libraries in the boost and nes_apu folders.

```
[andrea@manhattan src]$ rm -rf boost nes_apu Sound_Queue.*
[andrea@manhattan src]$ cloc .
      24 text files.
      24 unique files.                              
       1 file ignored.

github.com/AlDanial/cloc v 1.70  T=0.03 s (780.3 files/s, 63170.2 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
C++                             11            210            110           1163
C/C++ Header                    12             87              7            285
-------------------------------------------------------------------------------
SUM:                            23            297            117           1448
-------------------------------------------------------------------------------
```

## Requirements
LaiNES should run on any Unix system that is compatible with the following tools.
- SConstruct
- C++11 compatible compiler (e.g. clang++)
- SDL2 (including sdl2-ttf and sdl2-image)

## Building and running
Install the dependencies:
```sh
# Debian-based systems:
sudo apt-get install clang scons libsdl2-dev libsdl2-image-dev libsdl2-ttf-dev

# Mac OS X:
brew install scons sdl2 sdl2_image sdl2_ttf
```

Compile and run:
```sh
git clone https://github.com/AndreaOrru/LaiNES && cd LaiNES
scons
./laines
```

## Usage
The emulator comes bundled with a simple GUI to navigate the filesystem and set preferences. Use arrow keys and Enter to operate it. ESC toggles between emulation and menu.

The size of the window and the controls are customizable. LaiNES supports multiple controllers and should work with joysticks as well. The default controls for the first player are as follows:

![Controls](./img/controls.png)

## Compatibility
LaiNES implements the most common mappers, which should be enough for a good percentage of the games:
- NROM
- MMC1 (SxROM)
- MMC3, MMC6 (TxROM)

You can check the compatibility for each ROM in the following list:
http://tuxnes.sourceforge.net/nesmapper.txt

## Technical notes
The 6502 CPU and the PPU are implemented in just 219 and 283 lines of code respectively.
Some clever meta-programming tricks are used to keep the codebase compact.
Here is a good example of how this is achieved:
```c++
/* Cycle emulation.
 *     For each CPU cycle, we call the PPU thrice, because it runs at 3 times the frequency. */
#define T   tick()
inline void tick() { PPU::step(); PPU::step(); PPU::step(); ... }
...

/* Addressing modes.
 *     These are all the possible ways instructions can access memory. */
typedef u16 (*Mode)(void);
inline u16 imm() { return PC++; }
...
inline u16 zpx() { T; return (zp() + X) % 0x100; }
...

/* Fetch parameter.
 *     Get the address of the opcode parameter in a, and the value in p. */
#define G  u16 a = m(); u8 p = rd(a)
...

/* Instruction emulation (LDx where x is in registers {A, X, Y}).
 *     upd_nz, not shown, just updates the CPU flags register. */
template<u8& r, Mode m> void ld() { G; upd_nz(r = p); }
...

/* Execute a CPU instruction.
 *     Opcodes are instantiated with the right template parameters
 *     (i.e. register and/or addressing mode).*/
void exec()
{
    switch (rd(PC++))  // Fetch the opcode.
    {
        // Select the right function to emulate the instruction:
         ...
         case 0xA0: return ld<Y,imm>();  case 0xA1: return ld<A,izx>();
         ...
    }
}
```

## Known issues
Some people are having problems with the audio emulation on Linux.
I couldn't reproduce it, but pull requests to fix it are more than welcome.

## External documentation
- Complete hardware reference: http://problemkaputt.de/everynes.htm
- Tick-by-tick breakdown of 6502 instructions: http://nesdev.com/6502_cpu.txt
- Diagram of the PPU state machine: http://wiki.nesdev.com/w/images/d/d1/Ntsc_timing.png
- blargg's APU sound chip emulator: http://blargg.8bitalley.com/libs/audio.html#Nes_Snd_Emu
