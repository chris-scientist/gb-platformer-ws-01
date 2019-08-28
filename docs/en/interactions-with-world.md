---
layout: step
title: "Interactions with the world"
permalink: /en/interactions-with-world/
next: /en/game-and-timer/
link-other-lang: /fr/monde/
lang: en
---

# Step 4 - Interactions with the world

## Introduction

For this fourth step we will implement interactions with the world. There will be a key to pick up, this one will open a door.

<!--*I invite you to download <a href="#" class="external-link" >the code</a> which is the result of the third step in order to start on a common basis.*-->

## The objects

### Creating objects

Let's start by adding constants in `Constants.h` :

<div class="filename" >Constants.h</div>
```
// Couleur des box
// ...
const Color KEY_COLOR = YELLOW;
const Color DOOR_CLOSED_COLOR = GRAY;
const Color DOOR_OPENED_COLOR = DARKGRAY;

// Pour une clé de 4 pixels de haut sur 8 pixels de large
const uint8_t HEIGHT_KEY = 4;
const uint8_t WIDTH_KEY = 8;
const uint8_t OVER_CENTER_X_KEY = 4;
const uint8_t OVER_CENTER_Y_KEY = 2;

// Pour une porte de 12 pixels de haut sur 10 pixels de large
const uint8_t HEIGHT_DOOR = 12;
const uint8_t WIDTH_DOOR = 10;
const uint8_t OVER_CENTER_X_DOOR = 5;
const uint8_t OVER_CENTER_Y_DOOR = 6;

// Nombre d'objets
const uint8_t NB_OF_OBJECTS = 2;

// Type d'objet
const uint8_t NO_OBJECT = 0;
const uint8_t KEY_OBJECT = 1;
const uint8_t DOOR_OBJECT = 2;

// État clé
const uint8_t KEY_ON_THE_PLATFORM = 0;
const uint8_t KEY_COLLECTED = 1;

// État porte
const uint8_t DOOR_CLOSED = 0;
const uint8_t DOOR_OPENED = 1;
```

We will define our objects by the following attributes:
* a `x` position ;
* a `y` position ;
* a type ;
* and a state.

In `Object.h`, let's create the structure for our objects :

<div class="filename" >Object.h</div>
```
#ifndef PLATFORMER_OBJECT
#define PLATFORMER_OBJECT

#include <Gamebuino-Meta.h>
#include "Constants.h"

struct Object {
  int8_t x;
  int8_t y;
  uint8_t type; // .... object type : see constants
  uint8_t state; // ... object state : see constants
};

#endif
```

In `Object.h`, let's add a function to create objects, the prototype of which is as follows :

<div class="filename" >Object.h</div>
```
Object createObject(
  int8_t aX, int8_t aY, 
  uint8_t aType, 
  uint8_t aState
);
```

In `Object.cpp`, let's define the function `createObject`, remember to include `Object.h` (in `Object.cpp`) :

<div class="filename" >Object.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
Object createObject(int8_t aX, int8_t aY, uint8_t aType, uint8_t aState) {
  Object anObject;

  anObject.x = aX;
  anObject.y = aY;
  anObject.type = aType;
  anObject.state = aState;

  return anObject;
}
```

In `Object.h`, let's add a function to initialize our objects, whose prototype is as follows :

<div class="filename" >Object.h</div>
```
void initObjects(Object * aSet);
```

In `Object.cpp`, let's define the function `initObjects` :

<div class="filename" >Object.cpp</div>
```
void initObjects(Object * aSet) {
  aSet[0] = createObject(75, 26, KEY_OBJECT, KEY_ON_THE_PLATFORM);
  aSet[1] = createObject(72, 54, DOOR_OBJECT, DOOR_CLOSED);
}
```

In the main program, you must first include `Object.h`.

Then, apart from the `setup` and `loop` functions, let's declare an array of objects, either :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes
#include "Object.h"

// Other globals variables
Object setOfObjects[NB_OF_OBJECTS];

void setup() {
  // ...
}

void loop() {
  // ...
}
```

Finally, let's initialize our objects in the `LAUNCH_PLAY_STATE` state :

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
      initObjects(setOfObjects); // ....... we reset the objects
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

### Displaying objects

