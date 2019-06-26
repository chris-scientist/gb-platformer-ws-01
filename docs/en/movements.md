---
layout: step
title: "Movement management"
permalink: /en/movements/
next: /en/free-fall-and-platforms/
link-other-lang: /fr/deplacements/
lang: en
---

# Step 2 - Movement management

## Introduction

In this second step, we will animate the character by implementing the moves :
* go right ;
* go left ;
* jump.

*I invite you to download <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v1.0.zip" class="external-link" >the code</a> which is the result of the first step in order to start on a common basis.*

## Move to the right or left

We will see here how to implement the move to the right and then to the left.

### Move to the right

The first thing to do is to add our character as a parameter of the `manageCommands` method. 
Remember the method is declared in the file `Commands.h`. Here is his new prototype :

<div class="filename" >Commands.h</div>
```
const uint8_t manageCommands(Character &aCharacter);
```

It is necessary to include `Character.h` in `Commands.h`.

Here is the pseudo code of the move to the right :

<div class="filename" >Pseudo code</div>
```
IF right button pressed THEN
  IF Character always within the limits of the screen THEN
    Increase the x-position of the character of a unit
    Indicate that the character is moving to the right
  END IF
END IF
```

For a "fast" movement, we will use the function <a href="https://gamebuino.com/academy/reference/gb-buttons-repeat" class="external-link" >gb.buttons.repeat</a>.

To make an increment of a unit we will use the following shortcut `aCharacter.x++;`, this is equivalent to `aCharacter.x += 1;`, or `aCharacter.x = aCharacter.x + 1;`.

In the definition of the `manageCommands` function, whose character must be added as a parameter, let's add the following code :

<div class="filename" >Commands.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t manageCommands(Character &aCharacter) {

  if(gb.buttons.repeat(BUTTON_RIGHT, 1)) {
    if(aCharacter.x < (gb.display.width() - UNDER_CENTER_X_HERO)) {
      aCharacter.x++;
      aCharacter.toTheLeft = false;
    }
  }
  
  return (gb.buttons.pressed(BUTTON_MENU) ? HOME_STATE : PLAY_STATE);
}
```

You can upload your program to your console. When you hold down the right button, you notice that your character will move to the right until he reaches the right edge of the screen. 
This is a first part of the movement is developed.

### Move to the left

I now propose that you code the movement to the left independently. If you can't do it, I'll give you some tips first. If you still don't see how to do it or you want to check your code : I'll give you the solution.

#### Move to the left : some leads

For the screen exit constraint, it must be checked that :

<div class="filename" >Code</div>
```
aCharacter.x > OVER_CENTER_X_HERO
```

Then to go to the left, you have to decrement the character's `x` position. To do this there is, as for incrementation, a shortcut which is :

<div class="filename" >Code</div>
```
aCharacter.x--;
```

Here are the two tips that coupled with the move code on the right should allow you to write the move on the left.

#### Move to the left : the solution

To go to the left, here is the code to write in the `manageCommands` function :

<div class="filename" >Commands.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t manageCommands(Character &aCharacter) {

  if(gb.buttons.repeat(BUTTON_RIGHT, 1)) {
    if(aCharacter.x < (gb.display.width() - UNDER_CENTER_X_HERO)) {
      aCharacter.x++;
      aCharacter.toTheLeft = false;
    }
  } else if(gb.buttons.repeat(BUTTON_LEFT, 1)) {
    if(aCharacter.x > OVER_CENTER_X_HERO) {
      aCharacter.x--;
      aCharacter.toTheLeft = true;
    }
  }
  
  return (gb.buttons.pressed(BUTTON_MENU) ? HOME_STATE : PLAY_STATE);
}
```

Before testing, remember to update the main program, especially the call of the `manageCommands` function in the `PLAY_STATE` state :

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
      stateOfGame = manageCommands(hero);
      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

### Test !

It is important to test your program every time you make a change. As we have just implemented the left and right move, upload your program, and see that the game reacts as you wish.

![Left and right movements](./../../img/E02/deplacements_LeftAndRight_v1.gif)

## Jump

For the jump, the character will have a different behavior depending on whether you have impulse or not :
* if the character does not gain impulse, then he will jump on the spot ;
* if the character gains impulse, then he will make a "bell jump".

