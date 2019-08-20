---
layout: step
title: "Gestion de la partie et chronomètre"
permalink: /fr/partie-et-chrono/
next: /fr/meilleur-score/
link-other-lang: /en/game-and-timer/
lang: fr
---

# Etape 5 - Gestion de la partie et chronomètre

## Introduction

Dans cette cinquième étape, nous allons implémenter : la fin de partie, un chronomètre et un écran de fin de partie et nous gérerons le "Game over" : la fin de partie lorsque le temps maximum autorisé pour compléter le niveau a été dépassé.

<!--*Je vous invite à télécharger <a href="#" class="external-link" >le code</a> qui est le résultat de la quatrième étape afin de partir sur des bases communes.*-->

## Fin de partie

Dans le fichier `Constants.h` ajoutons la constante `GAME_IS_FINISH_STATE`, un nouvel état indiquant que le jeu est fini :

<div class="filename" >Constants.h</div>
```
const uint8_t GAME_IS_FINISH_STATE = 4;
```

Créons le fichier `Game.h` où nous placerons les fonctions relatives à la gestion de la partie. 
A cette étape d'avancement, une partie est terminée lorsque la porte est ouverte. 
Ajoutons donc dans le fichier `Game.h` la fonction `isEndOfGame`  qui permettra de déterminer si la partie est terminée.

Voici le contenu du fichier :

<div class="filename" >Game.h</div>
```
#ifndef PLATFORMER_GAME
#define PLATFORMER_GAME

#include "Constants.h"
#include "Object.h"

bool isEndOfGame(const Object &aDoor);

#endif
```

Créons le fichier `Game.cpp` correspondant pour y coder la fonction `isEndOfGame` :

<div class="filename" >Game.cpp</div>
```
#include "Game.h"

bool isEndOfGame(const Object &aDoor) {
  return (aDoor.state == DOOR_OPENED);
}
```

N'oublions pas d'inclure le fichier `Game.h` dans le programme principal et ajoutons y aussi la redirection vers l'état `GAME_IS_FINISH_STATE` lorsque la partie est finie (à ajouter dans l'état `PLAY_STATE`) :

<div class="filename" >GBPlatformer01.ino</div>
```
// Autres includes...
#include "Game.h"

// Variables globales...

void setup() {
  // ...
}

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

Dans `Lang.h`, ajoutons les constantes suivantes :

<div class="filename" >Lang.h</div>
```
static const char * TRY_AGAIN_EN = "TRY AGAIN";
static const char * A_4_TRY_AGAIN_EN = "(A) Try again";
static const char * B_4_MENU_EN = "(B) Menu";
```

Dans `Display.h`, créons la fonction qui affiche l'écran de fin de jeu :

<div class="filename" >Display.h</div>
```
void paintEndOfGame();
```

Dans `Display.cpp`, implémentons la fonction `paintEndOfGame` :

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

Définissons une fonction de gestion des commandes qui a le comportement suivant :
* si le bouton A est pressé alors lancer une nouvelle partie ;
* si le bouton B est pressé alors retourner au menu ;
* sinon rester dans l'état actuel.

Dans `Commands.h`, ajoutons le prototype de la fonction `manageCommandsOutOfGame` :

<div class="filename" >Commands.h</div>
```
const uint8_t manageCommandsOutOfGame(const uint8_t aState);
```

Dans `Commands.cpp`, définissons la fonction `manageCommandsOutOfGame` qui réagit comme nous l'avons décrit ci-avant :

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

Dans le programme principal, ajoutons l'état fin de jeu soit `GAME_IS_FINISH_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
// Autres includes...
#include "Game.h"

// Variables globales...

void setup() {
  // ...
}

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

Vous pouvez téléverser le programme vers votre console, le résultat est le suivant :

![Fin de partie](./../../img/E05/fin_de_partie_v1.gif)

## Chronomètre

Avant de développer le timer, ajoutons des constantes dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
// Constantes timer
const uint8_t DAYS_NUMBER = 0;
const uint8_t HOURS_NUMBER = 1;
const uint8_t MINUTES_NUMBER = 2;
const uint8_t SECONDS_NUMBER = 3;
const uint8_t MILLISECONDS_NUMBER = 4;
```

Nous allons créer une structure pour le chronomètre, nous aurons besoin :
* d'un entier pour compter le temps passé ;
* d'un autre entier pour compter le temps passé depuis la dernière pause ;
* d'un tableau d'entiers pour avoir le temps total passé, découpé en jours, heures, minutes, secondes et millisecondes ;
* d'un booléen pour déterminer si le chronomètre est actif ou non.

Voici le contenu du fichier `Timer.h` :

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

Dans `Timer.h` ajoutons le prototype de la fonction `createTimer` :

<div class="filename" >Timer.h</div>
```
void createTimer(Timer &aTimer);
```

Dans `Timer.cpp` ajoutons le contenu suivant, dont la fonction `createTimer` :

<div class="filename" >Timer.cpp</div>
```
#include "Timer.h"

