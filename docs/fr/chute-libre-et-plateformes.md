---
layout: step
title: "Chute libre et plateformes"
permalink: /fr/chute-libre-et-plateformes/
next: /fr/monde/
link-other-lang: /en/introduction/
lang: fr
---

# Etape 3 - chute libre et plateformes

## Introduction

Dans cette troisième étape, nous allons implémenter :
* la chute libre ;
* et la gestion des plateformes.

*Je vous invite à télécharger [le code](https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v2.0.zip) qui est le résultat de la deuxième étape afin de partir sur des bases communes.*

## Chute libre

Implémentons une première version de la chute libre. En effet, nous l'adapterons après avoir ajouté les plateformes.

Dans `PhysicsEngine.h`, ajoutons le prototype de la fonction `gravity` :

<div class="filename" >PhysicsEngine.h</div>
```
void gravity(Character &aCharacter);
```

Implémentons la méthode `gravity` :

<div class="filename" >PhysicsEngine.cpp</div>
```
// Chute libre
void gravity(Character &aCharacter) {

  const uint8_t platformId = isOnOnePlatform(aCharacter);
  if( platformId == NO_ID ) {
    // Chute libre
    aCharacter.oldY = aCharacter.y;
    aCharacter.vy += GRAVITY;
    aCharacter.state = FREE_FALL_STATE;
    aCharacter.y += aCharacter.vy;
  } else {
    // en contact avec une structure
    rectifyPositionY(aCharacter);
    aCharacter.vy = 0;
    aCharacter.state = ON_THE_PLATFORM_STATE;
  }
  
}
```

Dans le programme principal, dans l'état `PLAY_STATE` remplaçons `gb.display.println("GRAVITY");` par `gravity(hero);`.

