---
layout: step
title: "What will be your best score ?"
permalink: /en/best-score/
next: /en/end/
link-other-lang: /fr/meilleur-score/
lang: en
---

# Step 6 - What will be your best score ?

## Introduction

In this sixth and final step, we will implement : the management of high scores, with the necessary interfaces to add this functionality.

<!--I invite you to download <a href="#" class="external-link" >the code</a> that is the result of the fifth step in order to start on a common basis.-->

## Some specifications

Our platform game will handle three high scores.

The score as you probably suspect will depend on the time between the beginning of the game and the opening of the door.
The score is therefore less than 20 seconds (remember the game over implemented in the previous step). 
The nickname will be a string of 5 characters.

We will set the following rule : if the time of the current game is equal to or lower than one of the high scores then we will have a new high score.

Now that the specifications have been defined, let's move on to data structuring.

## How is a high score defined ?

We will create two data structures to manage our high scores.

First we will need a structure that manages a high score with :
* a nickname that we will manage via a string (5 characters precisely) ;
* a score managed via an integer.

We will also need a structure to manage the three high scores containing :
* three high scores using the defined structure instead ;
* the number of high scores recorded managed via an integer ;
* the index of the new high score represented by an integer.

Here is the first structure that we will name `HighScore` in the file `HighScore.h` :

<div class="filename" >HighScore.h</div>
```
#ifndef PLATFORMER_HIGH_SCORE
#define PLATFORMER_HIGH_SCORE

#include <Gamebuino-Meta.h>
#include <cstring>

struct HighScore {
  char nameOfScore[6];
  int32_t score;
};

#endif
```

We will call the second structure `HighScoreManager` and place it in the file `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
#ifndef PLATFORMER_HIGH_SCORE_MANAGER
#define PLATFORMER_HIGH_SCORE_MANAGER

#include <Gamebuino-Meta.h>
#include "HighScore.h"
#include "Constants.h"

struct HighScoreManager {
  HighScore highScore1;
  HighScore highScore2;
  HighScore highScore3;
  uint8_t nbHighScore;
  uint8_t indexNewHighScore;
};

#endif
```

## Add features

### Initialization of the manager

To initialize the high score manager, we will need to initialize each of the high scores. 
So in `HighScore.h`, we will create the function `setHighScore` :

<div class="filename" >HighScore.h</div>
```
void setHighScore(
  HighScore &aHighScore, 
  char * aName, 
  int32_t aScore
);
```

In `HighScore.cpp`, after including `HighScore.h`, let's define the `setHighScore` function :

<div class="filename" >HighScore.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void setHighScore(HighScore &aHighScore, char * aName, int32_t aScore) {
  strncpy(aHighScore.nameOfScore, aName, 6);
  aHighScore.nameOfScore[5] = '\0';
  aHighScore.score = aScore;
}
```

We will start by initializing the high score manager ; let's add the prototype of the `initHighScoreManager` function in `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
void initHighScoreManager(HighScoreManager &aManager);
```

In `HighScoreManager.cpp`, let's include the file `HighScoreManager.h` and implement the function `initHighScoreManager` :

<div class="filename" >HighScoreManager.cpp</div>
```
void initHighScoreManager(HighScoreManager &aManager) {
  setHighScore(aManager.highScore1, "     ", 0);
  setHighScore(aManager.highScore2, "     ", 0);
  setHighScore(aManager.highScore3, "     ", 0);
  aManager.nbHighScore = 0;
  aManager.indexNewHighScore = 0;
}
```

In the main program, remember to include `HighScoreManager.h` and instantiate the score manager outside the `setup` and `loop` functions :

<div class="filename" >GBPlatformer01.ino</div>
```
// Other includes...
#include "HighScoreManager.h"

// Other global variables...
HighScoreManager highScoreManager;

void setup() {
  // ...
}

