---
layout: step
title: "Gestion des déplacements"
permalink: /fr/deplacements/
next: /fr/chute-libre-et-plateformes/
link-other-lang: /en/introduction/
lang: fr
---

# Etape 2 - Gestion des déplacements

## Introduction

Dans cette deuxième étape, nous allons animer le personnage en implémentant les déplacements :
* aller à droite ;
* aller à gauche ;
* sauter.

*Je vous invite à télécharger <a href="https://github.com/chris-scientist/gb-platformer-workshop-01/archive/v1.0.zip" class="external-link" >le code</a> qui est le résultat de la première étape afin de partir sur des bases communes.*

## Avancer à droite ou à gauche

Nous allons voir ici comment implémenter le déplacement vers la droite puis vers la gauche.


### Avancer à droite

La première chose à faire c'est d'ajouter notre personnage en paramètre de la méthode `manageCommands`. 
Rappelez-vous la méthode est déclarée dans le fichier `Commands.h`. Voici son nouveau prototype :

<div class="filename" >Commands.h</div>
```
const uint8_t manageCommands(Character &aCharacter);
```

Il est nécessaire d'inclure `Character.h` dans `Commands.h`.

Voici le pseudo code du déplacement vers la droite :

<div class="filename" >Pseudo code</div>
```
SI touche droite enfoncée ALORS
  SI Personnage toujours dans les limites de l'écran ALORS
    Incrémenter la position x du personnage d'une unité
    Indiquer que le personnage se déplace à droite
  FIN SI
FIN SI
```

Pour un déplacement "rapide", nous utiliserons la fonction <a href="https://gamebuino.com/fr/academy/reference/gb-buttons-repeat" class="external-link" >gb.buttons.repeat</a>.

Pour faire un incrément d'une unité nous utiliserons le raccourci suivant `aCharacter.x++;`, ceci est équivalent à `aCharacter.x += 1;`, ou à `aCharacter.x = aCharacter.x + 1;`.

Dans la définition de la fonction `manageCommands` dont il faut penser à ajouter le personnage en paramètre, ajoutons le code suivant :

<div class="filename" >Commands.cpp</div>
```
if(gb.buttons.repeat(BUTTON_RIGHT, 1)) {
  if(aCharacter.x < (gb.display.width() - UNDER_CENTER_X_HERO)) {
    aCharacter.x++;
    aCharacter.toTheLeft = false;
  }
}
```

Vous pouvez téléverser votre programme vers votre console. Quand vous maintenez le bouton droit enfoncé, vous constatez que votre personnage va se déplacer vers la droite jusqu'à ce qu'il atteigne le bord droit de l'écran. 
Voilà une première partie du déplacement est développée.


### Avancer à gauche

Je vous propose maintenant de coder le déplacement vers la gauche en autonomie. Si vous n'y arrivez pas, je vous donnerais dans un premier temps quelques astuces. Si vous ne voyez toujours pas comment faire ou que vous souhaitez vérifier votre code : je vous donne la solution.


#### Avancer à gauche : quelques pistes

Pour la contrainte de sortie d'écran il faut vérifier que :

<div class="filename" >Code</div>
```
aCharacter.x > OVER_CENTER_X_HERO
```

Ensuite pour aller à gauche, il faut décrémenter la position `x` du personnage. Pour ce faire il y a, comme pour l'incrémentation, un raccourci qui est :

<div class="filename" >Code</div>
```
aCharacter.x--;
```

Voici les deux astuces qui couplées au code de déplacement à droite devrait vous permettre d'écrire le déplacement à gauche.


#### Avancer à gauche : la solution

Pour aller à gauche, voici le code à écrire dans la fonction `manageCommands` :

<div class="filename" >Commands.cpp</div>
```
if(gb.buttons.repeat(BUTTON_LEFT, 1)) {
  if(aCharacter.x > OVER_CENTER_X_HERO) {
    aCharacter.x--;
    aCharacter.toTheLeft = true;
  }
}
```


### Tester !

Il est important de tester votre programme à chaque fois que vous apportez une modification. Comme nous venons d'implémenter le déplacement à gauche et à droite, téléverser votre programme, et constater que le jeu réagit comme vous le désirez.

![Déplacements gauche et droite](./../../img/E02/deplacements_LeftAndRight_v1.gif)

## Sauter

Pour le saut, le personnage aura un comportement différent selon que vous ayez de l'élan ou non :
* si le personnage ne prend pas d'élan, alors il sautera sur place ;
* si le personnage prend de l'élan, alors il réalisera un "saut en cloche".