Before embarking on the development of the jump that meets its specifications, let's look at the theory that will allow us to make such a jump.

### Jump : the theory

To implement the jump we will need to add to our character :
* A horizontal speed, which according to its value will allow us to do the bell jump or not, let us call it `vx`.
* A vertical speed, which will allow us to evolve in the air, let's call it `vy`.
* The old value in y that we will call `oldY` that we will use mainly in the next step.
* The character's state : on the ground (i.e. on a platform), if he gives an impulse to jump or if he jumps, let's call it `state'.

Let's see the main steps of the jump :
1. The player gives the order to jump, the character's state changes to `PUSH_FOR_JUMP_STATE`.
2. During the impulse, initialize the vertical speed with the initial vertical speed (we will use a constant for the latter).
3. Change the character's state to the `JUMP_STATE` state.
4. Start the jump.
5. As long as the character is in the air, continue the jump.
6. If the character is out of the screen, then the jump is cancelled and gravity is played (we will implement gravity in the next step).
7. If the character is in contact with a platform (for the moment the only platform managed is the ground) then stop the jump.

From a mathematical point of view, to make the jump you need :
1. Make the vertical velocity evolve with gravity, i.e. `vy += g;`, with `g` the gravity constant.
2. Move the horizontal position according to the horizontal speed, i.e. `x += vx;`.
3. Move the vertical position according to the vertical speed, i.e. `y += vy;`.

In addition, the vertical speed must be initialized, i.e. `vy = -vy0;` with `vy0` the initial vertical speed, to be performed only at the time of the pulse. In addition it is necessary to store 'y' in `oldY` before changing the value of 'y' but we will see why later on it will be easier with the example.

Don't be afraid of this whole theory, we'll move on to implementation.

### Jump : the pratrice

#### Evolution of the character

Before we change the character, let's add some constants in our `Constants.h` files :

<div class="filename" >Constants.h <span>/!\ Horizontal scrolling /!\</span></div>
```
// State of character
const uint8_t ON_THE_PLATFORM_STATE = 0; // ... the character is on the platform
const uint8_t FREE_FALL_STATE = 1; // ......... the character is free fall
const uint8_t PUSH_FOR_JUMP_STATE = 2; // ..... the player gives an impulse to jump
const uint8_t JUMP_STATE = 3; // .............. the player jump

// Platform IDs
const uint8_t NO_ID = 0;
const uint8_t ID_GROUND = 1;

// Gravity
const uint8_t GRAVITY = 2;

// Jump settings
const uint8_t HORIZONTAL_VELOCITY = 3; // ...... horizontal velocity
const uint8_t INIT_VERTICAL_VELOCITY = 10; // ... initial vertical velocity
```

Then, let's add the following attributes to the character (see `Character.h`) :

<div class="filename" >Character.h</div>
```
struct Character {
  /* ... */
  int8_t oldY; // ......... position y at last refresh
  int8_t vx; // ........... horizontal velocity
  int8_t vy; // ........... vertical velocity
  uint8_t state; // ....... state of character : see constants
};
```

Let's not forget to initialize the new attributes in `initCharacter` (which is in `Character.cpp`) :

<div class="filename" >Character.cpp</div>
```
// the previous position 'y' is initialized
aCharacter.oldY = aCharacter.y;

// the vertical and horizontal velocity is initialized
aCharacter.vx = 0;
aCharacter.vy = 0;

// by default, the player is on the ground
aCharacter.state = ON_THE_PLATFORM_STATE;
```

We still need to add a function that corrects the `y` position.

In `Character.h`, let's write the prototype of the function `rectifyPositionY` :

<div class="filename" >Character.h</div>
```
void rectifyPositionY(Character &aCharacter);
```

The code of the function `rectifyPositionY`, to be placed in `Character.cpp`, is as follows :

<div class="filename" >Character.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Correct the y-position (the function will evolve in the next step)
void rectifyPositionY(Character &aCharacter) {
  aCharacter.y = (gb.display.height() - UNDER_CENTER_Y_HERO);
}
```

Good ! Now that we have made the necessary modifications to the character, we need to look at whether we are colliding with a platform or jumping off the screen.

#### A story of collisions

Before implementing the jump, we will write two functions :
* the first one allows to detect a collision with the ground ;
* the second one allows to stop the jump if we jump out of the screen.