void loop() {
  // ...
}
```

Then, still in the main program, let's call the initialization in the `setup` function :

<div class="filename" >GBPlatformer01.ino</div>
```
void setup() {
  // initialization of the Gamebuino META
  gb.begin();

  createTimer(myTimer);
  initHighScoreManager(highScoreManager);

  stateOfGame = HOME_STATE;
}
```

### Load the high scores recorded

To load the high scores previously recorded, we will first add the following constants in `Constants.h` :

<div class="filename" >Constants.h</div>
```
// High score identifier
const uint8_t NO_HIGH_SCORE = 0;
const uint8_t HIGH_SCORE_1 = 1;
const uint8_t HIGH_SCORE_2 = 2;
const uint8_t HIGH_SCORE_3 = 3;

// Block to save high scores
const uint16_t NB_HIGH_SCORE_BLOCK = 0;
const uint16_t NAME_HIGH_SCORE_1_BLOCK = 1;
const uint16_t SCORE_HIGH_SCORE_1_BLOCK = 2;
const uint16_t NAME_HIGH_SCORE_2_BLOCK = 3;
const uint16_t SCORE_HIGH_SCORE_2_BLOCK = 4;
const uint16_t NAME_HIGH_SCORE_3_BLOCK = 5;
const uint16_t SCORE_HIGH_SCORE_3_BLOCK = 6;
```

In `HighScoreManager.h`, let's create the function `loadHighScore` :

<div class="filename" >HighScoreManager.h</div>
```
void loadHighScore(
  HighScore &aScore, 
  uint16_t aBlockName, 
  uint16_t aBlockScore
);
```

In `HighScoreManager.cpp`, let's define the function `loadHighScore` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void loadHighScore(HighScore &aScore, uint16_t aBlockName, uint16_t aBlockScore) {
  char temp[5];
  gb.save.get(aBlockName, temp, 6);
  setHighScore(aScore, temp, gb.save.get(aBlockScore));
}
```

Let's now go back to `HighScoreManager.h` and add the prototype of the `loadAllHighScore` function :

<div class="filename" >HighScoreManager.h</div>
```
void loadAllHighScore(HighScoreManager &aManager);
```

In `HighScoreManager.cpp`, let's implement the function `loadAllHighScore` :

<div class="filename" >HighScoreManaher.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void loadAllHighScore(HighScoreManager &aManager) {
  int32_t nbHighScoreSaved = gb.save.get(NB_HIGH_SCORE_BLOCK);

  if(nbHighScoreSaved > 0) {
    uint8_t nbScore = 0;

    // Load high score 1
    loadHighScore(aManager.highScore1, NAME_HIGH_SCORE_1_BLOCK, SCORE_HIGH_SCORE_1_BLOCK);
    nbScore++;

    // Load high score 2
    if(nbHighScoreSaved >= 2) {
      loadHighScore(aManager.highScore2, NAME_HIGH_SCORE_2_BLOCK, SCORE_HIGH_SCORE_2_BLOCK);
      nbScore++;
    }

    // Load high score 3
    if(nbHighScoreSaved >= 3) {
      loadHighScore(aManager.highScore3, NAME_HIGH_SCORE_3_BLOCK, SCORE_HIGH_SCORE_3_BLOCK);
      nbScore++;
    }

    aManager.nbHighScore = nbScore;
  }
}
```

In the main program, in the `setup` function, after the manager initialization, let's load the best scores :

<div class="filename" >GBPlatformer01.ino</div>
```
void setup() {
  // initialization of the Gamebuino META
  gb.begin();

  createTimer(myTimer);
  initHighScoreManager(highScoreManager);
  loadAllHighScore(highScoreManager);

  stateOfGame = HOME_STATE;
}
```

### Reset the manager

When a new part is launched, it is necessary to reset the high score manager to zero the index of the current high score (so that there is no "selected" score by default).

This function must be created in `HighScoreManager.h` like this :

<div class="filename" >HighScoreManager.h</div>
```
void resetIndexNewHighScore(HighScoreManager &aManager);
```

In `HighScoreManager.cpp`, let's define the function `resetIndexNewHighScore` :

<div class="filename" >HighScoreManager.cpp</div>
```
void resetIndexNewHighScore(HighScoreManager &aManager) {
  aManager.indexNewHighScore = NO_HIGH_SCORE;
}
```

In the main program, in the `LAUNCH_PLAY_STATE` state of the `loop` function, let's add the function we just defined :

<div class="filename" >GBPlatformer01.ino <span>/!\ Horizontal scrolling /!\</span></div>
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
      resetIndexNewHighScore(highScoreManager); // ... we reset the high score table
      resetTimer(myTimer); // ............. we reset the timer
      initObjects(setOfObjects); // ....... we reset objects
      initPlatforms(setOfPlatforms); // ... we reset platforms
      initCharacter(hero);
      stateOfGame = PLAY_STATE;
      break;
    // ...
    default:
      gb.display.println("Votre message");
  }
}
```

