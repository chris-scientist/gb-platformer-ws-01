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

Before developing the timer, let's add constants in `Constants.h` :

<div class="filename" >Constants.h</div>
```
// Constantes timer
const uint8_t DAYS_NUMBER = 0;
const uint8_t HOURS_NUMBER = 1;
const uint8_t MINUTES_NUMBER = 2;
const uint8_t SECONDS_NUMBER = 3;
const uint8_t MILLISECONDS_NUMBER = 4;
```

We will create a structure for the timer, we will need :
* of an integer to count the time spent ;
* of another integer to count the time spent since the last break ;
* an array of integers to have the total time spent, divided into days, hours, minutes, seconds and milliseconds ;
* a Boolean to determine if the timer is active or not.

Here is the content of the file `Timer.h` :

<div class="filename" >Timer.h</div>
```
#ifndef PLATFORMER_TIMER
#define PLATFORMER_TIMER

#include <Gamebuino-Meta.h>

#include "Constants.h"

struct Timer {
  uint32_t timeInMilliseconds;
  uint32_t tempTime;
  uint16_t valueOfTime[5] = {0, 0, 0, 0, 0};
  bool activateTimer;
};

#endif
```

In `Timer.h` let's add the prototype of the `createTimer` function :

<div class="filename" >Timer.h</div>
```
void createTimer(Timer &aTimer);
```

In `Timer.cpp` let's add the following content, including the function `createTimer` :

<div class="filename" >Timer.cpp</div>
```
#include "Timer.h"

// Create the timer
void createTimer(Timer &aTimer) {
  aTimer.activateTimer = false;
}
```

In the main program, let's include the file `Timer.h` and create the timer and then add the call to the `createTimer` function in the `setup` function :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes...
#include "Timer.h"

// Other globals variables...
Timer myTimer;

void setup() {
  // ... 

  createTimer(myTimer);
}

void loop() {
  // ...
}
```

In `Timer.h`, let's add the function `resetTimer` which will allow to reset the timer :

<div class="filename" >Timer.h</div>
```
void resetTimer(Timer &aTimer);
```

Then, as usual, let's code in `Timer.cpp` the function we just declared (`resetTimer`) :

<div class="filename" >Timer.cpp</div>
```
// Re-initialize the timer
void resetTimer(Timer &aTimer) {
  aTimer.timeInMilliseconds = 0;
  aTimer.tempTime = 0;
  aTimer.activateTimer = true;
}
```

In the main program, when the game is in the `LAUNCH_PLAY_STATE` state, the timer must be reset :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
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
      resetTimer(myTimer); // ............. we reset the timer
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

In `Timer.h`, let's add the prototype of the function `pauseForTimer` :

<div class="filename" >Timer.h</div>
```
void pauseForTimer(Timer &aTimer);
```

In `Timer.cpp`, let's develop the function `pauseForTimer` :

<div class="filename" >Timer.cpp</div>
```
// Pause the timer
void pauseForTimer(Timer &aTimer) {
  if(aTimer.tempTime != 0) {
    aTimer.timeInMilliseconds += aTimer.tempTime;
    aTimer.tempTime = 0;
  }
}
```

In `Timer.h`, let's add the prototype of the `computeTime` function which cuts a time in milliseconds into days, hours, minutes, seconds and milliseconds :

<div class="filename" >Timer.h</div>
```
void computeTime(Timer &aTimer);
```

In `Timer.cpp`, let's implement the function `computeTime` :

<div class="filename" >Timer.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Break down the elapsed time into days, hours, minutes and seconds
void computeTime(Timer &aTimer) {
  uint32_t rest = aTimer.timeInMilliseconds + aTimer.tempTime;

  const uint16_t DAYS_IN_FRAMES    = 24*60*60*1000;
  const uint16_t HOURS_IN_FRAMES   = 60*60*1000;
  const uint16_t MINUTES_IN_FRAMES = 60*1000;
  const uint16_t SECONDS_IN_FRAMES = 1000;

  uint16_t nbDays = 0;
  uint16_t nbHours = 0;
  uint16_t nbMinutes = 0;
  uint16_t nbSeconds = 0;

  // Compute days
  if(rest >= DAYS_IN_FRAMES) {
    nbDays = (uint16_t)(rest / DAYS_IN_FRAMES);
    rest = (rest - (nbDays * DAYS_IN_FRAMES));
  }
  // Compute hours
  if(rest >= HOURS_IN_FRAMES) {
    nbHours = (uint16_t)(rest / HOURS_IN_FRAMES);
    rest = (rest - (nbHours * HOURS_IN_FRAMES));
  }
  // Compute minutes
  if(rest >= MINUTES_IN_FRAMES) {
    nbMinutes = (uint16_t)(rest / MINUTES_IN_FRAMES);
    rest = (rest - (nbMinutes * MINUTES_IN_FRAMES));
  }
  // Compute seconds
  if(rest >= SECONDS_IN_FRAMES) {
    nbSeconds = (uint16_t)(rest / SECONDS_IN_FRAMES);
    rest = (rest - (nbSeconds * SECONDS_IN_FRAMES));
  }

  aTimer.valueOfTime[DAYS_NUMBER] = nbDays;
  aTimer.valueOfTime[HOURS_NUMBER] = nbHours;
  aTimer.valueOfTime[MINUTES_NUMBER] = nbMinutes;
  aTimer.valueOfTime[SECONDS_NUMBER] = nbSeconds;
  aTimer.valueOfTime[MILLISECONDS_NUMBER] = rest;
}
```