Let's start with the specific functions that draw our objects.

In `Display.h`, let's add the prototype of the `paintKey` function :

<div class="filename" >Display.h</div>
```
void paintKey(
  const int8_t aX, const int8_t aY, 
  const uint8_t aState
);
```

In `Display.cpp`, let's define the function `paintKey` :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Performs key display
void paintKey(const int8_t aX, const int8_t aY, const uint8_t aState) {
  if(aState == KEY_ON_THE_PLATFORM) {
    paintBox(aX - OVER_CENTER_X_KEY, aY - OVER_CENTER_Y_KEY, WIDTH_KEY, HEIGHT_KEY, KEY_COLOR);
  }
}
```

In `Display.h`, let's add the prototype of the `paintDoor` function :

<div class="filename" >Display.h</div>
```
void paintDoor(
  const int8_t aX, const int8_t aY, 
  const uint8_t aState
);
```

In `Display.cpp`, let's define the function `paintDoor` :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Performs door display
void paintDoor(const int8_t aX, const int8_t aY, const uint8_t aState) {
  paintBox(aX - OVER_CENTER_X_DOOR, aY - OVER_CENTER_Y_DOOR, WIDTH_DOOR, HEIGHT_DOOR, (aState == DOOR_OPENED ? DOOR_OPENED_COLOR : DOOR_CLOSED_COLOR));
}
```

Consider including `Object.h` in `Display.h`.

In `Display.h`, let's add the prototype of the `paintObject` function, which draws an object provided as a parameter :

<div class="filename" >Display.h</div>
```
void paintObject(const Object &anObject);
```

In `Display.cpp`, let's define the function `paintObject` :

<div class="filename" >Display.cpp</div>
```
// Performs the orientation to the specific drawing function
void paintObject(const Object &anObject) {
  switch(anObject.type) {
    case KEY_OBJECT:
      paintKey(anObject.x, anObject.y, anObject.state);
      break;
    case DOOR_OBJECT:
      paintDoor(anObject.x, anObject.y, anObject.state);
      break;
  }
}
```

In `Display.h`, let's add the prototype of the function `paintObjects` :

<div class="filename" >Display.h</div>
```
void paintObjects(Object * aSetOfObjects);
```

In `Display.cpp`, let's define the function `paintOjects` :

<div class="filename" >Display.cpp</div>
```
// Performs object display
void paintObjects(Object * aSetOfObjects) {
  for(uint8_t i = 0 ; i < NB_OF_OBJECTS ; i++) {
    paintObject(aSetOfObjects[i]);
  }
}
```

Let's modify, in `Display.h`, the prototype of the `painted` function to pass our objects through it :

<div class="filename" >Display.h</div>
```
void paint(
  Character &aCharacter, 
  Platform * aSetOfPlatforms, 
  Object * aSetOfObjects
);
```

Let's change, in `Display.cpp`, the function `painted` :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Performs all display
void paint(Character &aCharacter, Platform * aSetOfPlatforms, Object * aSetOfObjects) {
  paintPlatforms(aSetOfPlatforms);
  paintObjects(aSetOfObjects); // call the function to draw objects
  paintHero(aCharacter);
}
```

In the main program, in the `PLAY_STATE` report, modify the call of the `paint` function :

<div class="filename" >GBPlatformer01.ino</div>
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
      // ...

      paint(hero, setOfPlatforms, setOfObjects);
      break;
    default:
      gb.display.println("Votre message");
  }
)
```

You can upload your program to your console, and start a game. There are two new rectangles : yellow represents a key and grey a closed door. If you go pick up the key, nothing happens, it's normal ! We will code the interactions with objects in the following step.

![Display of objects, without interactions](./../../img/E04/objets_sans_interactions_v1.gif)

### Interactions with objects

Modify the character, in `Character.h`, let's add an attribute to indicate if the character has picked up the key :

<div class="filename" >Character.h</div>
```
struct Character {
  /* ... */

  bool haveKey; // ........ true if the character has the key, false if not
};
```

In `Character.cpp`, in the `initCharacter` function, we indicate that the character does not have the key :

<div class="filename" >Character.cpp</div>
```
void initCharacter(Character &aCharacter) {

  /* ... */

  // by default, the player does not have the key
  aCharacter.haveKey = false;
}
```

