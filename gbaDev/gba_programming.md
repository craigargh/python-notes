# Game Boy Advance Programming


## Setup


### Emulator

There are several emulators for the Game Boy Advance. On Ubuntu Linux mgba and vba-m are the most fully featured.

Overall, mgba is the better choice for development. It works out of the box with the gdb debugger and connects well to bluetooth controllers.

To install mgba:

1. Open a terminal
1. Run `sudo apt-get install mgba-qt`

Once it is installed you can open `.gba` files with `mgba-qt ./file_name.gba`


### Setting Up DevKitPro on Ubuntu

https://devkitpro.org/

Steps:
1. Download devkitpro's custom package manager from https://github.com/devkitPro/pacman/releases/
1. Update the available packages for DevKitPro's package manager. On the command line type `sudo dkp-pacman -Sy`
1. To install all packages required for gba-dev `sudo dkp-pacman -S gba-dev`. Press enter when prompted which packages you want to install all
1. Set the DEVKITARM env var using `sudo nano ~/.profile`

```
# devkitpro
export DEVKITPRO=/opt/devkitpro
export DEVKITARM=/opt/devkitpro/devkitARM
export DEVKITPPC=/opt/devkitpro/devkitPPC
```

1. Reload env vars from profile `. ~/.profile`


### 8BitDo/Xbox One Controller Setup with VisualBoy Advance M

The controller should work with mgba. However if you want to use Visual Boy Adance M from Snap the controller won't be detected. 

1. Install Visual Boy Advance M using via Ubuntu software
1. Connect joypad via Bluetooth

If the controller is not detected by Visual Boy Advance:
1. Install qjoypad
1. Import 8bitdo_qjoypad.lyt in qjoypad

### Other notes 


Debian Packages

https://github.com/devkitPro/pacman/releases/tag/v1.0.2


Docker Images

