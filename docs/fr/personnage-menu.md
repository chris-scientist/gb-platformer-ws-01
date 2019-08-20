---
layout: step
title: "Le personnage : création, affichage et gestion d'un menu"
permalink: /fr/personnage-menu/
next: /fr/deplacements/
link-other-lang: /en/character-menu/
lang: fr
---

# Etape 1 - Le personnage : création, affichage et gestion d'un menu


## Introduction

Dans cette étape, nous allons créer un personnage, l'afficher et nous verrons également comment créer un menu.


## Programme basique

Avant de créer le personnage, il faut écrire le programme de base comme nous l'avons vu dans l'<a href="https://gamebuino.com/fr/academy/workshop/tap-tap-how-fast-can-you-tap/inputs-update-draw" class="external-link" >étape 2 de TapTap, un jeu ou il faut aller très vite</a> !

Pour rappel, il nous faut :
* inclure le fichier `Gamebuino-Meta.h` en début de code ;
* penser à initialiser la Gamebuino META dans la fonction `setup` ;
* ajouter une boucle d'attente dans la fonction `loop` ;
* effacer l'écran toujours dans la fonction `loop`.

Pour tester que tout fonctionne bien, nous afficherons un message avec `gb.display.println("Votre message");`.

Voici le code de ce programme de base :

<div class="filename" >GBPlatformer01.ino</div>
```
#include <Gamebuino-Meta.h>
  
void setup() {
  // initialisation de la Gamebuino META
  gb.begin();
}

void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // Commencer par effacer l'écran
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

Téléversez votre programme pour constater l'affichage du message, voici le résultat :

![Programme de base](./../../img/E01/home_screen_v1.png)

<!--Vous pouvez télécharger <a href="#" class="external-link" >ici le programme de base</a>.-->

## Le personnage

### Créer la structure

Bien, maintenant que nous avons un programme qui fonctionne, regardons de quoi nous aurons besoin pour gérer le personnage.

Bien que nous ne ferons qu'afficher ce personnage sans gérer ses mouvements dans cette première étape, nous allons avoir besoin :
* de sa position horizontale que nous allons gérer avec un entier `x` ;
* de sa position verticale également gérée avec un entier `y` ;
* de la direction de déplacement du personnage gérée avec un booléen, `true` pour aller à gauche et `false` pour aller à droite.

Pour représenter le personnage nous allons utiliser une structure (struct), comme nous l'avons vu dans le tutoriel : <a href="https://gamebuino.com/fr/creations/structurer-les-objets-de-votre-programme" class="external-link" >Structurer les objets de votre programme</a>.

Nous allons appeler cette structure `Character` et nous allons la créer dans le fichier d'entête (ou header) `Character.h`.

> Pourquoi mets-tu une majuscule au nom de la structure ? Il s'agit d'une convention de développeur. Si votre structure comporte plusieurs mots, par exemple une structure qui stocke un meilleur score : nous la nommerons `HighScore` (chaque nouveau mot commençant ainsi par une majuscule).

> Un fichier d'entête est l'endroit idéal pour définir le prototype des fonctions, les types personnalisés et les structures. 

Si vous ne voyez pas comment implémenter la structure du personnage ou si vous avez réussi et que vous souhaitez comparer votre code, voici la structure que nous utiliserons au cours de ce workshop :

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


### Initialiser le personnage

Maintenant que nous avons la structure du personnage, il faut écrire une fonction qui initialise les données du personnage. Nous allons faire en sorte que notre personnage s'affiche en bas et au centre de l'écran, et qu'il se dirige vers la droite, pour cela nous devons initialiser :
* la position x du personnage à 40, soit la moitié de l'écran ;
* la position y du personnage à hauteur de l'écran - delta ;
* la direction (à ne pas oublier).

Remarque : le delta pour la position y du personnage, c'est le nombre de pixels en dessous du centre du personnage.

Nous allons également définir quelques constantes avant de voir la solution.

Créons un fichier `Constants.h`, et ajoutons-y les lignes suivantes :

<div class="filename" >Constants.h</div>
```
#ifndef PLATFORMER_CONSTANTS
#define PLATFORMER_CONSTANTS

#include <Gamebuino-Meta.h>

// Pour un personnage de 8 pixels de haut sur 6 pixels de large
const uint8_t WIDTH_HERO = 6;
const uint8_t HEIGHT_HERO = 8;
const uint8_t UNDER_CENTER_X_HERO = 3; // à droite du héro
const uint8_t UNDER_CENTER_Y_HERO = 4; // en dessous du héro
const uint8_t OVER_CENTER_X_HERO = 3; // à gauche du héro
const uint8_t OVER_CENTER_Y_HERO = 4; // au dessus du héro