This function will later be called by another function of the timer.

In `Timer.h`, let's add the function `incrementTimer` which will increment the counter with each call :

<div class="filename" >Timer.h</div>
```
void incrementTime(Timer &aTimer);
```

In `Timer.cpp`, let's define the function `incrementTime` :

<div class="filename" >Timer.cpp</div>
```
void incrementTime(Timer &aTimer) {
  aTimer.tempTime += gb.getTimePerFrame();
}
```

Like the `computeTime` function, the `incrementTime` function will be called by another function of the stopwatch that we will see now.

In `Timer.h`, let's create the function `runTimer` that we will call each time we pass through the `loop` function (of the main program), and when we are in the `PLAY_STATE` state :

<div class="filename" >Timer.h</div>
```
void runTimer(Timer &aTimer);
```

In `Timer.cpp`, let's expand the `runTimer` function :

<div class="filename" >Timer.cpp</div>
```
// Manage the timer
void runTimer(Timer &aTimer) {
  if(aTimer.activateTimer) {
    incrementTime(aTimer);
    computeTime(aTimer);
  } else {
    pauseForTimer(aTimer);
  }
}
```

In the main program, in the `PLAY_STATE` state, after command management, let's add the call of the `runTimer` function like this :

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

      runTimer(myTimer);

      if(hero.state != JUMP_STATE && hero.state != PUSH_FOR_JUMP_STATE) {
        gravity(hero, setOfPlatforms);
      } else if(hero.state == JUMP_STATE || hero.state == PUSH_FOR_JUMP_STATE) {
        jump(hero, setOfPlatforms);
      }

      // ...
      break;
    default:
      gb.display.println("Votre message");
  }
)
}
```

We will add a new state.

Let's start by creating this new state in `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint8_t SAVE_HIGH_SCORE_STATE = 5;
```

Let's go back to the main program, in the `PLAY_STATE` state, modify the redirection that is performed when the game is finished to the new state :

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

      if( isEndOfGame(setOfObjects[1]) ) {
        stateOfGame = SAVE_HIGH_SCORE_STATE;
      }
      break;
    case GAME_IS_FINISH_STATE:
      // ...
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

Still in the main program, let's add the state `SAVE_HIGH_SCORE_STATE`, for now, in this state :
* we will stop the timer ;
* and we will redirect to the `GAME_IS_FINISH_STATE` state.

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
      break;
    case GAME_IS_FINISH_STATE:
      // ...
      break;
    case SAVE_HIGH_SCORE_STATE:
      myTimer.activateTimer = false;
      runTimer(myTimer);

      stateOfGame = GAME_IS_FINISH_STATE;
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

The timer is at this functional stage. To see this, we will display it on the game screen.

Let's start by adding the prototype of the `paintTimer` function in `Display.h` :

<div class="filename" >Display.h</div>
```
void paintTimer(const uint16_t * aTime);
```

This function takes as parameter the array of integers which represents the time spent divided into days, hours, minutes, seconds and milliseconds.

In `Display.cpp`, let's implement the function `paintTimer` :

<div class="filename" >Display.cpp</div>
```
// Display the timer
void paintTimer(const uint16_t * aTime) {
  gb.display.setFontSize(1);
  gb.display.setColor(WHITE);
  
  uint16_t nbSeconds = aTime[SECONDS_NUMBER];
  if(nbSeconds < 10) {
    gb.display.print("0");
  }
  gb.display.printf("%d:", nbSeconds);
  uint16_t nbMilliseconds = aTime[MILLISECONDS_NUMBER];
  if(nbMilliseconds < 100) {
    gb.display.print("0");
  }
  if(nbMilliseconds < 10) {
    gb.display.print("0");
  }
  gb.display.printf("%d", nbMilliseconds);
}
```

In `Display.h`, let's modify the prototype of the `paint` function like this :

<div class="filename" >Display.h</div>
```
void paint(
  Character &aCharacter, 
  Platform * aSetOfPlatforms, 
  Object * aSetOfObjects, 
  const uint16_t * aTime
);
```

In `Display.cpp`, update the `paint` function :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void paint(Character &aCharacter, Platform * aSetOfPlatforms, Object * aSetOfObjects, const uint16_t * aTime) {
  /* display the game... */
  paintTimer(aTime);
}
```

