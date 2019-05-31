---
layout: step
title: "Next page"
permalink: /en/character-menu/
next: /en/end/
link-other-lang: /fr/personnage-menu/
lang: en
---

# Next step

Work in progress

<div class="filename" >GBPlatformer01.ino</div>
```
#include <Gamebuino-Meta.h>
  
void setup() {
  // initialisation de la Gamebuino META
  gb.begin();
}

void loop() {
  // boucle d'attente
  while(!gb.update());

  // Commencer par effacer l'écran
  gb.display.clear();

  gb.display.println("Hello world");
}
```

It's the end :)
