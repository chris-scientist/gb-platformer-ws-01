---
layout: step
title: "Free fall and platforms"
permalink: /en/free-fall-and-platforms/
next: /en/interactions-with-world/
link-other-lang: /fr/chute-libre-et-plateformes/
lang: en
---

# Step 3 - Free fall and platforms

## Introduction

In this third step, we will implement:
* the free fall;
* and platform management.

*I invite you to download <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v2.0.zip" class="external-link" >the code</a> which is the result of the second step in order to start on a common basis.*

## Free fall

Let's implement a first version of the free fall. Indeed, we will adapt it after adding the platforms.

In `PhysicsEngine.h`, let's add the prototype of the `gravity` function :

<div class="filename" >PhysicsEngine.h</div>
```
void gravity(Character &aCharacter);
```

Let's implement the `gravity` function :

<div class="filename" >PhysicsEngine.cpp</div>
```
// Free fall
void gravity(Character &aCharacter) {

  const uint8_t platformId = isOnOnePlatform(aCharacter);
  if( platformId == NO_ID ) {
    // Free fall
    aCharacter.oldY = aCharacter.y;
    aCharacter.vy += GRAVITY;
    aCharacter.state = FREE_FALL_STATE;
    aCharacter.y += aCharacter.vy;
  } else {
    // in contact with a structure
    rectifyPositionY(aCharacter);
    aCharacter.vy = 0;
    aCharacter.state = ON_THE_PLATFORM_STATE;
  }
  
}
```

In the main program, in the state `PLAY_STATE` let's replace `gb.display.println("GRAVITY");` with `gravity(hero);`.

<div class="filename" >GBPlatformer01.ino <span>/!\ Horizontal scrolling /!\</span></div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    case HOME_STATE:
      stateOfGame = paintMenu();
      break;
    case LAUNCH_PLAY_STATE:
      // ...
      break;
    case PLAY_STATE:
      if(hero.state == ON_THE_PLATFORM_STATE) {
        stateOfGame = manageCommands(hero);
      }

      if(hero.state != JUMP_STATE && hero.state != PUSH_FOR_JUMP_STATE) {
        gravity(hero);
      } else if(hero.state == JUMP_STATE || hero.state == PUSH_FOR_JUMP_STATE) {
        jump(hero);
      }

      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
)
```

Upload the program to your console and see the free fall (by jumping to the edge of the screen).

![Free fall](./../../img/E03/chute_libre_v1.gif)

The basics of free fall are now implemented, let's move on to platform management.

## The platforms

### Creation of the platforms

Before creating the platform structure, let's add constants in `Constants.h` :

<div class="filename" >Constants.h</div>
```
// Box colors
// ...
const Color GROUND_COLOR = BROWN;
const Color PLATFORM_T_COLOR = LIGHTGREEN;
const Color PLATFORM_NOT_T_COLOR = GREEN;

// Number of the platforms
const uint8_t NB_OF_PLATFORMS = 4;

// Platform IDs
// ...
const uint8_t ID_PLATFORM_1 = 2;
const uint8_t ID_PLATFORM_2 = 3;
const uint8_t ID_PLATFORM_3 = 4;

// Type of platforms
const uint8_t NO_PLATFORM_TYPE = 0;
const uint8_t GROUND_TYPE = 1;
const uint8_t PLATFORM_TYPE = 2;

// For a platform 4 pixels high by 8 pixels wide
const uint8_t HEIGHT_PLATFORM = 4;
const uint8_t WIDTH_PLATFORM = 8;
const uint8_t OVER_CENTER_X_PLATFORM = 4;
const uint8_t OVER_CENTER_Y_PLATFORM = 2;

