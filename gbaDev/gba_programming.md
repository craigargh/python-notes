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

Each block is 16kb in size, making the total 96kb for tile storage. These blocks are referred to as "tile blocks" or "char blocks". [THIS SEEMS INCORRECT COMPARED TO OTHER SOURCES AND THE VALUE IN THE CODE - IT MIGHT BE 32 KILOBYTES IN MODES 0-2, 16KILOBYTES IN MODES 3-5]


### Sprites

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
-------------------------------------
| FE |  Shape of Sprite: 00 = Square, 01 = Tall, 10 = Wide |
| D  | Colour Mode: 0 = 4bpp, 1 = 8bpp |
| C | Not used today |
| AB | Not used today |
| 89 | Not used today |
| 7654 3210 | Y Coordinate |


And the values for `attr1`:


| Attr 1 | 0x FEDC BA98 7654 3210 |
------------------------------------
| FE | Sprite Size (discussed below) |
| DCBA98 | Not Used Today |
| 7654 3210 | X coordinate |

The dimensions of a sprite is based on two values, bits FE in `attr0` for shape and bits FE in `attr1` for size. These two values are combined to find the dimensions, which are summarised in this table:


| Size 00 | Size 01 | Size 10 | Size 11 |
-----------------------------------------
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

