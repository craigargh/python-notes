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

This example builds on the previous sprite code to implement animations

```cpp
class AnimationFrame{
public:
    u8 duration;
    u8 tileOffset;

    AnimationFrame(u8 tile, u8 length){
        tileOffset = tile;
        duration = length;
    }
};

class Animation {
public:
    u16 tileIndex;
    std::vector<AnimationFrame> frames;
    u8 frame;

    Animation(){};

    Animation(u16 tileIndexValue){
        tileIndex = tileIndexValue;
        frame = 0;
    };

    void tick(){
        frame ++;
    }

    u16 tile(){
        u8 frameIndex = 0;
        u8 cumulativeFrames = 0;

        for(u8 i = 0; i <= frames.size(); i++){
            if (i == frames.size()){
                frame = 0;
                frameIndex = 0;
                break;
            }

            cumulativeFrames += frames[i].duration;

            if (frame < cumulativeFrames){
                frameIndex = i;
                break;
            }
        }

        return tileIndex + (frames[frameIndex].tileOffset * 4);
    }
};
```

main

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

Sprite makeCharacterSprite(u16 baseTile, u8 x, u8 y, u8 bufferIndex){
    Animation animation (baseTile);
    animation.frames.push_back(AnimationFrame(0, 8));
    animation.frames.push_back(AnimationFrame(1, 10));
    animation.frames.push_back(AnimationFrame(0, 8));
    animation.frames.push_back(AnimationFrame(2, 10));

    Sprite character (&oam_backbuffer[bufferIndex], x, y);
    character.animation = animation;
    return character;
};