In a new file, which we will name `PhysicsEngine.h`, we add the following code :

<div class="filename" >PhysicsEngine.h</div>
```
#ifndef PLATFORMER_PHYSICS_ENGINE
#define PLATFORMER_PHYSICS_ENGINE

#include <Gamebuino-Meta.h>

#include "Constants.h"
#include "Character.h"

const uint8_t isOnOnePlatform(const Character &aCharacter);

#endif
```

For the moment, the `isOnOnePlatform` function will only check whether we are in collision with the ground or not. We will evolve this function in the next step.

So here is the code to detect a collision with the ground, to be written in the file `PhysicsEngine.cpp`, remember to include `PhysicsEngine.h` (in `PhysicsEngine.cpp`) :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t isOnOnePlatform(const Character &aCharacter) {
  return ( (aCharacter.y + aCharacter.vy) >= (gb.display.height() - UNDER_CENTER_Y_HERO) ) ? ID_GROUND : NO_ID ;
}
```

Let's add the method that detects that the character is off the screen. This function must, in addition to detecting the "out of world", correct the character's `x` position, in order to avoid making it disappear.

We will add this last one in the file `PhysicsEngine.h` :

<div class="filename" >PhysicsEngine.h</div>
```
bool isOutOfWorld(Character &aCharacter);
```

Here is the code of the function `isOutOfWorld` (to be written in the file `PhysicsEngine.cpp`) :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
bool isOutOfWorld(Character &aCharacter) {
  if( aCharacter.x <= OVER_CENTER_X_HERO ) {
    // If the character comes out on the left of the screen
    // so we force his coordinates so he's always on the screen.
    aCharacter.x = OVER_CENTER_X_HERO;
    return true;
  } else if( aCharacter.x >= (gb.display.width() - UNDER_CENTER_X_HERO) ) {
    // If the character comes out on the right of the screen
    // so we force his coordinates so he's always on the screen.
    aCharacter.x = gb.display.width() - UNDER_CENTER_X_HERO;
    return true;
  }
  return false;
}
```

With the boundaries determined, let's move on to the development of the jump.

##### A little bit of physics

To manage the impulse, we will modify the `manageCommands` function (which is in the `Commands.cpp` file) :

<div class="filename" >Commands.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t manageCommands(Character &aCharacter) {
  aCharacter.vx = 0; // by default, no horizontal velocity

  if(gb.buttons.repeat(BUTTON_RIGHT, 1)) {
    if(aCharacter.x < (gb.display.width() - UNDER_CENTER_X_HERO)) {
      /* ... */
      aCharacter.vx = HORIZONTAL_VELOCITY; // positive horizontal velocity to go right
    }
  } else if(gb.buttons.repeat(BUTTON_LEFT, 1)) {
    if(aCharacter.x > OVER_CENTER_X_HERO) {
      /* ... */
      aCharacter.vx = -HORIZONTAL_VELOCITY; // negative horizontal speed to go left
    }
  }

  return (gb.buttons.pressed(BUTTON_MENU) ? HOME_STATE : PLAY_STATE);
}
```

Still in the `manageCommands` function, it is necessary to write the code so that when the player presses button A and the character does not jump, the initial impulse for the jump is given, either :

<div class="filename" >Commands.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const uint8_t manageCommands(Character &aCharacter) {
  aCharacter.vx = 0; // by default, no horizontal velocity

  if(gb.buttons.repeat(BUTTON_RIGHT, 1)) {
    /* ... */
  } else if(gb.buttons.repeat(BUTTON_LEFT, 1)) {
    /* ... */
  }

  if(gb.buttons.pressed(BUTTON_A) && aCharacter.state != JUMP_STATE) {
    aCharacter.state = PUSH_FOR_JUMP_STATE;
  }

  return (gb.buttons.pressed(BUTTON_MENU) ? HOME_STATE : PLAY_STATE);
}
```

Let's create, in `PhysicsEngine.h`, the prototype of the function `jumpMovement`, that is :

<div class="filename" >PhysicsEngine.h</div>
```
void jumpMovement(Character &aCharacter);
```

