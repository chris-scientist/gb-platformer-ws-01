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

Let's start by including `Platform.h` in `Display.h`.

In `Display.h`, let's add the prototype of the `paintGround` function :

<div class="filename" >Display.h</div>
```
void paintGround(const int8_t aX, const int8_t aY);
```

In `Display.cpp`, let's define the function `paintGround` :

<div class="filename" >Display.cpp</div>
```
// Performs ground display
void paintGround(const int8_t aX, const int8_t aY) {
  paintBox(aX, aY, WIDTH_GROUND, HEIGHT_GROUND, GROUND_COLOR);
}
```

In `Display.h`, let's add the prototype of the `paintPlatform` function which aims to draw a floating platform :

<div class="filename" >Display.h</div>
```
void paintPlatform(
  const int8_t aX, const int8_t aY, 
  const bool isGoThrough
);
```

In `Display.cpp`, let's define the function `paintPlatform` :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Performs the display of a floating platform
void paintPlatform(const int8_t aX, const int8_t aY, const bool isGoThrough) {
  paintBox(aX, aY, WIDTH_PLATFORM, HEIGHT_PLATFORM, (isGoThrough ? PLATFORM_T_COLOR : PLATFORM_NOT_T_COLOR));
}
```

In `Display.h`, let's add the prototype of the `paintPlatform` function which directs to the dedicated drawing function :

<div class="filename" >Display.h</div>
```
void paintPlatform(const Platform &aPlatform);
```

In `Display.cpp`, let's define the function `paintPlatform` :

<div class="filename" >Display.cpp</div>
```
// Orients to the dedicated drawing function
void paintPlatform(const Platform &aPlatform) {
  int8_t x = aPlatform.x - OVER_CENTER_X_PLATFORM;
  int8_t y = aPlatform.y - OVER_CENTER_Y_PLATFORM;
  switch(aPlatform.type) {
    case GROUND_TYPE:
      x = aPlatform.x - OVER_CENTER_X_GROUND;
      y = aPlatform.y - OVER_CENTER_Y_GROUND;
      break;
  }

  for(uint8_t i = 0 ; i < aPlatform.lengthPlatform ; i++) {

    switch(aPlatform.type) {
      case GROUND_TYPE:
        paintGround(x, y);
        x += WIDTH_GROUND;
        break;
      default:
        paintPlatform(x, y, aPlatform.isGoThrough);
        x += WIDTH_PLATFORM;
    }
    
  }
}
```

In `Display.h`, let's add the prototype of the `paintPlatforms` function that draws all platforms :

<div class="filename" >Display.h</div>
```
void paintPlatforms(Platform * aSet);
```

In `Display.cpp`, let's define the function `paintPlatforms` :

<div class="filename" >Display.cpp</div>
```
// Draw all platforms
void paintPlatforms(Platform * aSet) {
  for(int i = 0 ; i < NB_OF_PLATFORMS ; i++) {
    paintPlatform(aSet[i]);
  }
}
```

In `Display.h`, let's modify the prototype of the `paint` function such that :

<div class="filename" >Display.h</div>
```
void paint(Character &aCharacter, Platform * aSetOfPlatforms);
```

In `Display.cpp`, let's change the definition of the `paint` function :

<div class="filename" >Display.cpp</div>
```
void paint(Character &aCharacter, Platform * aSetOfPlatforms) {
  paintPlatforms(aSetOfPlatforms);
  // dessiner le personnage...
}
```

In the main program, let's modify the call of the `paint` method such that :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // wainting loop
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
      // ...

      paint(hero, setOfPlatforms);
      break;
    default:
      gb.display.println("Votre message");
  }
)
```

Before making our platforms "physical", we will modify the position of our character so that it is displayed above the ground.

In `Character.cpp`, modify the `initCharacter` function, in particular the `y` position such that :

<div class="filename" >Character.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void initCharacter(Character &aCharacter) {
  // we force the hero's initial position in the middle of the screen and tackle the ground
  aCharacter.x = 40;
  aCharacter.y = gb.display.height() - (UNDER_CENTER_Y_HERO + HEIGHT_GROUND);

  // ...
}
```

Still in `Character.cpp`, let's modify the `rectifyPositionY` function such that :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t isOnOnePlatform(const Character &aCharacter) {
  return ( (aCharacter.y + aCharacter.vy) >= (gb.display.height() - (UNDER_CENTER_Y_HERO + HEIGHT_GROUND)) ) ? ID_GROUND : NO_ID ;
}
```

You can upload the program to your console. You will see that your character is drawn on the ground, but you can jump on the platforms, but no reaction, you cross them. We will fix it right away.