Téléversez le programme vers votre console et constatez la chute libre (en sautant sur le bord de l'écran).

Les bases de la chute libre sont maintenant implémentées, passons à la gestion des plateformes.

## Les plateformes

### Création des plateformes

Avant de créer la structure des plateformes, ajoutons des constantes dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
// Couleurs des box
// ...
const Color GROUND_COLOR = BROWN;
const Color PLATFORM_T_COLOR = LIGHTGREEN;
const Color PLATFORM_NOT_T_COLOR = GREEN;

// Nombre de plateformes
const uint8_t NB_OF_PLATFORMS = 4;

// Identifiants des plateformes
// ...
const uint8_t ID_PLATFORM_1 = 2;
const uint8_t ID_PLATFORM_2 = 3;
const uint8_t ID_PLATFORM_3 = 4;

// Type de plateforme
const uint8_t NO_PLATFORM_TYPE = 0;
const uint8_t GROUND_TYPE = 1;
const uint8_t PLATFORM_TYPE = 2;

// Pour une plateforme de 4 pixels de haut sur 8 pixels de large
const uint8_t HEIGHT_PLATFORM = 4;
const uint8_t WIDTH_PLATFORM = 8;
const uint8_t OVER_CENTER_X_PLATFORM = 4;
const uint8_t OVER_CENTER_Y_PLATFORM = 2;

// Pour le sol de 4 pixels de haut sur 8 pixels de large
const uint8_t HEIGHT_GROUND = 4;
const uint8_t WIDTH_GROUND = 8;
const uint8_t OVER_CENTER_X_GROUND = 4;
const uint8_t OVER_CENTER_Y_GROUND = 2;
```

Dans le fichier `Platform.h`, ajoutons entre autre la structure gérant les plateformes :

<div class="filename" >Platform.h</div>
```
#ifndef PLATFORMER_PLATFORM
#define PLATFORMER_PLATFORM

#include "Constants.h"

struct Platform {
  int8_t x; // ................ position x du début de la plateforme (centre de la première plateforme)
  int8_t y; // ................ position y de la plateforme (centre de la plateforme)
  uint8_t lengthPlatform; // ... longueur de la plateforme en bloc (doit être au minimum égale à 2), taille des blocs dans le fichier de constantes
  uint8_t type; // ............. type de plateforme : voir constantes
  uint8_t id; // ............... identifiant (unique) de la plateforme
  bool isGoThrough; // ......... true pour indiquer que le joueur peut passer à travers la plateforme, sinon false
};

#endif
```

Toujours dans `Platform.h` ajoutons le prototype de la fonction `createPlatform` qui permet de créer une plateforme :

<div class="filename" >Platform.h</div>
```
Platform createPlatform(int8_t aX, int8_t aY, uint8_t aLength, uint8_t aType, uint8_t aId, bool goThrough);
```

Dans le fichier `Platform.cpp`, ajoutons la définition de la fonction `createPlatform` :

<div class="filename" >Platform.cpp</div>
```
Platform createPlatform(int8_t aX, int8_t aY, uint8_t aLength, uint8_t aType, uint8_t aId, bool goThrough) {
  // On force la taille de la plateforme à 2 blocs
  if(aLength < 2) {
    aLength = 2;
  }

  // On force le type de plateforme à une plateforme flottante
  if(aType < 1 || aType > 2) {
    aType = PLATFORM_TYPE;
  }

  // Création de la plateforme
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

Dans `Platform.h`, ajoutons le prototype de la fonction `initPlatforms` qui initialisera les plateformes :

<div class="filename" >Platform.h</div>
```
void initPlatforms(Platform * aSet);
```

Comme vous l'avez vu dans les constantes, nous allons créer quatre plateformes, dont le sol.

Dans `Platform.cpp`, définissons la plateforme `initPlatforms` :

<div class="filename" >Platform.cpp</div>
```
void initPlatforms(Platform * aSet) {
  aSet[0] = createPlatform(4, 62, 10, GROUND_TYPE, ID_GROUND, false);
  aSet[1] = createPlatform(4, 50, 2, PLATFORM_TYPE, ID_PLATFORM_1, true);
  aSet[2] = createPlatform(35, 40, 2, PLATFORM_TYPE, ID_PLATFORM_2, false);
  aSet[3] = createPlatform(68, 30, 2, PLATFORM_TYPE, ID_PLATFORM_3, false);
}
```

Dans le programme principal, en dehors des fonctions `setup` et `loop`, ajoutons un tableau de plateformes :

<div class="filename" >GBPlatformer01.ino</div>
```
Platform setOfPlatforms[NB_OF_PLATFORMS];
```

N'oubliez pas d'inclure `Platform.h` dans votre programme principal.

Toujours dans le programme principal, en particulier dans l'état `LAUNCH_PLAY_STATE`, initialisons les plateformes :

<div class="filename" >GBPlatformer01.ino</div>
```
initPlatforms(setOfPlatforms); // ... on réinitialise les plateformes
```

### Affichage des plateformes

Commençons par inclure `Platform.h` dans `Display.h`.

Dans `Display.h`, ajoutons le prototype de la fonction `paintGround` :

<div class="filename" >Display.h</div>
```
void paintGround(const int8_t aX, const int8_t aY);
```

Dans `Display.cpp`, définissons la fonction `paintGround` :

<div class="filename" >Display.cpp</div>
```
// Réalise l'affichage du sol
void paintGround(const int8_t aX, const int8_t aY) {
  paintBox(aX, aY, WIDTH_GROUND, HEIGHT_GROUND, GROUND_COLOR);
}
```

Dans `Display.h`, ajoutons le prototype de la fonction `paintPlatform` qui a pour but de dessiner une plateforme flottante :

<div class="filename" >Display.h</div>
```
void paintPlatform(const int8_t aX, const int8_t aY, const bool isGoThrough);
```

Dans `Display.cpp`, définissons la fonction `paintPlatform` :

<div class="filename" >Display.cpp</div>
```
// Réalise l'affichage d'une plateforme flottante
void paintPlatform(const int8_t aX, const int8_t aY, const bool isGoThrough) {
  paintBox(aX, aY, WIDTH_PLATFORM, HEIGHT_PLATFORM, (isGoThrough ? PLATFORM_T_COLOR : PLATFORM_NOT_T_COLOR));
}
```

Dans `Display.h`, ajoutons le prototype de la fonction `paintPlatform` qui oriente vers la fonction de dessin dédiée :

<div class="filename" >Display.h</div>
```
void paintPlatform(const Platform &aPlatform);
```

Dans `Display.cpp`, définissons la fonction `paintPlatform` :

<div class="filename" >Display.cpp</div>
```
// Oriente vers la fonction de dessin dédiée
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

Dans `Display.h`, ajoutons le prototype de la fonction `paintPlatforms` qui dessine toutes les plateformes :

<div class="filename" >Display.h</div>
```
void paintPlatforms(Platform * aSet);
```

Dans `Display.cpp`, définissons la fonction `paintPlatforms` :

<div class="filename" >Display.cpp</div>
```
// Dessine toutes les plateformes
void paintPlatforms(Platform * aSet) {
  for(int i = 0 ; i < NB_OF_PLATFORMS ; i++) {
    paintPlatform(aSet[i]);
  }
}
```

Dans `Display.h`, modifions le prototype de la fonction `paint` tel que :

<div class="filename" >Display.h</div>
```
void paint(Character &aCharacter, Platform * aSetOfPlatforms);
```

Dans `Display.cpp`, modifions la définition de la fonction `paint` :

<div class="filename" >Display.cpp</div>
```
void paint(Character &aCharacter, Platform * aSetOfPlatforms) {
  paintPlatforms(aSetOfPlatforms);
  // dessiner le personnage...
}
```

Dans le programme principal, modifions l'appel de la méthode `paint` tel que :

<div class="filename" >GBPlatformer01.ino</div>
```
paint(hero, setOfPlatforms);
```

Avant de rendre nos plateformes "physiques", nous allons modifier la position de notre personnage pour qu'il s'affiche au dessus du sol.

Dans `Character.cpp`, modifions la fonction `initCharacter`, en particulier la position `y` tel que :

<div class="filename" >Character.cpp</div>
```
aCharacter.y = gb.display.height() - (UNDER_CENTER_Y_HERO + HEIGHT_GROUND);
```

Toujours dans `Character.cpp`, modifions la fonction `rectifyPositionY` tel que :

<div class="filename" >Character.cpp</div>
```
aCharacter.y = (gb.display.height() - (UNDER_CENTER_Y_HERO + HEIGHT_GROUND));
```

Enfin, dans `PhysicsEngine.cpp`, modifions la fonction `isOnOnePlatform` tel que :

<div class="filename" >PhysicsEngine.cpp</div>
```
return ( (aCharacter.y + aCharacter.vy) >= (gb.display.height() - (UNDER_CENTER_Y_HERO + HEIGHT_GROUND)) ) ? ID_GROUND : NO_ID ;
```

Vous pouvez téléverser le programme vers votre console. Vous verrez que votre personnage est dessiné sur le sol, par contre vous pouvez sauter sur les plateformes, mais aucune réaction, vous les traversez. Nous allons y remédier tout de suite.

### Collision avec les plateformes

Pour commencer, nous avons besoin d'une fonction qui indique si le personnage monte ou descend.

Dans `Character.h`, ajoutons le prototype de la fonction `isFall` :

<div class="filename" >Character.h</div>
```
bool isFall(const Character &aCharacter);
```

Dans `Character.cpp`, définissons la fonction `isFall` :

<div class="filename" >Character.cpp</div>
```
// true si le personnage descend, false sinon
bool isFall(const Character &aCharacter) {
  return (aCharacter.oldY - aCharacter.y) < 0;
}
```

Nous allons également avoir besoin d'une fonction qui retourne la plateforme correspondante à l'identifiant donné en paramètre.

Dans `Platform.h`, ajoutons le prototype de la fonction `getPlatformFromId` :

<div class="filename" >Platform.h</div>
```
Platform getPlatformFromId(const uint8_t aId, Platform * aSet);
```

Dans `Platform.cpp`, définissons la fonction `getPlatformFromId` :

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

Ajoutons une fonction qui détermine s'il y a une collision avec la plateforme fournit en paramètre.

Dans `PhysicsEngine.h`, ajoutons le prototype de la fonction `isOnThePlatform` :

<div class="filename" >PhysicsEngine.h</div>
```
const uint8_t isOnThePlatform(const Character &aCharacter, const Platform &aPlatform);
```

Dans `PhysicsEngine.cpp`, définissons la fonction `isOnThePlatform` :

<div class="filename" >PhysicsEngine.cpp</div>
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

Dans `PhysicsEngine.h`, modifions le prototype de la fonction `isOnOnePlatform` afin de lui passer un ensemble de plateformes en paramètre, soit :

<div class="filename" >PhysicsEngine.h</div>
```
const uint8_t isOnOnePlatform(const Character &aCharacter, Platform * aSetOfPlatforms);
```

Modifions le code de la fonction `isOnOnePlatform` (dans `PhysicsEngine.cpp`) :

<div class="filename" >PhysicsEngine.cpp</div>
```
// Permet de détecter une collision avec une plateforme
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

Comme vous pouvez le remarquer, la fonction `isOnOnePlatform` parcourt l'ensemble des plateformes, et si elle détecte une collision alors elle renvoie l'identifiant de la plateforme avec laquelle le personnage est en collision.

Apportons maintenant une modification à la fonction `rectifyPositionY`, nous allons lui ajouter un paramètre, qui sera une plateforme, afin de corriger la position `y` en fonction de la plateforme avec laquelle le joueur est en contact.

Il est nécessaire d'inclure `Platform.h` dans `Character.h`.

Dans `Character.h`, modifions le prototype de la fonction `rectifyPositionY` :

<div class="filename" >Character.h</div>
```
void rectifyPositionY(Character &aCharacter, Platform &aPlatform);
```

Et adaptons le code de la fonction `rectifyPositionY` :

<div class="filename" >Character.cpp</div>
```
// Corriger la position y
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

Adaptons la fonction `gravity`.

Dans `PhysicsEngine.h`, modifions le prototype de la fonction `gravity` tel que :

<div class="filename" >PhysicsEngine.h</div>
```
void gravity(Character &aCharacter, Platform * aSetOfPlatforms);
```

Dans `PhysicsEngine.cpp`, modifions la fonction `gravity` tel que :

<div class="filename" >PhysicsEngine.cpp</div>
```
// Chute libre
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

De la même mannière, modifions le prototype de la fonction `jump`, dans `PhysicsEngine.h`, tel que :

<div class="filename" >PhysicsEngine.h</div>
```
void jump(Character &aCharacter, Platform * aSetOfPlatforms);
```

Dans `PhysicsEngine.h`, modifions la fonction `jump` :

<div class="filename" >PhysicsEngine.h</div>
```
// Implémentation du saut du personnage
void jump(Character &aCharacter, Platform * aSetOfPlatforms) {
  const uint8_t platformId = isOnOnePlatform(aCharacter, aSetOfPlatforms);

  if(aCharacter.state == PUSH_FOR_JUMP_STATE) {
    /* Inchangé... */
  } else if( platformId != NO_ID) {
    // Si on est en contact avec une plateforme

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
    /* Inchangé... */
  } else {
    /* Inchangé... */
  }
  
}
```

Enfin, dans le programme principal, apportons quelques modifications dans le code correspondant à l'état `PLAY_STATE` :

<div class="filename" >GBPlatformer01.ino</div>
```
/* ... */

if(hero.state != JUMP_STATE && hero.state != PUSH_FOR_JUMP_STATE) {
  gravity(hero, setOfPlatforms);
} else if(hero.state == JUMP_STATE || hero.state == PUSH_FOR_JUMP_STATE) {
  jump(hero, setOfPlatforms);
}

/* ... */
```

Téléversez votre porgramme vers la console, et amusez-vous à sauter sur les plateformes. Si vous avez conserver les valeurs fournies dans cette étape, autant les couleurs que la position des plateformes, vous pouvez être en-dessous de la plateformes verte claire et sauter, pour ainsi monter dessus.

## Conclusion

Vous voici arrivé à la fin de cette troisième étape : où nous avons ajouté la chute libre et la gestion des plateformes.

Si vous avez terminé ou si vous rencontrez des problèmes vous pouvez télécharger la solution [ici](https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v3.0.zip).

Dans la prochaine étape, c'est-à-dire la quatrième, nous aborderons les interactions avec le monde. Nous allons ainsi voir comment ajouter des objets comme des clés et une porte.

N'hésitez pas à me faire un retour : les améliorations que vous apporteriez (un regard extérieur est toujours bienvenu), les choses que vous n'avez pas compris, les fautes, etc. Pour laisser un retour, merci d'ajouter un commentaire à la création suivante :

{% include react-btn.html %}