Let's add in `PhysicsEngine.cpp` the code of `jumpMovement` :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
// Performs the character's movement during the jump
void jumpMovement(Character &aCharacter) {
  aCharacter.oldY = aCharacter.y; // ... the previous vertical position is recorded
  aCharacter.vy += GRAVITY; // ......... we change the vertical velocity
  aCharacter.x += aCharacter.vx; // .... we change the horizontal position
  aCharacter.y += aCharacter.vy; // .... we change the vertical position
}
```

In `PhysicsEngine.h`, let's add the prototype of the `jump` function, that is :

<div class="filename" >PhysicsEngine.h</div>
```
void jump(Character &aCharacter);
```

The implementation of the `jump` function, to be added in `PhysicsEngine.cpp`, is as follows :

<div class="filename" >PhysicsEngine.cpp</div>
```
// Implementation of the character jump
void jump(Character &aCharacter) {
  const uint8_t platformId = isOnOnePlatform(aCharacter);

  if(aCharacter.state == PUSH_FOR_JUMP_STATE) {
    // the player gives an impulse for the jump
    // we then initialize the data for the jump
    aCharacter.vy = -INIT_VERTICAL_VELOCITY;
    aCharacter.state = JUMP_STATE;

    jumpMovement(aCharacter);
  } else if( platformId != NO_ID) {
    rectifyPositionY(aCharacter);
    aCharacter.vy = 0;
    aCharacter.state = ON_THE_PLATFORM_STATE;
  } else if( isOutOfWorld(aCharacter) ) {
    // if the jump leads us out of the world
    // then we apply gravity
    aCharacter.state = FREE_FALL_STATE;
  } else {
    // the character jump
    jumpMovement(aCharacter);
  }
  
}
```

The character is ready to jump, almost ready to jump ! Indeed it is necessary to write some more code before the character jumps.

#### Final straight before jumping

Let's go back to the main program, and in particular to the `PLAY_STATE` report where it is necessary to make changes. It is necessary to include `PhysicsEngine.h` in the main program.
The code for this state must be :

<div class="filename" >GBPlatformer01.ino <span>/!\ Horizontal scrolling /!\</span></div>
```
// Autres includes...
#include "PhysicsEngine.h"

void setup() {
  // ...
}

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
      if(hero.state == ON_THE_PLATFORM_STATE) {
        stateOfGame = manageCommands(hero);
      }

      if(hero.state != JUMP_STATE && hero.state != PUSH_FOR_JUMP_STATE) {
        gb.display.println("GRAVITY");
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

As you can see, the player can only act on the controls if he is on a platform, which is currently on the ground.

![Movement : first version of the jump](./../../img/E02/deplacements_Jump_v1.gif
)

#### Some settings

Now you can upload your program to the console, and have fun jumping around. It should be noted that if you jump to the edge of the screen then the game is blocked. We will see in the next step how to solve this problem by implementing gravity / free fall.

Have fun changing the gravity, horizontal speed and initial vertical speed found in `Constants.h`. 
It is important that you play with it to find the right settings.

I insist ! Try to change these settings !

You may have noticed that the settings provided are too important for a platformer that must be played within the limits of the screen only.

Thus, in `Constants.h`, and after a calibration phase, we have chosen the following settings :

<div class="filename" >Constants.h <span>/!\ Horizontal scrolling /!\</span></div>
```
// Gravité
const uint8_t GRAVITY = 1;

// Paramétrage du saut
const uint8_t HORIZONTAL_VELOCITY = 2; // ...... horizontal velocity
const uint8_t INIT_VERTICAL_VELOCITY = 6; // ... initial vertical velocity
```

This is the jump before callibration :

![Movement : first version of the jump](./../../img/E02/deplacements_Jump_v1.gif
)

And here is the current jump (after callibration) :

![Movement : first version of the jump after callibration](./../../img/E02/deplacements_NewJump_v1.gif
)

## Conclusion

You are at the end of this second step where we have added the moves for the character : move left, move right and jump.

If you have finished or are having problems you can download the solution <a href="#" class="external-link" >here</a>.

In the next step, i.e. the third, we will address platform management from creation to display, including collision management. And we will also implement free fall.

Feel free to give me a feedback : the improvements you would make (an outside look is always welcome), the things you didn't understand, the mistakes, etc. To leave a feedback, please add a comment to the following creation :

{% include react-btn.html %}
