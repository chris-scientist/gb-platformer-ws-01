---
layout: step
title: Introduction
permalink: /fr/introduction/
next: /fr/personnage-menu/
link-other-lang: /en/introduction/
lang: fr
---

# Etape 0 - Introduction

N'avez-vous jamais rêvé de créer le prochain Mario ? Vous verrez, après avoir terminé ce workshop, que ce n'est pas si compliqué que ça. Le plus important est de ne surtout pas griller les étapes...

![Exemple de jeu de plateformes](https://zupimages.net/up/19/11/kqg7.png)

*L'image ci-dessus est une illustration, ce n'est pas une image issue du jeu que nous allons concevoir dans ce workshop.*

## Présentation des jeux de plateformes

### Brève définition

Dans les jeux de plateformes (ou *platformers*), le joueur contrôle un personnage qui doit sauter sur des plateformes et éviter des obstacles. Cependant, dans le premier platformer, pour se déplacer, le personnage grimpait à des échelles.

### Un peu d'histoire

C'est à la fin de l'année 1980, que le premier jeu de plateformes **Space Panic** est sorti, développé par la société *Universal Entertainment Corporation*.  Ce jeu, jouable sur bornes d'arcades, avait été conçu sur un écran fixe, comme celui que nous allons concevoir dans ce workshop. Le but était de piéger des aliens en creusant des trous dans le sol, ceci dans un temps limité par une jauge d'oxygène s'epuisant au fil de la partie. 
Voici un aperçu du jeu trouver sur <a href="https://www.youtube.com/watch?v=dLd1xABCsaQ" class="external-link" >YouTube</a>.

En 1981, est sorti **Apple Panic**, développé par *Brøderbund Software*. Il s'agit d'un clone de Space Panic, mais jouable sur des micro-ordinateurs tels que Apple 2, Atari, etc.

Toujours en 1981, est sorti un fameux **Donkey Kong**, développé par *Nintendo* et jouable sur bornes d'arcades.

Par la suite, et depuis les années 1980, de nombreux jeux de plateformes sont sortis, pour n'en citer que deux, nous pouvons citer **Crash Bandicoot** et **Super Mario Odyssey**. Si à l'origine les jeux étaient en 2D (et sur bornes d'arcades), aujourd'hui le genre a su se réinventer et avoir des graphismes en 3D.

Vous trouverez une liste complète des principaux jeux de plateformes sur <a href="https://fr.wikipedia.org/wiki/Liste_chronologique_de_jeux_de_plates-formes" class="external-link" >wikipédia</a>, du premier jeu jusqu'à nos jours.

### Sources

#### Wikipédia

<div class="sources" >
	<ul>
		<li><a href="https://fr.wikipedia.org/wiki/Jeu_de_plates-formes" class="external-link" >Jeu de plateformes</a></li>
		<li><a href="https://fr.wikipedia.org/wiki/Space_Panic" class="external-link" >Space Panic</a></li>
		<li><a href="https://fr.wikipedia.org/wiki/Apple_Panic" class="external-link" >Apple Panic</a></li>
		<li><a href="https://fr.wikipedia.org/wiki/Donkey_Kong_%28jeu_vidéo,_1981%29" class="external-link" >Donkey Kong</a></li>
	</ul>
</div>

#### Autres sources

<div class="sources" >
	<ul>
		<li><a href="https://www.millenium.org/news/137671.html?page=4" class="external-link" >Histoire du jeu de plateformes sur wwww.millenium.org</a></li>
	</ul>
</div>

## Présentation du workshop

Dans ce workshop, nous allons aborder les bases du jeu de plateformes.

Les étapes de ce workshop sont :
1. le personnage : création, affichage et gestion d'un menu ;
2. gestion des déplacements ;
3. chute libre et plateformes ;
4. interactions avec le monde ;
5. gestion de la partie et chronomètre ;
6. meilleurs scores : gestion et affichage.

Dans ce premier workshop, les éléments : plateformes et objets auront des coordonnées définies par un couple d'entiers. Nous verrons plus tard comment concevoir un monde basé sur du tilemapping. Un exemple de tilemapping est de stocker le niveau dans un tableau de chiffres, cela peut également être un tableau de caractères comme je l'ai montré dans <a href="https://gamebuino.com/fr/academy/workshop/sokoban-vs-poo" class="external-link" >Sokoban</a>. Où chaque élément du tableau (chiffre, caractère, ...) sera transformé en une petite image dont l'ensemble formera l'image globale affichée, mais nous reparlerons de cela dans un futur workshop. Et sachez que certains utilisent d'autres techniques comme une image de point de couleur pour définir la place des éléments.

Comme déjà dit, le jeu de plateformes, que nous allons réaliser dans ce premier workshop, sera conçu sur un écran fixe. C'est-à-dire que le personnage n'évoluera pas sur un monde plus grand que l'écran de la META mais nous verrons cette possibilité dans un prochain workshop.

De plus, nous aborderons l'aspect des graphismes dans un workshop ultérieur. Nous nous contenterons dans ce premier workshop, d'afficher des rectangles de couleurs. Je rappelle que son but est de poser les bases du jeu de plateforme. Dans cette optique, il n'est pas nécessaire d'avoir de jolis graphismes. Des rectangles suffisent amplement pour calibrer le jeu.

## Conclusion

Vous en savez maintenant plus sur les jeux de plateformes et sur les objectifs de ce workshop.

Voici un aperçu de ce que nous allons réaliser :

![Notre platformer](./../../img/Platformer01_Ecrans_v2.png)

Comme vous pouvez le voir ça ne ressemble pas à Mario... Allons-y étapes après étapes !

Pour laisser un retour, merci d'ajouter un commentaire à la création suivante :

{% include react-btn.html %}
