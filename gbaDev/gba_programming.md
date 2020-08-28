# Game Boy Advance Programming

## Kyle Halladay Tutorial

### Drawing Rectangles

There are five different video modes on the Game Boy Advance. Video mode 3 is suitable for doing basic drawings. The screen buffer for mode 3 is 240x160. It uses a single buffer (more on that when we cover buffers) which means it is relatively straightforward to set pixels on the screen.

To set the video mode you need to set the value of the display control register. The register is referenced using it's hexadecimal memoery location, which is `0x04000000`. The value needs to be set to `0x0003` for the video mode to be set to mode 3. In addition to this the background mode needs to be set to mode 2 in the same register, using the hex value of `0x0400`.

Here's the basic code to set this up:

```cpp
typedef unsigned int    uint32;

#define REG_DISPLAYCONTROL *((volatile uint32*)(0x04000000))
#define VIDEOMODE_3         0x0003
#define BGMODE_2            0x0400

int main()
{
    REG_DISPLAYCONTROL = VIDEOMODE_3 | BGMODE_2;
    while(1){}
}
```

The `REG_DISPLAYCONTROL` is a reference to the display control register that is cast to an unsigned 32-bit integer (16-bit in libgba). The `volatile` keyword ensures that even though we don't read the value of the register, the compiler doesn't optimise the code for this. Optimising this could lead it to not working correctly.

Let's break down this line `*((volatile uint32*)(0x04000000))` which defines the display control register location. First up `0x04000000` this is the memory address of the register written in hexadecimal. The `(volatile uint32*)` part casts this as a 32 bit integer pointer (16 bit in the libgba library). The pointer is then dereferenced with the surrounding `*()` so that the value of the memory location can be set.

The `REG_DISPLAYCONTROL = VIDEOMODE_3 | BGMODE_2` line uses a bitwise or operator (`|`) to combine the values of video mode 3 and background mode 2. Essentially the value that is set is `0x0403`, which is a combination of these two values.

The program uses an infinite loop in `while(1){}`. Usually game logic lives in here, but in this example it is empty. 

The libgba library contains header files for setting the video mode as well as custom types. This saves the need to write out the registers and values in your program. Here's the same code rewritten to take advantage of libgba:

```cpp
#include <gba_video.h>

int main()
{
    SetMode(MODE_3 | BG2_ON);
    while(1){}
}
```

#### Writing to the Mode 3 Screen Buffer 

The mode 3 screen buffer is a location in memory that is used to set pixels on the screen when using display mode 3. Its memory address is `0x06000000`. By setting indexes at this location to a hex values, you can draw coloured pixels to the screen:

```cpp
typedef unsigned char      uint8;
typedef unsigned short     uint16;
typedef unsigned int       uint32;

#define REG_DISPLAYCONTROL *((volatile uint32*)(0x04000000))
#define VIDEOMODE_3         0x0003
#define BGMODE_2            0x0400

#define SCREENBUFFER        ((volatile uint16*)0x06000000)
#define SCREEN_W            240
#define SCREEN_H            160

int main()
{
    REG_DISPLAYCONTROL = VIDEOMODE_3 | BGMODE_2;

    for (int i = 0; i < SCREEN_W * SCREEN_H; ++i)
    {
        SCREENBUFFER[i] = 0xFFFF;
    }

    while(1){}
    return 0;
}  
```

This program uses a for loop to set each pixel `for (int i = 0; i < SCREEN_W * SCREEN_H; ++i)`. In total there are 38400 pixels in this mode (240 width x 160 high). The top left corner of the screen is pixel 0, with values increasing as you move right. This mode loops back around so pixel index 240 is on the second line of the image.

The `SCREENBUFFER[i] = 0xFFFF;` line uses the index value from the for loop to set the colour of each pixel to white.

You may have noticed that the `REG_DISPLAYCONTROL` pointer `*((volatile uint32*)(0x04000000))` is dereferenced, whereas the `SCREENBUFFER` pointer `((volatile uint16*)0x06000000)` is not. This is so that we can index into the screen buffer to set individual pixel values with the `SCREENBUFFER[i] = 0xFFFF;` line in the for loop. The `0x06000000` value is the first index in the screen buffer array.

It's also worth noting that the memory location of the screen buffer in graphics mode 3 `0x06000000` has different uses in different graphics mode. The memory location `0x06000000` is actually just the location for Video RAM (VRAM), which is instead used for storing sprites and palettes in other modes.