https://hub.docker.com/u/devkitpro


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
        MODE3_FB[i][j] = RGB5(0,0,0);
    }

    int x = 0;

    while(1)
    {
        VBlankIntrWait();

        if ( x > SCREEN_WIDTH * (SCREEN_HEIGHT/10)) x = 0;
        if (x)
        {
            int last = x - 10;
            drawRect(last % SCREEN_WIDTH, (last / SCREEN_WIDTH) * 10, 10, 10, RGB5(0,0,0));
        }

        drawRect(x % SCREEN_WIDTH, (x / SCREEN_WIDTH) * 10, 10, 10, RGB5(31,31,31));
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

```cpp
irqInit();
irqEnable(IRQ_VBLANK);
```

Finally, in the game loop you add the `VBlankIntrWait()` function to wait for VBLANK. Just like the code above this function will wait until the screen drawing is in a VBLANK so that you can control when you draw to VRAM.

```cpp
while(1){
    VBlankIntrWait();
    // Do whatever
}
```


### Tiles and Palettes

In the GBA video modes, modes 0-2 are tiled modes. These modes allow the use of sprites. The sprites are stored in VRAM in 8x8 tiles. 

The tiles do not contain the colour values that will be displayed. Instead they contain indexes which are used to lookup the colour values from a separate palette array. 

Each program has access to a palette for background colours and another for tile colours. Each of these palettes can have a total of 256 colours. These colours are 15bit values, as mentioned earlier.

Tiles can either use 8 bits per pixel (giving a total of 256 possible colours values) or 4 bits per pixel (giving 16 possible colour values).

Sprites are made up of a rectangular collection of the 8x8 tiles in VRAM. Sprites on the GBA are referred to as "Objects". This is different to the concept of an object in object-oriented programming. Don't confuse the two.

Here's a sample sprite tile and palette from the Kyle Halladay tutorial:

```cpp
const unsigned int testTiles[16] __attribute__((aligned(4)))=
{
    0x00000000,0x00000000,
    0x00000001,0x00000000,
    0x00000000,0x00000000,
    0x00000000,0x00000000,
    0x00000000,0x00000000,
    0x00000000,0x00000000,
    0x02020102,0x02020202,
    0x00000000,0x00000000,
};

const unsigned int testPal[2] __attribute__((aligned(4)))=
{
    0x03E0001F,0x00007C00,
};
```

As mentioned before each tyle is 8x8 pixels. Four bytes make up each row, meaning each hexadecimal value in the array represents four pixels. Each pixel is made up of two bytes. 

The lowest digits in the hex values represent the left-most pixels, while the most significant hex values represent pixels further to the left. In other words each hex value is read from right to left. However each item in the array is read in order.

The value 00 in a tile is transparent pixel, while other values are indeces into the corresponding palette array (with 1-indexing instead of zero-indexing.

The `__attribute__((aligned(4)))=` part of the definition is a macro that ensures that data is correctly aligned and follows the 4 byte boundary in memory. The Tonc tutorial explains this in more depth about why this is necessary.

Tiles are stored in sequence in the same array, one after another. For example the Kyle Halladay tutorial uses four tiles to create a larger sprite:

```cpp
const unsigned int spriteTiles[64] __attribute__((aligned(4)))=
{
    0x00000000,0x01000000,0x00000000,0x01010000,0x00000000,0x01010100,0x00000000,0x01010101,
    0x01000000,0x01010101,0x01010000,0x01010101,0x01010100,0x01010101,0x01010101,0x01010101,
    0x00000003,0x00000000,0x00000303,0x00000000,0x00030303,0x00000000,0x03030303,0x00000000,
    0x03030303,0x00000003,0x03030303,0x00000303,0x03030303,0x00030303,0x03030303,0x03030303,
    0x04040404,0x04040404,0x04040400,0x04040404,0x04040000,0x04040404,0x04000000,0x04040404,
    0x00000000,0x04040404,0x00000000,0x04040400,0x00000000,0x04040000,0x00000000,0x04000000,
    0x02020202,0x02020202,0x02020202,0x00020202,0x02020202,0x00000202,0x02020202,0x00000002,
    0x02020202,0x00000000,0x00020202,0x00000000,0x00000202,0x00000000,0x00000002,0x00000000,
};

const unsigned int spritePal[3] __attribute__((aligned(4)))=
{
    0x001E0000,0x03E07FFF,0x00007C1F,
};
```

The corresponding header file for this would look like:

```cpp
#ifndef SPRITE_H
#define SPRITE_H

#define spriteTilesLen 256 //size in bytes
extern const unsigned int spriteTiles[64];

#define spritePalLen 12
extern const unsigned int spritePal[3];

#endif
```

The `extern` keyword is used to so that if multiple other files import from this header file, then the compiler doesn't complain that the values have been defined multiple times.

To copy the palette into memory, the `memcpy` function is used to copy the array in palette memory at `0x05000200`:

```cpp
#define MEM_PALETTE   ((uint16*)(0x05000200))
void UploadPaletteMem()
{
    memcpy(MEM_PALETTE, spritePal, spritePalLen);

}
```

The `memcpy` function takes three arguments:
1. A pointer to the destination location in memory
1. The data to copy
1. The size of the data in bytes

It is important to correctly set the size in bytes of the memory that is being copied as the function ignores types and just directly copies the underlying binary value of the data.


A similar method can be used to copy tile data into VRAM:

```cpp
typedef uint32 Tile[16];
typedef Tile TileBlock[256];

#define MEM_VRAM        ((volatile uint32*)0x06000000)
#define MEM_TILE        ( (TileBlock*)MEM_VRAM )

void UploadTileMem()
{
    memcpy(&MEM_TILE[4][1], spriteTiles, spriteTilesLen);
}
```

There six several blocks of memory for tiles. This code creates a pointer to the VRAM memory location so that each block can be treated as an item in an array. For example `&MEM_TILE[4]` is the fifth block of tile memory.  

There is 32 Kilobytes of memory dedicated to sprites on the GBA in modes 0, 1 and 2. There are 6 blocks with sprite memory: 4 blocks for background tiles; 2 blocks for sprite tiles. These blocks are referred to as "tile blocks" or "char blocks". 


#### Sprites

Sprites in the GBA are referred to as objects (I'll refer to them as sprite objects for clarity). Sprite objects have a number of attributes that allow for the programmer to set their tiles, palettes and location. These objects can then be moved around the screen without the need to clear their past location, like was required in the Kyle Halladay rectangle tutorial.

A sprite object use the following structure:

```cpp
typedef struct ObjectAttributes {
    uint16 attr0;
    uint16 attr1;
    uint16 attr2;
    uint16 pad;
} __attribute__((packed, aligned(4))) ObjectAttributes;

#define MEM_OAM  ((volatile ObjectAttributes *)0x07000000)
```

Sprite objects structs are referred to as `ObjectAttributes`. These `ObjectAttributes` are stored in Object Attribute Memory, or OAM for short. The location of OAM is `0x07000000`.

The values of `attr0`, `attr1` and `attr2` are responsible for managing a number of things on the sprite. For example the Y co-ordinate is set using bits 0-7 of `attr0`, bits FE are used to set the sprite shape, and bit D sets the colour mode:

```cpp
volatile ObjectAttributes *spriteAttribs = &MEM_OAM[0];
spriteAttribs->attr0 = 0x2032;
```

This is a square sprite (bits FE) that uses 8 bits per pixel, with a Y coordinate of 50.

| Attr 0 | 0x FEDC BA98 7654 3210 |
|---|---|
| FE |  Shape of Sprite: 00 = Square, 01 = Tall, 10 = Wide |
| D  | Colour Mode: 0 = 4bpp, 1 = 8bpp |
| C | Not used today |
| AB | Not used today |
| 89 | Not used today |
| 7654 3210 | Y Coordinate |


And the values for `attr1`:


| Attr 1 | 0x FEDC BA98 7654 3210 |
|---|---|
| FE | Sprite Size (discussed below) |
| DCBA98 | Not Used Today |
| 7654 3210 | X coordinate |

The dimensions of a sprite is based on two values, bits FE in `attr0` for shape and bits FE in `attr1` for size. These two values are combined to find the dimensions, which are summarised in this table:


| Size 00 | Size 01 | Size 10 | Size 11 |
|---|---|---|---|
| Shape 00 | 8x8  | 16x16 | 32x23 | 64x64 |
| Shape 01 | 16x8 | 32x8  | 32x16 | 64x32 |
| Shape 10 | 8x16 | 8x32  | 16x32 | 32x64 |


This code will create a 16x16 square sprite at coordindates x=100, y=50:

```cpp
volatile ObjectAttributes *spriteAttribs = &MEM_OAM[0];

spriteAttribs->attr0 = 0x2032; // 8bpp tiles, SQUARE shape
spriteAttribs->attr1 = 0x4064;
```

To set the location of the sprite tiles, `attr2` is used. It's bit value 0-9 are used to set to the index of the first sprite tile.

```cpp
volatile ObjectAttributes *spriteAttribs = &MEM_OAM[0];

spriteAttribs->attr0 = 0x2032; // 8bpp tiles, SQUARE shape
spriteAttribs->attr1 = 0x4064; // 16x16 size when using the SQUARE shape
spriteAttribs->attr2 = 2;      // Start at [4][1]
```

Since we're using 8bits per pixel and the tile is stored in index 1, the attrbute value for the sprite should be set to 2. You have to double the value for the index when using 8 bits per pixel. So a tile at index 4 in the array would be set as 8 in the sprite attribute.

If we were using 4 bits per pixel the index values in the attribute and the array would be the same.

To setup the Display Control register for sprite mapping we need to set the video mode to 1, enable the hardware to manage objects, and set the tile mapping mode to one dimensional:

```cpp
#define REG_DISPLAYCONTROL     *((volatile uint16*)(0x04000000))

#define VIDEOMODE_0    0x0000
#define ENABLE_OBJECTS 0x1000
#define MAPPINGMODE_1D 0x0040

int main()
{
    ...
    REG_DISPLAYCONTROL =  VIDEOMODE_0 | ENABLE_OBJECTS | MAPPINGMODE_1D;
    ...
}
```

A one-dimensional tile mapping means that the multiple tiles that make up a sprite are stored in sequence. Whereas in two-dimensional mapping the tiles will be stored on multiple rows close together, to almost appear in the same shape as the final sprite that will be displayed.

Here's the full Kyle Halladay code for moving a sprite across the screen (note the sprites array and palette is stored in `sprite.h`):

```cpp
#include "sprite.h"
#include <string.h>

typedef unsigned char      uint8;
typedef unsigned short     uint16;
typedef unsigned int       uint32;

typedef uint32 Tile[16];
typedef Tile   TileBlock[256];

#define VIDEOMODE_0    0x0000
#define ENABLE_OBJECTS 0x1000
#define MAPPINGMODE_1D 0x0040

#define REG_VCOUNT              (*(volatile uint16*) 0x04000006)
#define REG_DISPLAYCONTROL      (*(volatile uint16*) 0x04000000)

#define MEM_VRAM      ((volatile uint16*)0x6000000)
#define MEM_TILE      ((TileBlock*)0x6000000 )
#define MEM_PALETTE   ((uint16*)(0x05000200))
#define SCREEN_W      240
#define SCREEN_H      160

typedef struct ObjectAttributes {
    uint16 attr0;
    uint16 attr1;
    uint16 attr2;
    uint16 pad;
} __attribute__((packed, aligned(4))) ObjectAttributes;

#define MEM_OAM       ((volatile ObjectAttributes *)0x07000000)

inline void vsync()
{
    while (REG_VCOUNT >= 160);
    while (REG_VCOUNT < 160);
}

int main()
{
    memcpy(MEM_PALETTE, spritePal,  spritePalLen );
    memcpy(&MEM_TILE[4][1], spriteTiles, spriteTilesLen);

    volatile ObjectAttributes *spriteAttribs = &MEM_OAM[0];

    spriteAttribs->attr0 = 0x2032; // 8bpp tiles, SQUARE shape, at y coord 50
    spriteAttribs->attr1 = 0x4064; // 16x16 size when using the SQUARE shape
    spriteAttribs->attr2 = 2;      // Start at the first tile in tile

    REG_DISPLAYCONTROL =  VIDEOMODE_0 | ENABLE_OBJECTS | MAPPINGMODE_1D;

    int x = 0;
    while(1)
    {
        vsync();
        x = (x+1) % (SCREEN_W);
        spriteAttribs->attr1 = 0x4000 | (0x1FF & x);

    }
    return 0;
}
```




When using libgba, this is what the code will look like instead:

```cpp
#include "sprite.h"
#include <string.h>
#include <gba_video.h>
#include <gba_systemcalls.h>
#include <gba_interrupt.h>
#include <gba_sprites.h>


typedef u32 Tile[16];
typedef Tile TileBlock[256];


#define MEM_TILE ((TileBlock*)VRAM)


void UploadPaletteMem(){
    memcpy(SPRITE_PALETTE, spritePal, spritePalLen);
}

void UploadTileMem(){
    memcpy(&MEM_TILE[4][1], spriteTiles, spriteTilesLen);
}

int main() {
    SetMode(MODE_0 | OBJ_ENABLE | OBJ_1D_MAP);

    irqInit();
    irqEnable(IRQ_VBLANK);

    UploadPaletteMem();
    UploadTileMem();

    volatile OBJATTR *spriteAttribs = &OAM[0];

    spriteAttribs->attr0 = OBJ_Y(50) | ATTR0_SQUARE | ATTR0_COLOR_256;
    spriteAttribs->attr1 = OBJ_X(0) | ATTR1_SIZE_16;
    spriteAttribs->attr2 = 2;

    int x = 0;
    while(1){
        VBlankIntrWait();
        x = (x+1) % (SCREEN_WIDTH);
        spriteAttribs->attr1 = ATTR1_SIZE_16 | (0x1FF & x);
    }
    return 0;
}
```

The line `spriteAttribs->attr1 = ATTR1_SIZE_16 | (0x1FF & x);` (which is also in the original code) is used to update the sprites x position on the screen. It uses a mask `(0x1FF & x)` so that only bits 0-7 are set, which are the bits of the x co-ordinate. An `|` with the sprite size is used to make sure it retains the correct dimensions.

As before we use the `SetMode()` function from libgba. Here we are using the values that set mode 0, enable objects and set the object map to one dimensional

```
SetMode(MODE_0 | OBJ_ENABLE | OBJ_1D_MAP);
```

The following line sets the sprite object's y position to 50, it's shape to square and sets it to 8 bits per pixel aka 256 colours:
```
spriteAttribs->attr0 = OBJ_Y(50) | ATTR0_SQUARE | ATTR0_COLOR_256
```

This line sets the sprite object's x position to 0 and its size to 16 pixels:

```    
spriteAttribs->attr1 = OBJ_X(0) | ATTR1_SIZE_16;
```



### Backgrounds

Backgrounds on the GBA are similar to sprites. They are made up of tiles and palettes. Unlike sprites they can be very big, up to 1024x1024 pixels.

The tile palette for backgrounds can store 256 colours. 

Background tiles are stored as 8bit indexes into the colour palette, just like sprite tiles.


The main difference between sprite tiles and background tiles, is that backgrounds are made up using one or more screen blocks. A screen block is an array of 32x32 blocks, with each item in the array containing an index to a background tile. There are 1024 tiles in a tile block array (32 x 32 = 1024). Tile indexes in the the tile block are arranged row by row. 

There can be between 0 and 4 active backgrounds in use at once.


This code will upload a background palette into memory:

```cpp
#include "tiles.h"

#define MEM_BG_PALETTE    ((uint16*)(0x05000000))
#define MEM_OBJ_PALETTE   ((uint16*)(0x05000200))

void UploadPaletteMem()
{
    memcpy(MEM_BG_PALETTE, bgPal, bgPalLen);
}
```

And to load the background tiles into memory:

```cpp
typedef uint16 Tile[32];
typedef Tile TileBlock[256];

#define MEM_VRAM                ((volatile uint32*)0x6000000)
#define MEM_TILE                ((TileBlock*)0x6000000)

void UploadTileMem()
{
    memcpy(&MEM_TILE[0][0], bgTiles, bgTilesLen);
}
```

Backgrounds screen blocks and background tiles use the same memory for storage. Screen blocks use 2048 bytes, which means 8 screen blocks can fit in a background tile block at once. 

To load the tile block into memory:

```cpp
typedef uint16 ScreenBlock[1024];
#define MEM_SCREENBLOCKS        ((ScreenBlock*)0x6000000)

void UploadScreenBlock()
{
    //checkerBg is the ScreenBlock data from the gist
    memcpy(&MEM_SCREENBLOCKS[1], checkerBg, checkerBgLen);
}
```
To display backgrounds, they need to be enabled via the display control register, which as we have covered is also repsonsible to setting the graphics mode and other things. 

```cpp
#define VIDEOMODE_0    0x0000
#define BACKGROUND_0   0x0100
#define BACKGROUND_1   0x0200
#define BACKGROUND_3   0x0400
#define BACKGROUND_4   0x0800

#define REG_DISPLAYCONTROL     *((volatile uint16*)(0x04000000))

int main()
{
    REG_DISPLAYCONTROL = VIDEOMODE_0 | BACKGROUND_0 | BACKGROUND_1;
    return 0;
}
```

There are four registers that control the properties of each of the available backgrounds:

```cpp
#define REG_BG0_CONTROL        *((volatile uint16*)(0x04000008))
#define REG_BG1_CONTROL        *((volatile uint16*)(0x0400000A))
#define REG_BG2_CONTROL        *((volatile uint16*)(0x0400000C))
#define REG_BG3_CONTROL        *((volatile uint16*)(0x0400000E))
```

By setting the value of these registers your programs can control the size, starting screen block, etc.

This table summarises which bits set which properties:


| BG | 0x FEDC BA98 7654 3210 |
|---|---|
| FE | Size (defined below) |
| D |   Ignored today (see Tonc for info) |
| CBA98 |   What Screen Block to start at |
| 7 |   Color mode: (1 for 8bpp, 0 for 4bpp) |
| 6 |   Ignored today (see Tonc for info) |
| 54 |  Nothing, empty bits |
| 32 |  Tile Block to use |
| 10 |  Z Depth |


The size as set by bits FE is based on the values in this table:


| Value | Size (in Tiles) |
|---|---|
| 00 | 32x32 |
| 01 | 64x32 |
| 10 | 32x64 |
| 11 | 64x64 |


The following code will set background 0 to a 32x32 background (size 0), with colour mode 1, a tile block of 1, and a depth of 0:

```cpp
//Size 00, Screen Block 1, Color Mode 1, Tile Block 0, Depth 0
//0000 0001 1000 0000

REG_BG0_CONTROL = 0x0180;
```

The z depth attribute sets the priority of the background. A value of 0 is the highest so it will be drawn on top of any backgrounds with a higer value.

This code will create a 64x32 tile background that uses two screen blocks:

```cpp
// Size 01, Screen Block 2, Color Mode 1, Tile Block 1, Priority 1
 // 0100 0010 1000 0101
REG_BG1_CONTROL = 0x4285;
```

Since it has z depth of 1, it will have a lower priority and be drawn behind the other background.

This is what the code will look like when combined (excluding the `generateGradient()` function), which is used to generate one of the backgrounds in the Kyle Halladay tutorial:

```cpp
#include <string.h>
#include "tiles.h"
#include "bg.h"

typedef unsigned char      uint8;
typedef unsigned short     uint16;
typedef unsigned int       uint32;

typedef uint16 ScreenBlock[1024];
typedef uint16 Tile[32];
typedef Tile TileBlock[256];

#define VIDEOMODE_0    0x0000
#define BACKGROUND_0   0x0100
#define BACKGROUND_1   0x0200

#define REG_DISPLAYCONTROL     *((volatile uint16*)(0x04000000))
#define REG_BG0_CONTROL        *((volatile uint16*)(0x04000008))
#define REG_BG1_CONTROL        *((volatile uint16*)(0x0400000A))

#define MEM_VRAM                ((volatile uint32*)0x6000000)
#define MEM_TILE                ((TileBlock*)0x6000000)
#define MEM_SCREENBLOCKS        ((ScreenBlock*)0x6000000)

#define MEM_BG_PALETTE          ((uint16*)(0x05000000))
#define MEM_PALETTE             ((uint16*)(0x05000200))

inline uint16 MakeCol(uint32 red, uint32 green, uint32 blue)
{
    return red | (green<<5) | (blue<<10);
}

void GenerateGradient();

int main()
{
    //load data
    memcpy(MEM_BG_PALETTE, bgPal, bgPalLen );
    memcpy(&MEM_TILE[0][0], bgTiles, bgTilesLen);
    memcpy(&MEM_SCREENBLOCKS[1], checkerBg, checkerBgLen);

    GenerateGradient();

    REG_BG0_CONTROL = 0x0180;// 0000 0001 1000 0000;
    REG_BG1_CONTROL = 0x4285; // 0100 0010 1000 0101
    REG_DISPLAYCONTROL =  VIDEOMODE_0 | BACKGROUND_0 | BACKGROUND_1;
    while(1)
    {
    }
    return 0;
}
```

This can be rewritten using libgba like so:

```cpp
#include <string.h>
#include "tiles.h"
#include <gba_video.h>

typedef u16 ScreenBlock[1024];
typedef u16 Tile[32];
typedef Tile TileBlock[256];

#define MEM_TILE                ((TileBlock*)VRAM)
#define MEM_SCREENBLOCKS        ((ScreenBlock*)VRAM)

void GenerateGradient();

int main()
{
    memcpy(BG_PALETTE, bgPal, bgPalLen );
    memcpy(&MEM_TILE[0][0], bgTiles, bgTilesLen);
    memcpy(&MEM_SCREENBLOCKS[1], checkerBg, checkerBgLen);

    GenerateGradient();

    REG_BG0CNT = BG_SIZE_0 | SCREEN_BASE(1) | BG_256_COLOR | BG_TILE_BASE(0) | BG_PRIORITY(0);
    REG_BG1CNT = BG_SIZE_1 | SCREEN_BASE(2) | BG_256_COLOR | BG_TILE_BASE(1) | BG_PRIORITY(1);
    
    SetMode(MODE_0 | BG0_ENABLE | BG1_ENABLE);

    while(1)
    {
    }
    return 0;
}
```

Using the background horizontal and vertical scroll registers it is relatively straightforward to move each background independantly. All you need to do is set the value in the register and the GBA does the rest of the work.

This is the extra code for the Kyle Hallady example:

```cpp
#define REG_BG0_SCROLL_H       *((volatile uint16*)(0x04000010))
#define REG_BG0_SCROLL_V       *((volatile uint16*)(0x04000012))
#define REG_BG1_SCROLL_H       *((volatile uint16*)(0x04000014))
#define REG_BG1_SCROLL_V       *((volatile uint16*)(0x04000016))
#define REG_BG2_SCROLL_H       *((volatile uint16*)(0x04000018))
#define REG_BG2_SCROLL_V       *((volatile uint16*)(0x0400001A))
#define REG_BG2_SCROLL_H       *((volatile uint16*)(0x0400001C))
#define REG_BG2_SCROLL_V       *((volatile uint16*)(0x0400001E))

...

//inside main

int hScroll = 0;
int h2Scroll = 0;
while(1)
{
    vsync();

    REG_BG0_SCROLL_H = -hScroll;
    REG_BG1_SCROLL_H = h2Scroll;
    h2Scroll +=2;
    hScroll = h2Scroll/3;
}

```


Here it is rewritten with libgba:

```cpp
#include <string.h>
#include "tiles.h"
#include <gba_video.h>
#include <gba_systemcalls.h>
#include <gba_interrupt.h>

typedef u16 ScreenBlock[1024];
typedef u16 Tile[32];
typedef Tile TileBlock[256];

#define MEM_TILE                ((TileBlock*)VRAM)
#define MEM_SCREENBLOCKS        ((ScreenBlock*)VRAM)

void GenerateGradient();

int main()
{
    irqInit();
    irqEnable(IRQ_VBLANK);

    memcpy(BG_PALETTE, bgPal, bgPalLen );
    memcpy(&MEM_TILE[0][0], bgTiles, bgTilesLen);
    memcpy(&MEM_SCREENBLOCKS[1], checkerBg, checkerBgLen);

    GenerateGradient();

    REG_BG0CNT = BG_SIZE_0 | SCREEN_BASE(1) | BG_256_COLOR | BG_TILE_BASE(0) | BG_PRIORITY(0);
    REG_BG1CNT = BG_SIZE_1 | SCREEN_BASE(2) | BG_256_COLOR | BG_TILE_BASE(1) | BG_PRIORITY(1);
    
    SetMode(MODE_0 | BG0_ENABLE | BG1_ENABLE);

    int hScroll = 0;
    int h2Scroll = 0;
    while(1)
    {
        VBlankIntrWait();

        BG_OFFSET[0].x = -hScroll;
        BG_OFFSET[1].x = h2Scroll;

        h2Scroll +=2;
        hScroll = h2Scroll/3;
    }
    return 0;
}
```

The `BG_OFFSET` constants can be used to set the position of the the backgrounds. The index corresponds to the background layer. Each background has a x and y attrbute, which can be set to move the background.

Note that although libgba also provides `REG_BG0HOFS` and `REG_BG1HOFS` to set the horizontal offset of background 0 and background 1, they do not work in this context. This is might be because they are missing the volatile keyword, causing the compiler to stop them from working.


### Input

The Game Boy Advance has 10 buttons in total: 4 directional buttons, A+B, Start and Select, and two shoulder buttons L+R.

To get whether a button is pressed you can read from a register. The register's address is `0x4000130`. Each button corresponds to a bit in this register:

| Button     | Bit |
|---|---|
| A          | 0   |
| B          | 1   |
| Select     | 2   |
| Start      | 3   |
| Right      | 4   |
| Left       | 5   |
| Up         | 6   |
| Down       | 7   |
| R Shoulder | 8   |
| L Shoulder | 9   |


When a button is pressed the value at the corresponding bit will be set to 0. When a button is not pressed the bit will be set to 1.

Here's the code from the Kyle Halladay tutorial for the register and the corresponding key codes:

```cpp
#define REG_KEYINPUT  (* (volatile uint16*) 0x4000130)

#define KEY_A        0x0001
#define KEY_B        0x0002
#define KEY_SELECT   0x0004
#define KEY_START    0x0008
#define KEY_RIGHT    0x0010
#define KEY_LEFT     0x0020
#define KEY_UP       0x0040
#define KEY_DOWN     0x0080
#define KEY_R        0x0100
#define KEY_L        0x0200

#define KEY_MASK     0xFC00
```

To check that a button is pressed this function can be used:

```cpp
uint32 getKeyState(uint16 key_code)
{
    return !(key_code & (REG_INPUT | KEY_MASK) );
}
```

This uses a mask with the value of the register then combine it with a key code to check if a specific key is pressed. This will return 1 for pressed and 0 for not pressed, which is the opposite of what is stored in the register. The mask `0xFC00` is the value `0000 0011 1111 1111` in binary, which masks only the 10 bits used by the keys.

For example to check if the right should button is pressed you can use `getKeyState(KEY_R)`. 

A more advanced version stores the value of the previous buttons so that you can detect when a key is pressed and released. To get this to work, your program should call the `key_poll()` function once every game loop.

Here's the input code as a header file (`input.h`):

```cpp
#ifndef INPUT_H
#define INPUT_H

unsigned short input_cur = 0x03FF;
unsigned short input_prev = 0x03FF;

#define REG_KEYINPUT  (* (volatile unsigned short*) 0x4000130)

#define KEY_A        0x0001
#define KEY_B        0x0002
#define KEY_SELECT   0x0004
#define KEY_START    0x0008
#define KEY_RIGHT    0x0010
#define KEY_LEFT     0x0020
#define KEY_UP       0x0040
#define KEY_DOWN     0x0080
#define KEY_R        0x0100
#define KEY_L        0x0200

#define KEY_MASK     0xFC00

inline void key_poll()
{
    input_prev = input_cur;
    input_cur = REG_KEYINPUT | KEY_MASK;
}

inline unsigned short wasKeyPressed(unsigned short key_code)
{
    return (key_code) & (~input_cur & input_prev);
}

inline unsigned short wasKeyReleased(unsigned short key_code)
{
    return  (key_code) & (input_cur & ~input_prev);
}

inline unsigned short getKeyState(unsigned short key_code)
{
    return !(key_code & (input_cur) );
}
#endif
```

Here's what it looks like with the full program (excluding sprite and background data):

```cpp
#define VIDEOMODE_0    0x0000
#define ENABLE_OBJECTS 0x1000
#define MAPPINGMODE_1D 0x0040
#define BACKGROUND_0   0x0100
#define REG_DISPLAYCONTROL     *((volatile uint16*)(0x04000000))
#define REG_VCOUNT             *((volatile uint16*)(0x04000006))

inline void vsync()
{
  while (REG_VCOUNT >= 160);
  while (REG_VCOUNT < 160);
}

int main()
{
    CreateBackground();
    LoadTileData();

    REG_DISPLAYCONTROL =  VIDEOMODE_0 | ENABLE_OBJECTS | BACKGROUND_0 | MAPPINGMODE_1D;
    key_poll();
    ClearSprite();

    while(1)
    {
        vsync();
        key_poll();

        const uint16 keys[] = {KEY_A, KEY_B, KEY_SELECT,
                                KEY_START, KEY_RIGHT, KEY_LEFT,
                                KEY_UP, KEY_DOWN, KEY_L, KEY_R};
        ClearSprite();
        for (int i = 0; i < 10; ++i)
        {
            if (getKeyState(keys[i]))
            {
                DrawSprite(keys[i]);
            }
        }
    }

    return 0;
}
```



The code rewritten to use libgba (`input.h` file not needed):

```cpp

```


### Buffers and Sprite Animation

Buffers are a technique used to prevent issues with rendering. If you modify OAM outside of VBLANK you can cause issues like screen tearing. Buffers are used to prevent this.

A buffer is effectively a copy of OAM. Whenever we need to make changes to sprite objects, we make the change to the buffer. When the VBLANK is hit, we copy the buffer into OAM.

To define a buffer:

```cpp
ObjectAttributes oam_object_backbuffer[128];
```

This data is then copied into OAM during VBLANK:

```cpp
while(1)
{
    vsync();
    MEM_OAM[0] = oam_object_backbuffer[0];
}
```

In this code only the first sprite object is copied, but multiple sprite objects can be copied at once.

To setup a sprite, we use the buffer instead of OAM directly:

```cpp
ObjectAttributes *spriteAttribs = &oam_object_backbuffer[0];
spriteAttribs->attr0 = 0x2000;
spriteAttribs->attr1 = 0x4000;
spriteAttribs->attr2 = 0;
```

The sprite object attributes that were covered earlier have another two settings that can be used to flip the sprite  horizontally and vertically. These are the C and D bits of `attr1` respectively. 

The x and y position are set using the top left corner of the sprite.

To record the attributes of out sprite before we modify the buffer, we can use additional structs, for example:

```cpp
const int FLOOR_Y = 160-16;

typedef struct
{
    ObjectAttributes* spriteAttribs;
    int facingRight;
    int firstAnimCycleFrame;
    int animFrame;
    int posX;
    int posY;
    int velX;
    int velY;
    int framesInAir;
}HeroSprite;

void InitializeHeroSprite(HeroSprite* sprite, ObjectAttributes* attribs)
{
    sprite->spriteAttribs = attribs;
    sprite->facingRight = 1;
    sprite->firstAnimCycleFrame = 0;
    sprite->animFrame = 0;
    sprite->posX = 0;
    sprite->posY = FLOOR_Y;
    sprite->velX = 0;
    sprite->velY = 0;
    sprite->framesInAir = 0;
}
```

When calculating the index of sprite's starting tile for an animation, you need to consider two things. How big is the sprite? Am I use 4bpp or 8bpp. A 16x16 sprite is made up of four tiles, so the index position of the second frame of animation will be offset by 4. If using 8bpp this will be multiple by two as each tile uses twice as many bits as the previous.

Here's some code to animat the sprite:

```cpp
void tickSpriteAnimation(HeroSprite* sprite)
{
    ObjectAttributes* spriteAttribs = sprite->spriteAttribs;

    //set firstAnimCycleFrame and animFrame in code here

    spriteAttribs->attr2 = sprite->firstAnimCycleFrame + (sprite->animFrame * 4 * 2);
}
```

Here's the full code:

```cpp
#include "charsprites.h"
#include <string.h>
#include "gba.h"
#include "input.h"

ObjectAttributes oam_object_backbuffer[128];

#define min(x,y) (x > y ? y : x)
#define max(x,y) (x < y ? y : x)

const int FLOOR_Y = 160-16;
const int GRAVITY = 2;
const int WALK_SPEED = 4;
const int JUMP_VI = -10;

typedef struct {
    ObjectAttributes* spriteAttribs;
    int facingRight;
    int firstAnimCycleFrame;
    int animFrame;
    int posX;
    int posY;
    int velX;
    int velY;
    int framesInAir;
}HeroSprite;

void InitializeHeroSprite(HeroSprite* sprite, ObjectAttributes* attribs){
    sprite->spriteAttribs = attribs;
    sprite->facingRight = 1;
    sprite->firstAnimCycleFrame = 0;
    sprite->animFrame = 0;
    sprite->posX = 0;
    sprite->posY = FLOOR_Y;
    sprite->velX = 0;
    sprite->velY = 0;
    sprite->framesInAir = 0;
}

void updateSpritePosition(HeroSprite* sprite) {
    if (getKeyState(KEY_LEFT))
    {
        sprite->facingRight = 0;
        sprite->velX = -WALK_SPEED;
    }
    else if (getKeyState(KEY_RIGHT))
    {
        sprite->facingRight = 1;
        sprite->velX = WALK_SPEED;
    }
    else sprite->velX = 0;

    int isMidAir = sprite->posY != FLOOR_Y;

    if (getKeyState(KEY_A))
    {
        if (!isMidAir)
        {
            sprite->velY = JUMP_VI;
            sprite->framesInAir = 0;
        }
    }

    if (isMidAir)
    {
        sprite->velY = JUMP_VI + (GRAVITY * sprite->framesInAir);
        sprite->velY = min(5, sprite->velY);
        sprite->framesInAir++;
    }

    sprite->posX += sprite->velX;
    //clamp to Screen
    sprite->posX = min(240-16, sprite->posX);
    sprite->posX = max(0, sprite->posX);

    sprite->posY += sprite->velY;
    sprite->posY = min(sprite->posY, FLOOR_Y);

    sprite->spriteAttribs->attr0 = 0x2000 + sprite->posY;
    sprite->spriteAttribs->attr1 = (sprite->facingRight? 0x4000 : 0x5000) + sprite->posX;
}

void tickSpriteAnimation(HeroSprite* sprite){
    ObjectAttributes* spriteAttribs = sprite->spriteAttribs;
    int isMidAir = sprite->posY != FLOOR_Y;

    if (isMidAir) {
        sprite->firstAnimCycleFrame = 56;
        sprite->animFrame = sprite->velY > 0 ? 1 : 0;
    } 
    else if (sprite->velX != 0) {
            sprite->firstAnimCycleFrame = 32;
            sprite->animFrame = (++sprite->animFrame) % 3;
    } 
    else {
        sprite->firstAnimCycleFrame = 0;
        sprite->animFrame = (++sprite->animFrame) % 4;
    }

    spriteAttribs->attr2 = sprite->firstAnimCycleFrame + (sprite->animFrame * 8);
}

int main() {
    memcpy(&MEM_TILE[4][0], charspritesTiles, charspritesTilesLen);
    memcpy(MEM_PALETTE, charspritesPal, charspritesPalLen);

    REG_DISPLAYCONTROL = VIDEOMODE_0 | BACKGROUND_0 | ENABLE_OBJECTS | MAPPINGMODE_1D;

    HeroSprite sprite;
    InitializeHeroSprite(&sprite, &oam_object_backbuffer[0]);

    while(1) {
        for (int i = 0; i < 4; i++){
            key_poll();
            vsync();
        }

        updateSpritePosition(&sprite);
        tickSpriteAnimation(&sprite);
        MEM_OAM[0] = oam_object_backbuffer[0];
        
    }

    return 0;
}
```


Here's the code rewritten to use libgba:

```cpp

```


## Sounds and Music

The maxmod library is a sound library that makes it easy to play music and small samples on the GBA. The library is available with devkitpro and is compatible with libgba.

There are a number of music formats that are playing using maxmod. These include are MOD, S3M, XM, and IT. The `.xm` file format is probably the most common, however it is an unusual format that is not supported by many modern applications. It is however a popular chip-tune format so it is supported by several chip-tune focussed editors. The milkytracker software is one of the more popular editors.

The maxmod library also supports conversion of `.wav` files. However due to the limits on the cartridge's size, these files are better suited for sound effects or very short pieces of music. Longer pieces of music should use `.xm`  or a similar format mentioned above as they require less storage.

Setup for the maxmod library is minimal for a project that already uses the default libgba make file. Most of the required steps are already setup.

In order to maxmod to a project follow these steps:
1. Create a folder in your project called `audio` (you can also call it whatever you like)
1. Open your project's `Makefile`
1. Change the line `MUSIC :=` to `MUSIC := audio`

Now when you add your music and sound effects files to the `audio` folder maxmod will automatically add them so that your project has access. To do this you can just run the following:

1. Run `make clean`
1. Run `make`


When maxmod runs during the make process, it will generate a number of files. The main ones that you need to use in your C/C++ are `soundbank.h` and `soundbank_bin.h`.

The headers in `soundbank.h` will be mostly based on the file names used in the `audio` folder. The `MOD_` prefix will be added to any `.xm` or `.mod` sound files, while `SFX_` will prefix any `.wav` files.

In this program a sound file called `snd1.wav`, which has been added to the soundbank header file as `SFX_SND1`. The program will play the sound once every 100 frames:

```cpp

#include <gba_interrupt.h>
#include <gba_systemcalls.h>
#include <maxmod.h>
#include "soundbank.h"
#include "soundbank_bin.h"


int main(void) {
    irqInit();
    irqEnable(IRQ_VBLANK);
    irqSet(IRQ_VBLANK, mmVBlank);

    mmInitDefault((mm_addr)soundbank_bin, 8);


    int count = 10;

    while (1) {
        mmFrame();
        VBlankIntrWait();

        count++;


        if (count % 100 == 0){
            mmEffect( SFX_SND1 );
        }
    }
}
```

The first part of the program includes the required maxmod and soundbank headers:

```cpp
#include <maxmod.h>
#include "soundbank.h"
#include "soundbank_bin.h"
```

Next, the `mmVBlank` is set as an interrupt handler:

```cpp
irqInit();
irqEnable(IRQ_VBLANK);
irqSet(IRQ_VBLANK, mmVBlank);
```

Maxmod is then initialised with 8 audio channels:

```cpp
mmInitDefault((mm_addr)soundbank_bin, 8);
```
The `mmFrame()` function needs to be called just before the interrupt wait to keep the sound in sync with the game:

```cpp
 while (1) {
    mmFrame();
    VBlankIntrWait();
    // ...
}
```

Finally to play the sound the `mmEffect()` function is called with the name of the effect from the `soundbank.h` header:

```cpp
mmEffect( SFX_SND1 );
```

If you want to play a song instead of a short sound effect, the `mmStart()` function can be used. The function takes the music file name from the `soundbin.h` header and argument for how it should repeat. 

The options are `MM_PLAY_LOOP` which repeats a song until it is stopped, or `MM_PLAY_ONCE` which plays it once.

Here's the program earlier modified to play a song instead:

```cpp
#include <gba_interrupt.h>
#include <gba_systemcalls.h>
#include <maxmod.h>
#include "soundbank.h"
#include "soundbank_bin.h"


int main(void) {
    irqInit();
    irqEnable(IRQ_VBLANK);
    irqSet(IRQ_VBLANK, mmVBlank);

    mmInitDefault((mm_addr)soundbank_bin, 8);


    int count = 10;

    mmStart( MOD_MUS0, MM_PLAY_LOOP );

    while (1) {
        count++;

        mmFrame();
        VBlankIntrWait();
    }
}
```

To stop and resume music playbook (e.g. when you press the pause button on the game) you can use the `mmPause()` and `mmResume()` functions.


Sound effects have a number of different attributes which can be set to control their playback. These attributes can be accessed by creating a new `mm_sound_effect()` object.

For example:

```cpp
mm_sound_effect sound;
sound.id = SFX_SND1;
sound.rate = 512; // 1024 is original speed. This also affects pitch
sound.volume = 200; 
sound.panning = 0; //0 is left, 128 is centered, 255 is right

``` 

The use of these attributes allows the same sound to be used in different ways. For example changing the pitch of a sound allows it totally different noise to be created.

After a sound has started playing it can be stopped with the `mmEffectCancel()` function.

```cpp
mm_sfxhand sound = mmEffect( SFX_SND1 );
mmEffectCancel(sound);
```

There are also other controls for the sound while it is playing:

```cpp
// Copied from maxmod docs:
sound = mmEffect( SFX_SND1 );

// Change pitch to +1 octave
mmEffectRate( sound, 1024*2 );

// Change volume to half level (128/255)
mmEffectVolume( sound, 128 );

// Change panning to far-right
mmEffectPanning( sound, 255 );
```


## Save Data

Save data on the GBA is stored on the cartridge in SRAM. The amount of space that is available for saves depends on the cartridge. Some cartridges have no built-in SRAM, meaning no save data can be saved. Others with built-in SRAM generally vary in size between 4KB and 64KB. 64KB is general max size for save storage, however Some cartridges such as those used by Pokemon Ruby and Sapphire used advanced techniques execeed this limit with a size of 1Mbit (approx 122 KB). 

Cartridges generally tend to use one of two types of storage. An SRAM/FRAM chip which requires an additional battery to maintain the save date. If the battery runs out (which can take years), then the save data is lost. The other type of memory, EEPROM, does not require a battery and will stores the data almost indefinitely. 

For the sake of simiplicity, this section refers to save memory as SRAM, irrelevant of whether it uses a SRAM/FRAM chip of EEPROM.


When programming games on the GBA, the memory location between `0x0E000000` and `0x0E00FFFF` are available for writing and reading save data. This is 64KB of memory. Although these locations are available doesn't mean that all of the addresses can be used. Which addresses are available depends on the size of the SRAM installed in the cartridge. 

Irrelevant of the size of the SRAM available, the first memory address will always be `0x0E000000`.

Only 8 bits of data can be written to SRAM at once. So if you write 16 bit or 32 bit values, everything past the first 8 bits will be lost.

Here's a program that fills the screen with a number of blue lines. By pressing down you can fill more of the screen, by pressing up you fill less of the screen. The save feature allows you to save the number of lines being drawn by pressing select and load that save by pressing start: 

```cpp
#include <gba_interrupt.h>
#include <gba_systemcalls.h>
#include <gba_video.h>
#include <gba_input.h>


u8 *saveMemory = ((u8*)0x0E000000);
u16 keys_pressed = 0;

void fillScreen(u8 rows){
    for (int i = 0; i < SCREEN_HEIGHT; ++i)
        {
            u16 colour = 0x0000;
            if (i > rows){
                colour = 0x6666;
            }

            for (int j = 0; j < SCREEN_WIDTH; ++j)
            {

                MODE3_FB[i][j] = colour;
            }
        }
}

u16 keyPressed(u16 keyCode){
    return keyCode & keys_pressed;
}

int main(void) {
    irqInit();
    irqEnable(IRQ_VBLANK);

    SetMode(MODE_3 | BG2_ON);

    u8 rows = 100;

    while (1) {
        VBlankIntrWait();
        scanKeys();

        keys_pressed = keysHeld();

        if (keyPressed(KEY_UP)){
            rows -= 1;
        } else if (keyPressed(KEY_DOWN)) {
            rows += 1;
        } 

        if (keyPressed(KEY_SELECT)){
            saveMemory[0] = rows;
        } else if (keyPressed(KEY_START)) {
            rows = saveMemory[0];
        }

        fillScreen(rows);
    }
}
```

There are three lines that are important for saving and loading. First we define a pointer to save memory:

```cpp
u8 *saveMemory = ((u8*)0x0E000000);
```

Then when the select key is pressed we copy the value of the `rows` variable into index/address 0 of `saveMemory`:

```cpp
if (keyPressed(KEY_SELECT)){
            saveMemory[0] = rows;
```

Then when the start button is pressed we load the value in memory back into the `rows` variable:

```cpp
else if (keyPressed(KEY_START)) {
    rows = saveMemory[0];
}
```













































































## Sprite and Background Attributes




### Sprites

#### Creating a Sprite

[libgba struct]

#### attr0

| bit | setting |
| --- | ---     |
| 0   | Something | 
| 1   | Something |
| 2   | Something |
| 3   | Something |
| 4   | Something |
| 5   | Something |
| 6   | Something |
| 7   | Something |
| 8   | Something |
| 9   | Something |
| A   | Something |
| B   | Something |
| C   | Something |
| D   | Something |
| E   | Something |
| F   | Something |


X setting (bits 0-??) options:
- [ligba constant]: explanation
- [ligba constant]: explanation


#### attr1

| bit | setting |
| --- | ---     |
| 0   | Something | 
| 1   | Something |
| 2   | Something |
| 3   | Something |
| 4   | Something |
| 5   | Something |
| 6   | Something |
| 7   | Something |
| 8   | Something |
| 9   | Something |
| A   | Something |
| B   | Something |
| C   | Something |
| D   | Something |
| E   | Something |
| F   | Something |


X setting (bits 0-??) options:
- [ligba constant]: explanation
- [ligba constant]: explanation

#### attr2

| bit | setting |
| --- | ---     |
| 0   | Something | 
| 1   | Something |
| 2   | Something |
| 3   | Something |
| 4   | Something |
| 5   | Something |
| 6   | Something |
| 7   | Something |
| 8   | Something |
| 9   | Something |
| A   | Something |
| B   | Something |
| C   | Something |
| D   | Something |
| E   | Something |
| F   | Something |

X setting (bits 0-??) options:
- [ligba constant]: explanation
- [ligba constant]: explanation


### Backgrounds


| bit | setting |
| --- | ---     |
| 0   | Something | 
| 1   | Something |
| 2   | Something |
| 3   | Something |
| 4   | Something |
| 5   | Something |
| 6   | Something |
| 7   | Something |
| 8   | Something |
| 9   | Something |
| A   | Something |
| B   | Something |
| C   | Something |
| D   | Something |
| E   | Something |
| F   | Something |

X setting (bits 0-??) options:
- [ligba constant]: explanation
- [ligba constant]: explanation



| libgba | Value | Size (in Tiles) |
|---|---|---|
| ??? | 00 | 32x32 |
| ??? | 01 | 64x32 |
| ??? | 10 | 32x64 |
| ??? | 11 | 64x64 |














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