// Créer le timer
void createTimer(Timer &aTimer) {
  aTimer.activateTimer = false;
}
```

Dans le programme principal, incluons le fichier `Timer.h` et créons le chronomètre puis ajoutons l'appel à la fonction `createTimer` dans la fonction `setup` :

<div class="filename" >GBPlatformer01.ino</div>
```
// Autres includes...
#include "Timer.h"

// Autres variables globales...
Timer myTimer;

void setup() {
  // ... 

  createTimer(myTimer);
}

void loop() {
  // ...
}
```

Dans `Timer.h`, ajoutons la fonction `resetTimer` qui permettra de réinitialiser le chronomètre :

<div class="filename" >Timer.h</div>
```
void resetTimer(Timer &aTimer);
```

Puis comme d'habitude, codons dans `Timer.cpp` la fonction que nous venons de déclarer (`resetTimer`) :

<div class="filename" >Timer.cpp</div>
```
// Réinitialiser le timer
void resetTimer(Timer &aTimer) {
  aTimer.timeInMilliseconds = 0;
  aTimer.tempTime = 0;
  aTimer.activateTimer = true;
}
```

Dans le programme principal, quand le jeu est à l'état `LAUNCH_PLAY_STATE`, il faut réinitialiser le chronomètre :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
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
      resetTimer(myTimer); // ............. on réinitialise le chronomètre
      initObjects(setOfObjects); // ....... on réinitialise les objets
      initPlatforms(setOfPlatforms); // ... on réinitialise les plateformes
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

Dans `Timer.h`, ajoutons le prototype de la fonction `pauseForTimer` :

<div class="filename" >Timer.h</div>
```
void pauseForTimer(Timer &aTimer);
```

Dans `Timer.cpp`, développons la fonction `pauseForTimer` :

<div class="filename" >Timer.cpp</div>
```
// Mettre en pause le timer
void pauseForTimer(Timer &aTimer) {
  if(aTimer.tempTime != 0) {
    aTimer.timeInMilliseconds += aTimer.tempTime;
    aTimer.tempTime = 0;
  }
}
```

Dans `Timer.h`, ajoutons le prototype de la fonction `computeTime` qui découpe un temps en millisecondes en jours, heures, minutes, secondes et millisecondes :

<div class="filename" >Timer.h</div>
```
void computeTime(Timer &aTimer);
```

Dans `Timer.cpp`, implémentons la fonction `computeTime` :

<div class="filename" >Timer.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
// Décomposer le temps écoulé en jours, heures, minutes et secondes
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

  // Calculer les jours
  if(rest >= DAYS_IN_FRAMES) {
    nbDays = (uint16_t)(rest / DAYS_IN_FRAMES);
    rest = (rest - (nbDays * DAYS_IN_FRAMES));
  }
  // Calculer les heures
  if(rest >= HOURS_IN_FRAMES) {
    nbHours = (uint16_t)(rest / HOURS_IN_FRAMES);
    rest = (rest - (nbHours * HOURS_IN_FRAMES));
  }
  // Calculer les minutes
  if(rest >= MINUTES_IN_FRAMES) {
    nbMinutes = (uint16_t)(rest / MINUTES_IN_FRAMES);
    rest = (rest - (nbMinutes * MINUTES_IN_FRAMES));
  }
  // Calculer les secondes
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

Cette fonction sera appelée plus tard par une autre fonction du chronomètre.

Dans `Timer.h`, ajoutons la fonction `incrementTimer` qui incrémentera le compteur à chaque appel  :

<div class="filename" >Timer.h</div>
```
void incrementTime(Timer &aTimer);
```

Dans `Timer.cpp`, définissons la fonction `incrementTime` :

<div class="filename" >Timer.cpp</div>
```
void incrementTime(Timer &aTimer) {
  aTimer.tempTime += gb.getTimePerFrame();
}
```

Comme la fonction `computeTime`, la fonction `incrementTime` sera appelée par une autre fonction du chronomètre que nous allons voir maintenant.

Dans `Timer.h`, créons la fonction `runTimer` que nous appelerons à chaque passage dans la fonction `loop` (du programme principal), et lorsque nous serons dans l'état `PLAY_STATE` :

<div class="filename" >Timer.h</div>
```
void runTimer(Timer &aTimer);
```

Dans `Timer.cpp`, développons la fonction `runTimer` :

<div class="filename" >Timer.cpp</div>
```
// Gérer le chronomètre
void runTimer(Timer &aTimer) {
  if(aTimer.activateTimer) {
    incrementTime(aTimer);
    computeTime(aTimer);
  } else {
    pauseForTimer(aTimer);
  }
}
```

Dans le programme principal, dans l'état `PLAY_STATE`, après la gestion des commandes, ajoutons l'appel de la fonction `runTimer` comme ceci :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
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

Nous allons ajouter un nouvel état.

Commencons par créer ce nouvel état dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint8_t SAVE_HIGH_SCORE_STATE = 5;
```