![Display of platforms, without collision management](./../../img/E03/platformes_sans_collisions_v1.gif)

### Collision with platforms

To begin with, we need a function that indicates whether the character is going up or down.

In `Character.h`, let's add the prototype of the `isFall` function :

<div class="filename" >Character.h</div>
```
bool isFall(const Character &aCharacter);
```

In `Character.cpp`, let's define the function `isFall` :

<div class="filename" >Character.cpp</div>
```
// true if the character go down, false if not
bool isFall(const Character &aCharacter) {
  return (aCharacter.oldY - aCharacter.y) < 0;
}
```

We will also need a function that returns the platform corresponding to the identifier given as a parameter.

In `Platform.h`, let's add the prototype of the function `getPlatformFromId` :

<div class="filename" >Platform.h</div>
```
Platform getPlatformFromId(const uint8_t aId, Platform * aSet);
```

In `Platform.cpp`, let's define the function `getPlatformFromId` :

<div class="filename" >Platform.cpp</div>
```
Platform getPlatformFromId(const uint8_t aId, Platform * aSet) {

  for(uint8_t i = 0 ; i < NB_OF_PLATFORMS ; i++) {
    if(aSet[i].id == aId) {
      return aSet[i];
    }
  }

  Platform nullPlatform;
  nullPlatform.id = NO_ID;
  return nullPlatform;
}
```

Let's add a function that determines if there is a collision with the platform provided as a parameter.

In `PhysicsEngine.h`, let's add the prototype of the function `isOnThePlatform`, it is necessary to include `Platform.h` in `PhysicsEngine.h` :

<div class="filename" >PhysicsEngine.h</div>
```
const uint8_t isOnThePlatform(
  const Character &aCharacter, 
  const Platform &aPlatform
);
```

In `PhysicsEngine.cpp`, let's define the function `isOnThePlatform` :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Détecter si une collision à lieu avec la plateforme en paramètre
const uint8_t isOnThePlatform(const Character &aCharacter, const Platform &aPlatform) {
  int8_t xCharacter = aCharacter.x - OVER_CENTER_X_HERO;
  int8_t yCharacter = aCharacter.y - OVER_CENTER_Y_HERO;

  int8_t xPlatform = aPlatform.x - OVER_CENTER_X_PLATFORM;
  int8_t yPlatform = aPlatform.y - OVER_CENTER_Y_PLATFORM;
  uint8_t widthPlatform = WIDTH_PLATFORM;
  uint8_t heightPlatform = HEIGHT_PLATFORM;
  switch(aPlatform.type) {
    case GROUND_TYPE:
      xPlatform = aPlatform.x - OVER_CENTER_X_GROUND;
      yPlatform = aPlatform.y - OVER_CENTER_Y_GROUND;
      widthPlatform = WIDTH_GROUND;
      heightPlatform = HEIGHT_PLATFORM;
      break;
  }

  if(aCharacter.state == JUMP_STATE && !isFall(aCharacter) && aPlatform.isGoThrough) {
    // Si l'on saute, que l'on monte et que la plateforme peut être traversé
    // alors on ne déclenche pas la détection de collision
    return NO_ID;
  } else if(
      aCharacter.state == JUMP_STATE &&
      gb.collideRectRect(xCharacter, yCharacter + aCharacter.vy, WIDTH_HERO, HEIGHT_HERO, xPlatform, yPlatform, widthPlatform * aPlatform.lengthPlatform, heightPlatform)
    )
  {
    return aPlatform.id;
  } else {

    if( (aCharacter.y + UNDER_CENTER_Y_HERO + aCharacter.vy) >= yPlatform ) {
      return gb.collideRectRect(xCharacter, yCharacter + aCharacter.vy, WIDTH_HERO, HEIGHT_HERO, xPlatform, yPlatform - 1, widthPlatform * aPlatform.lengthPlatform, heightPlatform) ? aPlatform.id : NO_ID;
    }
    
  }

  return NO_ID;
}
```

In `PhysicsEngine.h`, let's modify the prototype of the `isOnOnePlatform` function to pass it a set of platforms as parameters, i.e. :

<div class="filename" >PhysicsEngine.h</div>
```
const uint8_t isOnOnePlatform(
  const Character &aCharacter, 
  Platform * aSetOfPlatforms
);
```

Modify the code of the function `isOnOnePlatform` (in `PhysicsEngine.cpp`) :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Allows to detect a collision with a platform
const uint8_t isOnOnePlatform(const Character &aCharacter, Platform * aSetOfPlatforms) {
  for(uint8_t i = 0 ; i < NB_OF_PLATFORMS ; i++) {
    const uint8_t platformId = isOnThePlatform(aCharacter, aSetOfPlatforms[i]);
    if(platformId != NO_ID) {
      return platformId;
    }
  }
  return NO_ID ;
}
```