int main() {
    initGameboy();

    u8 halfWidth = SCREEN_WIDTH >> 1;
    u8 halfHeight = SCREEN_HEIGHT >> 1;

    std::vector<Sprite> sprites;
    sprites.push_back(makeCharacterSprite(512, halfWidth - 24, halfHeight - 8, 0));
    sprites.push_back(makeCharacterSprite(524, halfWidth - 8, halfHeight - 8, 1));
    sprites.push_back(makeCharacterSprite(536, halfWidth + 8, halfHeight - 8, 2));

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



**Sprite sheets** are an optimised way to store images that are used by games. Instead of loading a single image for each sprite and each animation for that sprite, a sprite sheet contains all of the images for several sprites in a single file.


## Chapter 5: Input


Input allows interactivity with games.

A **digital** input is an input type that is either on or off, for example a button

An **analog** input is an input type that has a range of values, such as a joystick.

Inputs can be processed as: 
- Individual events: such as a single button press
- Chords: multiple button pressed together
- Sequences: multiple buttons pressed in a specific order


To make easier to represent key value, a Key code is a type of variable that maps to a specific key. When the key is pressed, the keycode can be used to check whether the key is pressed.

If we just checked if a key was pressed every frame, we may end causing an action to trigger to frequently. For example if the user presses the start button to open a menu, they'd need to press it exactly for one frame otherwise the menu opening and closing would trigger for each frame the button is pressed. This is not an ideal situation.

Key states are used to record whether a key is pressed, released or held. To do this the code records the current state and previous state of the key to work out whether the key is just pressed, just released, held or still released. There are different ways to represent this in the code, including as an enum.


One way to get input is to poll the input each frame and return the state of each key after they are polled.


Event based input is used as a way of decoupling the polling of input from the actions required as a result of the input. Instead of each action that requires input individually polling the input each frame, the input is polled once per frame and events are dispatched to trigger any actions that rely on the input.

The basic components of an event-based input system:
- Input Bindings: A map of keycodes to the event names that they trigger.
- Event Bindings
  - Event: The type of event triggered by the input, such as "up_pressed". Usually stored as a map, with event as the key, while state and callable are stored in a list of tuples.
  - State type: The state that the event is applicable for. For example certain bindings are only applicable when a menu is open, while others will apply while the main game is running.
  - Callable: A function of method that will be called when the event triggers. For example open a menu or make an attack.
- Poll Input:
  - Iterates through each key binding to check if they are pressed
  - If they are pressed filters the event bindings that match the current game state (this step is necessary to prevent bugs where an event changes the game state and accidentally causes multiple actions to run)
  - Iterates through each active event binding to call the action
- Add Binding: A method for registering a new event binding


## Chapter 6: Sound

The number of sounds that can be played simultaneously is determined by the number of **channels** available in the audio engine. Therefore to ensure that the most important sounds are played, a prioritisation system can be used. Each sound receives a priority to determine which sounds are played when there are more sounds to play than channels.

**Source data** refers to the original sound files. There are two main methods to play the sounds:
- Load them into memory if the sound file is short
- Stream the sound file if it is a larger file

**Sound cues** are what triggers sounds to play. Instead of referring directly to a sound file in every location in the code, the code refers to a sound cue, which then maps to one or more sound files that can be play. This decouples the parts of the code that trigger the sounds from the actual sound files that are played.

This is an example data structure for storing a sound cue:

```json
{
    "name": "sword_hit",
    "priority": "1",

    "sources": [
        "sword_1.wav",
        "sword_2.wav",
    ]
}
```

Additional metadata can be stored in the data structure, such as falloff, which detemines the volume of a sound based on how far away the player is from the source.

This can be represented as a class as well.

```cpp
class SoundCue {
    std::string name;
    unsigned int priority;

    std::vector sources;

    void play();
};
```


The class declares a `play()` method, which will randomly select which sound file to play if there are multiple sources.

For some sound cues the sound that is played is context specific. For example a sword that hits a stond wall will sound different to a sword that hits squishy mud. One approach for this is to have a map of sound types to sound sources. 

This is what the structure might look like as json:

```json
{
    "name": "sword_hit",
    "priority": "1",

    "sources": {
        "stone": [
            "sword_stone_1.wav",
            "sword_stone_2.wav",
        ],
        "mud": [
            "sword_mud_1.wav",
            "sword_mud_2.wav",
        ]
    }
}
```

Here's a declaration for a `SwitchableSoundCue` class:

```cpp
class SoundCue {
    std::string name;
    unsigned int priority;

    std::map<std::string, std::vector<std::string>> sources;

    void play();
};
```

Certain sounds will have different volumes depending on how far away they are. The entity that makes the sound is known as the **emitter**, while the entity that hears it is called the **listener**. The listener is usually the player's character. To calculate the volume of the sound, the distance between the listener and emitter is calculated, taking into account the sound's **dropoff** value.

**Digital Sound Processing** (DSP) is used to maninpulate sounds using software. Adding reverb or changing the pitch of a sound are two examples of DSP effects.


## Chapter 3: Linear Algebra

Linear algebra is used widely in game development. Although linear algebra is a very large field of mathematics, only knowledge of a small subset is essential for game development. Specifically vectors and matrices.

A **vector** is a representation of magnitude and direction. Vectors can be in any number of dimensions, though in games they are usually 2D or 3D. 

A 2D vector is made up of two values, `x` and `y`, which are usually decimal values.

```cpp
class Vector2D{
    float x;
    float y;

    Vector2D(float xValue, float yValue)(
        x(xValue),
        y(yValue)
    ){};
};
```

Vectors do not represent a position. They represent a magnitude. Two vectors are identical if they have the same magnitude and length, irrespective of their position.

### Addition

Vector addition is calculated by adding together each dimension of the two vectors. This is an implementation of adding two 2D vectors:

```cpp
float add2DVectors(Vector2D a, Vector2D b){
    return Vector(a.x + b.x, a.y + b.y);
};
```
This could also be implemented on the `Vector2D` class by overloading the `+` and `+=` operators:

```cpp
class Vector2D{
    float x;
    float y;

    // ...

    Vector2D operator+(const Vector2D& b){
        return return Vector(x + b.x, y + b.y);
    };

    Vector2D operator+=(const Vector2D& b){
        x += b.x;
        y += b.y;

        return this;
    };
```

### Subtraction

Subtraction is similar to addtion, though the order of subtraction is important. The dimensions of the second vector are substracted from those of the first:

```cpp
float subtract2DVectors(Vector2D a, Vector2D b){
    return Vector(b.x - a.x, b.y - a.y);
};
```

This can also be implemented by overloading the `operator-` and `operator-=` methods.

Subtraction allows you to calculate a new vector between two points.


### Vector Length

The length a vector can be calculated using Pythagoras' theorem. This is a function to calculate the length of a 2D vector:

```cpp
#include <math.h>

float length2DVector(Vector2D a){
    return sqrt(pow(a.x, 2) + pow(a.y, 2));
};
```

Square root can be quite a taxing operation on the CPU. 

When comparing the length of two vectors, an optimisation can be used to avoid the use of the square root. The solution is just to use the length squared for the comparison:

```cpp
float lengthSquared2DVector(Vector2D a){
    return pow(a.x, 2) + pow(a.y, 2);
}

bool isLengthGreater(Vector2D a, Vector2D b){
    return lengthSquared2DVector(a) > lengthSquared2DVector(b);
}
```
Using length squared will return the same result as fully calculating length when comparing two vectors.

A **unit vector** is a vector whose length has been **normalised** to a length of 1. To normalise a vector, each dimension is divided by the length of the vector:

```cpp
Vector2D normalise(Vector2D a){
    float length = length2DVector(a);

    return Vector2D(a.x / length, a.y / length);
}
```

Magnitude information is lost when a vector is normalised.

Division can also be very taxing for CPUs, especially with floating point numbers. The Quake source code implemented a work around for this by using an approximate square root function that uses multiplication instead of division:

```cpp
float quake_sqrt(float number){
    int i;
    float x2, y;
    const float threehalfs = 1.5F;

    x2 = number * 0.5F;
    y = number;

    i = * (int *) &y;

    i = 0x5f3759df - (i >> 1);

    y = * (float * ) &i;
    y = y * (threehalfs - (x2 * y * y));

    return y;
}
```

I have no idea how this works, but it gives a good approximate square root value.

### Scalar Multiplication

The length of a vector can be increased by multiplying it be a scalar, which can be a integer or decimal value.

```cpp
Vector2D scale(Vector2D a, float scalar){
    return Vector2D(scalar * a.x, scalar * a.y);
}
```

Multiplying by a positive scalar value will increase the magnitude of the vector. Multiplying by a negative scalar value will invert the direction of the vector.

### Vector Multiplication: Dot Product

There are two ways to multiply vectors: **dot product** and **cross product**. The dot product results in a scalar, while the cross product results in a vector.

```cpp
float dotProduct(Vector2D a, Vector2D b){
    return (a.x * b.x) + (a.y * b.y);
}
```

Dot product is commonly used as part of the formula to calculate the angle between two vectors. The angle is calculated by normalising the vectors, calculating the dot product, then calculating the arc cosine of the result:

```cpp
float angle(Vector2D a, Vector2D b){
    Vector2D normalisedA = normalise(a);
    Vector2D normalisedB = normalise(b);

    float dotP = dotProduct(normalisedA, normalisedB);

    return acos(dotP);
}
``` 

Alternatively the angle can be calculated by calculating the dot product, dividing that they the lengths of the vectors multiplied together, the calculating the arc cosine of the result:

```cpp
float angle(Vector2D a, Vector2D b){
    float dotP = dotProduct(a, b);
    float abLength = length2DVector(a) * length2DVector(b);

    float result = dotP / abLength;

    return acos(result);
}
```

If the dot product of two vectors is 0, then the angle between the two is 90 degrees. If the dot product is 1, then the vectors are parallel, if it is -1 they are parallel but facing the opposite direction.

A **scalar projection** can be used to create a right angled triangle between two vectors. One side should be a vector and the other a unit vector. Calculating the dot product between these two vectors will return a scalar. Multiplying the unit vector by the scalar will align the two vector to form a right angled triangle. 

By subtracting the two vectors from one another a vector for the third side can be calculated. The third side which will be at 90 degrees to the vector that was originally a unit vector.

Here are the steps for a scalar projection:
1. Take two vectors
1. Convert one to a unit vector
1. Calculate the dot product to get a scalar
1. Multiply the unit vector by the scalar

```cpp
Vector scalarProjection(Vector2D a, Vector2D toProject){
    Vector2D unitVector = normalise(toProject);

    float scalar = dotProduct(a, unitVector);

    return scale(unitVector, scalar);
}
```

### Vector Reflection

In games such as pong, vector dot products can be applied to calculated the angle that the ball bounces off an object. To do this we need to get the vector that the ball hit the object and then calculate the inverse of that vector.

If the object that is hit is parallel to the x or y axis of the co-ordinates system it is easy to calculate the inverse. If the object that is hit is parallel to the y-axis, then the x dimension of the vector is simply multiplied by -1. Likewise if the object that is hit is parallel to the x-axis, then the y dimension of the vector is multiplied by -1.

If the object that is hit is not parallel to an axis, the calculation is more complex.

These are the steps:
1. Calculate a vector perpendicular to the one being hit
1. Convert the perpendicular vector to a unit vector
1. Invert the direction of the vector hitting the wall
1. Calculate the scalar projection for the perpendicular vector and scale it
1. Subtract the ???

[COME BACK TO THIS]


### Vector Multiplication: Cross Product

A cross product is another way to multiply two vectors. Cross product is primarily used to calculated a vector against a plane in 3D. Though it does have some application for 2D vectors, as it can be used to determine the direction of a rotation.

When working with 2D vectors, they need to be converted to 3D vectors by setting the third dimension to 0.

To calculate the cross product of two vectors:

```cpp
Vector2D crossProduct(Vector2D a, Vector2D b){
    float x = (a.y * b.z) - (a.z * b.y);
    float y = (a.z * b.x) - (a.x * b.z);
    float z = (a.x * b.y) - (a.y * b.x);
}
```

The direction of the vector is determined by the right hand rule [ADD MORE DETAIL ABOUT THIS]

When working with a 2D vector, the cross product can be used to determine the direction of the angle calculated using the dot product. If the z dimension is positive, the rotation is counter-clockwise, whereas if it is negative, the rotation is clockwise.

### Linear Interpolation

Linear interpolation (also called lerp) is used to calculate a point inbetween two other points. For example it can be used to calculate the point that is 10% of the distance along a line.

Linear interpolation for two 2D points can be calculated as (I think this is correct):

```cpp
Vector2D lerp(Vector2D a, Vector2D b, Float fraction){
    Vector2D aScaled = scale(a, 1 - fraction);
    Vector2D bScaled = scale(b, f);

    return aScaled + bScaled;
};
```

## Matrices

Matrices are primarily used in 3D games.

[COME BACK TO THIS]


# Chapter 7: Physics

## Planes, Rays and Line Segments

A plane is a flat surface.



## Collision Geometry




## Collision Detection




## Physics-Based Movement




# Chapter 9: Artificial Intelligence



## Pathfinding



## State-Based Behaviour



## Strategy and Planning