### Is the time achieved one of the high scores ?

We will now see all the functions that determine and record the new time if it is one of the high scores.

Let's start with the function that creates a temporary high score, corresponding to a new high score. Let's call this function `createNewHighScore`. The only data needed for this function is the score, which we will pass as a parameter, the nickname will be entered by the user via a call to the `paintInputPseudoWindow` function.

Before writing the function that instantiates this new high score, let's see how to create the function `paintInputPseudoWindow` returning the nickname we will add in `Display.h` :

<div class="filename" >Display.h</div>
```
void paintInputPseudoWindow(char * pseudo);
```

In `Display.cpp`, let's define the function `paintInputPseudoWindow` :

<div class="filename" >Display.cpp</div>
```
void paintInputPseudoWindow(char * pseudo) {
  gb.gui.keyboard("Save your score!", pseudo, 5);
  gb.display.clear();
}
```

Let's place the prototype of `createNewHighScore` in `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
const HighScore createNewHighScore(const uint32_t aTimeOfPart);
```

Let's develop the `createNewHighScore` function in `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp</div>
```
const HighScore createNewHighScore(const uint32_t aTimeOfPart) {
  char pseudo[6] = { '\0' };
  
  paintInputPseudoWindow(pseudo);

  HighScore newHighScore;
  strncpy(newHighScore.nameOfScore, pseudo, 6);
  newHighScore.nameOfScore[5] = '\0';
  newHighScore.score = aTimeOfPart;

  return newHighScore;
}
```

Consider including `Display.h` in `HighScoreManager.h`.

We now need a function that exchanges scores, for example : the high score 2 is replaced by the high score 1, or the high score 3 is replaced by the high score 2, etc. To do this, we will add the `swapHighScore` function, this function takes as parameter the references of two high scores : the one to replace and the new one.

We will create the function `swapHighScore` in `HighScoreManager.h`:

<div class="filename" >HighScoreManager.h</div>
```
void swapHighScore(
  HighScore &aHighScore, 
  const HighScore & aNewHighScore
);
```

In `HighScoreManager.cpp`, let's define the function `swapHighScore` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void swapHighScore(HighScore &aHighScore, const HighScore & aNewHighScore) {
  strncpy(aHighScore.nameOfScore, aNewHighScore.nameOfScore, 6);
  aHighScore.nameOfScore[5] = '\0';
  aHighScore.score = aNewHighScore.score;
}
```

Let's move on to the `compareTime` function which compares the duration of a high score with another duration (the new time achieved, candidate for a new record).
This function returns a number between -1 and 1 (bounds included), i.e. `a` the duration of a high score and `b` a duration:
* If a < b then return -1
* If a == b then return 0
* If a > b then return 1

Here is the prototype of the `compareTime` function, to be written in `HighScore.h` :

<div class="filename" >HighScore.h</div>
```
int8_t compareTime(
  const HighScore &aScore, 
  const int32_t aTimeInSeconds
);
```

Let's implement this function :

<div class="filename" >HighScore.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
int8_t compareTime(const HighScore &aScore, const int32_t aTimeInSeconds) {
  if(aScore.score == aTimeInSeconds) {
    return 0;
  } else if(aScore.score > aTimeInSeconds) {
    return 1;
  }
  return -1;
}
```