Retournons dans le programme principal, dans l'état `PLAY_STATE`, modifiez la redirection qui est réalisée lorsque la partie est finie vers le nouvel état  :

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

Toujours dans le programme principal, ajoutons l'état `SAVE_HIGH_SCORE_STATE`, pour l'instant, dans cet état :
* nous arrêterons le chronomètre ;
* et nous redirigerons vers l'état `GAME_IS_FINISH_STATE`.

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

Le chronomètre est à ce stade fonctionnel. Pour le constater, nous allons l'afficher sur l'écran de jeu.

Commençons par ajouter le prototype de la fonction `paintTimer` dans `Display.h` :

<div class="filename" >Display.h</div>
```
void paintTimer(const uint16_t * aTime);
```

Cette fonction prend en paramètre le tableau d'entiers qui représente le temps passé découpé en jours, heures, minutes, secondes et millisecondes.

Dans `Display.cpp`, implémentons la fonction `paintTimer` :

<div class="filename" >Display.cpp</div>
```
// Afficher le chronomètre
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

Dans `Display.h`, modifions le prototype de la fonction `paint` comme ceci :

<div class="filename" >Display.h</div>
```
void paint(
  Character &aCharacter, 
  Platform * aSetOfPlatforms, 
  Object * aSetOfObjects, 
  const uint16_t * aTime
);
```

Dans `Display.cpp`, mettons à jour la fonction `paint` :

<div class="filename" >Display.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void paint(Character &aCharacter, Platform * aSetOfPlatforms, Object * aSetOfObjects, const uint16_t * aTime) {
  /* affichage du jeu... */
  paintTimer(aTime);
}
```

Dans le programme principal, adaptons l'appel de la fonction `paint` (dans l'état `PLAY_STATE`) :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
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

Avant de passer au "game over", nous allons afficher le temps sur l'écran de fin de jeu.

Pour cela, modifions le prototype de la fonction `paintEndOfGame` :

<div class="filename" >Display.h</div>
```
void paintEndOfGame(const uint16_t * aTime);
```

Dans `Display.cpp`, adaptons la fonction `paintEndOfGame` :

<div class="filename" >Display.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void paintEndOfGame(const uint16_t * aTime) {
  gb.display.setFontSize(2);
  gb.display.setColor(BROWN);
  gb.display.println("");
  gb.display.println("");
  gb.display.println(TRY_AGAIN_EN);
  paintTimer(aTime); // ............. ajouter l'affichage du chronomètre ici
  gb.display.setFontSize(1);
  gb.display.setColor(WHITE);
  
  gb.display.println("");
  gb.display.println("");
  gb.display.println(A_4_TRY_AGAIN_EN);
  gb.display.println(B_4_MENU_EN);
}
```

Dans le programme principal, dans l'état `GAME_IS_FINISH_STATE`, modifions l'appel de la fonction `paintEndOfGame` :

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

Vous pouvez téléverser le programme vers votre console, vous constaterez alors que le chronomètre "redémarre à zéro" au-delà d'une minute. C'est normal, comme nous affichons que les secondes et les millisecondes, la précision est perdu. C'est un détail que nous règlerons dans la suite de ce workshop. En effet, au-delà de 20 000 millisecondes (c'est-à-dire 20 secondes) nous allons déclencher le "game over".

Attention dans la capture ci-dessous le chronomètre ne démarre pas à zéro ! C'est parce que je lance manuellement la capture et donc le temps c'est écoulé entre le début réel de la partie puis le début de la capture.

![Ajout du chronomètre](./../../img/E05/chrono_v1.gif)

## Game over

Pour la gestion du game over, nous allons commencer par ajouter le temps maximum de la partie en millisecondes et l'état pour le game over, dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint32_t MAX_TIME_OF_GAME = 20000;

const uint8_t GAME_OVER_STATE = 6;
```

Dans `Game.h`, incluons le fichier `Timer.h`, puis ajoutons la fonction `isGameOver` (dans `Game.h`) :