#endif
```

Comme vous pouvez le voir notre personnage fera 6 pixels de large sur 8 pixels de haut. Et pour le delta dont nous avons précédement parlé, il s'agit de la constante `UNDER_CENTER_Y_HERO`.

Voici la signature de la fonction servant à initialiser le personnage, fonction que nous allons déclarer dans `Character.h` :

<div class="filename" >Character.h</div>
```
void initCharacter(Character &aCharacter);
```

> Vous remarquerez que nous utilisons un passage par référence grâce au symbole `&`. Sans ce passage par référence nous aurions un problème : en effet un passage par copie que nous aurions obtenu avec `void initCharacter(Character aCharacter);` n'effectuerait l'initialisation qu'à l'intérieur de cette fonction. Une fois de retour dans le programme principal ce serait comme si rien n'avait été fait. De plus, un passage par copie est plus gourmand en mémoire.

Le code de cette fonction, à placer dans `Character.cpp`, est le suivant :

<div class="filename" >Character.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
#include "Character.h"
#include "Constants.h"

void initCharacter(Character &aCharacter) {
  // on force la position intiale du héro au milieu de l'écran et plaqué au sol
  aCharacter.x = 40;
  aCharacter.y = gb.display.height() - UNDER_CENTER_Y_HERO;

  // on force la direction du personnage vers la droite
  aCharacter.toTheLeft = false;
}
```

### Programme principal

Modifions le programme principal pour créer un personnage, l'initialiser et afficher ces informations. Mais avant nous allons en profiter pour ajouter un système d'états à notre jeu et ainsi moduler l'affichage en fonction de l'état.

Ajoutons d'abord les constantes suivantes dans `Constants.h`, qui correspondent aux états dont nous aurons besoin pour commencer :

<div class="filename" >Constants.h</div>
```
const uint8_t HOME_STATE = 1;
const uint8_t LAUNCH_PLAY_STATE = 2;
const uint8_t PLAY_STATE = 3;
```

N'oublions pas d'inclure les constantes dans le programme principal puis ajoutons y une variable pour gérer l'état. Elle doit être déclarée en dehors des fonctions `setup` et `loop` :

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

Dans la fonction `setup`, après l'initialisation de la console, initialisons l'état du jeu à la valeur `HOME_STATE`, comme ceci :

<div class="filename" >GBPlatformer01.ino</div>
```
void setup() {
  // initialisation de la Gamebuino META
  gb.begin();

  stateOfGame = HOME_STATE;
}
```

Il faut instancier la structure pour notre personnage, comme pour la variable qui gère l'état, en dehors des fonctions `setup` et `loop` :

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

Enfin, modifions le programme dans la fonction `loop` pour :
* passer à l'état `LAUNCH_PLAY_STATE` lorsque nous sommes dans l'état `HOME_STATE` (nous ajouterons le menu à cet état à la fin de cette étape) ;
* lorsque nous sommes dans l'état `LAUNCH_PLAY_STATE` nous devons initialiser les informations du personnage grâce à la fonction `initCharacter` et passer à l'état `PLAY_STATE` ;
* lorsque nous sommes dans l'état `PLAY_STATE`, il faut afficher les informations du personnage ;
* par défaut, si nous ne sommes dans aucun de ces états (ce qui ne devrait pas ce produire) alors nous afficherons le texte de départ.

Nous utiliserons la structure conditionnelle `switch` pour faire ce travail.

Pour afficher les informations du personnage nous allons utiliser `gb.display.printf`, cette fonction s'utilise comme ceci :

<div class="filename" >Exemple</div>
```
gb.display.printf("x = %d", hero.x);
```

Voici à quoi ressemble notre gestion des états :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Vous pouvez téléverser votre programme vers la console, qui devrait afficher quelque chose comme :

<div class="filename" >Affichage console</div>
```
x,y = 40,60
to the left = 0
```

> Notons qu'un booléen affiche 0 s'il vaut `false` et 1 s'il vaut `true`.


### Afficher le personnage

Avant de passer à l'affichage du personnage, définissons deux nouvelles constantes :
* une constante qui définit la couleur quand le personnage va à gauche (valant par exemple, `LIGHTBLUE`) ;
* une autre constante qui définit la couleur quand le personnage va à droite (valant par exemple, `BLUE`).

Pour ce faire, ajoutons dans le fichier `Constants.h` les constantes suivantes :

<div class="filename" >Constants.h</div>
```
const Color HERO_L_COLOR = LIGHTBLUE;
const Color HERO_R_COLOR = BLUE;
```

Nous allons gérer l'affichage du jeu dans les fichiers `Display.h` et `Display.cpp`.

Voici le contenu du fichier `Display.h` :

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

Quelques précisions sur les fonctions qui sont déclarées dans `Display.h` :
* `paint` sera la seule fonction utilisée dans le programme principal et c'est elle qui se chargera de faire tout l'affichage ;
* `paintHero` réalise l'affichage du personnage via `paintBox` ;
* `paintBox` dessine un rectangle en fonction des paramètres qui lui sont passés.

La fonction `paint` ne fera pour l'instant qu'un appel à `paintHero`.

Toutes ces fonctions doivent être définies dans `Display.cpp`, pensez à inclure `Display.h` (dans `Display.cpp`).

Pour changer la couleur il faut utiliser <a href="https://gamebuino.com/fr/academy/reference/graphics-setcolor" class="external-link" >gb.display.setColor</a> et pour dessiner un rectangle plein il faut utiliser <a href="https://gamebuino.com/fr/academy/reference/graphics-fillrect" class="external-link" >gb.display.fillRect</a>.