We will now write a function that returns `true` if the score obtained is better or equal to a given high score. 
This function, which we will call `isBetterOrEqualToScore`, will use the result of `compareTime`.

Let's go back to `HighScoreManager.h`, and create the function `isBetterOrEqualToScore` :

<div class="filename" >HighScoreManager.h</div>
```
const bool isBetterOrEqualToScore(int8_t aValue);
```

In `HighScoreManager.cpp`, let's define the function :

<div class="filename" >HighScoreManager.cpp</div>
```
const bool isBetterOrEqualToScore(int8_t aValue) {
  return (aValue == 0 || aValue == 1);
}
```

We will discuss an important function for the management of the high scores, this function determines if the score obtained (for the current game) is a high score or not. There are a number of tests to do, but don't be afraid ! We will name this function `setHighScore4Time`.

The prototype of this function is as follows :

<div class="filename" >HighScoreManager.h</div>
```
const uint8_t setHighScore4Time(
  HighScoreManager &aManager, 
  const int32_t aTimeOfPart
);
```

It takes the high score manager as a parameter since we will probably have to modify it and the duration of the game that the player has just finished.

Before providing you with its implementation, here is its pseudo code :

<div class="filename" >Pseudo code <span>/!\ Horizontal scrolling /!\</span></div>
```
// Legend :
// nb(HS) for many of the high scores
// SC for score of the current part
// S1 for high score 1
// S2 for high score 2
// S3 for high score 3
// index to indicate the position of the new high score

index = NO_HIGH_SCORE

IF ( nb(HS) > 0 ) THEN // If HS are already registered
    IF ( SC better than/equal to S1 ) THEN
        SWITCH nb(HS) THEN
            DO IF (3 OR 2 HS)
                swap S3 = S2
                swap S2 = S1
                swap S1 = SC
                IF ( nb(HS) = 2 ) THEN
                    nb(HS) = 3
                END IF
            END DO IF
            DO IF (1 HS)
                swap S2 = S1
                swap S1 = SC
                nb(HS) = 2
            END DO IF
        END SWITCH
        index = HIGH_SCORE_1
    END IF
    ELSE IF ( nb(HS) = 3 OR 2) ET ( SC better than/equal to S2 ) THEN
        swap S3 = S2
        swap S2 = SC
        IF ( nb(HS) = 2 ) THEN
            nb(HS) = 3
        END IF
        index = HIGH_SCORE_2
    END ELSE IF
    ELSE IF ( nb(HS) = 3 ET SC better than/equal to S3 ) OR ( nb(HS) = 2 ) THEN
        swap S3 = SC
        IF ( nb(HS) = 2 ) THEN
            nb(HS) = 3
        END IF
        index = HIGH_SCORE_3
    END ELSE IF
    ELSE IF nb(HS) = 1 THEN
        swap S2 = SC
        nb(HS) = 2
        index = HIGH_SCORE_2
    END ELSE IF
END IF
ELSE
    swap S1 = SC
    nb(HS) = 1
    index = HIGH_SCORE_1
END ELSE

return index
```

