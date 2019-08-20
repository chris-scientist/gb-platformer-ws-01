---
layout: step
title: "Quel sera votre meilleur score ?"
permalink: /fr/meilleur-score/
next: /fr/fin/
link-other-lang: /en/best-score/
lang: fr
---

# Etape 6 - Quel sera votre meilleur score ?

## Introduction

Dans cette sixième et dernière étape, nous allons implémenter : la gestion des meilleurs scores, avec les interfaces nécessaires à l'ajout de cette fonctionnalité.

<!--*Je vous invite à télécharger <a href="#" class="external-link" >le code</a> qui est le résultat de la cinquième étape afin de partir sur des bases communes.*-->

## Quelques spécifications

Notre jeu de plateformes va gérer trois meilleurs scores.

Le score comme vous devez surêment vous en doutez dépendra du temps entre le début de la partie et l'ouverture de la porte. 
Le score est donc inférieur à 20 secondes (souvenez vous du game over implémenté lors de l'étape précédente). 
Le pseudo sera une chaîne de caractères comptant 5 caractères.

Nous allons fixer la règle suivante : si le temps de la partie courante et égale ou inférieure à l'un des meilleurs scores alors nous aurons un nouveau meilleur score.

Les spécifications étant maintenant définies, passons à la structuration des données.

## Comment se définit un meilleur score ?

Nous allons créer deux structures de données pour gérer nos meilleurs scores.

D'abord nous aurons besoins d'une structure qui gère un meilleur score avec :
* un pseudo que nous allons gérer via une chaîne de caractère (5 caractères précisément) ;
* un score géré via un entier.

Nous aurons également besoin d'une structure pour gérer les trois meilleurs scores contenant :
* trois meilleurs scores utilisant la structure définie plutôt ;
* le nombre de meilleurs scores enregistrés gérés via un entier ;
* l'index du nouveau meilleur score représenté par un entier.

Voici la pemière structure que nous nommerons `HighScore` dans le fichier `HighScore.h` :

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

Nous appelerons la seconde structure  `HighScoreManager` et nous la placerons dans le fichier `HighScoreManager.h`:

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

## Ajouter les fonctionnalités

### Initialisation du gestionnaire

Pour initialiser le gestionnaire de meilleurs scores, nous aurons besoin d'initialiser chacun des meilleurs scores. 
Ainsi dans `HighScore.h`, nous allons créer la fonction `setHighScore` :

<div class="filename" >HighScore.h</div>
```
void setHighScore(
  HighScore &aHighScore, 
  char * aName, 
  int32_t aScore
);
```

Dans `HighScore.cpp`, après avoir inclus `HighScore.h`, définissons la fonction `setHighScore` :

<div class="filename" >HighScore.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void setHighScore(HighScore &aHighScore, char * aName, int32_t aScore) {
  strncpy(aHighScore.nameOfScore, aName, 6);
  aHighScore.nameOfScore[5] = '\0';
  aHighScore.score = aScore;
}
```

Nous allons commencer par initialiser le gestionnaire de meilleurs scores ; ajoutons le prototype de la fonction `initHighScoreManager` dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
void initHighScoreManager(HighScoreManager &aManager);
```

Dans `HighScoreManager.cpp`, incluons le fichier `HighScoreManager.h` et implémentons la fonction `initHighScoreManager` :

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

Dans le programme principal, pensez à inclure `HighScoreManager.h` et instancions le gestionnaire de scores en dehors des fonctions `setup` et `loop` :

<div class="filename" >GBPlatformer01.ino</div>
```
// Autres includes...
#include "HighScoreManager.h"

// Autres variables gloables...
HighScoreManager highScoreManager;

void setup() {
  // ...
}

void loop() {
  // ...
}
```

Puis, toujours dans le programme principal,  appelons l'initialisation dans la fonction `setup` :

<div class="filename" >GBPlatformer01.ino</div>
```
void setup() {
  // initialisation de la Gamebuino META
  gb.begin();

  createTimer(myTimer);
  initHighScoreManager(highScoreManager);

  stateOfGame = HOME_STATE;
}
```

### Charger les meilleurs scores enregistrés

Pour charger les meilleurs scores préalablement enregistrés, nous allons d'abord ajouter les constantes suivantes dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
// Identifiant des meilleurs scores
const uint8_t NO_HIGH_SCORE = 0;
const uint8_t HIGH_SCORE_1 = 1;
const uint8_t HIGH_SCORE_2 = 2;
const uint8_t HIGH_SCORE_3 = 3;

// Bloc pour sauvegarder les meilleurs scores
const uint16_t NB_HIGH_SCORE_BLOCK = 0;
const uint16_t NAME_HIGH_SCORE_1_BLOCK = 1;
const uint16_t SCORE_HIGH_SCORE_1_BLOCK = 2;
const uint16_t NAME_HIGH_SCORE_2_BLOCK = 3;
const uint16_t SCORE_HIGH_SCORE_2_BLOCK = 4;
const uint16_t NAME_HIGH_SCORE_3_BLOCK = 5;
const uint16_t SCORE_HIGH_SCORE_3_BLOCK = 6;
```

Dans `HighScoreManager.h`, créons la fonction `loadHighScore` :

<div class="filename" >HighScoreManager.h</div>
```
void loadHighScore(
  HighScore &aScore, 
  uint16_t aBlockName, 
  uint16_t aBlockScore
);
```

Dans `HighScoreManager.cpp`, définissons la fonction `loadHighScore` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void loadHighScore(HighScore &aScore, uint16_t aBlockName, uint16_t aBlockScore) {
  char temp[5];
  gb.save.get(aBlockName, temp, 6);
  setHighScore(aScore, temp, gb.save.get(aBlockScore));
}
```

Retournons maintenant dans `HighScoreManager.h` et ajoutons y le prototype de la fonction `loadAllHighScore` :

<div class="filename" >HighScoreManager.h</div>
```
void loadAllHighScore(HighScoreManager &aManager);
```

Dans `HighScoreManager.cpp`, implémentons la fonction `loadAllHighScore` :

<div class="filename" >HighScoreManaher.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void loadAllHighScore(HighScoreManager &aManager) {
  int32_t nbHighScoreSaved = gb.save.get(NB_HIGH_SCORE_BLOCK);

  if(nbHighScoreSaved > 0) {
    uint8_t nbScore = 0;

    // Charger le meilleur score 1
    loadHighScore(aManager.highScore1, NAME_HIGH_SCORE_1_BLOCK, SCORE_HIGH_SCORE_1_BLOCK);
    nbScore++;

    // Charger le meilleur score 2
    if(nbHighScoreSaved >= 2) {
      loadHighScore(aManager.highScore2, NAME_HIGH_SCORE_2_BLOCK, SCORE_HIGH_SCORE_2_BLOCK);
      nbScore++;
    }

    // Charger le meilleur score 3
    if(nbHighScoreSaved >= 3) {
      loadHighScore(aManager.highScore3, NAME_HIGH_SCORE_3_BLOCK, SCORE_HIGH_SCORE_3_BLOCK);
      nbScore++;
    }

    aManager.nbHighScore = nbScore;
  }
}
```

Dans le programme principal, dans la fonction `setup`, après l'initialisation du gestionnaire, chargons les meilleurs scores :

<div class="filename" >GBPlatformer01.ino</div>
```
void setup() {
  // initialisation de la Gamebuino META
  gb.begin();

  createTimer(myTimer);
  initHighScoreManager(highScoreManager);
  loadAllHighScore(highScoreManager);

  stateOfGame = HOME_STATE;
}
```

### Réinitialiser le gestionnaire

Lorsqu'une nouvelle partie est lancée, il est nécessaire de réinitialiser le gestionnaire de meilleurs scores afin de remettre à zéro l'index du meilleur score actuel (pour qu'il n'y ait aucun score de "sélectionné" par défaut).

Cette fonction doit être créée dans `HighScoreManager.h` comme ceci :

<div class="filename" >HighScoreManager.h</div>
```
void resetIndexNewHighScore(HighScoreManager &aManager);
```

Dans `HighScoreManager.cpp`, définissons la fonction `resetIndexNewHighScore` :

<div class="filename" >HighScoreManager.cpp</div>
```
void resetIndexNewHighScore(HighScoreManager &aManager) {
  aManager.indexNewHighScore = NO_HIGH_SCORE;
}
```

Dans le programme principal, dans l'état `LAUNCH_PLAY_STATE` de la fonction `loop`, ajoutons la fonction que nous venons de définir :

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
      resetIndexNewHighScore(highScoreManager); // ... on réinitialise le tableau de meilleurs scores
      resetTimer(myTimer); // ............. on réinitialise le chronomètre
      initObjects(setOfObjects); // ....... on réinitialise les objets
      initPlatforms(setOfPlatforms); // ... on réinitialise les plateformes
      initCharacter(hero);
      stateOfGame = PLAY_STATE;
      break;
    // ...
    default:
      gb.display.println("Votre message");
  }
}
```

### Le temps réalisé fait-il partie des meilleurs scores ?

Nous allons voir maintenant l'ensemble des fonctions qui déterminent et enregistrent le nouveau temps si celui-ci fait partie des meilleurs scores.

Commençons par la fonction qui créé un meilleur score temporaire, correspondant à un nouveau meilleur score. Appelons cette fonction `createNewHighScore`. La seule donnée dont a besoin cette fonction est le score, que nous allons donc passer en paramètre, le pseudo sera saisit par l'utilisateur via un appel à la fonction `paintInputPseudoWindow`.

Avant d'écrire la fonction qui instancie ce nouveau meilleur score, voyons comment créer la fonction `paintInputPseudoWindow` retournant le pseudo que nous ajouterons dans `Display.h` :

<div class="filename" >Display.h</div>
```
void paintInputPseudoWindow(char * pseudo);
```

Dans `Display.cpp`, définissons la fonction `paintInputPseudoWindow` :

<div class="filename" >Display.cpp</div>
```
void paintInputPseudoWindow(char * pseudo) {
  gb.gui.keyboard("Save your score!", pseudo, 5);
  gb.display.clear();
}
```

Plaçons le prototype de `createNewHighScore` dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
const HighScore createNewHighScore(const uint32_t aTimeOfPart);
```

Développons la fonction `createNewHighScore` dans `HighScoreManager.cpp` :

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

Pensez à inclure `Display.h` dans `HighScoreManager.h`.

Il nous faut maintenant une fonction qui échange les scores, par exemple : le meilleur score 2 est remplacé par le meilleur score 1, ou le meilleur score 3 est remplacé par le meilleur score 2, etc. Pour ce faire, nous allons ajouter la fonction `swapHighScore`, cette fonction prend en paramètre les références de deux meilleurs scores : celui à remplacer et le nouveau.

Nous allons créer la fonction `swapHighScore` dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
void swapHighScore(
  HighScore &aHighScore, 
  const HighScore & aNewHighScore
);
```

Dans `HighScoreManager.cpp`, définissons la fonction `swapHighScore` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void swapHighScore(HighScore &aHighScore, const HighScore & aNewHighScore) {
  strncpy(aHighScore.nameOfScore, aNewHighScore.nameOfScore, 6);
  aHighScore.nameOfScore[5] = '\0';
  aHighScore.score = aNewHighScore.score;
}
```

Passons à la fonction `compareTime` qui compare la durée d'un meilleur score à une autre durée (le nouveau temps réalisé, candidat à un nouveau record).
Cette fonction renvoie un nombre compris entre -1 et 1 (bornes incluses), soit `a` la durée d'un meilleur score et `b` une durée :
* Si a < b alors retourner -1
* Si a == b alors retourner 0
* Si a > b alors retourner 1

Voici le prototype de la fonction `compareTime`, à écrire dans `HighScore.h` :

<div class="filename" >HighScore.h</div>
```
int8_t compareTime(
  const HighScore &aScore, 
  const int32_t aTimeInSeconds
);
```

Implémentons cette fonction :

<div class="filename" >HighScore.cpp <span>/!\ Scroll horizontal /!\</span></div>
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

Nous allons maintenant écrire une fonction qui renvoie `true` si le score obtenu est meilleur ou égal à un meilleur score donné. 
Cette fonction que nous nommerons `isBetterOrEqualToScore`, utilisera le résultat de `compareTime`.

Retournons dans `HighScoreManager.h`, et créons y la fonction `isBetterOrEqualToScore` :

<div class="filename" >HighScoreManager.h</div>
```
const bool isBetterOrEqualToScore(int8_t aValue);
```

Dans `HighScoreManager.cpp`, définissons la fonction :

<div class="filename" >HighScoreManager.cpp</div>
```
const bool isBetterOrEqualToScore(int8_t aValue) {
  return (aValue == 0 || aValue == 1);
}
```

Nous allons aborder une fonction importante pour la gestion des meilleurs score, cette fonction détermine si le score obtenu (pour la partie en cours) est un meilleur score ou non. Il y a un certain nombre de tests à réaliser, mais n'ayez pas peur ! Nous nommerons cette fonction `setHighScore4Time`.

Le prototype de cette fonction est le suivant :

<div class="filename" >HighScoreManager.h</div>
```
const uint8_t setHighScore4Time(
  HighScoreManager &aManager, 
  const int32_t aTimeOfPart
);
```

Elle prend en paramètre le gestionnaire de meilleurs scores puisque nous aurons probablement à le modifier et la durée de la partie que le joueur vient de finir.

Avant de vous fournir son implémentation, voici son pseudo code :

<div class="filename" >Pseudo code <span>/!\ Scroll horizontal /!\</span></div>
```
// Légende :
// nb(HS) pour nombre de meilleurs scores
// SC pour score de la partie courante
// S1 pour meilleur score 1
// S2 pour meilleur score 2
// S3 pour meilleur score 3
// index pour indiquer la position du nouveau meilleur score

index = NO_HIGH_SCORE

SI ( nb(HS) > 0 ) ALORS // Si des HS sont déjà enregistré
    SI ( SC meilleur que/égale à S1 ) ALORS
        SELON nb(HS) ALORS
            FAIRE SI (3 ou 2 HS)
                swap S3 = S2
                swap S2 = S1
                swap S1 = SC
                SI ( nb(HS) = 2 ) ALORS
                    nb(HS) = 3
                FIN SI
            FIN FAIRE SI
            FAIRE SI (1 HS)
                swap S2 = S1
                swap S1 = SC
                nb(HS) = 2
            FIN FAIRE SI
        FIN SELON
        index = HIGH_SCORE_1
    FIN SI
    SINON SI ( nb(HS) = 3 OU 2) ET ( SC meilleur que/égale à S2 ) ALORS
        swap S3 = S2
        swap S2 = SC
        SI ( nb(HS) = 2 ) ALORS
            nb(HS) = 3
        FIN SI
        index = HIGH_SCORE_2
    FIN SINON SI
    SINON SI ( nb(HS) = 3 ET SC meilleur que/égale à S3 ) OU ( nb(HS) = 2 ) ALORS
        swap S3 = SC
        SI ( nb(HS) = 2 ) ALORS
            nb(HS) = 3
        FIN SI
        index = HIGH_SCORE_3
    FIN SINON SI
    SINON SI nb(HS) = 1 ALORS
        swap S2 = SC
        nb(HS) = 2
        index = HIGH_SCORE_2
    FIN SINON SI
FIN SI
SINON
    swap S1 = SC
    nb(HS) = 1
    index = HIGH_SCORE_1
FIN SINON

retourner index
```

Développons cette fonction :

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

Voyons les fonctions permettant de sauvegarder sur la META les meilleurs scores, et ainsi les conserver d'une exécution à l'autre.

D'abord, créons la fonction qui effectue la sauvergade unitaire d'un meilleur score. Nous allons pour cela créer la fonction `saveHighScore` dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
void saveHighScore(
  char * aName, 
  int32_t aScore, 
  uint16_t aBlockName, 
  uint16_t aBlockScore
);
```

Dans `HighScoreManager.cpp`, définissons cette fonction :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
void saveHighScore(char * aName, int32_t aScore, uint16_t aBlockName, uint16_t aBlockScore) {
  gb.save.set(aBlockName, aName);
  gb.save.set(aBlockScore, aScore);
}
```

Enfin, écrivons la fonction qui sauvegarde l'intégralité des meilleurs scores. Le prototype de cette fonction que nous appelerons  `saveAllHighScore`, est à placer dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
void saveAllHighScore(HighScoreManager &aManager);
```

Implémentons cette fonction dans `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
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

Avant de passer à l'affichage des meilleurs scores, écrivons la fonction qui déclenche la sauvegarde des meilleurs scores s'il y a un nouveau meilleur score. Pour ce faire nous allons créer la fonction suivante dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
bool saveScoreIfNewHighScore(
  HighScoreManager &aManager, 
  const int32_t aTimeOfPart
);
```

Définissons cette fonction dans `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
```
bool saveScoreIfNewHighScore(HighScoreManager &aManager, const int32_t aTimeOfPart) {
  // Comparer le score actuel aux scores en mémoire
  const uint8_t highScoreIndex = setHighScore4Time(aManager, aTimeOfPart);

  // S'il y a un nouveau meilleur score alors on le sauvegarde
  bool haveNewHighScore = (highScoreIndex != NO_HIGH_SCORE);
  if(haveNewHighScore) {
    saveAllHighScore(aManager);
  }
  aManager.indexNewHighScore = highScoreIndex;

  return haveNewHighScore;
}
```

C'est cette dernière fonction que nous appelerons dans le programme principal. Nous verrons où et comment un peu plus tard dans cette étape. Voyons d'abord quelles sont les fonctions nécessaires pour l'affichage des meilleurs scores.


### Gérer l'affichage des meilleurs scores

Pour gérer l'affichage, nous allons avoir besoin d'une fonction qui retourne le meilleur score associé à l'index. Voici le prototype de cette fonction à écrire dans `HighScoreManager.h` :

<div class="filename" >HighScoreManager.h</div>
```
const HighScore& getHighScore(
  const HighScoreManager &aManager, 
  uint8_t anIndex
);
```

Développons cette fonction dans `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
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

Dans `HighScoreManager.h`, créons la fonction qui affiche les meilleurs scores :

<div class="filename" >HighScoreManager.h</div>
```
void paintHighScoreWindow(const HighScoreManager& aScoreManager);
```

Définissons cette fonction dans `HighScoreManager.cpp` :

<div class="filename" >HighScoreManager.cpp <span>/!\ Scroll horizontal /!\</span></div>
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

  // Afficher en bas de l'écran "retourner au menu"
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


### Finaliser la gestion des meilleurs scores

Pour commencer, ajoutons un nouvel état dans `Constants.h` :

<div class="filename" >Constants.h</div>
```
const uint8_t HIGH_SCORE_STATE = 9;
```

Nous utiliserons cette constante pour afficher le tableau de meilleurs scores.

Nous allons maintenant modifier le programme principal.

Nous allons gérer l'état `HIGH_SCORE_STATE` qui permet d'afficher les meilleurs scores :

<div class="filename" >GBPlatformer01.ino</div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Dans l'état `SAVE_HIGH_SCORE_STATE`, ajoutons la redirection vers l'écran d'affichage des meilleurs scores s'il y a un nouveau meilleur score :

<div class="filename" >GBPlatformer01.ino <span>/!\ Scroll horizontal /!\</span></div>
```
void loop() {
  // boucle d'attente
  gb.waitForUpdate();

  // effacer l'écran
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

Dans `Lang.h`, créons l'item du menu pour accéder aux meilleurs scores :

<div class="filename" >Lang.h</div>
```
static const char * HIGH_SCORE_EN = "High score";
```

Dans `Display.cpp`, modifions la fonction `paintMenu` afin d'ajouter l'item pour aller au meilleur score soit :

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

Pensez à téléverser le programme vers votre console pour tester cette dernière fonctionnalité.


## Conclusion

<!--Vous pouvez télécharger le <a href="#" class="external-link" >code source final</a>.--> Vous avez maintenant un jeu de plateformes complet ! Oui c'est pas transcendant au niveau des graphismes mais soyez patient, nous verrons cela ultérieurement, mais (spoiler alerte) ce ne sera pas l'objectif du prochain workshop. Le but avec ce premier workshop était de poser les bases d'un jeu de plateformes, mais ce n'est que le début d'une série...

N'hésitez pas à me faire un retour sur cette étape et sur le workshop : 

{% include react-btn.html %}

Vous voici arrivés à la fin de ce workshop, mais restez dans le coin pour réaliser d'autres workshops en attendant la suite de celui-ci. *Ne soyez pas trop impatient de voir la suite de ce workshop, en effet j'ai écris celui-ci en plus de 6 mois.*

C'est en forgeant que l'on devient forgeron ;)
