# System Info
## Memory Layout
    - 4096 memory locations (All 1 byte ie 8bits long)
        - bottom 512 bytes used for the interpreter
        - top 256 bytes used by the display
        - 96 bytes below also used for internal use and call stack
## Registers
    - 16 registers, V0 to VF (8 bits long)
        - VF used as a flag for addition carry, subtration no borrow, collision detection
    - I register is the address register and used to for memory operations (12bits long)
## Timers
    - Two both count down for 60hz (8bits long)
        - Delay Timer ie used for game stuff
        - Sound timer beeps when non zero
## Stack
    - Used to mem addr of subroutines. 
    - 48bytes which can hold 12 levels of nesting
## Graphics
    - MonoChrome display
    - 64 x 32 screen and sprites (8pixel x 1..=15pixel) that are XOR'd with the screen
        - When collision happens, you invert the screen
## Input
    - hexkeyboard from 0 to f
    - Three opcodes track input
        - One skips instruction if a button is pressed
        - One skips instructions if a specific button is not pressed
        - One waits for input and adds that to a register 
## Opcode table
    - 35 Opcodes [2 bytes] stored as big endian
    - Symbols
        - NNN is the address
        - NN is an 8bit constant
        - N is a 4 bit constant
        - X and Y are the 4 bit register identifiers
        - PC is the Program Counter
        - I is the 12 memory address register
        - VN  is one of the 16 variables where N is a variable from 0 to F 

# Book
## ROMS
- ROMS aka read only memory are our game files. 
- Cant be opened with a regular editor because they dont store text or ascii
- They store hexadecimal numbers and can be opened with a hex-editor
- Each instruction is two bytes which is they book groups them in pairs

## CPU
- Hardware that does math. Addition, subtraction, equality, jumping to different places, fetching/saving numbers
- All these math operations have a corresponding number called an opcode
- When its time for the CPU to perform a new operation, it grabs the next opcode from the ROM and does 
    whatever it says
- Opcode table uses N's to indicate literal hexadecimal numbers. N for single digit. NN for two digits. NNN for three.
- X and Y are register identifiers
- Program Counter controls which instruction to execute next. (it is also a register)
- Starts at the first byte then moves onto the third (each instruction is two bytes)
- There might also be jump instructions that tell the PC to go elsewhere but otherwise executed in linear order

# Registers
Can store values. You have 16 of them from V0 to VF

# RAM 
- We have only 16 registers and if you think about it, itd be nice to store more than 16 numbers
- This is where RAM comes in. Its a large array of numbers and you can do wtv you like with it 
- 4KB or 4096 bytes or 4096 8 bit places to store numbers
- Chip8 doesnt directly read from the game files from ROM, instead the game is copied into RAM 
    and read from there. Most games use much much less than 4KB fit into RAM
- Also ROM data isnt copied into the start of the RAM. Its copied from byte 512 ie 0x200
- This is because first 512 bytes used for chip8 interpreter. We probably gonna use that data for fonts tho :D

We haven’t yet delved into how the Chip-8 screen display works, but the gist for now is that it renders sprites
which are stored in memory to the screen, one line at a time. It is up to the game developer to correctly load their
sprites before copying them over. However wouldn’t it be nice if the system automatically had sprites for commonly
used things, such as numbers? I mentioned earlier that our PC will begin at address 0x200, leaving the first 512
intentionally empty. Most modern emulators will use that space to store the sprite data for font characters of all the
hexadecimal digits, that is characters of 0-9 and A-F. We could store this data at any fixed position in RAM, but this
space is already defined as empty anyway. Each character is made up of eight rows of five pixels, with each row using
a byte of data, meaning that each letter altogether takes up five bytes of data. The following diagram illustrates how
a character is stored as bytes.

Sprites that are stored in memory are rendered to the screen 
Can use initial memory to store commonly used font data like all hexadecimal characters (0-F)
8 rows each using 1 byte

One thing im a little confused about is the memory layout of the font data. 
Theres 16 characters. 0 - F that we want to add to RAM. 
Each character takes 5 bytes of data so we end up with 80 bytes 