Avant de se lancer dans le développement du saut répondant à ses spécifications, voyons la théorie qui nous permettra de réaliser un tel saut.


### Sauter : la théorie

Pour implémenter le saut nous allons avoir besoin d'ajouter à notre personnage :
* Une vitesse horizontale, qui selon sa valorisation nous permettra de faire le saut en cloche ou non, nommons la `vx`.
* Une vitesse verticale, qui nous permettra d'évoluer dans l'air, nommons la `vy`.
* L'ancienne valeur en y que nous appelerons `oldY`que nous utiliserons surtout dans la prochaine étape.
* L'état du personnage : sur le sol (c'est-à-dire sur une plateforme), s'il donne une impulsion pour sauter ou s'il saute, nommons le `state`.

Voyons les principales étapes du saut :
1. Le joueur donne l'ordre de sauter, l'état du joueur passe à l'état `PUSH_FOR_JUMP_STATE`.
2. Lors de l'impulsion, initialiser la vitesse verticale avec la vitesse verticale initiale (nous utiliserons une constante pour cette dernière).
3. Changer l'état du joueur à l'état `JUMP_STATE`.
4. Commencer le saut.
5. Tant que le personnage est en l'air, continuer le saut.
6. Si le personnage est en dehors de l'écran, alors on annule le saut et on fait jouer la gravité (nous implémenterons la gravité dans la prochaine étape).
7. Si le personnage est en contact avec une plateforme (pour l'instant la seule plateforme gérée est le sol) alors arrêter le saut.

D'un point de vue mathématique, pour réaliser le saut il faut :
1. Faire évoluer la vitesse verticale avec la gravité, soit `vy += g;`, avec `g` la constante de gravité.
2. Faire évoluer la position horizontale selon la vitesse horizontale, soit `x += vx;`.
3. Faire évoluer la position verticale selon la vitesse verticale, soit `y += vy;`.

De plus, il faut initialiser la vitesse verticale soit `vy = -vy0;` avec `vy0` la vitesse verticale initiale, à réaliser lors de l'impulsion seulement. De plus il est nécessaire de mémoriser 'y' dans `oldY` avant de modifier la valeur de 'y' mais nous verrons pourquoi plus tard, ce sera plus simple avec l'exemple.

N'ayez pas peur de toute cette théorie, nous allons passer à l'implémentation.


### Sauter : la pratique


#### Evolution du personnage

Avant de faire évoluer le personnage, ajoutons quelques constantes dans notre fichiers `Constants.h` :

<div class="filename" >Constants.h <span>/!\ Scroll horizontal /!\</span></div>
```
// Etat du personnage
const uint8_t ON_THE_PLATFORM_STATE = 0; // ... le joueur est sur le sol
const uint8_t FREE_FALL_STATE = 1; // ......... le joueur est en chute libre
const uint8_t PUSH_FOR_JUMP_STATE = 2; // ..... le joueur donne une impulsion pour sauter
const uint8_t JUMP_STATE = 3; // .............. le joueur saute

// Identifiants des plateformes
const uint8_t NO_ID = 0;
const uint8_t ID_GROUND = 1;

// Gravité
const uint8_t GRAVITY = 2;

// Paramétrage du saut
const uint8_t HORIZONTAL_VELOCITY = 3; // ...... vitesse horizontale
const uint8_t INIT_VERTICAL_VELOCITY = 10; // ... vitesse verticale initiale
```

Ensuite, ajoutons les attributs suivants au personnage (voir `Character.h`) :

<div class="filename" >Character.h</div>
```
struct Character {
  /* ... */
  int8_t oldY; // ......... position y au dernier rafraîchissement
  int8_t vx; // ........... vitesse horizontale
  int8_t vy; // ........... vitesse verticale
  uint8_t state; // ....... état du personnage : voir constantes
};
```

N'oublions pas d'initialiser les nouveaux attributs dans `initCharacter` (qui se trouve dans `Character.cpp`) :

<div class="filename" >Character.cpp</div>
```
// on initialise la position 'y' précédente
aCharacter.oldY = aCharacter.y;

// on initialise la vitesse verticale et horizontale
aCharacter.vx = 0;
aCharacter.vy = 0;

// par défaut, le joueur est sur le sol
aCharacter.state = ON_THE_PLATFORM_STATE;
```

Nous devons encore ajouter une fonction qui corrige la position `y`.

Dans `Character.h`, écrivons le prototype de la fonction `rectifyPositionY` :

<div class="filename" >Character.h</div>
```
void rectifyPositionY(Character &aCharacter);
```

Le code de la fonction `rectifyPositionY`, à placer dans `Character.cpp`, est le suivant :

<div class="filename" >Character.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
// Corriger la position y (la fonction évoluera à la prochaine étape)
void rectifyPositionY(Character &aCharacter) {
  aCharacter.y = (gb.display.height() - UNDER_CENTER_Y_HERO);
}
```

Bien ! Maintenant que nous avons apporter les modifications nécessaires au personnage, nous devons regarder si nous sommes en collision avec une plateforme ou si nous sautons en dehors de l'écran.


#### Une histoire de collisions

Avant d'implémenter le saut, nous allons écrire deux fonctions :
* la première permet de détecter une collision avec le sol ;
* la seconde permet d'interrompre le saut si nous sautons en dehors de l'écran.

Dans un nouveau fichier, que nous nommerons `PhysicsEngine.h`, ajoutons le code suivant :

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

Pour l'instant, la fonction `isOnOnePlatform` contrôlera uniquement si nous sommes en collision avec le sol ou non. Nous ferons évoluer cette fonction lors de la prochaine étape.

Voici donc le code pour détecter une collision avec le sol, à écrire dans le fichier `PhysicsEngine.cpp`, pensez à inclure `PhysicsEngine.h` (dans `PhysicsEngine.cpp`) :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
const uint8_t isOnOnePlatform(const Character &aCharacter) {
  return ( (aCharacter.y + aCharacter.vy) >= (gb.display.height() - UNDER_CENTER_Y_HERO) ) ? ID_GROUND : NO_ID ;
}
```

Ajoutons la méthode qui détecte que le personnage est en dehors de l'écran. Cette fonction doit, en plus de la détection du "hors monde", corriger la position `x` du personnage, afin d'éviter de le faire disparaître.

Nous allons ajouter cette dernière dans le fichier `PhysicsEngine.h` :

<div class="filename" >PhysicsEngine.h</div>
```
bool isOutOfWorld(Character &aCharacter);
```

Voici le code de la fonction `isOutOfWorld` (à écrire dans le fichier `PhysicsEngine.cpp`) :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
bool isOutOfWorld(Character &aCharacter) {
  if( aCharacter.x <= OVER_CENTER_X_HERO ) {
    // Si le personnage sort à gauche de l'écran
    // alors on force ses coordonnées pour qu'il soit toujours sur l'écran
    aCharacter.x = OVER_CENTER_X_HERO;
    return true;
  } else if( aCharacter.x >= (gb.display.width() - UNDER_CENTER_X_HERO) ) {
    // Si le personnage sort à droite de l'écran
    // alors on force ses coordonnées pour qu'il soit toujours sur l'écran
    aCharacter.x = gb.display.width() - UNDER_CENTER_X_HERO;
    return true;
  }
  return false;
}
```

Les limites étant déterminées, passons au développement du saut.


##### Un peu de physique

Pour gérer l'élan, nous allons modifier la fonction `manageCommands` (qui est dans le fichier `Commands.cpp`) :

<div class="filename" >Commands.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
aCharacter.vx = 0; // par défaut, pas de vitesse horizontale

if(gb.buttons.repeat(BUTTON_RIGHT, 1)) {
  if(aCharacter.x < (gb.display.width() - UNDER_CENTER_X_HERO)) {
    /* ... */
    aCharacter.vx = HORIZONTAL_VELOCITY; // vitesse horizontale positive pour aller à droite
  }
} else if(gb.buttons.repeat(BUTTON_LEFT, 1)) {
  if(aCharacter.x > OVER_CENTER_X_HERO) {
    /* ... */
    aCharacter.vx = -HORIZONTAL_VELOCITY; // vitesse horizontale négative pour aller à gauche
  }
}
```

Toujours dans la fonction `manageCommands`, il faut écrire le code pour que lorsque le joueur appuye sur le bouton A et que le personnage ne saute pas, on donne l'impulsion initiale pour le saut, soit :

<div class="filename" >Commands.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
if(gb.buttons.pressed(BUTTON_A) && aCharacter.state != JUMP_STATE) {
  aCharacter.state = PUSH_FOR_JUMP_STATE;
}
```

Créons, dans `PhysicsEngine.h`, le prototype de la fonction `jumpMovement`, soit :

<div class="filename" >PhysicsEngine.h</div>
```
void jumpMovement(Character &aCharacter);
```

Ajoutons dans `PhysicsEngine.cpp` le code de `jumpMovement` :

<div class="filename" >PhysicsEngine.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
// Réalise le déplacement du personnage lors du saut
void jumpMovement(Character &aCharacter) {
  aCharacter.oldY = aCharacter.y; // ... on enregistre la position verticale précédente
  aCharacter.vy += GRAVITY; // ......... on fait évoluer la vitesse verticale
  aCharacter.x += aCharacter.vx; // .... on fait évoluer la position horizontale
  aCharacter.y += aCharacter.vy; // .... on fait évoluer la position verticale
}
```

Dans `PhysicsEngine.h`, ajoutons le prototype de la fonction `jump`, soit :

<div class="filename" >PhysicsEngine.h</div>
```
void jump(Character &aCharacter);
```

L'implémentation de la fonction `jump`, à ajouter dans `PhysicsEngine.cpp`, est la suivante :

<div class="filename" >PhysicsEngine.cpp</div>
```
// Implémentation du saut du personnage
void jump(Character &aCharacter) {
  const uint8_t platformId = isOnOnePlatform(aCharacter);

  if(aCharacter.state == PUSH_FOR_JUMP_STATE) {
    // le joueur donne une impulsion pour le saut
    // on initialise alors les données pour le saut
    aCharacter.vy = -INIT_VERTICAL_VELOCITY;
    aCharacter.state = JUMP_STATE;

    jumpMovement(aCharacter);
  } else if( platformId != NO_ID) {
    rectifyPositionY(aCharacter);
    aCharacter.vy = 0;
    aCharacter.state = ON_THE_PLATFORM_STATE;
  } else if( isOutOfWorld(aCharacter) ) {
    // si le saut nous conduit en dehots du monde
    // alors on applique la gravité
    aCharacter.state = FREE_FALL_STATE;
  } else {
    // le personnage saute
    jumpMovement(aCharacter);
  }
  
}
```

Le personnage est prêt à sauter, enfin presque ! Effectivement il faut écrire encore un peu de code avant que le personnage ne saute.


#### Dernière ligne droite avant de sauter

Retournons dans le programme principal, et en particulier dans l'état `PLAY_STATE` où il est nécessaire d'apporter des modifications. Il est nécessaire d'inclure `PhysicsEngine.h` dans le programme principal.
Le code pour cet état doit être :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
```
// Autres includes...
#include "PhysicsEngine.h"

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

Comme vous pouvez le remarquez, le joueur peut agir sur les commandes uniquement s'il est sur une plateforme, soit pour l'instant sur le sol.

![Déplacement : première version du saut](./../../img/E02/deplacements_Jump_v1.gif
)

#### Quelques réglages

Vous pouvez désormais téléverser votre programme vers la console, et vous amuser à sauter partout. Il est à noter que si vous sauter sur le bord de l'écran alors le jeu est bloqué. Nous verrons dans la prochaine étape comment régler ce problème en implémentant la gravité / chute libre.

Amusez vous à changer la gravité, la vitesse horizontale et la vitesse verticale initiale qui se trouve dans `Constants.h`. 
Il est important que vous jouiez avec pour trouver le bon paramétrage.

J'insiste ! Essayez de changer ces paramètres !

Vous avez sûrement remarqué que le paramétrage fourni est trop important pour un jeu de plateformes qui doit se jouer dans les limites de l'écran uniquement.

Ainsi, dans `Constants.h`, et après une phase de calibrage, nous avons choisis le paramétrage suivant :

<div class="filename" >Constants.h <span>/!\ Scroll horizontal /!\</span></div>
```
// Gravité
const uint8_t GRAVITY = 1;

// Paramétrage du saut
const uint8_t HORIZONTAL_VELOCITY = 2; // ...... vitesse horizontale
const uint8_t INIT_VERTICAL_VELOCITY = 6; // ... vitesse verticale initiale
```

Voici le saut avant callibrage :

![Déplacement : première version du saut](./../../img/E02/deplacements_Jump_v1.gif
)

Et voici le saut actuel (après callibrage) :

![Déplacement : première version du saut](./../../img/E02/deplacements_NewJump_v1.gif
)

## Conclusion

Vous voici arrivé à la fin de cette deuxième étape où nous avons ajouté les déplacements pour le personnage : avancer à gauche, avancer à droite et sauter.

Si vous avez terminé ou si vous rencontrez des problèmes vous pouvez télécharger la solution <a href="" class="external-link" >ici</a>.

Dans la prochaine étape, c'est-à-dire la troisième, nous aborderons la gestion des plateformes de la création à l'affichage, en passant par la gestion des collisions. Et nous implémenterons également la chute libre.

N'hésitez pas à me faire un retour : les améliorations que vous apporteriez (un regard extérieur est toujours bienvenu), les choses que vous n'avez pas compris, les fautes, etc.  Pour laisser un retour, merci d'ajouter un commentaire à la création suivante :

{% include react-btn.html %}