Let's develop this function :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
const uint8_t setHighScore4Time(HighScoreManager &aManager, const int32_t aTimeOfPart) {
  uint8_t highScoreIndex = NO_HIGH_SCORE;

  if( aManager.nbHighScore > 0) {
    
    const int cmpTime1 = compareTime(aManager.highScore1, aTimeOfPart);
    if( isBetterOrEqualToScore(cmpTime1) ) {
      HighScore newHighScore;
      switch(aManager.nbHighScore) {
        case 3:
        case 2:
          swapHighScore(aManager.highScore3, aManager.highScore2);
          swapHighScore(aManager.highScore2, aManager.highScore1);

          newHighScore = createNewHighScore(aTimeOfPart);

          swapHighScore(aManager.highScore1, newHighScore);

          if(aManager.nbHighScore == 2) {
            aManager.nbHighScore = 3;
          }
          
          break;
        case 1:
          swapHighScore(aManager.highScore2, aManager.highScore1);

          newHighScore = createNewHighScore(aTimeOfPart);

          swapHighScore(aManager.highScore1, newHighScore);
          aManager.nbHighScore = 2;
          break;
      }

      highScoreIndex = HIGH_SCORE_1;
    } else if( ( aManager.nbHighScore == 3 || aManager.nbHighScore == 2 ) &&
      isBetterOrEqualToScore( compareTime(aManager.highScore2, aTimeOfPart) )
    ) {
      swapHighScore(aManager.highScore3, aManager.highScore2);

      HighScore newHighScore;
      newHighScore = createNewHighScore(aTimeOfPart);

      swapHighScore(aManager.highScore2, newHighScore);
      if(aManager.nbHighScore == 2) {
        aManager.nbHighScore = 3;
      }
      highScoreIndex = HIGH_SCORE_2;
    } else if( ( aManager.nbHighScore == 3 && isBetterOrEqualToScore( compareTime(aManager.highScore3, aTimeOfPart) ) ) ||
      aManager.nbHighScore == 2
    ) {
      HighScore newHighScore;
      newHighScore = createNewHighScore(aTimeOfPart);

      swapHighScore(aManager.highScore3, newHighScore);
      if( aManager.nbHighScore == 2) {
        aManager.nbHighScore = 3;
      }
      highScoreIndex = HIGH_SCORE_3;
    } else if( aManager.nbHighScore == 1 ) {
      HighScore newHighScore;
      newHighScore = createNewHighScore(aTimeOfPart);

      swapHighScore(aManager.highScore2, newHighScore);
      aManager.nbHighScore = 2;
      highScoreIndex = HIGH_SCORE_2;
    }
    
  } else {
    HighScore newHighScore;
    newHighScore = createNewHighScore(aTimeOfPart);

    swapHighScore(aManager.highScore1, newHighScore);
    aManager.nbHighScore = 1;
    highScoreIndex = HIGH_SCORE_1;
  }
  
  return highScoreIndex;
}
```

Let's see the functions that allow us to save the best scores on the META, and thus keep them from one execution to the next.

First, let's create the function that performs the unit saving of a better score. To do this, we will create the `saveHighScore` function in `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
void saveHighScore(
  char * aName, 
  int32_t aScore, 
  uint16_t aBlockName, 
  uint16_t aBlockScore
);
```

In `HighScoreManager.cpp`, let's define this function :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void saveHighScore(char * aName, int32_t aScore, uint16_t aBlockName, uint16_t aBlockScore) {
  gb.save.set(aBlockName, aName);
  gb.save.set(aBlockScore, aScore);
}
```

Finally, let's write the function that saves all the high scores. The prototype of this function, which we will call `saveAllHighScore`, is to be placed in `HighScoreManager.h`:

<div class="filename" >HighScoreManager.h</div>
```
void saveAllHighScore(HighScoreManager &aManager);
```