As you can see, the `isOnOnePlatform` function scans all platforms, and if it detects a collision then it returns the identifier of the platform with which the character is in collision.

Let's now make a modification to the `rectifyPositionY` function, we will add a parameter, which will be a platform, to correct the `y` position according to the platform with which the player is in contact.

It is necessary to include `Platform.h` in `Character.h`.

In `Character.h`, let's modify the prototype of the `rectifyPositionY` function :

<div class="filename" >Character.h</div>
```
void rectifyPositionY(Character &aCharacter, Platform &aPlatform);
```

And let's adapt the code of the function `rectifyPositionY` :

<div class="filename" >Character.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Correct the position y
void rectifyPositionY(Character &aCharacter, Platform &aPlatform) {
  int8_t overCenterY = OVER_CENTER_Y_PLATFORM;
  switch(aPlatform.type) {
    case GROUND_TYPE:
      overCenterY = OVER_CENTER_Y_GROUND;
      break;
  }
  aCharacter.y = (aPlatform.y - (UNDER_CENTER_Y_HERO + overCenterY));
}
```

Let's adapt the `gravity` function.

In `PhysicsEngine.h`, let's modify the prototype of the `gravity` function such that :

<div class="filename" >PhysicsEngine.h</div>
```
void gravity(Character &aCharacter, Platform * aSetOfPlatforms);
```

In `PhysicsEngine.cpp`, let's modify the `gravity` function such that :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Free fall
void gravity(Character &aCharacter, Platform * aSetOfPlatforms) {

  const uint8_t platformId = isOnOnePlatform(aCharacter, aSetOfPlatforms);
  if( platformId == NO_ID ) {
    /* ... */
  } else {
    Platform aPlatform = getPlatformFromId(platformId, aSetOfPlatforms);
    rectifyPositionY(aCharacter, aPlatform);
    /* ... */
  }
  
}
```

In the same way, let's modify the prototype of the `jump` function, in `PhysicsEngine.h`, such that :

<div class="filename" >PhysicsEngine.h</div>
```
void jump(Character &aCharacter, Platform * aSetOfPlatforms);
```

In `PhysicsEngine.h`, let's modify the `jump` function :

<div class="filename" >PhysicsEngine.h <span>/!\ Horizontal scrolling /!\</span></div>
```
// Implementation of the character jump
void jump(Character &aCharacter, Platform * aSetOfPlatforms) {
  const uint8_t platformId = isOnOnePlatform(aCharacter, aSetOfPlatforms);

  if(aCharacter.state == PUSH_FOR_JUMP_STATE) {
    /* Unchanged... */
  } else if( platformId != NO_ID) {
    // If you are in contact with a platform

    Platform aPlatform = getPlatformFromId(platformId, aSetOfPlatforms);

    if(aPlatform.isGoThrough || (isFall(aCharacter) && aCharacter.y <= aPlatform.y) || aPlatform.type == GROUND_TYPE) {
      rectifyPositionY(aCharacter, aPlatform);
      aCharacter.vy = 0;
      aCharacter.state = ON_THE_PLATFORM_STATE;
    } else {
      aCharacter.vy = 0;
      jumpMovement(aCharacter);
    }
    
  } else if( isOutOfWorld(aCharacter) ) {
    /* Unchanged... */
  } else {
    /* Unchanged... */
  }
  
}
```

Finally, in the main program, let's make some changes in the code corresponding to the `PLAY_STATE` state :

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
        gravity(hero, setOfPlatforms);
      } else if(hero.state == JUMP_STATE || hero.state == PUSH_FOR_JUMP_STATE) {
        jump(hero, setOfPlatforms);
      }

      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
)
```

Upload your porgram to the console, and have fun jumping on the platforms. If you have kept the values provided in this step, both the colors and the position of the platforms, you can be below the light green platform and jump, thus climbing on it.

![Display of platforms, with collision management](./../../img/E03/platformes_avec_collisions_v1.gif)

## Conclusion

You have reached the end of this third step : where we have added free fall and platform management.

If you have finished or are having problems you can download the solution <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v3.0.zip" class="external-link" >here</a>.

In the next step, the fourth, we will discuss interactions with the world. We will see how to add objects like keys and a door.

Feel free to give me a feedback : the improvements you would make (an outside look is always welcome), the things you didn't understand, the mistakes, etc. To leave a feedback, please add a comment to the following creation :

{% include react-btn.html %}