In the main program, let's adapt the call of the `paint` function (in the `PLAY_STATE` state) :

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
      // ...

      interactionsWithWorld(hero, setOfObjects);

      paint(hero, setOfPlatforms, setOfObjects, myTimer.valueOfTime);

      if( isEndOfGame(setOfObjects[1]) ) {
        stateOfGame = SAVE_HIGH_SCORE_STATE;
      }
      break;
    case GAME_IS_FINISH_STATE:
      // ...
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

Before moving on to the "game over", we will display the time on the end of the game screen.

To do this, let's modify the prototype of the `paintEndOfGame` function :

<div class="filename" >Display.h</div>
```
void paintEndOfGame(const uint16_t * aTime);
```

In `Display.cpp`, let's adapt the function `paintEndOfGame` :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void paintEndOfGame(const uint16_t * aTime) {
  gb.display.setFontSize(2);
  gb.display.setColor(BROWN);
  gb.display.println("");
  gb.display.println("");
  gb.display.println(TRY_AGAIN_EN);
  paintTimer(aTime); // ............. add the timer display here
  gb.display.setFontSize(1);
  gb.display.setColor(WHITE);
  
  gb.display.println("");
  gb.display.println("");
  gb.display.println(A_4_TRY_AGAIN_EN);
  gb.display.println(B_4_MENU_EN);
}
```

In the main program, in the `GAME_IS_FINISH_STATE` report, modify the call of the `paintEndOfGame` function :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case GAME_IS_FINISH_STATE:
      paintEndOfGame(myTimer.valueOfTime);
      stateOfGame = manageCommandsOutOfGame(stateOfGame);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

You can upload the program to your console, you will then see that the timer "restarts at zero" after one minute. This is normal, as we display that seconds and milliseconds, accuracy is lost. This is a detail that we will solve in the rest of this workshop. Indeed, beyond 20,000 milliseconds (i.e. 20 seconds) we will trigger the "game over".

Be careful in the capture below the stopwatch does not start at zero ! This is because I manually start the capture and therefore the time has elapsed between the actual start of the game and the start of the capture.

![Adding the timer](./../../img/E05/chrono_v1.gif)

## Game over

For the management of the game over, we will start by adding the maximum game time in milliseconds and the state for the game over, in `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint32_t MAX_TIME_OF_GAME = 20000;

const uint8_t GAME_OVER_STATE = 6;
```

In `Game.h`, let's include the file `Timer.h`, then add the function `isGameOver` (in `Game.h`) :

<div class="filename" >Game.h</div>
```
bool isGameOver(Timer &aTimer);
```

In `Game.cpp`, let's implement the function `isGameOver` :

<div class="filename" >Game.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
bool isGameOver(Timer &aTimer) {
  return ( (aTimer.timeInMilliseconds + aTimer.tempTime) >= MAX_TIME_OF_GAME );
}
```

Before modifying the main program, we will create a game over screen.

In `Lang.h`, let's add a constant for the game over screen :

<div class="filename" >Lang.h</div>
```
static const char * GAME_OVER_EN = "GAME OVER";
```

In `Display.h`, let's declare a function, which displays a game over screen, we will name this function `paintGameOverScreen`:

<div class="filename" >Display.h</div>
```
void paintGameOverScreen();
```

In `Display.cpp`, let's define the function `paintGameOverScreen` :

<div class="filename" >Display.cpp</div>
```
void paintGameOverScreen() {
  gb.display.setFontSize(2);
  gb.display.setColor(BROWN);
  gb.display.println("");
  gb.display.println("");
  gb.display.println(GAME_OVER_EN);

  gb.display.setFontSize(1);
  gb.display.setColor(WHITE);
  gb.display.println("");
  gb.display.println("");
  gb.display.println(A_4_TRY_AGAIN_EN);
  gb.display.println(B_4_MENU_EN);
}
```

Let's change the main program.

First, add the report `GAME_OVER_STATE`, in this report we will use the command management we created previously and then we will display the "game over" screen, either :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case GAME_OVER_STATE:
      stateOfGame = manageCommandsOutOfGame(stateOfGame);
      paintGameOverScreen();
      
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