Let's create a file named `Interactions.h`, it will allow to manage interactions with objects, let's add the function `isContactWithObject` :

<div class="filename" >Interactions.h</div>
```
#ifndef PLATFORMER_INTERACTIONS
#define PLATFORMER_INTERACTIONS

#include <Gamebuino-Meta.h>

#include "Constants.h"
#include "Character.h"
#include "Object.h"

const uint8_t isContactWithObject(
  const Character &aCharacter, 
  const Object &anObject
);

#endif
```

In `Interactions.cpp`, let's add the definition of the function `isContactWithObject`, consider including `Interactions.h` (in `Interactions.cpp`) :

<div class="filename" >Interactions.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Indicates if there is a collision with the object in parameter
const uint8_t isContactWithObject(const Character &aCharacter, const Object &anObject) {
  int8_t xObject = anObject.x;
  int8_t yObject = anObject.y;
  uint8_t widthObj = WIDTH_KEY;
  uint8_t heightObj = HEIGHT_KEY;
  switch(anObject.type) {
    case KEY_OBJECT:
      xObject -= OVER_CENTER_X_KEY;
      yObject -= OVER_CENTER_Y_KEY;
      break;
    case DOOR_OBJECT:
      xObject -= OVER_CENTER_X_DOOR;
      yObject -= OVER_CENTER_Y_DOOR;
      widthObj = WIDTH_DOOR;
      heightObj = HEIGHT_DOOR;
      break;
  }

  if(gb.collideRectRect(aCharacter.x - OVER_CENTER_X_HERO, aCharacter.y - OVER_CENTER_Y_HERO, WIDTH_HERO, HEIGHT_HERO, xObject, yObject, widthObj, heightObj)) {
    switch(anObject.type) {
      case KEY_OBJECT:
        return (anObject.state == KEY_ON_THE_PLATFORM) ? KEY_OBJECT : NO_OBJECT;
        break;
      case DOOR_OBJECT:
        return DOOR_OBJECT;
        break;
    }
  }

  return NO_OBJECT;
}
```

In `Interactions.h`, let's add the prototype of the `interactionsWithWorld` function :

<div class="filename" >Interactions.h</div>
```
void interactionsWithWorld(
  Character &aCharacter, 
  Object * aSetOfObjects
);
```

In `Interactions.cpp`, let's define the function `interactionsWithWorld` :

<div class="filename" >Interactions.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Performs interactions with objects in the event of a collision
void interactionsWithWorld(Character &aCharacter, Object * aSetOfObjects) {
  for(uint8_t i = 0 ; i < NB_OF_OBJECTS ; i++) {
    const uint8_t aTypeOfObject = isContactWithObject(aCharacter, aSetOfObjects[i]);
    switch(aTypeOfObject) {
      case KEY_OBJECT:
        aSetOfObjects[i].state = KEY_COLLECTED;
        aCharacter.haveKey = true;
        break;
      case DOOR_OBJECT:
        if(aCharacter.haveKey) {
          aSetOfObjects[i].state = DOOR_OPENED;
          aCharacter.haveKey = false;
        }
        break;
    }
  }
}
```

Let's start by including `Interactions.h` in the main program.

Finally, in the `PLAY_STATE` state, before displaying the game, let's add the call of the `interactionsWithWorld` function :

<div class="filename" >GBPlatformer01.ino</div>
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
      // ...

      interactionsWithWorld(hero, setOfObjects);

      paint(hero, setOfPlatforms, setOfObjects);
      break;
    default:
      gb.display.println("Votre message");
  }
)
```

Upload your program to your console, and now find that you can pick up the key and open the door.

![Object display, with interactions](./../../img/E04/objets_avec_interactions_v1.gif)

## Conclusion

You have reached the end of this fourth step : we have created, displayed and interacted with objects.

<!--If you have finished or are having problems you can download the solution <a href="#" class="external-link" >here</a>.-->

In the next step, i.e. the fifth step, we will approach the management of the game and add a timer.

Feel free to give me a feedback : the improvements you would make (an outside look is always welcome), the things you didn't understand, the mistakes, etc. To leave a feedback, please add a comment to the following creation :

{% include react-btn.html %}