<div class="filename" >Game.h</div>
```
bool isGameOver(Timer &aTimer);
```

Dans `Game.cpp`, implémentons la fonction `isGameOver` :

<div class="filename" >Game.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
bool isGameOver(Timer &aTimer) {
  return ( (aTimer.timeInMilliseconds + aTimer.tempTime) >= MAX_TIME_OF_GAME );
}
```

Avant de modifier le programme principal, nous allons créer un écran game over.

Dans `Lang.h`, ajoutons une constante pour l'écran game over :

<div class="filename" >Lang.h</div>
```
static const char * GAME_OVER_EN = "GAME OVER";
```

Dans `Display.h`, déclarons une fonction, qui affiche un écran game over, nous nommerons cette fonction `paintGameOverScreen` :

<div class="filename" >Display.h</div>
```
void paintGameOverScreen();
```

Dans `Display.cpp`, définissons la fonction `paintGameOverScreen` :

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

Modifions le programme principal.

Dans un premier temps, ajouter l'état `GAME_OVER_STATE`, dans cet état nous utiliserons la gestion des commandes que nous avons créé préalablement puis nous afficherons l'écran "game over", soit :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Dans un second temps, ajoutons la route vers le game over, ceci dans l'état `PLAY_STATE` :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Vous pouvez téléverser le programme sur votre console, lancez une partie, et laissez le temps passer : au-delà de 20 secondes l'écran game over s'affiche.

J'ai commencé la capture ci-dessous après 16 secondes, mon objectif ici est de montrer l'affichage de l'écran "game over" :

![Game over](./../../img/E05/game_over_v1.gif)

## Navigation

Nous allons ajouter un écran de pause, au lieu de retourner sans condition au menu.

Pour ce faire déclarons deux nouveaux états dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint8_t PAUSE_STATE = 7;
const uint8_t GO_BACK_GAME_STATE = 8;
```

Dans `Lang.h`, ajoutons deux constantes, que nous utiliserons pour l'écran :

<div class="filename" >Lang.h</div>
```
static const char * A_4_GO_TO_GAME_EN = "(A) Go to game";
static const char * DO_YOU_WANT_EN = "Do you want ?";
```

Dans `Display.h`, ajoutons le prototype de la fonction `paintPause` :

<div class="filename" >Display.h</div>
```
void paintPause();
```

Dans `Display.cpp`, implémentons la fonction `paintPause` :

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

La gestion des commandes pour l'état `PAUSE_STATE` doit être le suivant :
* si le bouton A est pressé alors retourner au jeu (via l'état `GO_BACK_GAME_STATE`) ;
* si le bouton B est pressé alors aller au menu ;
* sinon rester en pause.

Dans `Commands.h`, déclarons la fonction `manageCommandsForPause` :

<div class="filename" >Commands.h</div>
```
const uint8_t manageCommandsForPause();
```

Dans `Commands.cpp`, définissons la fonction `manageCommandsForPause` :

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

Toujours dans `Commands.cpp`, modifions la fonction `manageCommands` pour rediriger vers l'écran de pause lorsqu'on appuye sur le bouton menu :

<div class="filename" >Commands.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
const uint8_t manageCommands(Character &aCharacter) {

  // ...
  
  return (gb.buttons.pressed(BUTTON_MENU) ? PAUSE_STATE : PLAY_STATE);
}
```

Apportons des modifications au programme principal.

Dans l'état `PLAY_STATE`, il faut arrêter le chronomètre si l'utilisateur appuye sur le bouton menu :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Ajoutons l'état `PAUSE_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Ajoutons l'état `GO_BACK_GAME_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Lorsque l'état est indéfini, nous allons ajouter une redirection vers l'état d'accueil :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Vous pouvez téléverser le programme vers votre console, le dernier ajout c'est l'écran de pause :

![Gestion pause](./../../img/E05/pause_v1.gif)

## Conclusion

Vous voici arrivé à la fin de cette cinquième étape où nous avons ajouté la gestion de la partie.

<!--Si vous avez terminé ou si vous rencontrez des problèmes vous pouvez télécharger la solution <a href="#" class="external-link" >ici</a>.-->

Dans la prochaine et dernière étape, c'est-à-dire la sixième, nous aborderons la gestion d'un tableau de meilleurs scores.

N'hésitez pas à me faire un retour : les améliorations que vous apporteriez (un regard extérieur est toujours bienvenu), les choses que vous n'avez pas compris, les fautes, etc. Pour laisser un retour, merci d'ajouter un commentaire à la création suivante :

{% include react-btn.html %}
