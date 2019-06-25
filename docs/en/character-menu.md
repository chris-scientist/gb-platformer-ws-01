---
layout: step
title: "The character : creation, display and management of a menu"
permalink: /en/character-menu/
next: /en/movements/
link-other-lang: /fr/personnage-menu/
lang: en
---

# Step 1 - The character : creation, display and management of a menu

## Introduction

In this step, we will create a character, display it and we will also see how to create a menu.

## Basic program

Before creating the character, you have to write the basic program as we saw in the <a href="https://gamebuino.com/academy/workshop/tap-tap-how-fast-can-you-tap/inputs-update-draw" class="external-link" >step 2 of TapTap, a game where you have to go very fast</a> !

As a reminder, we need :
* include the file `Gamebuino-Meta.h` at the beginning of the code ;
* think about initializing the Gamebuino META in the `setup` function ;
* add a waiting loop in the `loop` function ;
* clear the screen always in the `loop` function.

To test that everything is working well, we will display a message with `gb.display.println("Your message");`.

Here is the code of this basic program :

<div class="filename" >GBPlatformer01.ino</div>
```
#include <Gamebuino-Meta.h>
  
void setup() {
  // initialization of the Gamebuino META
  gb.begin();
}

void loop() {
  // wainting loop
  gb.waitForUpdate();

  // Start by erasing the screen
  gb.display.clear();

  /*************************************************/
  /**/   /* A REMPLACER PAR VOTRE PROGRAMME */   /**/
  /**/ gb.display.println("");                   /**/
  /**/ gb.display.println("   - Platformer -");  /**/
  /**/ gb.display.println("");                   /**/
  /**/ gb.display.println("");                   /**/
  /**/ gb.display.println("   Workshop 01 :");   /**/
  /**/ gb.display.println("     Les bases");     /**/
  /*************************************************/
}
```

Upload your program to see the message displayed, here is the result :

![Basic program](./../../img/E01/home_screen_v1.png)

You can download <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v0.0.zip" class="external-link" >here the basic program</a>.

## The character

### Create the structure

Well, now that we have a program that works, let's look at what we'll need to manage the character.

Although we will only display this character without managing his movements in this first step, we will need :
* of its horizontal position that we will manage with an integer `x` ;
* from its vertical position also managed with an integer `y` ;
* of the character's direction of movement managed with a boolean, `true` to go left and `false` to go right.