Voici le code de la fonction `paintBox` :

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

Pour l'affichage du personnage nous allons utiliser l'opérateur ternaire, ceci afin de choisir la couleur.

Voici un code classique :

<div class="filename" >Exemple</div>
```
char signe = '-';
if(nb > 0) {
  signe = '+';
}
```

Ce dernier peut-être remplacé par l'opérateur ternaire que voici :

<div class="filename" >Exemple</div>
```
char signe = (nb > 0) ? '+' : '-';
```

Voici le code de la fonction `paintHero` :

<div class="filename" >Display.cpp <span>/!\ Scroll horizontal /!\</span></div>
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

Et voici le code de la fonction `paint` :

<div class="filename" >Display.cpp</div>
```
void paint(Character &aCharacter) {
  paintHero(aCharacter);
}
```

Retournons dans le programme principale. N'oubliez pas d'inclure `Display.h` ! Remplaçons l'affichage des informations du personnage par l'affichage du jeu, soit le code suivant :

<div class="filename" >GBPlatformer01.ino</div>
```
// Autres includes...
#include "Display.h"

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

Si vous téléversez votre programme, vous devriez voir un joli rectangle bleu en bas de l'écran : c'est votre personnage ! Voici une capture d'écran du jeu :

![Affichage du personnage](./../../img/E01/character_v1.png)

## Ajouter un menu

Nous allons ajouter un menu à notre jeu, nous le compléterons au fil de l'avancée du workshop.

Créons un fichier nommé `Lang.h`, nous nous limiterons à l'anglais. Nous verrons dans les bonus comment gérer plusieurs langues. Voici le contenu du fichier :

<div class="filename" >Lang.h</div>
```
#ifndef PLATFORMER_LANG
#define PLATFORMER_LANG

static const char * PLAY_EN = "Play";

#endif
```

Nous allons ajouter la gestion du menu dans `Display.h` et dans `Display.cpp`. D'abord, il faut inclure `Lang.h` dans `Display.h`. Ensuite ajoutons le prototype de la fonction suivante (toujours dans `Display.h`) :

<div class="filename" >Display.h</div>
```
const uint8_t paintMenu();
```

Pour le menu nous allons utiliser la fonction <a href="https://gamebuino.com/fr/academy/reference/gb-gui-menu" class="external-link" >gb.gui.menu</a>. Cette fonction prend en paramètre une chaîne de caractères pour le nom du menu, et un tableau de chaîne de caractères pour les items. Pour l'instant le menu ne comporte qu'un seul item.

Voici le code de la fonction `paintMenu` à placé dans `Display.cpp` :

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

Dans le programme principal, remplacez `stateOfGame = LAUNCH_PLAY_STATE;` par `stateOfGame = paintMenu();`, rappelez vous c'est dans l'état `HOME_STATE`, voici le code :

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

En complément de ce menu, nous allons ajouter une commande permettant de revenir au menu lorsque le jeu est en cours. Pour cela, créons le fichier `Commands.h` avec le code suivant :

<div class="filename" >Commands.h</div>
```
#ifndef PLATFORMER_COMMANDS
#define PLATFORMER_COMMANDS

#include <Gamebuino-Meta.h>

#include "Constants.h"

const uint8_t manageCommands();

#endif
```

Le code de la fonction `manageCommands` est relativement simple, si le bouton menu est pressé alors on retourne au menu, sinon on reste dans le jeu, soit le code suivant à écrire dans `Commands.cpp` :

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

Comme vous le voyez nous utilisons à nouveau un opérateur ternaire.

Dans le programme principal, incluons `Commands.h`, dans l'état `PLAY_STATE` et avant l'affichage du jeu, ajoutons l'appel à la fonction `manangeCommands` soit le code suivant :

<div class="filename" >GBPlatformer01.ino</div>
```
// Autres includes...
#include "Commands.h"

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
      stateOfGame = manageCommands();
      paint(hero);
      break;
    default:
      gb.display.println("Votre message");
  }
}
```

Vous pouvez téléverser votre programme sur la console, vous aurez au démarrage du jeu un menu avec l'item "Play", qui permet de lancer le jeu. Pendant le jeu vous pouvez revenir au menu grâce au bouton menu.


## Conclusion

Vous voici arrivé à la fin de cette première étape : nous avons créé et affiché un personnage et nous avons géré les commandes basiques pour naviguer dans notre jeu, en partie grâce à un menu.

<!--Si vous avez terminé ou si vous rencontrez des problèmes vous pouvez télécharger la solution <a href="#" class="external-link" >ici</a>.-->

Dans la prochaine étape, c'est-à-dire la deuxième, nous aborderons les déplacements du personnage.

N'hésitez pas à me faire un retour : les améliorations que vous apporteriez (un regard extérieur est toujours bienvenu), les choses que vous n'avez pas compris, les fautes, etc. Pour laisser un retour, merci d'ajouter un commentaire à la création suivante :

{% include react-btn.html %}
