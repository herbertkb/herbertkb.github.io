---
layout: post
title:  "Authoring a Hunt the Wumpus clone in RHPAM 7.3"
date:   2019-05-30
categories: tech business-automation
---

This post will walk through authoring a (relatively) simple game in RHPAM 7.3.
We will implement a clone of the '70s minicomputer classic [Hunt the
Wumpus](https://www.atariarchives.org/bcc1/showpage.php?page=247). If you want
to give a it a try before we start, run `wump` from the `bsd-games` package in
Fedora.

```
$ sudo dnf install bsd-games
$ wump
Instructions? (y-n) n

You're in a cave with 20 rooms and 3 tunnels leading from each room.
There are 3 bats and 3 pits scattered throughout the cave, and your
quiver holds 5 custom super anti-evil Wumpus arrows.  Good luck.

You are in room 11 of the cave, and have 5 arrows left.
There are tunnels to rooms 4, 16, and 18.
Move or shoot? (m-s)
```  

In our game, the player goes into a cave full of rooms, some giant bats, some
bottomless pits, and one big mean Wumpus. Each turn, the player can either move
or shoot an arrow into one of three rooms leading from their current room. If
their arrow hits the Wumpus, they win and go home a hero. If they step into a
room with the Wumpus, the Wumpus gets a yummy snack. Stepping into a room with a
giant bat gets the player a free ride to another random room in the cave, and
stepping into a bottomless pit sends them to the other side of the Earth. Fun!

Like any good project, the first step is to white board (paper board?) a rough 
draft of the overall flow for our process, and sketch out our data model. 
![paper diagram]({{ site.url }}/assets/pam-wumpus/paperdiagram.jpg)

Note that here I sketched the Wumpus, Bats, Pits, and Player as their own
objects. It would be simpler to keep that bit of state as global process 
variables or embedded in the state of the rooms (in fact, that's what the `bsd-games` 
[implementation](https://github.com/vattam/BSDGames/blob/master/wump/wump.c) 
does). But I felt that a more realistic business process will have entities with
much more than single fields and this approach is more extendable if we'd like
to create new behaviors for these entities in the future. I think I finally understand what "loose coupling" means. 