Let's implement this function in `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void saveAllHighScore(HighScoreManager &aManager) {
  switch(aManager.nbHighScore) {
    case 1:
      saveHighScore((char*)aManager.highScore1.nameOfScore, aManager.highScore1.score, NAME_HIGH_SCORE_1_BLOCK, SCORE_HIGH_SCORE_1_BLOCK);
      break;
    case 2:
      saveHighScore((char*)aManager.highScore1.nameOfScore, aManager.highScore1.score, NAME_HIGH_SCORE_1_BLOCK, SCORE_HIGH_SCORE_1_BLOCK);
      saveHighScore((char*)aManager.highScore2.nameOfScore, aManager.highScore2.score, NAME_HIGH_SCORE_2_BLOCK, SCORE_HIGH_SCORE_2_BLOCK);
      break;
    case 3:
      saveHighScore((char*)aManager.highScore1.nameOfScore, aManager.highScore1.score, NAME_HIGH_SCORE_1_BLOCK, SCORE_HIGH_SCORE_1_BLOCK);
      saveHighScore((char*)aManager.highScore2.nameOfScore, aManager.highScore2.score, NAME_HIGH_SCORE_2_BLOCK, SCORE_HIGH_SCORE_2_BLOCK);
      saveHighScore((char*)aManager.highScore3.nameOfScore, aManager.highScore3.score, NAME_HIGH_SCORE_3_BLOCK, SCORE_HIGH_SCORE_3_BLOCK);
      break;
  }
  gb.save.set(NB_HIGH_SCORE_BLOCK, aManager.nbHighScore);
}
```

Before we go to the high score display, let's write the function that triggers the saving of the high scores if there is a new high score. To do this we will create the following function in `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
bool saveScoreIfNewHighScore(
  HighScoreManager &aManager, 
  const int32_t aTimeOfPart
);
```

Let's define this function in `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
bool saveScoreIfNewHighScore(HighScoreManager &aManager, const int32_t aTimeOfPart) {
  // Compare the current score with the scores in memory
  const uint8_t highScoreIndex = setHighScore4Time(aManager, aTimeOfPart);

  // If there is a new higher score then it is saved
  bool haveNewHighScore = (highScoreIndex != NO_HIGH_SCORE);
  if(haveNewHighScore) {
    saveAllHighScore(aManager);
  }
  aManager.indexNewHighScore = highScoreIndex;

  return haveNewHighScore;
}
```

It is this last function that we will call in the main program. We will see where and how later in this step. Let's first look at the functions needed to display the highest scores.

### Manage the display of high scores

To manage the display, we will need a function that returns the high score associated with the index. Here is the prototype of this function to write in `HighScoreManager.h`:

<div class="filename" >HighScoreManager.h</div>
```
const HighScore& getHighScore(
  const HighScoreManager &aManager, 
  uint8_t anIndex
);
```

Let's develop this function in `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
const HighScore& getHighScore(const HighScoreManager &aManager, uint8_t anIndex) {
  switch(anIndex) {
    case HIGH_SCORE_2:
      return aManager.highScore2;
      break;
    case HIGH_SCORE_3:
      return aManager.highScore3;
      break;
  }
  return aManager.highScore1;
}
```

In `HighScoreManager.h`, let's create the function that displays the highest scores :

<div class="filename" >HighScoreManager.h</div>
```
void paintHighScoreWindow(const HighScoreManager& aScoreManager);
```

Let's define this function in `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Horizontal scrolling /!\</span></div>
```
void paintHighScoreWindow(const HighScoreManager& aScoreManager) {
  gb.display.setFontSize(1);
  gb.display.setColor(BROWN);
  gb.display.println("High score");
  gb.display.println("");
  const uint8_t listHighScore[3] = { HIGH_SCORE_1, HIGH_SCORE_2, HIGH_SCORE_3 };
  for(int i=0 ; i<aScoreManager.nbHighScore ; i++) {
    uint8_t index = listHighScore[i];
    const HighScore& highScore = getHighScore(aScoreManager, index);
    if(index == aScoreManager.indexNewHighScore) {
      gb.display.setColor(BROWN);
    } else {
      gb.display.setColor(WHITE);
    }
    gb.display.printf("%s ", highScore.nameOfScore);

    int32_t rest = highScore.score;

    const uint16_t MINUTES_IN_FRAMES = 60*1000;
    const uint16_t SECONDS_IN_FRAMES = 1000;
  
    uint16_t nbMinutes = 0;
    uint16_t nbSeconds = 0;
  
    // Compute the minutes
    if(rest >= MINUTES_IN_FRAMES) {
      nbMinutes = (uint16_t)(rest / MINUTES_IN_FRAMES);
      rest = (rest - (nbMinutes * MINUTES_IN_FRAMES));
    }
    // Compute the seconds
    if(rest >= SECONDS_IN_FRAMES) {
      nbSeconds = (uint16_t)(rest / SECONDS_IN_FRAMES);
      rest = (rest - (nbSeconds * SECONDS_IN_FRAMES));
    }

    gb.display.cursorX = 30;
    if(nbSeconds < 10) {
      gb.display.print("0");
    }
    gb.display.printf("%d s ", nbSeconds);
    uint16_t nbMilliseconds = rest;
    if(nbMilliseconds < 100) {
      gb.display.print("0");
    }
    if(nbMilliseconds < 10) {
      gb.display.print("0");
    }
    gb.display.printf("%d", nbMilliseconds);
    gb.display.println("");
  }

  // Display at the bottom of the screen "go to the menu"
  int l = aScoreManager.nbHighScore;
  while(l<3) {
    gb.display.println();
    l++;
  }
  gb.display.println();
  gb.display.println();
  gb.display.println();
  gb.display.setColor(WHITE);
  gb.display.println("(A) Try again");
  gb.display.println("(B) Menu");
}
```

