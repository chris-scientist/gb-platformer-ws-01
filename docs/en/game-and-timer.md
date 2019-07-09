---
layout: step
title: "Game management and timer"
permalink: /en/game-and-timer/
next: /en/best-score/
link-other-lang: /fr/partie-et-chrono/
lang: en
---

# Step 5 - Game management and timer

## Introduction

In this fifth step, we will implement : the end of the game, a timer and an end of game screen and we will manage the "Game over" : the end of the game when the maximum time allowed to complete the level has been exceeded.

*I invite you to download <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v4.0.zip" class="external-link" >the code</a> which is the result of the fourth step in order to start on a common basis.*

## End of the game

In the file `Constants.h` let's add the constant `GAME_IS_FINISH_STATE`, a new state indicating that the game is finished :

<div class="filename" >Constants.h</div>
```
const uint8_t GAME_IS_FINISH_STATE = 4;
```

Let's create the file `Game.h` where we will place the functions related to the management of the game. 
At this stage of progress, a part is finished when the door is open. 
So let's add in the `Game.h` file the function `isEndOfGame` which will determine if the game is finished.

Here is the content of the file :

<div class="filename" >Game.h</div>
```
#ifndef PLATFORMER_GAME
#define PLATFORMER_GAME

#include "Constants.h"
#include "Object.h"

bool isEndOfGame(const Object &aDoor);

#endif
```

Let's create the corresponding `Game.cpp` file to code the `isEndOfGame` function :

<div class="filename" >Game.cpp</div>
```
#include "Game.h"

bool isEndOfGame(const Object &aDoor) {
  return (aDoor.state == DOOR_OPENED);
}
```

Let's not forget to include the file `Game.h` in the main program and let's also add the redirection to the `GAME_IS_FINISH_STATE` state when the game is finished (to be added in the `PLAY_STATE` state) :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes...
#include "Game.h"

// Globals variables...

void setup() {
  // ...
}

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

      if( isEndOfGame(setOfObjects[1]) ) {
        stateOfGame = GAME_IS_FINISH_STATE;
      }
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

In `Lang.h`, let's add the following constants :

<div class="filename" >Lang.h</div>
```
static const char * TRY_AGAIN_EN = "TRY AGAIN";
static const char * A_4_TRY_AGAIN_EN = "(A) Try again";
static const char * B_4_MENU_EN = "(B) Menu";
```

In `Display.h`, let's create the function that displays the end of game screen :

<div class="filename" >Display.h</div>
```
void paintEndOfGame();
```

In `Display.cpp`, let's implement the function `paintEndOfGame` :

<div class="filename" >Display.cpp</div>
```
void paintEndOfGame() {
  gb.display.setFontSize(2);
  gb.display.setColor(BROWN);
  gb.display.println("");
  gb.display.println("");
  gb.display.println(TRY_AGAIN_EN);
  gb.display.setFontSize(1);
  gb.display.setColor(WHITE);
  
  gb.display.println("");
  gb.display.println("");
  gb.display.println(A_4_TRY_AGAIN_EN);
  gb.display.println(B_4_MENU_EN);
}
```

Let's define a command management function that has the following behavior :
* if button A is pressed then launch a new game ;
* if button B is pressed then return to the menu ;
* otherwise stay in the current state.

In `Commands.h`, let's add the prototype of the `manageCommandsOutOfGame` function :

<div class="filename" >Commands.h</div>
```
const uint8_t manageCommandsOutOfGame(const uint8_t aState);
```

In `Commands.cpp`, let's define the `manageCommandsOutOfGame` function that reacts as described above :

<div class="filename" >Commands.cpp</div>
```
const uint8_t manageCommandsOutOfGame(const uint8_t aState) {
  if(gb.buttons.pressed(BUTTON_A)) {
    return LAUNCH_PLAY_STATE;
  } else if(gb.buttons.pressed(BUTTON_B)) {
    return HOME_STATE;
  }

  return aState;
}
```

In the main program, let's add the end state of the game `GAME_IS_FINISH_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes...
#include "Game.h"

// Globals variables...

void setup() {
  // ...
}

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
      break;
    case GAME_IS_FINISH_STATE:
      paintEndOfGame();
      stateOfGame = manageCommandsOutOfGame(stateOfGame);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

You can upload the program to your console, the result is as follows :

![End of the game](./../../img/E05/fin_de_partie_v1.gif)

## Timer

**Work in progress**