In a second step, let's add the road to the game over, this in the state `PLAY_STATE` :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case PLAY_STATE:
      // ...

      interactionsWithWorld(hero, setOfObjects);

      paint(hero, setOfPlatforms, setOfObjects, myTimer.valueOfTime);

      if( isEndOfGame(setOfObjects[1]) ) {
        stateOfGame = SAVE_HIGH_SCORE_STATE;
      } else {
        stateOfGame = ( isGameOver(myTimer) ? GAME_OVER_STATE : PLAY_STATE );
      }
      break;
    // ...
    default:
      gb.display.println("Votre message");
  }
}
```

You can upload the program to your console, start a game, and let the time pass : after 20 seconds the game over screen appears.

I started the capture below after 16 seconds, my objective here is to show the display of the "game over" screen :

![Game over](./../../img/E05/game_over_v1.gif)

## Navigation

We will add a pause screen, instead of returning unconditionally to the menu.

To do this, let's declare two new states in `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint8_t PAUSE_STATE = 7;
const uint8_t GO_BACK_GAME_STATE = 8;
```

In `Lang.h`, let's add two constants, which we will use for the screen :

<div class="filename" >Lang.h</div>
```
static const char * A_4_GO_TO_GAME_EN = "(A) Go to game";
static const char * DO_YOU_WANT_EN = "Do you want ?";
```

In `Display.h`, let's add the prototype of the `paintPause` function :

<div class="filename" >Display.h</div>
```
void paintPause();
```

In `Display.cpp`, let's implement the `paintPause` function :

<div class="filename" >Display.cpp</div>
```
void paintPause() {
  gb.display.setFontSize(1);
  gb.display.setColor(BROWN);
  gb.display.println("");
  gb.display.println("");
  gb.display.println("");
  gb.display.println("");
  gb.display.println(DO_YOU_WANT_EN);

  gb.display.setColor(WHITE);
  gb.display.println("");
  gb.display.println("");
  gb.display.println("");
  gb.display.println(A_4_GO_TO_GAME_EN);
  gb.display.println(B_4_MENU_EN);
}
```

The command management for the `PAUSE_STATE` state must be as follows :
* if the A button is pressed then return to the game (via the `GO_BACK_GAME_STATE` state) ;
* if button B is pressed then go to the menu ;
* otherwise stay on break.

In `Commands.h`, let's declare the function `manageCommandsForPause` :

<div class="filename" >Commands.h</div>
```
const uint8_t manageCommandsForPause();
```

In `Commands.cpp`, let's define the `manageCommandsForPause` function :

<div class="filename" >Commands.cpp</div>
```
const uint8_t manageCommandsForPause() {
  if(gb.buttons.pressed(BUTTON_A)) {
    return GO_BACK_GAME_STATE;
  } else if(gb.buttons.pressed(BUTTON_B)) {
    return HOME_STATE;
  }
  return PAUSE_STATE;
}
```

Still in `Commands.cpp`, let's modify the `manageCommands` function to redirect to the pause screen when the menu button is pressed :

<div class="filename" >Commands.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t manageCommands(Character &aCharacter) {

  // ...
  
  return (gb.buttons.pressed(BUTTON_MENU) ? PAUSE_STATE : PLAY_STATE);
}
```

Let's make some changes to the main program.

In the `PLAY_STATE` state, the timer must be stopped if the user presses the menu button :

<div class="filename" >GBPlatformer01.ino <span>/!\ Horizontal scrolling /!\</span></div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case PLAY_STATE:
      if(hero.state == ON_THE_PLATFORM_STATE) {
        stateOfGame = manageCommands(hero);
        if(stateOfGame == PAUSE_STATE) {
          myTimer.activateTimer = false;
        }
      }
      
      // ...

      if( stateOfGame != PAUSE_STATE ) {
        if( isEndOfGame(setOfObjects[1]) ) {
          stateOfGame = SAVE_HIGH_SCORE_STATE;
        } else {
          stateOfGame = ( isGameOver(myTimer) ? GAME_OVER_STATE : PLAY_STATE );
        }
      }
      break;
    // ...
    default:
      gb.display.println("Votre message");
  }
}
```

Let's add the state `PAUSE_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case PAUSE_STATE:
      stateOfGame = manageCommandsForPause();
      paintPause();
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

Let's add the state `GO_BACK_GAME_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case GO_BACK_GAME_STATE:
      myTimer.activateTimer = true;
      stateOfGame = PLAY_STATE;
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

When the state is undefined, we will add a redirection to the host state :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    default:
      // afficher un texte...

      delay(3000);
      stateOfGame = HOME_STATE;
  }
}
```

You can upload the program to your console, the last addition is the pause screen :

![Pause management](./../../img/E05/pause_v1.gif)

## Conclusion

You have reached the end of this fifth step where we have added game management.

If you have finished or are having problems you can download the solution <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v5.0.zip" class="external-link" >here</a>.

In the next and last step, i.e. the sixth, we will discuss the management of a high score table.

Feel free to give me a feedback : the improvements you would make (an outside look is always welcome), the things you didn't understand, the mistakes, etc. To leave a feedback, please add a comment to the following creation :

{% include react-btn.html %}