To represent the character we will use a structure (struct), as we saw in the tutorial : <a href="https://gamebuino.com/fr/creations/structurer-les-objets-de-votre-programme" class="external-link" >Structure the objects of your program</a> (sorry, it's french tutorial).

We will call this structure `Character` and we will create it in the header file `Character.h`.

> Why do you capitalize the name of the structure ? This is a developer agreement. If your structure has several words, for example a structure that stores a higher score : we will name it `HighScore` (each new word starts with a capital letter).

> A header file is the ideal place to define the prototype functions, custom types and structures.

If you don't see how to implement the character structure or if you have succeeded and want to compare your code, here is the structure we will use in this workshop :

<div class="filename" >Character.h</div>
```
#ifndef PLATFORMER_CHARACTER
#define PLATFORMER_CHARACTER

#include <Gamebuino-Meta.h>

struct Character {
  int8_t x;
  int8_t y;
  bool toTheLeft;
};

#endif
```

### Initialize the character

Now that we have the character structure, we need to write a function that initializes the character data. We will make our character appear at the bottom and center of the screen, and move to the right, for this we must initialize :
* the x-position of the character at 40, half the screen;
* the character's y-position at screen height - delta ;
* the direction (not to be forgotten).

Note : the delta for the character's y-position is the number of pixels below the character's center.

We will also define some constants before we see the solution.

Let's create a file `Constants.h`, and add the following lines to it :

<div class="filename" >Constants.h</div>
```
#ifndef PLATFORMER_CONSTANTS
#define PLATFORMER_CONSTANTS

#include <Gamebuino-Meta.h>

// For a character 8 pixels high by 6 pixels wide
const uint8_t WIDTH_HERO = 6;
const uint8_t HEIGHT_HERO = 8;
const uint8_t UNDER_CENTER_X_HERO = 3; // à droite du héro
const uint8_t UNDER_CENTER_Y_HERO = 4; // en dessous du héro
const uint8_t OVER_CENTER_X_HERO = 3; // à gauche du héro
const uint8_t OVER_CENTER_Y_HERO = 4; // au dessus du héro

#endif
```

As you can see, our character will be 6 pixels wide by 8 pixels high. And for the delta we mentioned earlier, it is the constant `UNDER_CENTER_Y_HERO`.

Here is the signature of the function used to initialize the character, a function that we will declare in `Character.h` :

<div class="filename" >Character.h</div>
```
void initCharacter(Character &aCharacter);
```

> You will notice that we use a reference passage using the `&` symbol. Without this reference passage we would have a problem : indeed a copy passage that we would have obtained with `void initCharacter(Character aCharacter);` would only perform the initialization inside this function. Once back in the main program it would be as if nothing had been done. In addition, a copy pass is more memory-intensive.

The code for this function, to be placed in `Character.cpp`, is as follows :

<div class="filename" >Character.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
#include "Character.h"
#include "Constants.h"

void initCharacter(Character &aCharacter) {
  // we force the hero's initial position in the middle of the screen and tackle the ground
  aCharacter.x = 40;
  aCharacter.y = gb.display.height() - UNDER_CENTER_Y_HERO;

  // we force the character's direction to the right
  aCharacter.toTheLeft = false;
}
```

### Main program

Let's modify the main program to create a character, initialize it and display this information. But first we will take the opportunity to add a state system to our game and thus modulate the display according to the state.

First let's add the following constants in `Constants.h`, which correspond to the states we will need to start with

<div class="filename" >Constants.h</div>
```
const uint8_t HOME_STATE = 1;
const uint8_t LAUNCH_PLAY_STATE = 2;
const uint8_t PLAY_STATE = 3;
```

Let's not forget to include constants in the main program and then add a variable to manage the state. It must be declared outside the `setup` and `loop` functions :

<div class="filename" >GBPlatformer01.ino</div>
```
#include "Constants.h"

uint8_t stateOfGame;

void setup() {
  // ...
}

void loop() {
  // ...
}
```

In the `setup` function, after the console initialization, let's initialize the game state to the value `HOME_STATE`, like this :

<div class="filename" >GBPlatformer01.ino</div>
```
void setup() {
  // initialization of Gamebuino META
  gb.begin();

  stateOfGame = HOME_STATE;
}
```

It is necessary to instantiate the structure for our character, as for the variable that manages the state, apart from the `setup` and `loop` functions :

<div class="filename" >GBPlatformer01.ino</div>
```
#include "Constants.h"
#include "Character.h"

uint8_t stateOfGame;
Character hero;

void setup() {
  // ...
}

void loop() {
  // ...
}
```

Finally, let's modify the program in the `loop` function to :
* switch to the `LAUNCH_PLAY_STATE` state when we are in the `HOME_STATE` state (we will add the menu to this state at the end of this step) ;
* when we are in the `LAUNCH_PLAY_STATE` state we must initialize the character information using the `initCharacter` function and switch to the `PLAY_STATE` state ;
* when we are in the `PLAY_STATE` state, it is necessary to display the character information ;
* by default, if we are not in any of these states (which should not happen) then we will display the original text.

We will use the conditional structure `switch` to do this work.

To display the character information we will use `gb.display.printf`, this function is used as follows :

<div class="filename" >Exemple</div>
```
gb.display.printf("x = %d", hero.x);
```

Here is what our state management looks like :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    case HOME_STATE:
      stateOfGame = LAUNCH_PLAY_STATE;
      break;
    case LAUNCH_PLAY_STATE:
      initCharacter(hero);
      stateOfGame = PLAY_STATE;
      break;
    case PLAY_STATE:
      gb.display.printf("x,y = %d,%d", hero.x, hero.y);
      gb.display.println("");
      gb.display.printf("to the left = %d", hero.toTheLeft);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

You can upload your program to the console, which should display something like :

<div class="filename" >Console display</div>
```
x,y = 40,60
to the left = 0
```

> Note that a Boolean displays 0 if it is `false` and 1 if it is `true`.

### Display the character

Before going to the character display, let's define two new constants :
* a constant that defines the color when the character goes to the left (worth for example, `LIGHTBLUE`);
* another constant that defines the color when the character goes to the right (worth for example, `BLUE`).

To do this, let's add the following constants in the `Constants.h` file :

<div class="filename" >Constants.h</div>
```
const Color HERO_L_COLOR = LIGHTBLUE;
const Color HERO_R_COLOR = BLUE;
```

We will manage the display of the game in the `Display.h` and `Display.cpp` files.

Here is the content of the file `Display.h` :

<div class="filename" >Display.h</div>
```
#ifndef PLATFORMER_DISPLAY
#define PLATFORMER_DISPLAY

#include <Gamebuino-Meta.h>

#include "Constants.h"
#include "Character.h"

void paint(Character &aCharacter);
void paintHero(Character &aCharacter);
void paintBox(
    const int8_t aX, const int8_t aY, 
    const uint8_t aWidth, const uint8_t aHeight, 
    const Color aColor
  );

#endif
```

Some details about the functions that are declared in `Display.h` :
* `paint` will be the only function used in the main program and it will be in charge of doing all the display ;
* `paintHero` performs the character display via `paintBox` ;
* `paintBox` draws a rectangle according to the parameters passed to it.

The `paint` function will only call `paintHero` for the moment.

All these functions must be defined in `Display.cpp`, remember to include `Display.h` (in `Display.cpp`).

To change the color you must use <a href="https://gamebuino.com/academy/reference/graphics-setcolor" class="external-link" >gb.display.setColor</a> and to draw a full rectangle you must use <a href="https://gamebuino.com/academy/reference/graphics-fillrect" class="external-link" >gb.display.fillRect</a>.

Here is the code of the `paintBox` function :

<div class="filename" >Display.cpp</div>
```
void paintBox(
    const int8_t aX, const int8_t aY, 
    const uint8_t aWidth, const uint8_t aHeight, 
    const Color aColor
) {
  gb.display.setColor(aColor);
  gb.display.fillRect(aX, aY, aWidth, aHeight);
}
```

For the character display we will use the ternary operator, this in order to choose the color.

Here is a classic code :

<div class="filename" >Example</div>
```
char signe = '-';
if(nb > 0) {
  signe = '+';
}
```

The latter can be replaced by the following ternary operator :

<div class="filename" >Example</div>
```
char signe = (nb > 0) ? '+' : '-';
```

Here is the code of the function `paintHero` :

<div class="filename" >Display.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void paintHero(Character &aCharacter) {
  Color heroColor = (aCharacter.toTheLeft ? HERO_L_COLOR : HERO_R_COLOR);
  paintBox(
    aCharacter.x - OVER_CENTER_X_HERO, aCharacter.y - OVER_CENTER_Y_HERO, 
    WIDTH_HERO, HEIGHT_HERO, 
    heroColor
  );
}
```

And here is the code of the `painted` function :

<div class="filename" >Display.cpp</div>
```
void paint(Character &aCharacter) {
  paintHero(aCharacter);
}
```

Let's go back to the main program. Don't forget to include `Display.h` ! Let's replace the display of character information by the display of the game, which is the following code :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes...
#include "Display.h"

// Global variables...

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
      stateOfGame = LAUNCH_PLAY_STATE;
      break;
    case LAUNCH_PLAY_STATE:
      // ...
      break;
    case PLAY_STATE:
      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

If you upload your program, you should see a nice blue rectangle at the bottom of the screen : it's your character! Here is a screenshot of the game :

![Character display](./../../img/E01/character_v1.png)

## Add a menu

We will add a menu to our game, we will complete it as the workshop progresses.

Let's create a file named `Lang.h`, we will limit ourselves to English. We will see in the bonuses how to manage several languages. Here is the content of the file :

<div class="filename" >Lang.h</div>
```
#ifndef PLATFORMER_LANG
#define PLATFORMER_LANG

static const char * PLAY_EN = "Play";

#endif
```

We will add menu management in `Display.h` and `Display.cpp`. First, you must include `Lang.h` in `Display.h`. Then let's add the prototype of the following function (still in `Display.h`) :

<div class="filename" >Display.h</div>
```
const uint8_t paintMenu();
```

For the menu we will use the function <a href="https://gamebuino.com/academy/reference/gb-gui-menu" class="external-link" >gb.gui.menu</a>. This function takes as parameter a string for the menu name, and a string table for the items. For the moment the menu contains only one item.

Here is the code of the function `paintMenu` to be placed in `Display.cpp` :

<div class="filename" >Display.cpp</div>
```
const uint8_t paintMenu() {
  const char* items[] = {
    PLAY_EN
  };

  const uint8_t indexItem = gb.gui.menu("Menu", items);
  uint8_t choice = HOME_STATE;
  if(items[indexItem] == PLAY_EN) {
    choice = LAUNCH_PLAY_STATE;
  }
  return choice;
}
```

In the main program, replace `stateOfGame = LAUNCH_PLAY_STATE;` with `stateOfGame = paintMenu();` remember, it's in the `HOME_STATE` state, here is the code :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
  gb.display.clear();

  switch(stateOfGame) {
    case HOME_STATE:
      stateOfGame = paintMenu();
      break;
    case LAUNCH_PLAY_STATE:
      // ...
      break;
    case PLAY_STATE:
      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

In addition to this menu, we will add a command to return to the menu while the game is running. To do this, let's create the file `Commands.h` with the following code :

<div class="filename" >Commands.h</div>
```
#ifndef PLATFORMER_COMMANDS
#define PLATFORMER_COMMANDS

#include <Gamebuino-Meta.h>

#include "Constants.h"

const uint8_t manageCommands();

#endif
```

The code of the `manageCommands` function is relatively simple, if the menu button is pressed then we return to the menu, otherwise we stay in the game, i.e. the following code to write in `Commands.cpp` :

<div class="filename" >Commands.cpp</div>
```
#include "Commands.h"

const uint8_t manageCommands() {
  return ( gb.buttons.pressed(BUTTON_MENU) ?
      HOME_STATE
    :
      PLAY_STATE
    );
}
```

As you can see, we are using a ternary operator again.

In the main program, let's include `Commands.h`, in the state `PLAY_STATE` and before displaying the game, let's add the call to the `manangeCommands` function either the following code :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes...
#include "Commands.h"

// Global variables...

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
      stateOfGame = manageCommands();
      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

You can upload your program to the console, you will have a menu with the item "Play", which allows you to launch the game. During the game you can return to the menu with the menu button.

## Conclusion

You have reached the end of this first step : we have created and displayed a character and we have managed the basic commands to navigate in our game, partly through a menu.

If you have finished or are having problems you can download the solution <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v1.0.zip" class="external-link" >here</a>.

In the next step, i.e. the second, we will discuss the character's movements.

Feel free to give me a feedback : the improvements you would make (an outside look is always welcome), the things you didn't understand, the mistakes, etc. To leave a feedback, please add a comment to the following creation :

{% include react-btn.html %}