Here's the code rewritten using libgba:

```cpp
#include <gba_video.h>

int main()
{
    SetMode(MODE_3 | BG2_ON);

    for (int i = 0; i < SCREEN_HEIGHT; ++i)
    {
        for (int j = 0; j < SCREEN_WIDTH; ++j)
        {

            MODE3_FB[i][j] = 0x6666;
        }
        
    }

    while(1){}
    return 0;
}
```

The main difference between the Kyle Halladay code and the libgba library is that the Mode 3 frame buffer is an 240 length array in libgba. This makes it slightly easier to to work out which row and column you're working with. For example `MODE3_FB[10][32]` is pixel on row 10, column 32 (if you count from 0).

This program will draw a gradient:

```cpp
#include <gba_video.h>

int main()
{
    SetMode(MODE_3 | BG2_ON);

    for (int i = 0; i < SCREEN_HEIGHT; ++i)
    {
        for (int j = 0; j < SCREEN_WIDTH; ++j)
        {

            MODE3_FB[i][j] = 0x6666 + (int) (i >> 3);
        }
        
    }

    while(1){}
    return 0;
}
```

Note that the bit shift operator `>>` is used here instead of a division. This is because the GBA has no hardware optimised division built-in unlike other arithmetic operations, so can be quite slow when doing divisions.


#### 15 Bit Colours

Colours on the GBA are stored at 16 bit integers (2 bytes), with each RGB value using 5 bits. This means that there are 32 values (0-31) for red, green and blue. The 16th bit is unused. 

The break down of colours/bits looks like this:

```
XBBB BBGG GGGR RRRR

X = unused
```

When using hexadecimal values use two bytes like so:

```cpp
blueish = 0x6666;
```

A utility Function to create 15 bit colours is outlined in Tonc and the Kyle Halladay tutorials:

```cpp
inline uint16 MakeCol(uint8 red, uint8 green, uint8 blue)
{
    return (red & 0x1F) | (green & 0x1F) << 5 | (blue & 0x1F) << 10;
}
```

This uses the the `&` binary operator to to ensure that values do not execeed 31. The bit shift operators are used to move the green and blue values into the correct bit positions.


#### VSync 

When drawing pixels to the screen, the GBA draws one row at a time. When it reaches the end of each row the GBA pauses momentarily before moving onto the next row. This pause is known as an HBLANK. After drawing each row the GBA will pause momentarily again. This pause is known as VBLANK. 

You should avoid writing to screenbuffer while the GBA is drawing to the screen as this can cause weird glitches like screen tearing or different parts of the screen being in different frames of animation. Instead you should wait until the drawing reaches a VBLANK as nothing is being drawn to the screen during this time.

There are different ways to detect this. One basic way is to read from the VCOUNT register at `0x04000006`. If the value of this register is 0-159, then it is currently drawing to one of the rows. It is is 160+ then it is in a VBLANK.

```cpp
#define REG_VCOUNT      (* (volatile uint16*) 0x04000006)
inline void vsync()
{
  while (REG_VCOUNT >= 160);
  while (REG_VCOUNT < 160);
}
```

This function has two main parts: `while (REG_VCOUNT >= 160);` will wait if the function was called during a VBLANK, meaning the function won't return until the next VBLANK; and `while (REG_VCOUNT < 160);` which will cause the function to wait until the GBA has stopped drawing to the screen and has reached a VBLANK. 

During the time that the GBA is drawing to the screen, it is still OK to do other calculations. As long as you don't modify the framebuffer everything should be OK.

This Kyle Halladay program will draw a square that moves across and down the screen:

