---
layout: step
title: Introduction
permalink: /en/introduction/
next: /en/end/
link-other-lang: /fr/introduction/
lang: en
---

# Step 0 - Introduction

Have you ever dreamed of creating the next Mario ? You will see, after completing this workshop, that it is not that complicated. The most important thing is not to fry the steps...

![Example of platformer](https://zupimages.net/up/19/11/kqg7.png)

*The image above is an illustration, it is not an image from the game we will design in this workshop.*

## Presentation of platformers

### Short definition

In platformers, the player controls a character who must jump on platforms and avoid obstacles. However, in the first platformer, to move, the character climbed ladders.

### A bit of history

It was at the end of 1980 that the first platformer **Space Panic** was released, developed by the company *Universal Entertainment Corporation*.  This game, which can be played on arcade terminals, was designed on a fixed screen, like the one we will design in this workshop. The goal was to trap aliens by digging holes in the ground, within a limited time by an oxygen gauge running out as the game progressed. 
Here is an overview of the game found on <a href="https://www.youtube.com/watch?v=dLd1xABCsaQ" class="external-link" >YouTube</a>.

In 1981, **Apple Panic**, developed by *Brøderbund Software*, was released. It is a Space Panic clone, but can be played on microcomputers such as Apple 2, Atari, etc.

Also in 1981, a famous **Donkey Kong**, developed by *Nintendo* and playable on arcade terminals, was released.

Subsequently, and since the 1980s, many platformers have been released, to name only two, we can mention **Crash Bandicoot** and **Super Mario Odyssey**. If at the beginning the games were in 2D (and on arcade terminals), today the genre has been able to reinvent itself and have 3D graphics.

You will find a complete list of the main platformers on <a href="https://fr.wikipedia.org/wiki/Liste_chronologique_de_jeux_de_plates-formes" class="external-link" >wikipedia</a> (sorry, it's a French page), from the first game to the present day.

### Sources

Sorry, it's a french sources.

#### Wikipedia

<div class="sources" >
	<ul>
		<li><a href="https://fr.wikipedia.org/wiki/Jeu_de_plates-formes" class="external-link" >Platformers</a></li>
		<li><a href="https://fr.wikipedia.org/wiki/Space_Panic" class="external-link" >Space Panic</a></li>
		<li><a href="https://fr.wikipedia.org/wiki/Apple_Panic" class="external-link" >Apple Panic</a></li>
		<li><a href="https://fr.wikipedia.org/wiki/Donkey_Kong_%28jeu_vidéo,_1981%29" class="external-link" >Donkey Kong</a></li>
	</ul>
</div>

#### Other sources

<div class="sources" >
	<ul>
		<li><a href="https://www.millenium.org/news/137671.html?page=4" class="external-link" >History of platformer on wwww.millenium.org</a></li>
	</ul>
</div>

## Presentation of the workshop

In this workshop, we will discuss the basics of platformer.

The steps of this workshop are :
1. the character : creation, display and management of a menu ;
2. movement management ;
3. free fall and platforms ;
4. interactions with the world ;
5. game management and timer ;
6. high scores : management and display.

In this first workshop, the elements : platforms and objects will have coordinates defined by a pair of integers. We will see later how to design a world based on tilemapping. An example of tilemapping is to store the level in a table of numbers, it can also be a table of characters as I showed in <a href="https://gamebuino.com/fr/academy/workshop/sokoban-vs-poo" class="external-link" >Sokoban</a> (sorry, I did not translate this workshop into English). Where each element of the painting (number, character, ...) will be transformed into a small image whose totality will form the overall image displayed, but we will discuss this again in a future workshop. And be aware that some use other techniques such as a color point image to define the place of the elements.

As already mentioned, the platformer, which we will realize in this first workshop, will be designed on a fixed screen. That is to say, the character will not evolve on a world larger than the META screen but we will see this possibility in a future workshop.

In addition, we will discuss the aspect of graphics in a later workshop. In this first workshop, we will content ourselves with displaying rectangles of colors. I remind you that its purpose is to lay the foundations of the platformer. In this perspective, it is not necessary to have pretty graphics. Rectangles are more than enough to calibrate the game.

## Conclusion

You now know more about platformers and the objectives of this workshop.

Here is an overview of what we will achieve (sorry, it's in french) :

![Our platformer](./../../img/Platformer01_Ecrans_v2.png)

As you can see, it doesn't look like Mario... Let's go step by step !

To leave a feedback, please add a comment to the following creation :

{% include react-btn.html %}