### Finalize the management of high scores

To start, let's add a new state in `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint8_t HIGH_SCORE_STATE = 9;
```

We will use this constant to display the high score table.

We will now modify the main program.

We will manage the `HIGH_SCORE_STATE` report which displays the high scores :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case HIGH_SCORE_STATE:
      stateOfGame = manageCommandsOutOfGame(stateOfGame);
      
      paintHighScoreWindow(highScoreManager);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

In the `SAVE_HIGH_SCORE_STATE` report, let's add the redirection to the high score display screen if there is a new high score :

<div class="filename" >GBPlatformer01.ino <span>/!\ Horizontal scrolling /!\</span></div>
```
void loop() {
  // waiting loop
  gb.waitForUpdate();

  // clear screen
  gb.display.clear();

  switch(stateOfGame) {
    // ...
    case SAVE_HIGH_SCORE_STATE:
      myTimer.activateTimer = false;
      runTimer(myTimer);

      stateOfGame = (saveScoreIfNewHighScore(highScoreManager, myTimer.timeInMilliseconds) ? HIGH_SCORE_STATE : GAME_IS_FINISH_STATE);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

In `Lang.h`, let's create the menu item to access the high scores :

<div class="filename" >Lang.h</div>
```
static const char * HIGH_SCORE_EN = "High score";
```

In `Display.cpp`, let's modify the `paintMenu` function to add the item to go to the best score either :

<div class="filename" >Display.cpp</div>
```
const uint8_t paintMenu() {
  const char* items[] = {
    PLAY_EN,
    HIGH_SCORE_EN
  };

  const uint8_t indexItem = gb.gui.menu("Menu", items);
  uint8_t choice = HOME_STATE;
  if(items[indexItem] == PLAY_EN) {
    choice = LAUNCH_PLAY_STATE;
  } else if(items[indexItem] == HIGH_SCORE_EN) {
    choice = HIGH_SCORE_STATE;
  }
  return choice;
}
```

Remember to upload the program to your console to test this last feature.

## Conclusion

<!--You can download the <a href="#" class="external-link" >final source code</a>.--> You now have a complete platform set ! Yes it is not transcendent in terms of graphics but be patient, we will see this later, but (spoiler alert) it will not be the objective of the next workshop. The goal with this first workshop was to lay the foundations of a platform game, but this is only the beginning of a series...

Feel free to give me a feedback on this step and on the workshop :

{% include react-btn.html %}

You have reached the end of this workshop, but stay in the area to do other workshops while waiting for the next one. *Don't be too impatient to see the rest of this workshop, indeed I wrote this one in more than 6 months.*

It is by forging that one becomes a blacksmith ;)
