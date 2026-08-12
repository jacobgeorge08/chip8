## System Info
### Memory Layout
    - 4096 memory locations (All 1 byte ie 8bits long)
        - bottom 512 bytes used for the interpreter
        - top 256 bytes used by the display
        - 96 bytes below also used for internal use and call stack
### Registers
    - 16 registers, V0 to VF (8 bits long)
        - VF used as a flag for addition carry, subtration no borrow, collision detection
    - I register is the address register and used to for memory operations (12bits long)
### Timers
    - Two both count down for 60hz (8bits long)
        - Delay Timer ie used for game stuff
        - Sound timer beeps when non zero
### Stack
    - Used to hold mem addr of subroutines. 
    - 48bytes which can hold 12 levels of nesting
### Graphics
    - MonoChrome display
    - 64 x 32 screen and sprites (8pixel x 1..=15pixel) that are XOR'd with the screen
        - When collision happens, you invert the screen
## Input
    - hexkeyboard from 0 to f
    - Three opcodes track input
        - One skips instruction if a button is pressed
        - One skips instructions if a specific button is not pressed
        - One waits for input and adds that to a register 
### Opcode table
    - 35 Opcodes [2 bytes] stored as big endian
    - Symbols
        - NNN is the address
        - NN is an 8bit constant
        - N is a 4 bit constant
        - X and Y are the 4 bit register identifiers
        - PC is the Program Counter
        - I is the 12 memory address register
        - VN  is one of the 16 variables where N is a variable from 0 to F 

## Thoughts while impl Emu
### ROMS
    - ROMS aka read only memory are our game files. 
    - Cant be opened with a regular editor because they dont store text or ascii
    - They store hexadecimal numbers and can be opened with a hex-editor
    - Each instruction is two bytes long
### CPU
    - Hardware that does math. Addition, subtraction, equality, jumping to different places, 
        fetching/saving numbers
    - All these math operations have a corresponding number called an opcode
    - When its time for the CPU to perform a new operation, it grabs the next opcode from the ROM 
        and does whatever it says
    - Opcode table uses N's to indicate literal hexadecimal numbers. 
        - N for single digit. NN for two digits. NNN for three.
    - X and Y are register identifiers
    - Program Counter controls which instruction to execute next. (it is also a register)
    - Starts at the first byte then moves onto the third (each instruction is two bytes)
    - There might also be jump instructions that tell the PC to go elsewhere but 
        otherwise executed in sequential order
### RAM 
    - We have only 16 registers and if you think about it, itd be nice to store more than 
        16 numbers
    - This is where RAM comes in. Its a large array of numbers and you can do wtv you 
        like with it 
    - 4KB or 4096 bytes or 4096 8 bit places to store numbers
    - Chip8 doesnt directly read from the game files from ROM, instead the game is copied into RAM 
        and read from there. Most games use much much less than 4KB fit into RAM
    - Also ROM data isnt copied into the start of the RAM. Its copied from byte 512 ie 0x200
    - This is because first 512 bytes used for chip8 interpreter. We probably gonna use 
        that space for font/sprite data tho :D
### Display
    - The gist of how the display works is that it renders sprites which are stored in memory
        one line at a time. 
    - Since our PC begins at 0x200, we're gonna be using those first 512 bytes to store sprite
        data. Just the 16 hexadecimal characters ie 0-9 and A-F.
    - Each character takes 5 bytes of data.
    - 0 sprite after hexadecimal [0xF0, 0x90, 0x90, 0x90, 0xF0] is encoded to binary. Looks like 0 if you squint :P
      11110000
      10010000
      10010000
      10010000
      11110000
    - Sprite for 5 [0xF0, 0x80, 0xF0, 0x10, 0xF0] 
      11110000
      10000000
      11110000
      00010000
      11110000
### Fetch 
    - Each instruction is two bytes long. Also everything is big endian.
    - We get the first and second bytes from the ram via pc (converted to u16)
    - Bit shift the first byte by 8 (<< 8) and OR it with the second byte
        to get our current opcode that we need to execute
    - Pass this to our decode function and let it pattern match
### Decode
    - Just a huge match statement that we implement by looking at the spec
    - One thing that was new to me was how individual digits are extracted from the opcode
    - We basically & the opcode with a bitmask and then right shift (>>) the digits to get 
        the units place we want 
### Calling Subroutines
    - We push the current instruction from the PC to the stack and we set the PC to 
        NNN (subroutine instruction)

### Unorganized notes
    - The v_registers have their data in 8 bit values
        - To get the least significant bit from the a particular register, we can & the 8bit val with 1
        - To get the most significant bit, we right shift the byte by & and then & with 1
    - For opcode FX0A where we wait for a keypress, we reset the pc to the 
        previous instruction if a key is not pressed after looping through the keys
        array because this avoid us missing a key that is pressed if we were to have
        instead used an infinite loop 
    - For opcode FX29, we store our font data (0-9 and A-F) starting at the beginning of RAM (index 0)
        - Each character takes 5 bytes of data each. So all we have to do to find the character in ram
            is take the character from the v_reg, multiply by 5 and set that as the i_reg value.
    - Opcode FX33 is where we convert our hexadecimal numbers into base10 decimals because when
        we print out things like high scores of our players etc, itd be a little jarring to 
        see 0x64 instead of a high score of 100. We just use division, modulo and .floor() to pull
        out and convert to the numbers we need
    - Opcode DXYN
        - Probably the hardest opcode becuase theres so much going on. 
        - We get the x and y coordinates from the v_registers and the height from digit4 of opcode
        - Iterate through the rows and get the mem_addr for ram which is stored in i_reg + an offset
        - Get the pixels for a row using the ram addr and compare with a bitmask to see if its a 1
        - If it is, we attempt to draw to screen.