// For the ground 4 pixels high by 8 pixels wide
const uint8_t HEIGHT_GROUND = 4;
const uint8_t WIDTH_GROUND = 8;
const uint8_t OVER_CENTER_X_GROUND = 4;
const uint8_t OVER_CENTER_Y_GROUND = 2;
```

In the file `Platform.h`, let's add among other things the structure managing the platforms :

<div class="filename" >Platform.h <span>/!\ Horizontal scrolling /!\</span></div>
```
#ifndef PLATFORMER_PLATFORM
#define PLATFORMER_PLATFORM

#include <Gamebuino-Meta.h>
#include "Constants.h"

struct Platform {
  int8_t x; // ................ x-position of the beginning of the platform (center of the first platform)
  int8_t y; // ................ y-position of the platform (center of the platform)
  uint8_t lengthPlatform; // ... length of the platform in block (must be at least equal to 2), size of the blocks in the constant file
  uint8_t type; // ............. type of platforms : see constants
  uint8_t id; // ............... (unique) platform identifier
  bool isGoThrough; // ......... true to indicate that the character can pass through the platform, otherwise false
};

#endif
```

Still in `Platform.h` let's add the prototype of the `createPlatform` function which allows to create a platform :

<div class="filename" >Platform.h</div>
```
Platform createPlatform(
  int8_t aX, int8_t aY, 
  uint8_t aLength, 
  uint8_t aType, 
  uint8_t aId, 
  bool goThrough
);
```

In the file `Platform.cpp`, let's add the definition of the function `createPlatform` :

<div class="filename" >Platform.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
#include "Platform.h"

Platform createPlatform(int8_t aX, int8_t aY, uint8_t aLength, uint8_t aType, uint8_t aId, bool goThrough) {
  // We force the size of the platform to 2 blocks
  if(aLength < 2) {
    aLength = 2;
  }

  // The platform type is forced to a floating platform
  if(aType < 1 || aType > 2) {
    aType = PLATFORM_TYPE;
  }

  // Creation of the platform
  Platform aPlatform;
  aPlatform.x = aX;
  aPlatform.y = aY;
  aPlatform.lengthPlatform = aLength;
  aPlatform.type = aType;
  aPlatform.id = aId;
  aPlatform.isGoThrough = goThrough;

  return aPlatform;
}
```

In `Platform.h`, let's add the prototype of the `initPlatforms` function that will initialize the platforms :

<div class="filename" >Platform.h</div>
```
void initPlatforms(Platform * aSet);
```

As you saw in the constants, we will create four platforms, including the ground.

In `Platform.cpp`, let's define the function `initPlatforms`:

<div class="filename" >Platform.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void initPlatforms(Platform * aSet) {
  aSet[0] = createPlatform(4, 62, 10, GROUND_TYPE, ID_GROUND, false);
  aSet[1] = createPlatform(4, 50, 2, PLATFORM_TYPE, ID_PLATFORM_1, true);
  aSet[2] = createPlatform(35, 40, 2, PLATFORM_TYPE, ID_PLATFORM_2, false);
  aSet[3] = createPlatform(68, 30, 2, PLATFORM_TYPE, ID_PLATFORM_3, false);
}
```

In the main program, apart from the `setup` and `loop` functions, let's add an array of platforms:

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes
#include "Platform.h"

// Other globals variables
Platform setOfPlatforms[NB_OF_PLATFORMS];

void setup() {
  // ...
}

void loop() {
  // ...
}
```

Don't forget to include `Platform.h` in your main program.

Still in the main program, especially in the `LAUNCH_PLAY_STATE` report, let's initialize the platforms :

<div class="filename" >GBPlatformer01.ino <span>/!\ Horizontal scrolling /!\</span></div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    case HOME_STATE:
      stateOfGame = paintMenu();
      break;
    case LAUNCH_PLAY_STATE:
      initPlatforms(setOfPlatforms); // ... we reset the platforms
      initCharacter(hero);
      stateOfGame = PLAY_STATE;
      break;
    case PLAY_STATE:
      // ...
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

### Display of platforms

**Work in progress...**