```cpp
typedef unsigned char      uint8;
typedef unsigned short     uint16;
typedef unsigned int       uint32;


#define REG_VCOUNT      (* (volatile uint16*) 0x04000006)
#define REG_DISPLAYCONTROL *((volatile uint32*)(0x04000000))
#define VIDEOMODE_3         0x0003
#define BGMODE_2            0x0400

#define SCREENBUFFER        ((volatile uint16*)0x06000000)
#define SCREEN_W            240
#define SCREEN_H            160


void drawRect(int left, int top, int width, int height, uint16 clr)
{
    for (int y = 0; y < height; ++y)
    {
        for (int x = 0; x < width; ++x)
        {
           SCREENBUFFER[(top + y) * SCREEN_W + left + x] = clr;
        }
    }
}

inline void vsync()
{
  while (REG_VCOUNT >= 160);
  while (REG_VCOUNT < 160);
}

int main()
{
    REG_DISPLAYCONTROL = VIDEOMODE_3 | BGMODE_2;

    for (int i = 0; i < SCREEN_W * SCREEN_H; ++i)
    {
        SCREENBUFFER[i] = MakeCol(0,0,0);
    }

    int x = 0;
    while(1)
    {
        vsync();

        if ( x > SCREEN_W * (SCREEN_H/10)) x = 0;
        if (x)
        {
            int last = x - 10;
            drawRect(last % SCREEN_W, (last / SCREEN_W) * 10, 10, 10,MakeCol(0,0,0));
        }

        drawRect(x % SCREEN_W, (x / SCREEN_W) * 10, 10, 10,MakeCol(31,31,31));
        x += 10;

    }
}
```


Here is the same program rewritten to use features of libgba:

```cpp
#include <gba_video.h>
#include <gba_systemcalls.h>
#include <gba_interrupt.h>


inline uint16_t MakeCol(uint8_t red, uint8_t green, uint8_t blue)
{
    return red | green << 5 | blue << 10;
}

void drawRect(int left, int top, int width, int height, uint16_t clr)
{
    for (int y = 0; y < height; ++y)
    {
        for (int x = 0; x < width; ++x)
        {
           MODE3_FB[top + y][SCREEN_WIDTH + left + x] = clr;
        }
    }
}

int main()
{
    SetMode(MODE_3 | BG2_ON);
    
    irqInit();
    irqEnable(IRQ_VBLANK);

    for (int i = 0; i < SCREEN_HEIGHT; ++i)
    {
        for (int j = 0; j < SCREEN_WIDTH; ++j)
        MODE3_FB[i][j] = MakeCol(0,0,0);
    }

    int x = 0;

    while(1)
    {
        VBlankIntrWait();

        if ( x > SCREEN_WIDTH * (SCREEN_HEIGHT/10)) x = 0;
        if (x)
        {
            int last = x - 10;
            drawRect(last % SCREEN_WIDTH, (last / SCREEN_WIDTH) * 10, 10, 10,MakeCol(0,0,0));
        }

        drawRect(x % SCREEN_WIDTH, (x / SCREEN_WIDTH) * 10, 10, 10,MakeCol(31,31,31));
        x += 10;

    }

    return 0;
}
```

The main difference is how libgba waits for VBLANK. Instead of reading from the the VCOUNT register, it waits for a VBLANK interrupt from the hardware. 

There are a couple of steps to set this up. First you need to include the system call and interrupts headers from libgba:

```cpp
#include <gba_systemcalls.h>
#include <gba_interrupt.h>
```

Then at the start of the program you use `irqInit()` to initialise the interrupts and `irqEnable(IRQ_VBLANK)` to enable the VBLANK interrupt. 

Finally, in the game loop you add the `VBlankIntrWait()` function to wait for VBLANK. Just like the code above this function will wait until the screen drawing is in a VBLANK so that you can control when you draw to VRAM.


### Sprites

























## Debugger



To try:
- Use regular gdb instead of the bundled arm one
- Make a list of gdb commands and try them with gdb
- Use mgba's built-in debugger
- Make a list of mgba's built-in debugger commands + try them with gdb
- Use the built-in debugger in Visual Studio Code


### Connecting GDB to a Compiled GBA Game

Setup:

1. Does gdb need to be installed?
1. Install Ncurses library `apt install libncurses5` if you want to use the gdb bundled with devkitpro


Running:

1. Compile the game
1. Open VBA-M and load game
1. Start GDB server with `Tools > GDB > Break into GDB`
1. Start GDB either with `gdb` or the bundled with DevKitPro via the terminal `/opt/devkitpro/devkitARM/bin/arm-none-eabi-gdb`
1. Connect to GDB Server running in VBA-M `target remote localhost:55555`
1. Load the corresponding `.elf` file for the `.gba` file to load the symbols with `file <path_to.elf>`

### GDB Commands


View the code with `layout next`. Nothing will be displayed as the program is not running yet.

Start the program with `next`


Notes:
- `run` command does not work


### Using Debugger with Visual Studio Code

https://sausage-factory.games/dev-blog/Gameboy-Advance-Dev-Workflow/

