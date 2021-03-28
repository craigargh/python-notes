# Game Algorithms

## Game Programming Algorithms and Techniques

Notes taken from Game Programming Algorithms and Techniques by Sanjay Madhav.


### Chapter 1: Game Loop and Objects

**Game loops** are used to update the game in a cycle. They repeat continuously while the game is running. There are three main stages in a game loop:
1. Read inputs
1. Update game state
1. Generate outputs

One iteration of the game loop is referred to as a single frame.

For single threaded games, such as most retro games, there is only one processor available so the game loop usually follows this pattern. For multi-threaded games that have access to multople processors it is common for updating the game state and generating graphics to be managed on two or more separate threads.

Timing in games is important. The number of times a game loop runs is measured in frames per second (fps). For example 60 fps means that the game loop completes 60 times a second, whereas 30fps means that it completes 30 times a second. 

Game logic can behave weirdly is running at a different framerate. For example moving a game from 30 fps to 60fps will cause the game to appear to run at twice the speed, meaning the player's character will move twice as fast. There are several techniques to manage these timing issues:

- **Delta time:** a variable is used to measure the time since the last frame. This can be used to scale different variables in the game logic. For example it can be used to manage how many pixels the player character moves in a single frame. Irrelevant of whether the game is running at 30fps or 60fps, the character will still move a consistent distance per second.
- **Frame limiting:** The game will have a maximum frame rate. If the loop finishes processing early then the game loop will wait before moving onto the next frame

An example of using delta time to move a character:
```python

while True:
    time_delta = # time since last frame
    player_distance = 10 * time_delta

    player.move(player_distance)
```

An example using frame limiting:

```python

while True:
    player.move(10)

    while not frame_interrupt:
        sleep(0.0001)
```

Anything in a game that needs to be updated and/or drawn is called a **Game object**. This is different to an object in a programming language.

There are three primary types of game objects:
- Updateable and drawable: These are objects that can be updated and drawn, such as the player's character or enemies
- Static Objects: These are objects that are drawn, but not updated. Such as a floor tile or building.
- Triggers: Triggers are onjects that are updated, but not drawn. For example the game's camera or a location that will spawn monsters when the player walks over it.

Game objects can be represented with class inheritance in programming languages. For example you can have a `GameObject` base class and two interfaces, `Updateable` and `Drawable` that have `update()` and `draw()` methods respectively. 

The game objects can then be stored in lists as part of the game world. For example:

```python
class GameWorld:
    drawableObjects = []
    updateableObjects = []
```

This is what the game loop would look like with the objects implementation:

```python
while True:
    input = readInput()

    delta = getTimeDelta()

    for updateable in game_world.updateableObjects:
        updateable.update(delta)

    for drawable in game_world.drawableObjects:
        drawable.draw(delta)

    limitFrameRate()
```


### Chapter 2: 2D Graphics

2D graphics are made up of pixels. 

Pixels are usually drawn onto screens using **scanlines**. Each scanline draws a single row of pixels onto the screen, one at a time. After the last scan line has been drawn onto the screen the display moves back up to the top row to start again. This move is called a vertical blank or a **VBLANK**. 

In single threaded games, a single frame will be drawn per screen drawing cycle. So each VBLANK corresponds to a single frame.

Pixels that are being drawn to the screen are usually stored in a **buffer**. It is important not to change the values in this buffer as it is being drawn to the screen otherwise it can result in screen tearing. Thefore a technique called **double buffering** is used where there are two buffers: one that is currently being written to the screen, and another which can be modified by the game loop to contain the data the next frame. When the VBLANK occurs, these two buffers swap so that the next frame is written to the screen and so on. 

Using double buffering can lead to **input lag** of one frame, though this is not noticeable by most players when games are running at a high frame rate.


A **sprite** is a 2D graphic in a game. It can represent the player, enemies, level objects and so on. Sprites can be animated by showing different images per frame.

When drawing sprites it is a good idea to have a draw order. This allows sprites to be layered on top of each other. When rendering these sprites they are then drawn from front-to-back based on the priority of each sprite. This is known as the **painter's algorithm**.

When implementing sprites using classes, sprites are a type of game object so they should have a `draw()` and `update()` method.


An example implementation of game object update and draw with a buffer on the GBA:


`gameobjects.hpp`
```cpp
#include <gba_base.h>
#include <gba_sprites.h>

class Sprite {

public:
    u16 tileIndex;
    u8 width;
    u8 height;
    u8 x;
    u8 y;
    OBJATTR* oam;

    Sprite(OBJATTR* oamRef, const u16 tileIndexStart, u8 xPos, u8 yPos){
        oam = oamRef;
        tileIndex = tileIndexStart;
        x = xPos;
        y = yPos;
    };

    void draw(){
        oam->attr0 = OBJ_16_COLOR | ATTR0_SQUARE | OBJ_Y(y);
        oam->attr1 = ATTR1_SIZE_16 | OBJ_X(x);
        oam->attr2 = tileIndex | ATTR2_PALETTE(0);
    };

    void update(){
        x += 1;
    };
};
```

`game.cpp`
```cpp
#include <vector>

#include <string.h>
#include <gba_video.h>
#include <gba_systemcalls.h>
#include <gba_interrupt.h>
#include <gba_sprites.h>

#include "asuka.h"
#include "gameobjects.hpp"

OBJATTR oam_backbuffer[128];


void uploadPaletteMem(){
    CpuFastSet(asukaPal, SPRITE_PALETTE, (asukaPalLen >> 2) | COPY32);
}

void uploadTileMem(){
    CpuFastSet(asukaTiles, TILE_BASE_ADR(5), (asukaTilesLen >> 2) | COPY32);
}

void initGameboy(){
    irqInit();
    irqEnable(IRQ_VBLANK);

    SetMode(MODE_0 | OBJ_ENABLE | OBJ_1D_MAP);

    uploadPaletteMem();
    uploadTileMem();
}

Sprite makeCharacterSprite(){
    u16 tileIndex = 512;
    u8 x = 0;
    u8 y = (SCREEN_HEIGHT >> 1) - 8;

    Sprite character (&oam_backbuffer[0], tileIndex, x, y);
    return character;
};

int main() {
    initGameboy();

    std::vector<Sprite> sprites;
    sprites.push_back(makeCharacterSprite());

    while(1){
        for (u8 i = 0; i < sprites.size(); i++) {
            sprites[i].update();
        }
        
        for (u8 i = 0; i < sprites.size(); i++) {
            sprites[i].draw();
        }

        VBlankIntrWait();
        CpuFastSet(oam_backbuffer, OAM, ((sizeof(OBJATTR)*128)>>2) | COPY32);
    }
    return 0;
}
```

Sprite animations are similar to traditional flip books. Animation is created by changing images between frames to create the illusion of movement. 

An animated sprite class requires extra attributes compared to a non-animated sprite class. This is usually represented as an array of animation data objects. The animation data object contains data on the start frame and the number of frames. This is then wrapped in another class that also includes image data.

In order for the animation to work, the sprite needs to keep track of frames using an attribute. It is also wise for the sprite class to provide `updateAnimation()` and `changeAnimation()` methods. 

[IMPLEMENT THIS ON THE GBA]

**Sprite sheets** are an optimised way to store images that are used by games. Instead of loading a single image for each sprite and each animation for that sprite, a sprite sheet contains all of the images for several sprites in a single file.


